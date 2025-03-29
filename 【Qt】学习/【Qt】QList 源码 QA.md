##### 一、QA：
 
1. QList 内部的数据结构是 QListData::Data。Data 结构体有个 QtPrivate::RefCount 类型的成员变量 ref，该成员变量记录着该内存块的引用。也就是说，QList 采用了 Copy On Write 的技术优化了存放数据的内存块。

2. 可以从 QList<T>::QList(const QList<T> &l)、QList<T>::append(const T &t) 的实现看出的确是采用了 COW 技术。

3. QList 的拷贝构造函数中调用的 QList<T>::node_copy 函数有一个小小的优化，就是通过判断 QTypeInfo<T>::isLarge、QTypeInfo<T>::isStatic、QTypeInfo<T>::isComplex 来决定是逐个元素调用 T 的构造函数还是直接 memcpy。（但是 QTypeInfo<T>::isStatic 是 true，也就是说永远都是调用构造函数？！）

4. 对 QByteArray 和 QString 模板偏特化的作用是啥？
- 例如，QStringList 有一些个性化的方法。

5. QList 有普通链表那种 next 指针吗？
- 没有。从 QList<T>::operator[] 函数的实现可以看出，它的下标访问并没有遍历链表的操作，它只是简单的指针相加操作。

---

##### 二、source code：

```cpp
template <typename T> struct QListSpecialMethods
{
protected:
    ~QListSpecialMethods() = default;
};
template <> struct QListSpecialMethods<QByteArray>;
template <> struct QListSpecialMethods<QString>;

struct Q_CORE_EXPORT QListData {
    ...
    struct Data {
        QtPrivate::RefCount ref;
        int alloc, begin, end;
        void *array[1];
    };
    ...
};
template <typename T>
class QList
#ifndef Q_QDOC
    : public QListSpecialMethods<T>
#endif
{
    ...
    union { QListData p; QListData::Data *d; };
    ...
};
```
- 关键语句：template <> struct QListSpecialMethods<QByteArray>;、template <> struct QListSpecialMethods<QString>;

```cpp
template <typename T>
class QTypeInfo
{
public:
    enum {
        isSpecialized = std::is_enum<T>::value, // don't require every enum to be marked manually
        isPointer = false,
        isIntegral = std::is_integral<T>::value,
        isComplex = !qIsTrivial<T>(),
        isStatic = true,
        isRelocatable = qIsRelocatable<T>(),
        isLarge = (sizeof(T)>sizeof(void*)),
        isDummy = false, //### Qt6: remove
        sizeOf = sizeof(T)
    };
};

QListData::Data *QListData::detach(int alloc)
{
    Data *x = d;
    Data* t = static_cast<Data *>(::malloc(qCalculateBlockSize(alloc, sizeof(void*), DataHeaderSize)));
    Q_CHECK_PTR(t);

    t->ref.initializeOwned();
    t->alloc = alloc;
    if (!alloc) {
        t->begin = 0;
        t->end = 0;
    } else {
        t->begin = x->begin;
        t->end   = x->end;
    }
    d = t;

    return x;
}
    
template <typename T>
Q_INLINE_TEMPLATE void QList<T>::node_copy(Node *from, Node *to, Node *src)
{
    Node *current = from;
    if (QTypeInfo<T>::isLarge || QTypeInfo<T>::isStatic) {
        QT_TRY {
            while(current != to) {
                current->v = new T(*reinterpret_cast<T*>(src->v));
                ++current;
                ++src;
            }
        } QT_CATCH(...) {
            while (current-- != from)
                delete reinterpret_cast<T*>(current->v);
            QT_RETHROW;
        }

    } else if (QTypeInfo<T>::isComplex) {
        QT_TRY {
            while(current != to) {
                new (current) T(*reinterpret_cast<T*>(src));
                ++current;
                ++src;
            }
        } QT_CATCH(...) {
            while (current-- != from)
                (reinterpret_cast<T*>(current))->~T();
            QT_RETHROW;
        }
    } else {
        if (src != from && to - from > 0)
            memcpy(from, src, (to - from) * sizeof(Node));
    }
}

template <typename T>
Q_OUTOFLINE_TEMPLATE QList<T>::QList(const QList<T> &l)
    : QListSpecialMethods<T>(l), d(l.d)
{
    if (!d->ref.ref()) {
        p.detach(d->alloc);

        QT_TRY {
            node_copy(reinterpret_cast<Node *>(p.begin()),
                    reinterpret_cast<Node *>(p.end()),
                    reinterpret_cast<Node *>(l.p.begin()));
        } QT_CATCH(...) {
            QListData::dispose(d);
            QT_RETHROW;
        }
    }
}
```
- 关键语句：if (!d->ref.ref())
- 关键语句：if (QTypeInfo<T>::isLarge || QTypeInfo<T>::isStatic)、else if (QTypeInfo<T>::isComplex)

```cpp
template <typename T>
Q_OUTOFLINE_TEMPLATE QList<T>::~QList()
{
    if (!d->ref.deref())
        dealloc(d);
}
```
- 关键语句：if (!d->ref.deref())

```cpp
template <typename T>
Q_INLINE_TEMPLATE QList<T> &QList<T>::operator=(const QList<T> &l)
{
    if (d != l.d) {
        QList<T> tmp(l);
        tmp.swap(*this);
    }
    return *this;
}
```
- 关键语句：if (d != l.d) 

```cpp
QListData::Data *QListData::detach_grow(int *idx, int num)
{
    Data *x = d;
    int l = x->end - x->begin;
    int nl = l + num;
    auto blockInfo = qCalculateGrowingBlockSize(nl, sizeof(void *), DataHeaderSize);
    Data* t = static_cast<Data *>(::malloc(blockInfo.size));
    Q_CHECK_PTR(t);
    t->alloc = int(uint(blockInfo.elementCount));

    t->ref.initializeOwned();
    // The space reservation algorithm's optimization is biased towards appending:
    // Something which looks like an append will put the data at the beginning,
    // while something which looks like a prepend will put it in the middle
    // instead of at the end. That's based on the assumption that prepending
    // is uncommon and even an initial prepend will eventually be followed by
    // at least some appends.
    int bg;
    if (*idx < 0) {
        *idx = 0;
        bg = (t->alloc - nl) >> 1;
    } else if (*idx > l) {
        *idx = l;
        bg = 0;
    } else if (*idx < (l >> 1)) {
        bg = (t->alloc - nl) >> 1;
    } else {
        bg = 0;
    }
    t->begin = bg;
    t->end = bg + nl;
    d = t;

    return x;
}

template <typename T>
Q_OUTOFLINE_TEMPLATE typename QList<T>::Node *QList<T>::detach_helper_grow(int i, int c)
{
    Node *n = reinterpret_cast<Node *>(p.begin());
    QListData::Data *x = p.detach_grow(&i, c);
    QT_TRY {
        node_copy(reinterpret_cast<Node *>(p.begin()),
                  reinterpret_cast<Node *>(p.begin() + i), n);
    } QT_CATCH(...) {
        p.dispose();
        d = x;
        QT_RETHROW;
    }
    QT_TRY {
        node_copy(reinterpret_cast<Node *>(p.begin() + i + c),
                  reinterpret_cast<Node *>(p.end()), n + i);
    } QT_CATCH(...) {
        node_destruct(reinterpret_cast<Node *>(p.begin()),
                      reinterpret_cast<Node *>(p.begin() + i));
        p.dispose();
        d = x;
        QT_RETHROW;
    }

    if (!x->ref.deref())
        dealloc(x);

    return reinterpret_cast<Node *>(p.begin() + i);
}

template <typename T>
Q_OUTOFLINE_TEMPLATE void QList<T>::append(const T &t)
{
    if (d->ref.isShared()) {
        Node *n = detach_helper_grow(INT_MAX, 1);
        QT_TRY {
            node_construct(n, t);
        } QT_CATCH(...) {
            --d->end;
            QT_RETHROW;
        }
    } else {
        if (QTypeInfo<T>::isLarge || QTypeInfo<T>::isStatic) {
            Node *n = reinterpret_cast<Node *>(p.append());
            QT_TRY {
                node_construct(n, t);
            } QT_CATCH(...) {
                --d->end;
                QT_RETHROW;
            }
        } else {
            Node *n, copy;
            node_construct(&copy, t); // t might be a reference to an object in the array
            QT_TRY {
                n = reinterpret_cast<Node *>(p.append());;
            } QT_CATCH(...) {
                node_destruct(&copy);
                QT_RETHROW;
            }
            *n = copy;
        }
    }
}
```
- 关键语句：if (d->ref.isShared())
- 关键语句：if (!x->ref.deref())

```cpp
struct Q_CORE_EXPORT QListData {
    ...
    inline void **at(int i) const noexcept { return d->array + d->begin + i; }
    ...
};
template <typename T>
inline const T &QList<T>::operator[](int i) const
{ Q_ASSERT_X(i >= 0 && i < p.size(), "QList<T>::operator[]", "index out of range");
 return reinterpret_cast<Node *>(p.at(i))->t(); }
```
- 关键语句：return d->array + d->begin + i;

```cpp
struct Q_CORE_EXPORT QListData {
    // tags for tag-dispatching of QList implementations,
    // based on QList's three different memory layouts:
    struct NotArrayCompatibleLayout {};
    struct NotIndirectLayout {};
    struct ArrayCompatibleLayout   : NotIndirectLayout {};                           // data laid out like a C array
    struct InlineWithPaddingLayout : NotArrayCompatibleLayout, NotIndirectLayout {}; // data laid out like a C array with padding
    struct IndirectLayout          : NotArrayCompatibleLayout {};                    // data allocated on the heap
    ...
};

template <typename T>
class QList
#ifndef Q_QDOC
    : public QListSpecialMethods<T>
#endif
{
public:
    struct MemoryLayout
        : std::conditional<
            // must stay isStatic until ### Qt 6 for BC reasons (don't use !isRelocatable)!
            QTypeInfo<T>::isStatic || QTypeInfo<T>::isLarge,
            QListData::IndirectLayout,
            typename std::conditional<
                sizeof(T) == sizeof(void*),
                QListData::ArrayCompatibleLayout,
                QListData::InlineWithPaddingLayout
             >::type>::type {};
    ...
};

template <typename T>
Q_OUTOFLINE_TEMPLATE bool QList<T>::operator==(const QList<T> &l) const
{
    if (d == l.d)
        return true;
    if (p.size() != l.p.size())
        return false;
    return this->op_eq_impl(l, MemoryLayout());
}

template <typename T>
inline bool QList<T>::op_eq_impl(const QList &l, QListData::NotArrayCompatibleLayout) const
{
    Node *i = reinterpret_cast<Node *>(p.begin());
    Node *e = reinterpret_cast<Node *>(p.end());
    Node *li = reinterpret_cast<Node *>(l.p.begin());
    for (; i != e; ++i, ++li) {
        if (!(i->t() == li->t()))
            return false;
    }
    return true;
}

template <typename T>
inline bool QList<T>::op_eq_impl(const QList &l, QListData::ArrayCompatibleLayout) const
{
    const T *lb = reinterpret_cast<const T*>(l.p.begin());
    const T *b  = reinterpret_cast<const T*>(p.begin());
    const T *e  = reinterpret_cast<const T*>(p.end());
    return std::equal(b, e, QT_MAKE_CHECKED_ARRAY_ITERATOR(lb, l.p.size()));
}
```
