> 整理自《【C++】《C++ 并发编程实战 (第2版) 》笔记-Chapter9-高级线程管理》

1、线程池类，包括 AJoinThreads、AQueueThreadSafe、ATaskType、AStealTaskQueue、AMyThreadPool。其中，

AJoinThreads：RAII思想，析构时调用 join() 自动等待所有线程完成。

AQueueThreadSafe：基于 std::mutex 的线程安全的 queue，用于存放任务。

ATaskType：任务包装类。

AStealTaskQueue：基于 std::mutex 的线程安全的 deque，用于存放各个线程自己的任务，是一个可被其他线程窃取任务的队列。

AMyThreadPool：对外使用的主类，外面通过 submit() 函数提交任务给线程池并得到对应任务的 future。

```
#ifndef AMYTHREADPOOL_H
#define AMYTHREADPOOL_H

#include <functional>
#include <atomic>
#include <vector>
#include <thread>
#include <future>
#include <queue>
#include <memory>
#include <mutex>
#include <condition_variable>

// RAII思想，析构时调用 join() 自动等待所有线程完成。
class AJoinThreads
{
public:
    explicit AJoinThreads(std::vector<std::thread> &threads)
        : m_threads(threads)
    {}

    ~AJoinThreads()
    {
        for(std::thread& t : m_threads)
        {
            if(t.joinable())
                t.join();
        }
    }

private:
    std::vector<std::thread> &m_threads;
};

// 基于 std::mutex 的线程安全的 queue，用于存放任务。
template<typename T>
class AQueueThreadSafe
{
public:
    AQueueThreadSafe()
    {}

    AQueueThreadSafe(const AQueueThreadSafe<T> &other)
    {
        std::lock_guard<std::mutex> lk(other.m_mtx);
        m_queueData = other.m_queueData;
    }

    void push(T newValue)
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        m_queueData.push(std::move(newValue));
        m_cond.notify_one();
    }

    void wait_and_pop(T &value)
    {
        std::unique_lock<std::mutex> lk(m_mtx);
        m_cond.wait(lk, [this]{ return false == m_queueData.empty(); });
        value = m_queueData.front();
        m_queueData.pop();
    }

    std::shared_ptr<T> wait_and_pop()
    {
        std::unique_lock<std::mutex> lk(m_mtx);
        m_cond.wait(lk, [this]{ return false == m_queueData.empty(); });
        std::shared_ptr<T> res(std::make_shared<T>(m_queueData.front()));
        m_queueData.pop();
        return res;
    }

    bool try_pop(T &value)
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        if(m_queueData.empty())
            return false;
        value = std::move(m_queueData.front());
        m_queueData.pop();
        return true;
    }

    std::shared_ptr<T> try_pop()
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        if(m_queueData.empty())
            return std::shared_ptr<T>();
        std::shared_ptr<T> res(std::make_shared<T>(m_queueData.front()));
        m_queueData.pop();
        return res;
    }

    bool empty() const
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        return m_queueData.empty();
    }

private:
    mutable std::mutex m_mtx;     // 互斥必须用 mutable 修饰（针对 const 对象，准许其数据成员发生变动）
    std::queue<T> m_queueData;
    std::condition_variable m_cond;
};

// 任务包装类。
// 由于 std::packaged_task<> 的实例仅能移动而不可复制，但 std::function<> 要求本身所含的函数对象能进行拷贝构造，因此任务队列的元素不能用 std::function<> 充当。
// 我们必须定制自己的类作为代替，以包装函数，并处理只移型别。
// 这个类其实就是一个包装可调用对象的简单类，对外可消除该对象的型别。这个类还具备函数调用操作符。
// 实现基类 ATaskTypeImplBase 是有必要的，不然 m_impl 就必须为 std::unique_ptr<ATaskTypeImpl<F>>，这就需要类 ATaskType 是个模板类，这违背了“对外可消除该对象的型别”的初衷。
class ATaskType
{
    struct ATaskTypeImplBase
    {
        virtual ~ATaskTypeImplBase() {}
        virtual void call() {}
    };

    template<typename F>
    struct ATaskTypeImpl : public ATaskTypeImplBase
    {
        F f;
        ATaskTypeImpl(F&& f_)
            : f(std::move(f_))
        {}

        void call()
        {
            f();
        }
    };

public:
    template<typename F>
    ATaskType(F&& f)
        : m_impl(new ATaskTypeImpl<F>(std::move(f)))
    {}

    void operator() ()
    {
        m_impl->call();
    }

    ATaskType() = default;
    ATaskType(ATaskType&& other)
        : m_impl(std::move(other.m_impl))
    {}

    ATaskType& operator=(ATaskType&& other)
    {
        m_impl = std::move(other.m_impl);
        return *this;
    }

private:
    ATaskType(const ATaskType&) = delete;
    ATaskType(ATaskType&) = delete;
    ATaskType& operator=(const ATaskType&) = delete;

private:
    std::unique_ptr<ATaskTypeImplBase> m_impl;
};

// 基于 std::mutex 的线程安全的 deque，用于存放各个线程自己的任务，是一个可被其他线程窃取任务的队列。
class AStealTaskQueue
{
public:
    AStealTaskQueue()
    {}
    AStealTaskQueue(const AStealTaskQueue& other) = delete;
    AStealTaskQueue& operator=(const AStealTaskQueue& other) = delete;

    void push(ATaskType data)
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        m_dequeTask.push_front(std::move(data));
    }

    bool empty() const
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        return m_dequeTask.empty();
    }

    bool try_pop(ATaskType &res)
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        if(m_dequeTask.empty())
        {
            return false;
        }
        res = std::move(m_dequeTask.front());
        m_dequeTask.pop_front();
        return true;
    }

    bool try_steal(ATaskType &res)      // 窃取时，操作的是队列后端
    {
        std::lock_guard<std::mutex> lk(m_mtx);
        if(m_dequeTask.empty())
            return false;
        res = std::move(m_dequeTask.back());
        m_dequeTask.pop_back();
        return true;
    }

private:
    std::deque<ATaskType> m_dequeTask;
    mutable std::mutex m_mtx;
};

// 对外使用的主类，外面通过 submit() 函数提交任务给线程池并得到对应任务的 future。
class AMyThreadPool
{
public:
    AMyThreadPool()
        : m_bDone(false)
        , m_joiner(m_threads)
    {
        unsigned const nThreadCnt = std::thread::hardware_concurrency();
        try
        {
            // 构造每个线程自己的可被别的线程窃取的任务队列
            for(unsigned i=0; i<nThreadCnt; ++i)
            {
                m_vecTaskQueue.push_back(std::unique_ptr<AStealTaskQueue>(new AStealTaskQueue));
            }

            // 构造线程池中每个线程，并向线程函数传递参数 i 作为线程下标
            for(unsigned i=0; i<nThreadCnt; ++i)
            {
                m_threads.push_back(std::thread(&AMyThreadPool::workerThread, this, i));
            }
        }
        catch(...)
        {
            m_bDone = true;
            throw;
        }
    }

    ~AMyThreadPool()
    {
        m_bDone = true;
    }

    // 提交任务给线程池并返回对应任务的 future.
    template<typename FunctionType>
    std::future<typename std::result_of<FunctionType()>::type> submit(FunctionType f)
    {
        typedef typename std::result_of<FunctionType()>::type result_type;
        std::packaged_task<result_type()> task(f);
        std::future<result_type> res(task.get_future());
        if(tl_pLocalTaskQueue)
        {
            tl_pLocalTaskQueue->push(std::move(task));
        }
        else
        {
            m_queueTask.push(std::move(task));
        }
        return res;
    }

protected:
    void workerThread(unsigned nCurThreadIndex)
    {
        tl_nCurThreadIndex = nCurThreadIndex;
        tl_pLocalTaskQueue = m_vecTaskQueue[nCurThreadIndex].get();
        while(false == m_bDone)
        {
            runTask();
        }
    }

    void runTask()
    {
        ATaskType task;
        if(getTaskFromLocalThreadQueue(task)
                || getTaskFromTotalQueue(task)
                || getTaskFromOtherThreadQueue(task))
        {
            task();
        }
        else
        {
            std::this_thread::yield();
        }
    }

    bool getTaskFromLocalThreadQueue(ATaskType &task)
    {
        return tl_pLocalTaskQueue && tl_pLocalTaskQueue->try_pop(task);
    }

    bool getTaskFromTotalQueue(ATaskType &task)
    {
        return m_queueTask.try_pop(task);
    }

    bool getTaskFromOtherThreadQueue(ATaskType &task)
    {
        for(unsigned i=0; i<m_vecTaskQueue.size(); ++i)
        {
            // 从当前线程的下一个线程开始窃取它的任务
            const unsigned nIndex = (tl_nCurThreadIndex+i+1) % m_vecTaskQueue.size();
            if(m_vecTaskQueue[nIndex]->try_steal(task))
                return true;
        }
        return false;
    }

private:
    std::vector<std::thread> m_threads;                             // 用 vector 存放所有线程
    std::atomic_bool m_bDone;                                       // 停止线程工作的标记
    AJoinThreads m_joiner;                                          // 析构时等待所有线程完成
    AQueueThreadSafe<ATaskType> m_queueTask;                        // 总的任务队列
    std::vector<std::unique_ptr<AStealTaskQueue> > m_vecTaskQueue;  // 每个线程自己的任务队列

    static thread_local AStealTaskQueue* tl_pLocalTaskQueue;        // 当前线程的任务队列
    static thread_local unsigned tl_nCurThreadIndex;                // 当前线程的下标
};
thread_local AStealTaskQueue* AMyThreadPool::tl_pLocalTaskQueue;
thread_local unsigned AMyThreadPool::tl_nCurThreadIndex;

#endif // AMYTHREADPOOL_H
```

2、测试代码
```
#include "threadpool/amythreadpool.h"
#include <iostream>
#include <thread>
#include <chrono>

namespace test3
{
    void testMyThreadPool(int nHandleWay)
    {
        AMyThreadPool threadPool;
        auto start = std::chrono::steady_clock::now();

        for(int i=0; i<10; ++i)
        {
            auto f = threadPool.submit([=]()->int
            {
                std::this_thread::sleep_for(std::chrono::milliseconds(100));
                return i;
            });
            if(1 == nHandleWay)         // costMs:2009
            {
                std::this_thread::sleep_for(std::chrono::milliseconds(200));
                std::cout << f.get() << std::endl;
            }
            else if(2 == nHandleWay)    // costMs:1009
                f.get();
            else if(3 == nHandleWay)    // costMs:1009
                std::cout << f.get() << std::endl;
            else if(4 == nHandleWay)    // costMs:0
            {}
        }

        int costMs = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now()- start).count();
        std::cout << std::thread::hardware_concurrency() << " costMs:" << costMs << std::endl;
    }
}

int main(int argc, char *argv[])
{
    std::cout << "Test Begin...\n";

    // -----------------------------------------------------------------

    test3::testMyThreadPool(1);
    std::cout << "----------------------------\n";
    test3::testMyThreadPool(2);
    std::cout << "----------------------------\n";
    test3::testMyThreadPool(3);
    std::cout << "----------------------------\n";
    test3::testMyThreadPool(4);

    // -----------------------------------------------------------------

    std::cout << "Test End...\n";

    return 0;
}
```
