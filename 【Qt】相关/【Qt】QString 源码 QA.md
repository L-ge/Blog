##### 一、QA：

1. QString 内部的数据结构是 QTypedArrayData<ushort>，而 QTypedArrayData 继承自 QArrayData。QArrayData 有个 QtPrivate::RefCount 类型的成员变量 ref，该成员变量记录着该内存块的引用。也就是说，QString 采用了 Copy On Write 的技术优化了存放字符串的内存块。

2. 可以从 QString::QString(const QString &other)、QString::append(const QString &str) 的实现看出的确是采用了 COW 技术。

---

##### 二、source code：

```cpp
typedef QTypedArrayData<ushort> QStringData;
typedef QStringData Data;
Data *d;
```

```cpp
QString::QString(int size, QChar ch)
{
   if (size <= 0) {
        d = Data::allocate(0);
    } else {
        d = Data::allocate(size + 1);
        Q_CHECK_PTR(d);
        d->size = size;
        d->data()[size] = '\0';
        ushort *i = d->data() + size;
        ushort *b = d->data();
        const ushort value = ch.unicode();
        while (i != b)
           *--i = value;
    }
}
```
- 关键语句：d = Data::allocate(size + 1);、d->data()[size] = '\0';

```cpp
inline QString::QString(const QString &other) noexcept : d(other.d)
{ Q_ASSERT(&other != this); d->ref.ref(); }

QString &QString::operator=(const QString &other) noexcept
{
    other.d->ref.ref();
    if (!d->ref.deref())
        Data::deallocate(d);
    d = other.d;
    return *this;
}
```
- 关键语句：d->ref.ref();

```cpp
struct Q_CORE_EXPORT QArrayData
{
    QtPrivate::RefCount ref;
    int size;
    uint alloc : 31;
    uint capacityReserved : 1;

    qptrdiff offset; // in bytes from beginning of header

    void *data()
    {
        Q_ASSERT(size == 0
                || offset < 0 || size_t(offset) >= sizeof(QArrayData));
        return reinterpret_cast<char *>(this) + offset;
    }
    ...
};

template <class T>
struct QTypedArrayData
    : QArrayData
{
    ...
    class AlignmentDummy { QArrayData header; T data; };
    
    Q_REQUIRED_RESULT static QTypedArrayData *allocate(size_t capacity,
            AllocationOptions options = Default)
    {
        Q_STATIC_ASSERT(sizeof(QTypedArrayData) == sizeof(QArrayData));
        return static_cast<QTypedArrayData *>(QArrayData::allocate(sizeof(T),
                    Q_ALIGNOF(AlignmentDummy), capacity, options));
    }
    ...
};
```
- 关键语句：sizeof(T)

```cpp
template <typename T> inline
typename std::enable_if<std::is_unsigned<T>::value || std::is_signed<T>::value, bool>::type
add_overflow(T v1, T v2, T *r)
{ return __builtin_add_overflow(v1, v2, r); }

template <typename T> inline
typename std::enable_if<std::is_unsigned<T>::value || std::is_signed<T>::value, bool>::type
mul_overflow(T v1, T v2, T *r)
{ return __builtin_mul_overflow(v1, v2, r); }

size_t qCalculateBlockSize(size_t elementCount, size_t elementSize, size_t headerSize) noexcept
{
    unsigned count = unsigned(elementCount);
    unsigned size = unsigned(elementSize);
    unsigned header = unsigned(headerSize);
    Q_ASSERT(elementSize);
    Q_ASSERT(size == elementSize);
    Q_ASSERT(header == headerSize);

    if (Q_UNLIKELY(count != elementCount))
        return std::numeric_limits<size_t>::max();

    unsigned bytes;
    if (Q_UNLIKELY(mul_overflow(size, count, &bytes)) ||
            Q_UNLIKELY(add_overflow(bytes, header, &bytes)))
        return std::numeric_limits<size_t>::max();
    if (Q_UNLIKELY(int(bytes) < 0))     // catches bytes >= 2GB
        return std::numeric_limits<size_t>::max();

    return bytes;
}

static inline size_t calculateBlockSize(size_t &capacity, size_t objectSize, size_t headerSize,
                                        uint options)
{
    // Calculate the byte size
    // allocSize = objectSize * capacity + headerSize, but checked for overflow
    // plus padded to grow in size
    if (options & QArrayData::Grow) {
        auto r = qCalculateGrowingBlockSize(capacity, objectSize, headerSize);
        capacity = r.elementCount;
        return r.size;
    } else {
        return qCalculateBlockSize(capacity, objectSize, headerSize);
    }
}

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
```
- 关键语句：header->ref.atomic.storeRelaxed(1);

```cpp
class RefCount
{
    ...
    bool isShared() const noexcept
    {
        int count = atomic.loadRelaxed();
        return (count != 1) && (count != 0);
    }
    ...
};

struct Q_CORE_EXPORT QArrayData
{
    static const QArrayData shared_null[2];
    static QArrayData *sharedNull() noexcept { return const_cast<QArrayData*>(shared_null); }

};

template <class T>
struct QTypedArrayData
    : QArrayData
{
    ...
    static QTypedArrayData *sharedNull() noexcept
    {
        Q_STATIC_ASSERT(sizeof(QTypedArrayData) == sizeof(QArrayData));
        return static_cast<QTypedArrayData *>(QArrayData::sharedNull());
    }
    ...
};

inline QString::QString() noexcept : d(Data::sharedNull()) {}

void QString::reallocData(uint alloc, bool grow)
{
    auto allocOptions = d->detachFlags();
    if (grow)
        allocOptions |= QArrayData::Grow;

    if (d->ref.isShared() || IS_RAW_DATA(d)) {
        Data *x = Data::allocate(alloc, allocOptions);
        Q_CHECK_PTR(x);
        x->size = qMin(int(alloc) - 1, d->size);
        ::memcpy(x->data(), d->data(), x->size * sizeof(QChar));
        x->data()[x->size] = 0;
        if (!d->ref.deref())
            Data::deallocate(d);
        d = x;
    } else {
        Data *p = Data::reallocateUnaligned(d, alloc, allocOptions);
        Q_CHECK_PTR(p);
        d = p;
    }
}

QString &QString::append(const QString &str)
{
    if (str.d != Data::sharedNull()) {
        if (d == Data::sharedNull()) {
            operator=(str);
        } else {
            if (d->ref.isShared() || uint(d->size + str.d->size) + 1u > d->alloc)
                reallocData(uint(d->size + str.d->size) + 1u, true);
            memcpy(d->data() + d->size, str.d->data(), str.d->size * sizeof(QChar));
            d->size += str.d->size;
            d->data()[d->size] = '\0';
        }
    }
    return *this;
}
```
- 关键语句：Data::deallocate(d);
- 关键语句：reallocData(uint(d->size + str.d->size) + 1u, true);、memcpy(d->data() + d->size, str.d->data(), str.d->size * sizeof(QChar));
