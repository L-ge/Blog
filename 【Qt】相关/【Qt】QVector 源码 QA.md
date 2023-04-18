##### 一、QA：

1. 1. QVector 内部的数据结构是 QTypedArrayData<T>，而 QTypedArrayData 继承自 QArrayData。QArrayData 有个 QtPrivate::RefCount 类型的成员变量 ref，该成员变量记录着该内存块的引用。也就是说，QVector 采用了 Copy On Write 的技术优化了存放数据的内存块。

2. 可以从 QVector<T>::QVector(const QVector<T> &v)、QVector<T>::append(const T &t) 的实现看出的确是采用了 COW 技术。

3. QVector 的拷贝构造函数中调用的 QVector<T>::copyConstruct 有一个小小的优化，就是通过判断 T 是否是 is_trivial 来决定是逐个元素调用 T 的构造函数还是直接 memcpy。

---

##### 二、source code：

```cpp
struct Q_CORE_EXPORT QArrayData
{
    QtPrivate::RefCount ref;
    int size;
    uint alloc : 31;
    uint capacityReserved : 1;

    qptrdiff offset; // in bytes from beginning of header

    QArrayData *QArrayData::allocate(size_t objectSize, size_t alignment,
            size_t capacity, AllocationOptions options) noexcept
    {
        // Alignment is a power of two
        Q_ASSERT(alignment >= Q_ALIGNOF(QArrayData)
                && !(alignment & (alignment - 1)));

        // Don't allocate empty headers
        if (!(options & RawData) && !capacity) {
    #if !defined(QT_NO_UNSHARABLE_CONTAINERS)
            if (options & Unsharable)
                return const_cast<QArrayData *>(&qt_array_unsharable_empty);
    #endif
            return const_cast<QArrayData *>(&qt_array_empty);
        }

        size_t headerSize = sizeof(QArrayData);

        // Allocate extra (alignment - Q_ALIGNOF(QArrayData)) padding bytes so we
        // can properly align the data array. This assumes malloc is able to
        // provide appropriate alignment for the header -- as it should!
        // Padding is skipped when allocating a header for RawData.
        if (!(options & RawData))
            headerSize += (alignment - Q_ALIGNOF(QArrayData));

        if (headerSize > size_t(MaxAllocSize))
            return nullptr;

        size_t allocSize = calculateBlockSize(capacity, objectSize, headerSize, options);
        QArrayData *header = static_cast<QArrayData *>(::malloc(allocSize));
        if (header) {
            quintptr data = (quintptr(header) + sizeof(QArrayData) + alignment - 1)
                    & ~(alignment - 1);

    #if !defined(QT_NO_UNSHARABLE_CONTAINERS)
            header->ref.atomic.storeRelaxed(bool(!(options & Unsharable)));
    #else
            header->ref.atomic.storeRelaxed(1);
    #endif
            header->size = 0;
            header->alloc = capacity;
            header->capacityReserved = bool(options & CapacityReserved);
            header->offset = data - quintptr(header);
        }

        return header;
    }
};

template <class T>
struct QTypedArrayData
    : QArrayData
{
    ...
    Q_REQUIRED_RESULT static QTypedArrayData *allocate(size_t capacity,
            AllocationOptions options = Default)
    {
        Q_STATIC_ASSERT(sizeof(QTypedArrayData) == sizeof(QArrayData));
        return static_cast<QTypedArrayData *>(QArrayData::allocate(sizeof(T),
                    Q_ALIGNOF(AlignmentDummy), capacity, options));
    }
    ...
};

template <typename T>
QVector<T>::QVector(int asize, const T &t)
{
    Q_ASSERT_X(asize >= 0, "QVector::QVector", "Size must be greater than or equal to 0.");
    if (asize > 0) {
        d = Data::allocate(asize);
        Q_CHECK_PTR(d);
        d->size = asize;
        T* i = d->end();
        while (i != d->begin())
            new (--i) T(t);
    } else {
        d = Data::sharedNull();
    }
}
```
- 关键语句：d = Data::allocate(asize);、new (--i) T(t);
- 关键语句：header->ref.atomic.storeRelaxed(1);

```cpp
template <typename T>
void QVector<T>::copyConstruct(const T *srcFrom, const T *srcTo, T *dstFrom)
{
    if (QTypeInfo<T>::isComplex) {
        while (srcFrom != srcTo)
            new (dstFrom++) T(*srcFrom++);
    } else {
        ::memcpy(static_cast<void *>(dstFrom), static_cast<const void *>(srcFrom), (srcTo - srcFrom) * sizeof(T));
    }
}

template <typename T>
inline QVector<T>::QVector(const QVector<T> &v)
{
    if (v.d->ref.ref()) {
        d = v.d;
    } else {
        if (v.d->capacityReserved) {
            d = Data::allocate(v.d->alloc);
            Q_CHECK_PTR(d);
            d->capacityReserved = true;
        } else {
            d = Data::allocate(v.d->size);
            Q_CHECK_PTR(d);
        }
        if (d->alloc) {
            copyConstruct(v.d->begin(), v.d->end(), d->begin());
            d->size = v.d->size;
        }
    }
}

inline ~QVector() { if (!d->ref.deref()) freeData(d); }
```
- 关键语句：if (QTypeInfo<T>::isComplex)
- 关键语句：d = v.d;
- 关键语句：if (!d->ref.deref()) freeData(d);

```cpp
template <typename T>
QVector<T> &QVector<T>::operator=(const QVector<T> &v)
{
    if (v.d != d) {
        QVector<T> tmp(v);
        tmp.swap(*this);
    }
    return *this;
}
```
- 关键语句：QVector<T> tmp(v);

```cpp
inline bool isDetached() const { return !d->ref.isShared(); }

template<typename T>
void QVector<T>::realloc(int aalloc, QArrayData::AllocationOptions options)
{
    Q_ASSERT(aalloc >= d->size);
    Data *x = d;

    const bool isShared = d->ref.isShared();

    QT_TRY {
        // allocate memory
        x = Data::allocate(aalloc, options);
        Q_CHECK_PTR(x);
        // aalloc is bigger then 0 so it is not [un]sharedEmpty
#if !defined(QT_NO_UNSHARABLE_CONTAINERS)
        Q_ASSERT(x->ref.isSharable() || options.testFlag(QArrayData::Unsharable));
#endif
        Q_ASSERT(!x->ref.isStatic());
        x->size = d->size;

        T *srcBegin = d->begin();
        T *srcEnd = d->end();
        T *dst = x->begin();

        if (!QTypeInfoQuery<T>::isRelocatable || (isShared && QTypeInfo<T>::isComplex)) {
            QT_TRY {
                if (isShared || !std::is_nothrow_move_constructible<T>::value) {
                    // we can not move the data, we need to copy construct it
                    while (srcBegin != srcEnd)
                        new (dst++) T(*srcBegin++);
                } else {
                    while (srcBegin != srcEnd)
                        new (dst++) T(std::move(*srcBegin++));
                }
            } QT_CATCH (...) {
                // destruct already copied objects
                destruct(x->begin(), dst);
                QT_RETHROW;
            }
        } else {
            ::memcpy(static_cast<void *>(dst), static_cast<void *>(srcBegin), (srcEnd - srcBegin) * sizeof(T));
            dst += srcEnd - srcBegin;
        }

    } QT_CATCH (...) {
        Data::deallocate(x);
        QT_RETHROW;
    }
    x->capacityReserved = d->capacityReserved;

    Q_ASSERT(d != x);
    if (!d->ref.deref()) {
        if (!QTypeInfoQuery<T>::isRelocatable || !aalloc || (isShared && QTypeInfo<T>::isComplex)) {
            // data was copy constructed, we need to call destructors
            // or if !alloc we did nothing to the old 'd'.
            freeData(d);
        } else {
            Data::deallocate(d);
        }
    }
    d = x;

    Q_ASSERT(d->data());
    Q_ASSERT(uint(d->size) <= d->alloc);
#if !defined(QT_NO_UNSHARABLE_CONTAINERS)
    Q_ASSERT(d != Data::unsharableEmpty());
#endif
    Q_ASSERT(d != Data::sharedNull());
    Q_ASSERT(d->alloc >= uint(aalloc));
}

template <typename T>
void QVector<T>::append(const T &t)
{
    const bool isTooSmall = uint(d->size + 1) > d->alloc;
    if (!isDetached() || isTooSmall) {
        T copy(t);
        QArrayData::AllocationOptions opt(isTooSmall ? QArrayData::Grow : QArrayData::Default);
        realloc(isTooSmall ? d->size + 1 : d->alloc, opt);

        if (QTypeInfo<T>::isComplex)
            new (d->end()) T(std::move(copy));
        else
            *d->end() = std::move(copy);

    } else {
        if (QTypeInfo<T>::isComplex)
            new (d->end()) T(t);
        else
            *d->end() = t;
    }
    ++d->size;
}
```
- 关键语句：if (!d->ref.deref())、freeData(d);、d = x;
