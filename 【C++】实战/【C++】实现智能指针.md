1、智能指针类，包括 AMyUniquePtr、AMySharedPtr、AMyWeakPtr。
```C++
#ifndef AMYSMARTPOINTER_H
#define AMYSMARTPOINTER_H

#include <atomic>
#include <mutex>

class ACounter
{
public:
    ACounter()
        : m_nSharedCnt(0)
        , m_nWeakCnt(0)
    {}

    void add()
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        ++m_nSharedCnt;
        ++m_nWeakCnt;
    }
    void addSharedCnt()
    {
        ++m_nSharedCnt;
    }
    void addWeakCnt()
    {
        ++m_nWeakCnt;
    }

    void minus()
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        --m_nSharedCnt;
        --m_nWeakCnt;
    }
    void minusSharedCnt()
    {
        --m_nSharedCnt;
    }
    void minusWeakCnt()
    {
        --m_nWeakCnt;
    }

    int getSharedCnt() const
    {
        return m_nSharedCnt;
    }
    int getWeakCnt() const
    {
        return m_nWeakCnt;
    }

private:
    std::atomic<int> m_nSharedCnt;
    std::atomic<int> m_nWeakCnt;
    std::mutex m_mtx;
};

// ①利用RAII的思想取管理资源，即类初始化时，获取资源；类析构时，释放资源。
// ②为了保证资源独占性，禁用拷贝函数。
template <typename T>
class AMyUniquePtr
{
public:
    AMyUniquePtr(T* pPtr = nullptr)
        : m_pPtr(pPtr)
    {}

    AMyUniquePtr(AMyUniquePtr<T>&& other)
    {
        m_pPtr = other.m_pPtr;
        other.m_pPtr = nullptr;
    }

    AMyUniquePtr<T>& operator=(AMyUniquePtr<T>&& other)
    {
        clean();
        m_pPtr = other.m_pPtr;
        other.m_pPtr = nullptr;
        return *this;
    }

    ~AMyUniquePtr()
    {
        clean();
    }

    T* get() const
    {
        return m_pPtr;
    }

    T* release()
    {
        T* old = m_pPtr;
        m_pPtr = nullptr;
        return old;
    }

    void reset(T* pPtr)
    {
        clean();
        m_pPtr = pPtr;
    }

    T* operator->() const
    {
        return m_pPtr;
    }

    T& operator*() const
    {
        return *m_pPtr;
    }

    operator bool() const
    {
        return m_pPtr;
    }

private:
    AMyUniquePtr(const AMyUniquePtr<T>& other) = delete;
    AMyUniquePtr<T>& operator=(const AMyUniquePtr<T>& other) = delete;

    void clean()
    {
        if(nullptr != m_pPtr)
        {
            delete m_pPtr;
            m_pPtr = nullptr;
        }
    }

private:
    T* m_pPtr;
};

template <typename T>
class AMyWeakPtr;

template <typename T>
class AMySharedPtr
{
    friend class AMyWeakPtr<T>;
public:
    AMySharedPtr()
        : m_pPtr(nullptr)
        , m_pCnter(new ACounter())
    {}

    AMySharedPtr(T* pPtr)
        : m_pPtr(pPtr)
        , m_pCnter(new ACounter())
    {
        m_pCnter->addSharedCnt();
    }

    AMySharedPtr(const AMySharedPtr<T>& other)
    {
        m_pPtr = other.m_pPtr;
        other.m_pCnter->addSharedCnt();
        m_pCnter = other.m_pCnter;
    }

    AMySharedPtr(const AMyWeakPtr<T>& other)
    {
        m_pPtr = other.m_pPtr;
        other.m_pCnter->addWeakCnt();
        m_pCnter = other.m_pCnter;
    }

    AMySharedPtr(AMySharedPtr<T>&& other)
    {
        m_pPtr = other.m_pPtr;
        m_pCnter = other.m_pCnter;
        other.m_pPtr = nullptr;
        other.m_pCnter = nullptr;
    }

    AMySharedPtr<T>& operator=(const AMySharedPtr<T>& other)
    {
        if(this == &other)
            return *this;

        clean();
        m_pPtr = other.m_pPtr;
        other.m_pCnter->addSharedCnt();
        m_pCnter = other.m_pCnter;
        return *this;
    }

    AMySharedPtr<T>& operator=(AMySharedPtr<T>&& other)
    {
        clean();
        m_pPtr = other.m_pPtr;
        m_pCnter = other.m_pCnter;
        other.m_pPtr = nullptr;
        other.m_pCnter = nullptr;
        return *this;
    }

    ~AMySharedPtr()
    {
        clean();
    }

    int use_count()
    {
        return m_pCnter->getSharedCnt();
    }

    T* get() const
    {
        return m_pPtr;
    }

    T* operator->() const
    {
        return m_pPtr;
    }

    T& operator*() const
    {
        return *m_pPtr;
    }

    operator bool() const
    {
        return m_pPtr;
    }

private:
    void clean()
    {
        if(nullptr != m_pCnter)
        {
            m_pCnter->minusSharedCnt();
            if(m_pCnter->getSharedCnt() < 1)
            {
                if(nullptr != m_pPtr)
                {
                    delete m_pPtr;
                    m_pPtr = nullptr;
                }
                if(m_pCnter->getWeakCnt() < 1)
                {
                    delete m_pCnter;
                    m_pCnter = nullptr;
                }
            }
        }
    }

private:
    T* m_pPtr;
    ACounter* m_pCnter;
};

// ①weak_ptr 是为了配合 shared_ptr 而引入的一种智能指针，它指向一个由 shared_ptr 管理的对象而不影响所指对象的生命周期，也就是将一个 weak_ptr 绑定到一个 shared_ptr，但不会改变 shared_ptr 的引用计数。
// ②两个对象相互使用一个 shared_ptr 成员变量指向对方，会造成循环引用，使引用计数失效，从而导致内存泄漏。
// ③weak_ptr 一般通过 share_ptr 来构造，通过 expired 函数检查原始指针是否为空，lock 来转化为 share_ptr。
template <typename T>
class AMyWeakPtr
{
public:
    AMyWeakPtr()
        : m_pPtr(nullptr)
        , m_pCnter(nullptr)
    {}

    AMyWeakPtr(AMySharedPtr<T>& other)
    {
        m_pPtr = other.m_pPtr;
        m_pCnter = other.m_pCnter;
        m_pCnter->addWeakCnt();
    }

    AMyWeakPtr(AMyWeakPtr<T>& other)
    {
        m_pPtr = other.m_pPtr;
        m_pCnter = other.m_pCnter;
        m_pCnter->addWeakCnt();
    }

    AMyWeakPtr<T>& operator=(const AMySharedPtr<T>& other)
    {
        clean();
        m_pPtr = other.m_pPtr;
        m_pCnter = other.m_pCnter;
        m_pCnter->addWeakCnt();
        return *this;
    }

    AMyWeakPtr<T>& operator=(const AMyWeakPtr<T>& other)
    {
        if(this == &other)
            return *this;

        clean();
        m_pPtr = other.m_pPtr;
        m_pCnter = other.m_pCnter;
        m_pCnter->addWeakCnt();
        return *this;
    }

    AMySharedPtr<T> lock()
    {
        return AMySharedPtr<T>(*this);
    }

    bool expired()
    {
        if(nullptr != m_pCnter
                && m_pCnter->getSharedCnt() > 0)
        {
            return false;
        }
        return true;
    }

    ~AMyWeakPtr()
    {
        clean();
    }

private:
    void clean()
    {
        if(nullptr != m_pCnter)
        {
            m_pCnter->minusWeakCnt();
            if(m_pCnter->getSharedCnt()<1 && m_pCnter->getWeakCnt()<1)
            {
                m_pCnter = nullptr;
            }
        }
    }

private:
    T* m_pPtr;
    ACounter* m_pCnter;
};

#endif // AMYSMARTPOINTER_H
```

2、测试程序
```C++
#include <iostream>
#include "smartpointer/amysmartpointer.h"
struct A
{
    A() { std::cout << "A() \n"; }
    ~A() { std::cout << "~A() \n"; }
};

void testMyUniquePtr()
{
    AMyUniquePtr<A> ptr(new A);
}

void testMySharedPtr()
{
    A* a = new A;
    AMySharedPtr<A> ptr(a);
    {
        std::cout << ptr.use_count() << std::endl;
        AMySharedPtr<A> b = ptr;
        std::cout << ptr.use_count() << std::endl;
        AMySharedPtr<A> c = ptr;
        std::cout << ptr.use_count() << std::endl;
        AMySharedPtr<A> d = std::move(b);
        std::cout << ptr.use_count() << std::endl;
    }
    std::cout << ptr.use_count() << std::endl;
}

class B1;
class B2;
struct B1
{
    B1() { std::cout << "B1() \n"; }
    ~B1() { std::cout << "~B1() \n"; }
    //AMySharedPtr<B2> b2;  如果这里用 AMySharedPtr 就会有循环引用的问题
    AMyWeakPtr<B2> b2;
};
struct B2
{
    B2() { std::cout << "B2() \n"; }
    ~B2() { std::cout << "~B2() \n"; }
    AMySharedPtr<B1> b1;
};
void testMyWeakPtr()
{
    AMySharedPtr<B1> ptr1(new B1);
    AMySharedPtr<B2> ptr2(new B2);
    ptr1->b2 = ptr2;
    ptr2->b1 = ptr1;
    std::cout << ptr1.use_count() << std::endl;
    std::cout << ptr2.use_count() << std::endl;
}

int main(int argc, char *argv[])
{
    testMyUniquePtr();

    std::cout << "\n----------------------------\n\n";

    testMySharedPtr();

    std::cout << "\n----------------------------\n\n";

    testMyWeakPtr();

    return 0;
}
```

3、测试结果
```C++
A()
~A()

----------------------------

A()
1
2
3
3
1
~A()

----------------------------

B1()
B2()
2
1
~B2()
~B1()
```