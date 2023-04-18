##### 一、QA：

1. QSharedPointer 是线程安全的吗？
- 不是。例如， QSharedPointer 拷贝构造函数中 ref() 函数的实现。
    ```cpp
    void ref() const noexcept { d->weakref.ref(); d->strongref.ref(); }
    ```
    - 若需保证线程安全，应对成员变量 d 的两个成员变量锁住进行访问。
- QSharedPointer 的计数器 d 里面的两个成员变量 weakref 和 strongref 是 QBasicAtomicInt 类型的，它俩是线程安全的。

2. 计数器什么时候析构？
- QSharedPointer 的计数器即是它的成员变量 d，而 d 是 ExternalRefCountData 类型的指针。
- 从 QSharedPointer 的析构函数的 deref() 函数实现可以看出，strongref 计数为 0 时就析构管理的资源，weakref 计数为 0 时就析构计数器。

3. sizeof QSharedPointer 是多大？
- 64位平台下是16。因为 QSharedPointer 里面只有两个成员变量。
    ```cpp
    Type *value;
    Data *d;
    ```

4. Qt 也是通过设置一个 QWeakPointer 来解决循环引用的问题，QWeakPointer 中也有获取管理资源指针的函数：
    ```cpp
    // ### Qt 6: remove users of this API; no one should ever access
    // a weak pointer's data but the weak pointer itself
    inline T *internalData() const noexcept
    {
        return d == nullptr || d->strongref.loadRelaxed() == 0 ? nullptr : value;
    }
    ```

---

##### 二、source code：

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/tools/qsharedpointer_impl.h
namespace QtSharedPointer {
    ...
    struct ExternalRefCountData
    {
        typedef void (*DestroyerFn)(ExternalRefCountData *);
        QBasicAtomicInt weakref;
        QBasicAtomicInt strongref;
        DestroyerFn destroyer;

        inline ExternalRefCountData(DestroyerFn d)
            : destroyer(d)
        {
            strongref.storeRelaxed(1);
            weakref.storeRelaxed(1);
        }
        inline ExternalRefCountData(Qt::Initialization) { }
        ~ExternalRefCountData() { Q_ASSERT(!weakref.loadRelaxed()); Q_ASSERT(strongref.loadRelaxed() <= 0); }

        void destroy() { destroyer(this); }

#ifndef QT_NO_QOBJECT
        Q_CORE_EXPORT static ExternalRefCountData *getAndRef(const QObject *);
        Q_CORE_EXPORT void setQObjectShared(const QObject *, bool enable);
        Q_CORE_EXPORT void checkQObjectShared(const QObject *);
#endif
        inline void checkQObjectShared(...) { }
        inline void setQObjectShared(...) { }

        inline void operator delete(void *ptr) { ::operator delete(ptr); }
        inline void operator delete(void *, void *) { }
    };
    ...
};

template <class T> class QSharedPointer
{
    ...
    typedef QtSharedPointer::ExternalRefCountData Data;
    ...
    template <typename X, typename Deleter>
    inline void internalConstruct(X *ptr, Deleter deleter)
    {
        typedef QtSharedPointer::ExternalRefCountWithCustomDeleter<X, Deleter> Private;
# ifdef QT_SHAREDPOINTER_TRACK_POINTERS
        typename Private::DestroyerFn actualDeleter = &Private::safetyCheckDeleter;
# else
        typename Private::DestroyerFn actualDeleter = &Private::deleter;
# endif
        d = Private::create(ptr, deleter, actualDeleter);

#ifdef QT_SHAREDPOINTER_TRACK_POINTERS
        internalSafetyCheckAdd(d, ptr);
#endif
        d->setQObjectShared(ptr, true);
        enableSharedFromThis(ptr);
    }
    ...
public:
    ...
    template <class X, IfCompatible<X> = true>
    inline explicit QSharedPointer(X *ptr) : value(ptr) // noexcept
    { internalConstruct(ptr, QtSharedPointer::NormalDeleter()); }
    ...
    QSharedPointer(const QSharedPointer &other) noexcept : value(other.value), d(other.d)
    { if (d) ref(); }
    ...
    ~QSharedPointer() { deref(); }
    ...
private:
    ...
    void ref() const noexcept { d->weakref.ref(); d->strongref.ref(); }
    ...
    void deref() noexcept
    { deref(d); }
    static void deref(Data *dd) noexcept
    {
        if (!dd) return;
        if (!dd->strongref.deref()) {
            dd->destroy();
        }
        if (!dd->weakref.deref())
            delete dd;
    }
    ...
    Data *d;
```
- 关键语句：void ref() const noexcept { d->weakref.ref(); d->strongref.ref(); }
- 关键语句：if (!dd->weakref.deref()) delete dd;

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/thread/qbasicatomic.h
template <typename T>
class QBasicAtomicInteger
{
    ...
    void storeRelaxed(T newValue) noexcept { Ops::storeRelaxed(_q_value, newValue); }
    ...
};
// ./Qt/5.15.2/Src/qtbase/src/corelib/thread/qgenericatomic.h
// not really atomic...
template <typename BaseClass> struct QGenericAtomicOps
{
    ...
    template <typename T, typename X> static Q_ALWAYS_INLINE
    void storeRelaxed(T &_q_value, X newValue) noexcept
    {
        _q_value = newValue;
    }
    ...
};
// ./Qt/5.15.2/Src/qtbase/src/corelib/thread/qatomic_bootstrap.h
template <typename T> struct QAtomicOps: QGenericAtomicOps<QAtomicOps<T> >
{
    typedef T Type;

    static bool ref(T &_q_value) noexcept
    {
        return ++_q_value != 0;
    }
    static bool deref(T &_q_value) noexcept
    {
        return --_q_value != 0;
    }
    ...
};
```
