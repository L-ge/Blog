# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 协程是一种用户态的轻量级线程。
	- 轻量级：相对于线程而言，协程比较轻量。在一个服务器上线程的数量不能太多，也就是上百个线程的级别，否则会产生性能问题。而协程可以轻松达到达到百万级别。
	- 协作式：协程是用户态的概念，对操作系统无感知。操作系统感知不到协程的存在，只能感知到线程和进程的存在。协程的调度是在用户态自己调度。一般协程的调度是协作式的调度。
- 根据控制传递机制的不同区分出了对称协程和非对称协程。
	- 非对称协程知道它的调用者，其在挂起时转让控制权给它的调用者，然后调用者根据算法调用其他非对称协程进行工作。具体来讲，非对称协程是跟一个特定的调用者绑定的，协程让出 CPU 时，只能让回给原调用者。那到底是什么东西“不对称”呢？其实，非对称在于程序控制流转移到被调协程时使用的是 call/resume 操作，而当被调协程让出 CPU 时使用的却是 return/yield 操作。此外，协程间的地位也不对等，caller 与 callee 关系是确定的，不可更改的，非对称协程只能返回最初调用它的协程。
	- 对称协程都是等价的，控制权直接在对称协程之间进行传递，即对称协程在挂起时主动指明另外一个对称协程来接收控制权。对称协程与非对称协程不一样，启动之后就跟启动之前的协程没有任何关系了。协程的切换操作，一般而言只有一个操作，yield，用于将程序控制流转移给另外的协程。对称协程机制一般需要一个调度器的支持，按一定调度算法去选择 yield 的目标协程。
- sylar 的协程模块是基于 ucontext.h 实现的非对称协程库。结构体 ucontext_t 保存了程序当前运行的上下文，例如寄存器信息（这些寄存器记录了函数栈帧、代码的执行位置等信息）等。
- ucontext_t 结构体与具体平台相关，但至少会有以下四个成员：
	```C
	ucontext_t *uc_link     	// Pointer to the context that is resumed when this context returns. 
	sigset_t    uc_sigmask  	// The set of signals that are blocked when this context is active. 
	stack_t     uc_stack    	// The stack used by this context. 
	mcontext_t  uc_mcontext  	// A machine-specific representation of the saved context. 
  ```
- ucontext.h 还提供了以下四个函数：
	```C
	int getcontext(ucontext_t *ucp);
	- 获取当前上下文
	
	int setcontext(const ucontext_t *ucp);
	- 恢复 ucp 指向的上下文，但这个函数不会返回，而是会跳转到 ucp 上下文对应的函数中执行，相当于变相调用了函数。
	
	void makecontext(ucontext_t *ucp, void (*func)(), int argc, ...);
	- 修改由 getcontext 获取到的上下文指针 ucp，将其与一个函数 func 进行绑定，支持指定 func 运行时的参数。
	- 在调用 makecontext 之前，必须手动给 ucp 分配一段内存空间，存储在 ucp->uc_stack 中，这段内存空间将作为 func 函数运行时的栈空间。
	- 同时也可以指定 ucp->uc_link，表示函数运行结束后恢复 uc_link 指向的上下文。
	- 如果不赋值 uc_link，那 func 函数结束时必须调用 setcontext 或 swapcontext 以重新指定一个有效的上下文，否则程序就跑飞了。
	- makecontext 执行完后，ucp 就与函数 func 绑定了，调用 setcontext 或 swapcontext 激活 ucp 时，func 就会被运行。
	
	int swapcontext(ucontext_t *restrict oucp, const ucontext_t *restrict ucp);
	- 恢复 ucp 指向的上下文，同时将当前的上下文存储到 oucp 中。
	- 和 setcontext 一样，swapcontext 也不会返回，而是会跳转到 ucp 上下文对应的函数中执行，相当于调用了函数。
	- swapcontext 是 sylar 非对称协程实现的关键，线程主协程和子协程用这个接口进行上下文切换。
	```
- 协程与 IO 多路复用结合，成为高并发的解决方案。


# Fiber

- 协程类。


# 其他说明

- 因为单线程下协程并不是并发执行，而是顺序执行的，所以不要在协程里使用线程级别的锁来做协程同步，比如 pthread_mutex_t。如果一个协程在持有锁之后让出执行，那么同线程的其他任何协程一旦尝试再次持有这个锁，整个线程就锁死了，这和单线程环境下，连续两次对同一个锁进行加锁导致的死锁道理完全一样。
- 对于非对称协程来说，协程除了创建语句外，只有两种操作，一种是 resume，表示恢复协程运行，一种是 yield，表示让出执行。协程的结束没有专门的操作，协程函数运行结束时协程即结束，协程结束时会自动调用一次 yield 以返回主协程。
- 单线程环境下，协程的 yield 和 resume 一定是同步进行的，一个协程的 yield，必然对应另一个协程的 resume，因为线程不可能没有执行主体。并且，协程的 yield 和 resume 是完全由应用程序来控制的。
- 所谓创建协程，其实就是把一个函数包装成一个协程对象，然后再用协程的方式把这个函数跑起来；所谓协程调度，其实就是创建一批的协程对象，然后再创建一个调度协程，通过调度协程把这些协程对象一个一个消化掉（协程可以在被调度时继续向调度器添加新的调度任务）；所谓IO协程调度，其实就是在调度协程时，如果发现这个协程在等待IO就绪，那就先让这个协程让出执行权，等对应的IO就绪后再重新恢复这个协程的运行；所谓定时器，就是给调度协程预设一个协程对象，等定时时间到了就恢复预设的协程对象。
- sylar 借助了线程局部变量的功能来实现协程模块。对于每个线程的协程上下文，sylar 设计了两个线程局部变量来存储上下文信息（对应源码的 t_fiber 和 t_thread_fiber），也就是说，一个线程在任何时候最多只能知道两个协程的上下文。又由于 sylar 只使用 swapcontext 来做协程切换，那就意味着，这两个线程局部变量必须至少有一个是用来保存线程主协程的上下文的，如果这两个线程局部变量存储的都是子协程的上下文，那么不管怎么调用 swapcontext，都没法恢复主协程的上下文，也就意味着程序最终无法回到主协程去执行，程序也就跑飞了。
- READY 状态的协程会被调度器自动重新调度，而 HOLD 状态的协程需要显式地再次将协程加入调度。
- 因为 GetThis() 兼具初始化主协程的功能，在使用协程之前必须显式调用一次 GetThis()。
- 线程主协程代表线程入口函数或是main函数所在的协程，这两种函数都不是以协程的手段创建的，所以它们只有 ucontext_t 上下文，但没有入口函数，也没有分配栈空间。
- 非对称协程，子协程不能创建并运行新的子协程，不然程序就跑飞了。


# 部分相关代码
```C++
/**
 * @filename    fiber.h
 * @brief   协程模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-26
 */
#ifndef __SYLAR_FIBER_H__
#define __SYLAR_FIBER_H__

#include <memory>
#include <functional>
#include <ucontext.h>

namespace sylar
{

/**
 * @brief   协程类
 */
class Fiber : public std::enable_shared_from_this<Fiber>
{
public:
    typedef std::shared_ptr<Fiber> ptr;

    // 协程运行状态
    enum State
    {
        INIT,       // 初始化状态
        HOLD,       // 暂停状态
        EXEC,       // 执行中状态
        TERM,       // 结束状态
        READY,      // 可执行(就绪)状态
        EXCEPT,     // 异常状态
    };

private:
    Fiber();

public:
    Fiber(std::function<void()> cb, size_t stacksize = 0, bool use_caller = false);
    ~Fiber();

    void reset(std::function<void()> cb);
    void swapIn();
    void swapOut();
    void call();
    void back();
    
    uint64_t getId() const { return m_id; }
    void setState(State s) { m_state = s; }
    State getState() const { return m_state; }

public:
    static void SetThis(Fiber* f);
    static Fiber::ptr GetThis();
    
    static void YieldToReady();
    static void YieldToHold();
    
    static uint64_t TotalFibers();
    static uint64_t GetFiberId();

    static void MainFunc();
    static void CallerMainFunc();

private:
    /// 协程id
    uint64_t m_id = 0;
    /// 协程运行栈大小
    uint32_t m_stacksize = 0;
    /// 协程状态
    State m_state = INIT;
    /// 协程上下文
    ucontext_t m_ctx;
    /// 协程运行栈指针
    void* m_stack = nullptr;
    /// 协程运行函数
    std::function<void()> m_cb;
};

}

#endif 


#include "fiber.h"
#include "config.h"
#include "scheduler.h"
#include "macro.h"
#include <atomic>

namespace sylar
{

static Logger::ptr g_logger = SYLAR_LOG_NAME("system");

static std::atomic<uint64_t> s_fiber_id{0};         // 用于生成协程ID
static std::atomic<uint64_t> s_fiber_count{0};      // 用于统计协程数

static thread_local Fiber* t_fiber = nullptr;           // 当前线程正在运行的协程
static thread_local Fiber::ptr t_threadFiber = nullptr; // 当前线程的主协程

static ConfigVar<uint32_t>::ptr g_fiber_stack_size = 
    Config::Lookup<uint32_t>("fiber.stack_size", 128*1024, "fiber stack size");

class MallocStackAllocator
{
public:
    static void* Alloc(size_t size)
    {
        return malloc(size);
    }

    static void Dealloc(void* vp, size_t size)
    {
        return free(vp);
    }
};

using StackAllocator = MallocStackAllocator;

void Fiber::SetThis(Fiber* f)
{
    t_fiber = f;
}
    
/**
 * @brief  返回当前线程正在执行的协程。
 *         如果当前线程尚未创建协程，则创建该线程的第一个协程，且该协程为当前线程的主协程，其他协程都通过这个协程来调度。
 */
Fiber::ptr Fiber::GetThis()
{
    if(t_fiber)
    {
        return t_fiber->shared_from_this();
    }
    
    Fiber::ptr main_fiber(new Fiber);
    SYLAR_ASSERT(t_fiber == main_fiber.get());
    t_threadFiber = main_fiber;
    return t_fiber->shared_from_this();
}
    
/**
 * @brief   将当前协程切换到后台，并设置为 READY 状态
 */
void Fiber::YieldToReady()
{
    Fiber::ptr cur = GetThis();
    SYLAR_ASSERT(cur->m_state == EXEC);
    cur->m_state = READY;
    cur->swapOut();
}

/**
 * @brief   将当前协程切换到后台，并设置为 HOLD 状态
 */
void Fiber::YieldToHold()
{
    Fiber::ptr cur = GetThis();
    SYLAR_ASSERT(cur->m_state == EXEC);
    cur->swapOut();
}
    
uint64_t Fiber::TotalFibers()
{
    return s_fiber_count;
}
    
uint64_t Fiber::GetFiberId()
{
    if(t_fiber)
    {
        t_fiber->getId();
    }
    return 0;
}

/**
 * @brief   协程执行函数，执行完成返回到线程的主协程
 */
void Fiber::MainFunc()
{
    Fiber::ptr cur = GetThis(); 
    SYLAR_ASSERT(cur);
    try
    {
        cur->m_cb();
        cur->m_cb = nullptr;
        cur->m_state = TERM;
    }
    catch(std::exception& ex)
    {
        cur->m_state = EXCEPT;
        SYLAR_LOG_ERROR(g_logger) << "Fiber Except: " << ex.what()
            << " fiber_id=" << cur->getId()
            << std::endl
            << sylar::BacktraceToString();
    }
    catch(...)
    {
        cur->m_state = EXCEPT;
        SYLAR_LOG_ERROR(g_logger) << "Fiber Except"
            << " fiber_id=" << cur->getId()
            << std::endl
            << sylar::BacktraceToString();
    }

    auto raw_ptr = cur.get();
    cur.reset();        // 引用计数减1
    raw_ptr->swapOut();

    SYLAR_ASSERT2(false, "never reach fiber_id=" + std::to_string(raw_ptr->getId()));
}
    
/**
 * @brief   协程执行函数，执行完成返回到线程调度协程
 *          协程的调度是运行在 caller 线程的子协程中，
 *          因此这里返回到的是 caller 线程的主协程中。
 */
void Fiber::CallerMainFunc()
{
    Fiber::ptr cur = GetThis();
    try
    {
        cur->m_cb();
        cur->m_cb = nullptr;
        cur->m_state = TERM;
    }
    catch(std::exception& ex)
    {
        cur->m_state = EXCEPT;
        SYLAR_LOG_ERROR(g_logger) << "Fiber Except: " << ex.what()
            << " fiber_id=" << cur->getId()
            << std::endl
            << sylar::BacktraceToString();
    }
    catch(...)
    {
        cur->m_state = EXCEPT;
        SYLAR_LOG_ERROR(g_logger) << "Fiber Except"
            << " fiber_id=" << cur->getId()
            << std::endl
            << sylar::BacktraceToString();
    }

    auto raw_ptr = cur.get();
    cur.reset();
    raw_ptr->back();
    
    SYLAR_ASSERT2(false, "never reach fiber_id=" + std::to_string(raw_ptr->getId()));
}

/**
 * @brief   只用于创建线程的第一个协程，也就是线程主函数对应的协程
 */
Fiber::Fiber()
{
    m_state = EXEC;
    SetThis(this);

    if(getcontext(&m_ctx))
    {
        SYLAR_ASSERT2(false, "getcontext");        
    }

    ++s_fiber_count;

    SYLAR_LOG_DEBUG(g_logger) << "Fiber::Fiber main";
}

Fiber::Fiber(std::function<void()> cb, size_t stacksize, bool use_caller)
    : m_id(++s_fiber_id)
    , m_cb(cb)
{
    ++s_fiber_count;
    m_stacksize = stacksize ? stacksize : g_fiber_stack_size->getValue();

    m_stack = StackAllocator::Alloc(m_stacksize);
    if(getcontext(&m_ctx))
    {
        SYLAR_ASSERT2(false, "getcontext");        
    }

    m_ctx.uc_link = nullptr;
    m_ctx.uc_stack.ss_sp = m_stack;
    m_ctx.uc_stack.ss_size = m_stacksize;

    if(!use_caller)
    {
        makecontext(&m_ctx, &Fiber::MainFunc, 0);
    }
    else
    {
        makecontext(&m_ctx, &Fiber::CallerMainFunc, 0);
    }

    SYLAR_LOG_DEBUG(g_logger) << "Fiber::Fiber id=" << m_id;
}
    
Fiber::~Fiber()
{
    --s_fiber_count;
    if(m_stack)
    {
        StackAllocator::Dealloc(m_stack, m_stacksize);
    }
    else
    {
        SYLAR_ASSERT(!m_cb);
        SYLAR_ASSERT(m_state == EXEC);

        Fiber* cur = t_fiber;
        if(cur == this)
        {
            SetThis(nullptr);
        }
    }

    SYLAR_LOG_DEBUG(g_logger) << "Fiber::~Fiber id=" << m_id
                              << " total=" << s_fiber_count;
}

/**
 * @brief  重置协程，即重复利用已结束的协程，复用其栈空间，创建新协程 
 */
void Fiber::reset(std::function<void()> cb)
{
    m_cb = cb;
    if(getcontext(&m_ctx))
    {
        SYLAR_ASSERT2(false, "getcontext");        
    }

    m_ctx.uc_link = nullptr;
    m_ctx.uc_stack.ss_sp = m_stack;
    m_ctx.uc_stack.ss_size = m_stacksize;

    makecontext(&m_ctx, &Fiber::MainFunc, 0);
    m_state = INIT;
}

/**
 * @brief   将当前协程切换到运行状态
 */
void Fiber::swapIn()
{
    SetThis(this);
    m_state = EXEC;
    if(swapcontext(&Scheduler::GetMainFiber()->m_ctx, &m_ctx))
    {
        SYLAR_ASSERT2(false, "swapcontext");        
    }
}
    
/**
 * @brief   将当前协程切换到后台
 */
void Fiber::swapOut()
{
    SetThis(Scheduler::GetMainFiber());
    if(swapcontext(&m_ctx, &Scheduler::GetMainFiber()->m_ctx))
    {
        SYLAR_ASSERT2(false, "swapcontext");        
    }
}
    
/**
 * @brief   将当前线程切换到执行状态
 *          当前线程的主协程切换到当前协程
 */
void Fiber::call()
{
    SetThis(this);
    m_state = EXEC;
    if(swapcontext(&t_threadFiber->m_ctx, &m_ctx))
    {
        SYLAR_ASSERT2(false, "swapcontext");        
    }
}
    
/**
 * @brief   将当前线程切换到后台
 *          当前协程切换为当前线程的主协程
 */
void Fiber::back()
{
    SetThis(this);
    if(swapcontext(&m_ctx, &t_threadFiber->m_ctx))
    {
        SYLAR_ASSERT2(false, "swapcontext");        
    }
}

}
```

# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
