# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 一个 N-M 的协程调度器，N 个线程运行 M 个协程，协程可以在线程之间进行切换，协程也可以绑定到指定线程运行。
- 实现协程调度之后，可以解决前一章协程模块中子协程不能运行另一个子协程的缺陷，子协程可以通过向调度器添加调度任务的方式来运行另一个子协程。
- 协程调度器调度的是协程，函数（可执行对象）被包装成协程。


# Scheduler

- 协程调度器类。
- t_scheduler_fiber 保存当前线程的调度协程，加上 Fiber 模块的 t_fiber 和 t_thread_fiber，每个线程总共可以记录三个协程的上下文信息。


# SchedulerSwitcher

- 调度器切换类。


# 其他说明

- 协程调度最难理解的地方是当 caller 线程也参与调度时调度协程和主线程切换的情况。
	- 调度线程可以包含 caller 线程。
	- 在非 caller 线程里，调度协程就是调度线程的主协程；但在 caller 线程里，调度协程并不是 caller 线程的主协程，而是相当于 caller 线程的子协程。
	- 在非对称协程里，子协程只能和线程主协程切换，而不能和另一个子协程切换。而这里，调度协程和任务协程，都是子协程，也就是说，调度协程不能直接和任务协程切换。sylar 的解决方案是：给每个线程增加一个线程局部变量用于保存调度协程的上下文就可以了，这样，每个线程可以同时保存三个协程的上下文，一个是当前正在执行的协程上下文，另一个是线程主协程的上下文，最后一个是调度协程的上下文。
- 支持添加函数或协程作为调度对象，并且支持将函数或协程绑定到一个具体的线程上执行。
- 调度协程执行 run 方法，负责从调度器的任务队列中取任务执行，取出的任务即子协程。每个子协程执行完后都必须返回调度协程，由调度协程重新从任务队列中取新的协程并执行。如果任务队列空了，那么调度协程会切换到一个 idle 协程，这个 idle 协程什么也不做，等有新任务进来前，不断地与调度协程进行切换（这里其实是忙等）。
- 如果任务队列为空，那么在添加任务之后，要调用一次 tickle 方法以通知各调度线程的调度协程有新任务来了。
- 调度器的停止行为要分两种情况讨论，首先是 use_caller 为 false 的情况，这种情况下，由于没有使用 caller 线程进行调度，那么只需要简单地等各个调度线程的调度协程退出就行了。如果 use_caller 为 true，表示 caller 线程也参与了调度，这时，调度器初始化时记录的属于 caller 线程的调度协程就要起作用了，在调度器停止前，应该让这个 caller 线程的调度协程也运行一次，让 caller 线程完成调度工作后再退出。如果调度器只使用了 caller 线程进行调度，那么所有的调度任务要在调度器停止时才会被调度（因为只使用了 caller 线程进行调度的话，就意味着用的是 caller 线程的子协程进行调度，而只有在调度器停止时，该子协程才会被 call()）。
- sylar 的协程调度模块因为存任务队列空闲时调度线程忙等待的问题，所以实际上并不实用，真正实用的是后面基于 Scheduler 实现的 IOManager。


# 部分相关代码
```C++
/**
 * @filename    scheduler.h
 * @brief   协程调度模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-29
 */
#ifndef __SYLAR_SCHEDULER_H__
#define __SYLAR_SCHEDULER_H__

#include <memory>
#include <vector>
#include <list>
#include <iostream>
#include "fiber.h"
#include "thread.h"

namespace sylar
{

/**
 * @brief   协程调度器
 */
class Scheduler
{
public:
    typedef std::shared_ptr<Scheduler> ptr;
    typedef Mutex MutexType;

    /**
     * @brief   构造函数
     *
     * @param   threads     线程数量
     * @param   use_caller  是否使用当前调用线程
     * @param   name        协程调度器名称
     */
    Scheduler(size_t threads = 1, bool use_caller = true, const std::string& name = "");
    virtual ~Scheduler();

    const std::string& getName() const { return m_name; }
    
    static Scheduler* GetThis();

    /**
     * @brief   返回当前协程调度器的调度协程(不一定是主协程)
     */
    static Fiber* GetMainFiber();

    void start();
    void stop();

    template<class FiberOrCb>
    void schedule(FiberOrCb fc, int thread = -1)
    {
        bool need_tickle = false;
        {
            MutexType::Lock lk(m_mutex);
            need_tickle = scheduleNoLock(fc, thread);
        }

        if(need_tickle)
        {
            tickle();
        }
    }

    template<class InputIterator>
    void schedule(InputIterator begin, InputIterator end)
    {
        bool need_tickle = false;
        {
            MutexType::Lock lk(m_mutex);
            while(begin != end)
            {
                need_tickle = scheduleNoLock(&*begin, -1) || need_tickle;
                ++begin;
            }
        }

        if(need_tickle)
        {
            tickle();
        }
    }

    void switchTo(int thread = -1);
    std::ostream& dump(std::ostream& os);

protected:
    /**
     * @brief  通知协程调度器有任务了 
     */
    virtual void tickle();

    void run();

    virtual bool stopping();

    virtual void idle();

    void setThis();

    bool hasIdleThreads() { return m_idleThreadCount > 0; }

private:
    template<class FiberOrCb>
    bool scheduleNoLock(FiberOrCb fc, int thread)
    {
        bool need_tickle = m_fibers.empty();
        FiberAndThread ft(fc, thread);
        if(ft.fiber || ft.cb)
        {
            m_fibers.push_back(ft);
        }
        return need_tickle;
    }

private:
    struct FiberAndThread
    {
        Fiber::ptr fiber;
        std::function<void()> cb;
        int thread;

        FiberAndThread(Fiber::ptr f, int thr)
            : fiber(f)
            , thread(thr)
        {}

        FiberAndThread(Fiber::ptr* f, int thr)
            : thread(thr)
        {
            fiber.swap(*f);
        }

        FiberAndThread(std::function<void()> f, int thr)
            : cb(f)
            , thread(thr)
        {}

        FiberAndThread(std::function<void()>* f, int thr)
            : thread(thr)
        {
            cb.swap(*f);        
        }

        FiberAndThread()
            : thread(-1)
        {}

        void reset()
        {
            fiber = nullptr;
            cb = nullptr;
            thread = -1;
        }
    };

private:
    MutexType m_mutex;
    /// 线程池
    std::vector<Thread::ptr> m_threads;
    /// 任务队列
    std::list<FiberAndThread> m_fibers;
    /// use_caller为true时有效，调度器所在线程的调用协程(但它是子协程)
    Fiber::ptr m_rootFiber;
    /// 协程调度器名称
    std::string m_name;

protected:
    /// 协程下的线程ID数组
    std::vector<int> m_threadIds;
    size_t m_threadCount = 0;
    /// 工作线程的数量
    std::atomic<size_t> m_activeThreadCount = {0};
    /// 空闲线程的数量
    std::atomic<size_t> m_idleThreadCount = {0};
    bool m_stopping = true;
    bool m_autoStop = false;
    /// use_caller为true时，调度器所在线程的线程id
    int m_rootThread = 0;
};

class SchedulerSwitcher : public Noncopyable
{
public:
    SchedulerSwitcher(Scheduler* target = nullptr);
    ~SchedulerSwitcher();

private:
    Scheduler* m_caller;
};

}

#endif


#include "scheduler.h"
#include "fiber.h"
#include "macro.h"
#include "hook.h"

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");
/// 当前线程的调度器，同一个调度器下的所有线程指向同一个调度器实例
static thread_local Scheduler* t_scheduler = nullptr;
/// 当前线程的调度协程，每个线程都独有一份，包括caller线程(caller线程的是当前线程的子协程)
static thread_local Fiber* t_scheduler_fiber = nullptr;

Scheduler::Scheduler(size_t threads, bool use_caller, const std::string& name)
    : m_name(name)
{
    SYLAR_ASSERT(threads > 0);

    if(use_caller)
    {
        sylar::Fiber::GetThis();    // 这里面其实创建了当前线程的主协程
        --threads;

        SYLAR_ASSERT(GetThis() == nullptr);
        t_scheduler = this;
        
        // 创建调度协程（该线程的子协程）
        m_rootFiber.reset(new Fiber(std::bind(&Scheduler::run, this), 0, true));
        sylar::Thread::SetName(m_name);
        
        t_scheduler_fiber = m_rootFiber.get();
        m_rootThread = sylar::GetThreadId();
        m_threadIds.push_back(m_rootThread);
    }
    else
    {
        m_rootThread = -1;
    }
    m_threadCount = threads;
}

Scheduler::~Scheduler()
{
    SYLAR_ASSERT(m_stopping);
    if(GetThis() == this)
    {
        t_scheduler = nullptr;
    }
}

Scheduler* Scheduler::GetThis()
{
    return t_scheduler;
}

Fiber* Scheduler::GetMainFiber()
{
    return t_scheduler_fiber;
}

void Scheduler::start()
{
    MutexType::Lock lk(m_mutex);
    if(!m_stopping)
    {
        return;
    }

    m_stopping = false;
    SYLAR_ASSERT(m_threads.empty());

    // 初始化线程池
    m_threads.resize(m_threadCount);
    for(size_t i=0; i<m_threadCount; ++i)
    {
        m_threads[i].reset(new Thread(std::bind(&Scheduler::run, this), 
                    m_name + "_" + std::to_string(i)));
        m_threadIds.push_back(m_threads[i]->getId());
    }
}

void Scheduler::stop()
{
    m_autoStop = true;
    if(m_rootFiber
            && m_threadCount == 0
            && (m_rootFiber->getState() == Fiber::TERM
                || m_rootFiber->getState() == Fiber::INIT))
    {
        SYLAR_LOG_INFO(g_logger) << this << " stopped";
        m_stopping = true;

        if(stopping())
        {
            return;
        }
    }

    // 如果user_caller，则调用stop的线程也应该是它
    if(m_rootThread != -1)
    {
        SYLAR_ASSERT(GetThis() == this);
    }
    else
    {
        SYLAR_ASSERT(GetThis() != this);
    }

    m_stopping = true;
    for(size_t i=0; i<m_threadCount; ++i)
    {
        tickle();
    }

    if(m_rootFiber)
    {
        tickle();
    }

    if(m_rootFiber)
    {
        if(!stopping())
        {
            // 此时才切换到m_rootFiber这条协程执行
            m_rootFiber->call();
        }
    }

    std::vector<Thread::ptr> thrs;
    {
        MutexType::Lock lk(m_mutex);
        thrs.swap(m_threads);
    }

    for(auto& i : thrs)
    {
        i->join();
    }
}

/**
 * @brief   切换线程（也可切换调度器）
 *
 * @param   thread 线程id
 */
void Scheduler::switchTo(int thread)
{
    SYLAR_ASSERT(Scheduler::GetThis() != nullptr);
    if(Scheduler::GetThis() == this)
    {
        // 如果已经是当前调度器，而线程id未指定或已经是当前线程，则直接return掉
        if(thread == -1 || thread == sylar::GetThreadId())
        {
            return;
        }
    }

    // 再次加入任务队列里面，并让出当前协程的执行权
    schedule(Fiber::GetThis(), thread);
    Fiber::YieldToHold();
}

std::ostream& Scheduler::dump(std::ostream& os)
{
    os << "[Scheduler name=" << m_name
       << " size=" << m_threadCount
       << " active_count=" << m_activeThreadCount
       << " idle_count=" << m_idleThreadCount
       << " stopping=" << m_stopping
       << " ]" << std::endl << "    ";

    for(size_t i=0; i<m_threadIds.size(); ++i)
    {
        if(i)
        {
            os << ", ";
        }
        os << m_threadIds[i];
    }
    return os;
}

void Scheduler::tickle()
{
    SYLAR_LOG_INFO(g_logger) << "tickle";
}

void Scheduler::run()
{
    SYLAR_LOG_DEBUG(g_logger) << m_name << " run";

    set_hook_enable(true);  // 设置当前线程为hook的

    setThis();      // 设置当前调度器

    // 设置调度协程
    if(sylar::GetThreadId() != m_rootThread)
    {
        t_scheduler_fiber = Fiber::GetThis().get();
    }

    Fiber::ptr idle_fiber(new Fiber(std::bind(&Scheduler::idle, this)));
    Fiber::ptr cb_fiber;

    FiberAndThread ft;
    while(true)
    {
        ft.reset();
        bool tickle_me = false;
        bool is_active = false;
        {
            MutexType::Lock lk(m_mutex);
            auto it = m_fibers.begin();
            while(it != m_fibers.end())
            {
                // 指定了调度线程，但不是在当前线程上调度，标记一下需要通知其他线程进行调度，
                // 然后跳过这个任务，继续下一个
                if(it->thread != -1 && it->thread != sylar::GetThreadId())
                {
                    ++it;
                    tickle_me = true;
                    continue;
                }

                SYLAR_ASSERT(it->fiber || it->cb);

                if(it->fiber && it->fiber->getState() == Fiber::EXEC)
                {
                    ++it;
                    continue;
                }

                ft = *it;
                m_fibers.erase(it++);
                ++m_activeThreadCount;
                is_active = true;
                break;
            }

            // 当前线程拿完一个任务后，发现任务队列还有剩余，那么标记一下需要通知其他线程进行调度
            tickle_me |= it!=m_fibers.end();
        }

        if(tickle_me)
        {
            tickle();
        }

        if(ft.fiber && (ft.fiber->getState() != Fiber::TERM
                        && ft.fiber->getState() != Fiber::EXCEPT))
        {
            ft.fiber->swapIn();         // 切进去执行该协程
            --m_activeThreadCount;      // 等该协程出来的时候，活跃线程数便可以减1

            // 该协程出来后，如果还是READY状态，则再把它进入到任务队列中
            if(ft.fiber->getState() == Fiber::READY)
            {
                schedule(ft.fiber);
            }
            else if(ft.fiber->getState() != Fiber::TERM
                    && ft.fiber->getState() != Fiber::EXCEPT)
            {
                ft.fiber->setState(Fiber::HOLD);
            }
            ft.reset();
        }
        else if(ft.cb)      // 任务包装的是函数对象
        {
            if(cb_fiber)
            {
                cb_fiber->reset(ft.cb);
            }
            else
            {
                cb_fiber.reset(new Fiber(ft.cb));
            }
            ft.reset();
            cb_fiber->swapIn();
            --m_activeThreadCount;
            if(cb_fiber->getState() == Fiber::READY)
            {
                schedule(cb_fiber);
                cb_fiber.reset();
            }
            else if(cb_fiber->getState() == Fiber::EXCEPT
                    || cb_fiber->getState() == Fiber::TERM)
            {
                cb_fiber->reset(nullptr);
            }
            else
            {
                cb_fiber->setState(Fiber::HOLD);
                cb_fiber.reset();
            }
        }
        else
        {
            if(is_active)
            {
                --m_activeThreadCount;
                continue;
            }

            if(idle_fiber->getState() == Fiber::TERM)
            {
                SYLAR_LOG_INFO(g_logger) << "idle fiber term";
                break;
            }

            ++m_idleThreadCount;
            idle_fiber->swapIn();       // 执行idle协程，其实是在忙等
            --m_idleThreadCount;
            if(idle_fiber->getState() != Fiber::TERM
                    && idle_fiber->getState() != Fiber::EXCEPT)
            {
                idle_fiber->setState(Fiber::HOLD);
            }
        }
    }
}

bool Scheduler::stopping()
{
    MutexType::Lock lk(m_mutex);
    return m_autoStop && m_stopping
        && m_fibers.empty() && m_activeThreadCount == 0;
}

void Scheduler::idle()
{
    SYLAR_LOG_INFO(g_logger) << "idle";
    while(!stopping())
    {
        sylar::Fiber::YieldToHold();
    }
}

void Scheduler::setThis()
{
    t_scheduler = this;
}

SchedulerSwitcher::SchedulerSwitcher(Scheduler* target)
{
    m_caller = Scheduler::GetThis();
    if(target)
    {
        target->switchTo();
    }
}

SchedulerSwitcher::~SchedulerSwitcher()
{
    if(m_caller)
    {
        m_caller->switchTo();
    }
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
