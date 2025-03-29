自旋锁——代码在循环中“自旋”。

自旋锁（spin lock）是一种非阻塞锁，如果某线程需要获取锁，但该锁已经被其他线程占用时，该线程不会被挂起，而是在不断的消耗 CPU 的时间，不停的试图获取锁。

互斥量（mutex）是阻塞锁，当某线程无法获取锁时，该线程会被直接挂起，该线程不再消耗CPU时间，当其他线程释放锁后，操作系统会激活那个被挂起的线程，让其投入运行。

因此，多核 CPU 才能用自旋锁。

其他细节，详看下面代码注释。
```
#ifndef AMYSPINLOCK_H
#define AMYSPINLOCK_H

#include <atomic>

// 采用 std::atomic_flag 实现自旋锁互斥，即 TAS 算法（Test And Set）
class AMySpinLock1
{
public:
    AMySpinLock1() = default;
    AMySpinLock1(const AMySpinLock1&) = delete;
    AMySpinLock1& operator=(const AMySpinLock1&) = delete;
    void lock()
    {
        while(flag.test_and_set(std::memory_order_acquire));
    }
    void unlock()
    {
        flag.clear(std::memory_order_release);
    }

private:
    // std::atomic_flag 类型的对象必须由宏 ATOMIC_FLAG_INIT 初始化，它把标志初始化为置零状态。
    std::atomic_flag flag{ATOMIC_FLAG_INIT};
};

// 采用 std::atomic<bool> 和 compare_exchange_strong 实现自旋锁互斥，即 CAS 算法（Compare And Swap）
// 比较-交换操作是原子类型的编程基石。
// 使用者给定一个期望值，原子变量将它和自身的值比较，如果相等，就存入另一既定的值，否则，更新期望值所属的变量，向它赋予原子变量的值。
// 比较-交换函数返回布尔类型，如果完成了保存动作（前提是两值相等），则操作成功，函数返回 true；反之操作失败，函数返回 false。
class AMySpinLock2
{
public:
    AMySpinLock2() = default;
    AMySpinLock2(const AMySpinLock2&) = delete;
    AMySpinLock2& operator=(const AMySpinLock2&) = delete;
    void lock()
    {
        // 判断 flag 对象封装的 bool 值是否为期望值(false)：
        // ① 若 bool 值为 false，与期望值相等，说明自旋锁空闲。此时，flag 对象写入 true，返回 true，即上锁成功。
        // ② 若 bool 值为 true，与期望值不相等，说明自旋锁被锁。此时，while 将一直循环，直到返回 true 为止。
        bool expected = false;
        while(false == flag.compare_exchange_strong(expected, true))
        {
            // 当 compare_exchange_strong 返回 false 时，
            // 证明 expected 与 flag 不相等，此时 expected 为false，flag 为 true；
            // 则将 expected 赋值为 flag 的值，即此时 expected 为 true 了，
            // 因此要修改为 false 后，才能进入下一次循环。
            expected = false;
        }
    }
    void unlock()
    {
        flag.store(false);
    }

private:
    // flag 对象所封装的 bool 值为 false 时，说明自旋锁未被线程占有。
    std::atomic<bool> flag{false};
};

#endif // AMYSPINLOCK_H
```

测试程序如下：
```
#include <iostream>
#include <thread>
#include "./lock/amyspinlock.h"

AMySpinLock1 m;
//AMySpinLock2 m;

void testSpinLock()
{
    static int i = 0, j = 0;

    for(int k=0; k<1000; ++k)
    {
        m.lock();
        ++i;
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
        ++j;
        if(i != j)
            printf("[%d] i:%d, j:%d \n", std::this_thread::get_id(), i, j);
        m.unlock();
    }
}

int main(int argc, char *argv[])
{
    std::cout << "Test Begin...\n";

    std::thread t1(testSpinLock);
    std::thread t2(testSpinLock);

    if(t1.joinable())
        t1.join();
    if(t2.joinable())
        t2.join();

    std::cout << "Test End...\n";

    return 0;
}
```