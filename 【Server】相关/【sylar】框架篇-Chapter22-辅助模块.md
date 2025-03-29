# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述
- 一些辅助工具类或函数。


# endian.h
- 字节序操作函数(大端/小端)
- 部分相关代码
```C++
#ifndef __SYLAR_ENDIAN_H__
#define __SYLAR_ENDIAN_H__

#include <byteswap.h>
#include <stdint.h>

#define SYLAR_LITTLE_ENDIAN 1
#define SYLAR_BIG_ENDIAN 2

namespace sylar
{

template<class T>
typename std::enable_if<sizeof(T) == sizeof(uint64_t), T>::type
byteswap(T value)
{
    return (T)bswap_64((uint64_t)value);
}

template<class T>
typename std::enable_if<sizeof(T) == sizeof(uint32_t), T>::type
byteswap(T value)
{
    return (T)bswap_32((uint32_t)value);
}

template<class T>
typename std::enable_if<sizeof(T) == sizeof(uint16_t), T>::type
byteswap(T value)
{
    return (T)bswap_16((uint16_t)value);
}

#if BYTE_ORDER == BIG_ENDIAN
#define SYLAR_BYTE_ORDER SYLAR_BIG_ENDIAN
#else
#define SYLAR_BYTE_ORDER SYLAR_LITTLE_ENDIAN
#endif

#if SYLAR_BYTE_ORDER == SYLAR_BIG_ENDIAN

template<class T>
T byteswapOnLittleEndian(T t)
{
    return t;
}

template<class T>
T byteswapOnBigEndian(T t)
{
    return byteswap(t);
}

#else

template<class T>
T byteswapOnLittleEndian(T t)
{
    return byteswap(t);
}

template<class T>
T byteswapOnBigEndian(T t)
{
    return t;
}

#endif

}

#endif
```


# macro.h
- 常用宏的封装
	- 断言宏封装
	- 编译器优化条件判断的宏
- 部分相关代码
```C++
#ifndef __SYLAR_MACRO_H__
#define __SYLAR_MACRO_H__

#include <string.h>
#include <assert.h>
#include "log.h"
#include "util/system_util.h"

#if defined __GNUC__ || defined __llvm__
/// LIKCLY 宏的封装, 告诉编译器优化,条件大概率成立
#   define SYLAR_LIKELY(x)       __builtin_expect(!!(x), 1)
/// LIKCLY 宏的封装, 告诉编译器优化,条件大概率不成立
#   define SYLAR_UNLIKELY(x)     __builtin_expect(!!(x), 0)
#else
#   define SYLAR_LIKELY(x)      (x)
#   define SYLAR_UNLIKELY(x)      (x)
#endif

/// 断言宏封装
#define SYLAR_ASSERT(x) \
    if(SYLAR_UNLIKELY(!(x))) \
    { \
        SYLAR_LOG_ERROR(SYLAR_LOG_ROOT()) << "ASSERTION: " #x \
            << "\nbacktrace:\n" \
            << sylar::BacktraceToString(100, 2, "    "); \
        assert(x); \
    }

/// 断言宏封装
#define SYLAR_ASSERT2(x, w) \
    if(SYLAR_UNLIKELY(!(x))) \
    { \
        SYLAR_LOG_ERROR(SYLAR_LOG_ROOT()) << "ASSERTION: " #x \
            << "\n" << w \
            << "\nbacktrace:\n" \
            << sylar::BacktraceToString(100, 2, "    "); \
        assert(x); \
    }

#endif
```


# mutex.h
- 线程同步类。
- 对信号量、互斥量、读写锁、自旋锁、原子锁进行封装。
	- 信号量是基于 sem_t 实现的。
	- 互斥量是基于 pthread_mutex_t 实现的。
	- 读写锁是基于 pthread_rwlock_t 实现的。
	- 自旋锁是基于 pthread_spinlock_t 实现的。
	- 原子锁是基于 std::atomic_flag 实现的。
- 利用 RAII 思想对上述锁进行模板类实现，类似标准库中的 std::lockguard<T>。
- 部分相关代码
```C++
/**
 * @filename    mutex.h
 * @brief   封装信号量、互斥量、读写锁、自旋锁、原子锁以及利用RAII思想对它们的包装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-11
 */
#ifndef __SYLAR_MUTEX_H__
#define __SYLAR_MUTEX_H__

#include <memory>
#include <atomic>
#include <stdint.h>
#include <semaphore.h>
#include <pthread.h>

#include "noncopyable.h"

namespace sylar
{

template<class T>
struct ScopedLockImpl
{
    ScopedLockImpl(T& mutex)
        : m_mutex(mutex)
    {
        m_mutex.lock();
        m_locked = true;
    }

    ~ScopedLockImpl()
    {
        unlock();
    }

    void lock()
    {
        if(!m_locked)
        {
            m_mutex.lock();
            m_locked = true;
        }
    }

    void unlock()
    {
        if(m_locked)
        {
            m_mutex.unlock();
            m_locked = false;
        }
    }

private:
    T& m_mutex;
    bool m_locked;
};

template<class T>
struct ReadScopedLockImpl
{
    ReadScopedLockImpl(T& mutex)
        : m_mutex(mutex)
    {
        m_mutex.rdlock();
        m_locked = true;
    }

    ~ReadScopedLockImpl()
    {
        unlock();
    }

    void lock()
    {
        if(!m_locked)
        {
            m_mutex.rdlock();
            m_locked = true;
        }
    }

    void unlock()
    {
        if(m_locked)
        {
            m_mutex.unlock();
            m_locked = false;
        }
    }

private:
    T& m_mutex;
    bool m_locked;
};

template<class T>
struct WriteScopedLockImpl
{
    WriteScopedLockImpl(T& mutex)
        : m_mutex(mutex)
    {
        m_mutex.wrlock();
        m_locked = true;
    }

    ~WriteScopedLockImpl()
    {
        unlock();
    }

    void lock()
    {
        if(!m_locked)
        {
            m_mutex.wrlock();
            m_locked = true;
        }
    }

    void unlock()
    {
        if(m_locked)
        {
            m_mutex.unlock();
            m_locked = false;
        }
    }

private:
    T& m_mutex;
    bool m_locked;
};

/**
 * @brief   信号量
 */
class Semaphore : private Noncopyable
{
public:

    /**
     * @brief   构造函数
     *
     * @param   count   信号量值的大小
     */
    Semaphore(uint32_t count = 0);

    ~Semaphore();


    /**
     * @brief   获取信号量
     */
    void wait();


    /**
     * @brief   释放信号量
     */
    void notify();

private:
    sem_t m_semaphore;
};

class Mutex : private Noncopyable
{
public:
    typedef ScopedLockImpl<Mutex> Lock;

    Mutex()
    {
        pthread_mutex_init(&m_mutex, nullptr);
    }

    ~Mutex()
    {
        pthread_mutex_destroy(&m_mutex);
    }

    void lock()
    {
        pthread_mutex_lock(&m_mutex);
    }

    void unlock()
    {
        pthread_mutex_unlock(&m_mutex);
    }

private:
    pthread_mutex_t m_mutex;
};

class RWMutex : Noncopyable
{
public:
    typedef ReadScopedLockImpl<RWMutex> ReadLock;
    typedef WriteScopedLockImpl<RWMutex> WriteLock;

    RWMutex()
    {
        pthread_rwlock_init(&m_lock, nullptr);
    }

    ~RWMutex()
    {
        pthread_rwlock_destroy(&m_lock);
    }

    void rdlock()
    {
        pthread_rwlock_rdlock(&m_lock);
    }

    void wrlock()
    {
        pthread_rwlock_wrlock(&m_lock);
    }

    void unlock()
    {
        pthread_rwlock_unlock(&m_lock);
    }

private:
    pthread_rwlock_t m_lock;
};

class Spinlock : Noncopyable
{
public:
    typedef ScopedLockImpl<Spinlock> Lock;

    Spinlock()
    {
        pthread_spin_init(&m_lock, 0);
    }

    ~Spinlock()
    {
        pthread_spin_destroy(&m_lock);
    }

    void lock()
    {
        pthread_spin_lock(&m_lock);
    }

    void unlock()
    {
        pthread_spin_unlock(&m_lock);
    }

private:
    pthread_spinlock_t m_lock;
};

class CASLock : Noncopyable
{
public:
    typedef ScopedLockImpl<CASLock> Lock;

    CASLock()
    {
        m_flag.clear();
    }

    ~CASLock()
    {}

    void lock()
    {
        while(std::atomic_flag_test_and_set_explicit(&m_flag, std::memory_order_acquire));
    }

    void unlock()
    {
        std::atomic_flag_clear_explicit(&m_flag, std::memory_order_release);
    }

private:
    volatile std::atomic_flag m_flag;
};

}


#endif


#include "mutex.h"
#include <exception>

namespace sylar
{

Semaphore::Semaphore(uint32_t count)
{
    if(sem_init(&m_semaphore, 0, count))
    {
        throw std::logic_error("sem_init error");
    }
}

Semaphore::~Semaphore()
{
    sem_destroy(&m_semaphore);
}

void Semaphore::wait()
{
    if(sem_wait(&m_semaphore))
    {
        throw std::logic_error("sem_wait error");
    }
}

void Semaphore::notify()
{
    if(sem_post(&m_semaphore))
    {
        throw std::logic_error("sem_post error");
    }
}

}
```


# noncopyable.h
- 不可拷贝对象封装类。
- 通过继承该类实现对象的不可拷贝属性。
- 该类主要是把拷贝构造函数和赋值函数声明成 delete 来达到不可拷贝和不可赋值的效果。
- 部分相关代码
```C++
/**
 * @filename    noncopyable.h
 * @brief   不可拷贝对象封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-11
 */
#ifndef __SYLAR_NONCOPYABLE_H__
#define __SYLAR_NONCOPYABLE_H__

namespace sylar
{

class Noncopyable
{
public:
    Noncopyable() = default;

    ~Noncopyable() = default;

    Noncopyable(const Noncopyable&) = delete;

    Noncopyable& operator=(const Noncopyable&) = delete;
};

}

#endif
```


# singleton.h
- 单例模式封装类。
- 内含单例模式封装类和单例模式智能指针封装类。 
- 部分相关代码
```C++
/**
 * @filename    singleton.h
 * @brief   单例模式封装类
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-11
 */
#ifndef __SYLAR_SINGLETON_H__
#define __SYLAR_SINGLETON_H__

#include <memory>

namespace sylar
{

template<class T, class X, int N>
T& GetInstanceX()
{
    static T v;
    return v;
}

template<class T, class X, int N>
std::shared_ptr<T> GetInstancePtr()
{
    static std::shared_ptr<T> v(new T);
    return v;
}

/**
 * @brief   单例模式封装类
 *
 * @tparam t    类型
 * @tparam x    为了创造多个实例对应的tag
 * @tparam n    同一个tag创造多个实例索引
 */
template<class T, class X = void, int N = 0>
class Singleton
{
public:
    static T* GetInstance()
    {
        static T v;
        return &v;
    }
};

/**
 * @brief   单例模式智能指针封装类
 *
 * @tparam t    类型
 * @tparam x    为了创造多个实例对应的tag
 * @tparam n    同一个tag创造多个实例索引
 */
template<class T, class X = void, int N = 0>
class SingletonPtr
{
public:
    static std::shared_ptr<T> GetInstance()
    {
        static std::shared_ptr<T> v(new T);
        return v;
    }
};

}

#endif
```


# filestream_util.h
- 常用的文件流工具函数、文件操作函数的再次封装。
- 部分相关代码
```C++
/**
 * @filename    filestream_util.h
 * @brief   常用的文件流工具函数
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-11
 */
#ifndef __SYLAR_FILESTREAM_UTIL_H__
#define __SYLAR_FILESTREAM_UTIL_H__

#include <fstream>
#include <unistd.h>
#include <sys/types.h>
#include <sys/syscall.h>
#include <stdio.h>
#include <cxxabi.h>
#include <sys/stat.h>
#include <string.h>
#include <vector>

namespace sylar
{

class FSUtil
{
public:
    static bool Mkdir(const std::string& dirname);
    static std::string Dirname(const std::string& filename);
    static bool OpenForWrite(std::ofstream& ofs, const std::string& filename, std::ios_base::openmode mode);
    static void ListAllFile(std::vector<std::string>& files, const std::string& path, const std::string& subfix);
    static bool Unlink(const std::string& filename, bool exist = false);
    static bool IsRunningPidfile(const std::string& pidfile);
    static bool Rm(const std::string& path);
    static bool Mv(const std::string& from, const std::string& to);
};

}

#endif

#include "filestream_util.h"
#include <dirent.h>
#include <sys/types.h>
#include <signal.h>

namespace sylar
{

static int __lstat(const char* file, struct stat* st = nullptr)
{
    // lstat函数是获取参数file所指的文件状态，
    // 当文件为符合链接时，该函数会返回该link本身的状态。
    struct stat lst;
    int ret = lstat(file, &lst);
    if(st)
    {
        *st = lst;
    }

    return ret;
}

static int __mkdir(const char* dirname)
{
    if(access(dirname, F_OK) == 0)
    {
        return 0;
    }

    return mkdir(dirname, S_IRWXU | S_IRWXG | S_IROTH | S_IXOTH);
}

bool FSUtil::Mkdir(const std::string& dirname)
{
    if(__lstat(dirname.c_str()) == 0)
    {
        return true;
    }

    char* path = strdup(dirname.c_str());
    char* ptr = strchr(path+1, '/');
    do
    {
        for(; ptr; *ptr = '/', ptr = strchr(ptr+1, '/'))
        {
            *ptr = '\0';
            if(__mkdir(path) != 0)
            {
                break;
            }
        }

        if(ptr != nullptr)
        {
            break;
        }
        else if(__mkdir(path) != 0)
        {
            break;
        }

        free(path);
        return true;
    }while(0);

    free(path);
    return false;
}

std::string FSUtil::Dirname(const std::string& filename)
{
    if(filename.empty())
    {
        return ".";
    }
    auto pos = filename.rfind('/');
    if(pos == 0)
    {
        return "/";
    }
    else if(pos == std::string::npos)
    {
        return ".";
    }
    
    return filename.substr(0, pos);
}

bool FSUtil::OpenForWrite(std::ofstream& ofs, const std::string& filename, std::ios_base::openmode mode)
{
    ofs.open(filename.c_str(), mode);
    if(!ofs.is_open())
    {
        std::string dir = Dirname(filename);
        Mkdir(dir);
        ofs.open(filename.c_str(), mode);
    }

    return ofs.is_open();
}

void FSUtil::ListAllFile(std::vector<std::string>& files, const std::string& path, const std::string& subfix)
{
    if(access(path.c_str(), 0) != 0)
    {
        return;
    }
    DIR* dir = opendir(path.c_str());
    if(dir == nullptr)
    {
        return;
    }
    struct dirent* dp = nullptr;
    while((dp = readdir(dir)) != nullptr)
    {
        if(dp->d_type == DT_DIR)
        {
            if(!strcmp(dp->d_name, ".")
                    || !strcmp(dp->d_name, ".."))
            {
                continue;
            }
            ListAllFile(files, path+"/"+dp->d_name, subfix);
        }
        else if(dp->d_type == DT_REG)
        {
            std::string filename(dp->d_name);
            if(subfix.empty())
            {
                files.push_back(path+"/"+filename);
            }
            else
            {
                if(filename.size() < subfix.size())
                {
                    continue;
                }
                if(filename.substr(filename.length()-subfix.size()) == subfix)
                {
                    files.push_back(path+"/"+filename);
                }
            }
        }
    }
    closedir(dir);
}

bool FSUtil::Unlink(const std::string& filename, bool exist)
{
    if(!exist && __lstat(filename.c_str()))
    {
        return false;
    }
    // unlink函数：从文件系统中删除一个名称。
    // 如果名称是文件的最后一个连接，并且没有其他进程将文件打开，名称对应的文件会实际被删除。
    return ::unlink(filename.c_str()) == 0;
}

bool FSUtil::IsRunningPidfile(const std::string& pidfile) 
{
    if(__lstat(pidfile.c_str()) != 0) 
    {
        return false;
    }
    std::ifstream ifs(pidfile);
    std::string line;
    if(!ifs || !std::getline(ifs, line)) 
    {
        return false;
    }
    if(line.empty()) 
    {
        return false;
    }
    pid_t pid = atoi(line.c_str());
    if(pid <= 1) 
    {
        return false;
    }
    if(kill(pid, 0) != 0)
    {
        return false;
    }
    return true;
}

bool FSUtil::Rm(const std::string& path)
{
    struct stat st;
    if(lstat(path.c_str(), &st)) 
    {
        return true;
    }
    
    if(!(st.st_mode & S_IFDIR))
    {
        return Unlink(path);
    }

    DIR* dir = opendir(path.c_str());
    if(!dir) 
    {
        return false;
    }
    
    bool ret = true;
    struct dirent* dp = nullptr;
    while((dp = readdir(dir)))
    {
        if(!strcmp(dp->d_name, ".")
                || !strcmp(dp->d_name, "..")) 
        {
            continue;
        }
        std::string dirname = path + "/" + dp->d_name;
        ret = Rm(dirname);
    }
    closedir(dir);
    if(::rmdir(path.c_str()))
    {
        ret = false;
    }
    return ret;
}

bool FSUtil::Mv(const std::string& from, const std::string& to)
{
    if(!Rm(to))
    {
        return false;
    }
    return rename(from.c_str(), to.c_str()) == 0;
}

}
```


# language_util.h
- 常用的与语言相关的函数，例如 typeid 这种函数。
- 部分相关代码
```C++
#ifndef __SYLAR_LANGUAGE_UTIL_H__
#define __SYLAR_LANGUAGE_UTIL_H__

#include <cxxabi.h>

namespace sylar
{

template<class T>
const char* TypeToName()
{
    static const char* s_name = abi::__cxa_demangle(typeid(T).name(), nullptr, nullptr, nullptr);
    return s_name;
}

}

#endif
```


# string_util.h
- 常用的字符串函数的封装。
- 部分相关代码
```C++
#ifndef __SYLAR_STRING_UTIL_H__
#define __SYLAR_STRING_UTIL_H__

#include <string>

namespace sylar
{

class StringUtil
{
public:
    static std::string UrlEncode(const std::string& str, bool space_as_plus = true);
    static std::string UrlDecode(const std::string& str, bool space_as_plus = true);

    static std::string Trim(const std::string& str, const std::string& delimit = " \t\r\n");
    static std::string TrimLeft(const std::string& str, const std::string& delimit = " \t\r\n");
    static std::string TrimRight(const std::string& str, const std::string& delimit = " \t\r\n");

};

}
#endif


#include "string_util.h"

namespace sylar
{
static const char uri_chars[256] = 
{
    /* 0 */
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 1, 1, 0,
    1, 1, 1, 1, 1, 1, 1, 1,   1, 1, 0, 0, 0, 1, 0, 0,
    /* 64 */
    0, 1, 1, 1, 1, 1, 1, 1,   1, 1, 1, 1, 1, 1, 1, 1,
    1, 1, 1, 1, 1, 1, 1, 1,   1, 1, 1, 0, 0, 0, 0, 1,
    0, 1, 1, 1, 1, 1, 1, 1,   1, 1, 1, 1, 1, 1, 1, 1,
    1, 1, 1, 1, 1, 1, 1, 1,   1, 1, 1, 0, 0, 0, 1, 0,
    /* 128 */
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    /* 192 */
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0, 0, 0,
};

static const char xdigit_chars[256] =
{
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,1,2,3,4,5,6,7,8,9,0,0,0,0,0,0,
    0,10,11,12,13,14,15,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,10,11,12,13,14,15,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
};

#define CHAR_IS_UNRESERVED(c) \
    (uri_chars[(unsigned char)(c)])

//-.0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_abcdefghijklmnopqrstuvwxyz~
std::string StringUtil::UrlEncode(const std::string& str, bool space_as_plus) 
{
    static const char *hexdigits = "0123456789ABCDEF";
    std::string* ss = nullptr;
    const char* end = str.c_str() + str.length();
    for(const char* c = str.c_str() ; c < end; ++c) 
    {
        if(!CHAR_IS_UNRESERVED(*c))
        {
            if(!ss)
            {
                ss = new std::string;
                ss->reserve(str.size() * 1.2);
                ss->append(str.c_str(), c - str.c_str());
            }

            if(*c == ' ' && space_as_plus)
            {
                ss->append(1, '+');
            }
            else
            {
                ss->append(1, '%');
                ss->append(1, hexdigits[(uint8_t)*c >> 4]);
                ss->append(1, hexdigits[*c & 0xf]);
            }
        } 
        else if(ss)
        {
            ss->append(1, *c);
        }
    }

    if(!ss)
    {
        return str;
    }
    else
    {
        std::string rt = *ss;
        delete ss;
        return rt;
    }
}

std::string StringUtil::UrlDecode(const std::string& str, bool space_as_plus) 
{
    std::string* ss = nullptr;
    const char* end = str.c_str() + str.length();
    for(const char* c = str.c_str(); c < end; ++c) 
    {
        if(*c == '+' && space_as_plus) 
        {
            if(!ss) 
            {
                ss = new std::string;
                ss->append(str.c_str(), c - str.c_str());
            }
            ss->append(1, ' ');
        } 
        else if(*c == '%' && (c + 2) < end
                    && isxdigit(*(c + 1)) 
                    && isxdigit(*(c + 2)))
        {
            if(!ss) 
            {
                ss = new std::string;
                ss->append(str.c_str(), c - str.c_str());
            }
            ss->append(1, (char)(xdigit_chars[(int)*(c + 1)] << 4 | xdigit_chars[(int)*(c + 2)]));
            c += 2;
        } 
        else if(ss) 
        {
            ss->append(1, *c);
        }
    }

    if(!ss) 
    {
        return str;
    } 
    else 
    {
        std::string rt = *ss;
        delete ss;
        return rt;
    }
}

std::string StringUtil::Trim(const std::string& str, const std::string& delimit) 
{
    auto begin = str.find_first_not_of(delimit);
    if(begin == std::string::npos)
    {
        return "";
    }
    auto end = str.find_last_not_of(delimit);
    return str.substr(begin, end - begin + 1);
}

std::string StringUtil::TrimLeft(const std::string& str, const std::string& delimit) 
{
    auto begin = str.find_first_not_of(delimit);
    if(begin == std::string::npos) 
    {
        return "";
    }
    return str.substr(begin);
}

std::string StringUtil::TrimRight(const std::string& str, const std::string& delimit) 
{
    auto end = str.find_last_not_of(delimit);
    if(end == std::string::npos)
    {
        return "";
    }
    return str.substr(0, end);
}

}
```


# system_util.h
- 常用的系统函数的封装。
- 部分相关代码
```C++
/**
 * @filename    system_util.h
 * @brief   常用的系统工具函数
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-11
 */
#ifndef __SYLAR_SYSTEM_UTIL_H__
#define __SYLAR_SYSTEM_UTIL_H__

#include <unistd.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <stdint.h>
#include <sys/time.h>
#include <string>
#include <vector>

namespace sylar
{

pid_t GetThreadId();
uint32_t GetFiberId();

std::string demangle(const char* str);
void Backtrace(std::vector<std::string>& bt, int size, int skip);
std::string BacktraceToString(int size = 64, int skip = 2, const std::string& prefix = "");


/**
 * @brief  获取当前时间的毫秒 
 */
uint64_t GetCurrentMS();

/**
 * @brief   获取当前时间的微秒
 */
uint64_t GetCurrentUS();

std::string Time2Str(time_t ts = time(0), const std::string& format = "%Y-%m-%d %H:%M:%S");
time_t Str2Time(const char* str, const char* format = "%Y-%m-%d %H:%M:%S");

}

#endif


#include "system_util.h"
#include <execinfo.h>

#include "../log.h"
#include "../fiber.h"

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

pid_t GetThreadId()
{
    return syscall(SYS_gettid);
}

uint32_t GetFiberId()
{
    return sylar::Fiber::GetFiberId();
}

std::string demangle(const char* str)
{
    size_t size = 0;
    int status = 0;
    std::string rt;
    rt.resize(256);
    if(1 == sscanf(str, "%*[^(]%*[^_]%255[^)+]", &rt[0])) {
        char* v = abi::__cxa_demangle(&rt[0], nullptr, &size, &status);
        if(v) {
            std::string result(v);
            free(v);
            return result;
        }
    }
    if(1 == sscanf(str, "%255s", &rt[0])) {
        return rt;
    }
    return str;
}

void Backtrace(std::vector<std::string>& bt, int size, int skip)
{
    void** array = (void**)malloc((sizeof(void*) * size));
    size_t s = ::backtrace(array, size);

    char** strings = backtrace_symbols(array, s);
    if(strings == NULL)
    {
        SYLAR_LOG_ERROR(g_logger) << "backtrace_synbols error";
        return;
    }

    for(size_t i = skip; i < s; ++i) 
    {
        bt.push_back(demangle(strings[i]));
    }

    free(strings);
    free(array);
}

std::string BacktraceToString(int size, int skip, const std::string& prefix)
{
    std::vector<std::string> bt;
    Backtrace(bt, size, skip);
    std::stringstream ss;
    for(size_t i = 0; i < bt.size(); ++i) 
    {
        ss << prefix << bt[i] << std::endl;
    }
    return ss.str();
}

uint64_t GetCurrentMS()
{
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_sec * 1000ul + tv.tv_usec / 1000;
}

uint64_t GetCurrentUS()
{
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_sec * 1000 * 1000ul + tv.tv_usec;
}

std::string Time2Str(time_t ts, const std::string& format) 
{
    struct tm tm;
    localtime_r(&ts, &tm);
    char buf[64];
    strftime(buf, sizeof(buf), format.c_str(), &tm);
    return buf;
}

time_t Str2Time(const char* str, const char* format)
{
    struct tm t;
    memset(&t, 0, sizeof(t));
    if(!strptime(str, format, &t))
    {
        return 0;
    }
    return mktime(&t);
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
