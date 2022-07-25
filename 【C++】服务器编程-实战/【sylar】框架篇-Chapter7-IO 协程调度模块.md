# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 基于 epoll 的IO协程调度器。
- 支持读写事件。
- IO协程调度器还解决了上一章协程调度器在 idle 状态下忙等待导致 CPU 占用率高的问题。IO协程调度器使用一对管道 fd 来 tickle 调度协程，当调度器空闲时，idle 协程通过 epoll_wait 阻塞在管道的读描述符上，等管道的可读事件。添加新任务时，tickle 方法写管道，idle 协程检测到管道可读后退出，调度器执行调度。
- IO协程调度支持为描述符注册可读和可写事件的回调函数，当描述符可读或可写时，执行对应的回调函数（包装在 FdContext::EventContext 里面）。


# IOManager

- IO协程调度器类。
- 继承自协程调度器。
- 重载了 Scheduler 的 tickle 和 idle 方法。


# 其他说明

- 对每个 fd，sylar 支持两类事件，一类是可读事件（对应 EPOLLIN），一类是可写事件（对应EPOLLOUT），sylar的事件枚举值直接继承自 epoll。
- 当然 epoll 本身除了支持了 EPOLLIN 和 EPOLLOUT 两类事件外，还支持其他事件，比如 EPOLLRDHUP, EPOLLERR, EPOLLHUP 等，对于这些事件，sylar 的做法是将其进行归类，分别对应到 EPOLLIN 和 EPOLLOUT 中，也就是所有的事件都可以表示为可读或可写事件，甚至有的事件还可以同时表示可读及可写事件，比如 EPOLLERR 事件发生时，fd 将同时触发可读和可写事件。
- 在执行 epoll_ctl 时通过 epoll_event 的私有数据指针 data.ptr 来保存 FdContext 结构体信息，其中 FdContext 结构体信息包括描述符 fd、所含事件 events、回调函数 EventContext。
- IO协程调度器在 idle 时会 epoll_wait 所有注册的fd，如果有 fd 满足条件，epoll_wait 返回，从私有数据中拿到 fd 的上下文信息（也就是 data.ptr 里面存放的 FdContext），并且执行其中的回调函数（实际是 idle 协程只负责收集所有已触发的 fd 的回调函数并将其加入调度器的任务队列，真正的执行时机是 idle 协程退出后，调度器在下一轮调度时执行）。
- epoll 是线程安全的，即使调度器有多个调度线程，它们也可以共用同一个 epoll 实例，而不用担心互斥。由于空闲时所有线程都阻塞的 epoll_wait 上，所以也不用担心 CPU 占用问题。
- addEvent 是一次性的，比如说，注册了一个读事件，当 fd 可读时会触发该事件，但触发完之后，这次注册的事件就失效了，后面 fd 再次可读时，并不会继续执行该事件回调，如果要持续触发事件的回调，那每次事件处理完都要手动再 addEvent。这样在应对 fd 的 WRITE 事件时会比较好处理，因为 fd 可写是常态，如果注册一次就一直有效，那么可写事件就必须在执行完之后就删除掉。


# 部分相关代码
```C++
/**
 * @filename    iomanager.h
 * @brief   IO协程调度模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-30
 */
#ifndef __SYLAR_IOMANAGER_H__
#define __SYLAR_IOMANAGER_H__

#include "scheduler.h"
#include "timer.h"

namespace sylar
{

/**
 * @brief   基于 epoll 的 IO 协程调度器
 */
class IOManager : public Scheduler, public TimerManager
{
public:
    typedef std::shared_ptr<IOManager> ptr;
    typedef RWMutex RWMutexType;

    /**
     * @brief  IO事件，直接继承自 epoll 对事件的定义
     *         只关心读写事件，其他事件也会归类到这两类事件
     */
    enum Event
    {
        NONE    = 0x0,  // 无事件
        READ    = 0x1,  // 读事件(EPOLLIN)
        WRITE   = 0x4,  // 写事件(EPOLLOUT)   
    };

private:
    /**
     * @brief   socket事件上下文
     */
    struct FdContext
    {
        typedef Mutex MutexType;

        /**
         * @brief  事件上下文 
         */
        struct EventContext
        {
            Scheduler* scheduler = nullptr;
            Fiber::ptr fiber;
            std::function<void()> cb;
        };

        /**
         * @brief   获取对应事件的上下文
         */
        EventContext& getContext(Event event);
        
        void resetContext(EventContext& ctx);
        void triggerEvent(Event event);

        /// 读事件上下文
        EventContext read;
        /// 写事件上下文
        EventContext write;
        /// 描述符
        int fd = 0;
        /// 所含事件
        Event events = NONE;
        MutexType mutex;
    };

public:
    IOManager(size_t threads = 1, bool use_caller = true,  const std::string& name = "");
    ~IOManager();

    int addEvent(int fd, Event event, std::function<void()> cb = nullptr);
    bool delEvent(int fd, Event event);
    bool cancelEvent(int fd, Event event);
    bool cancelAll(int fd);

    static IOManager* GetThis();

protected:
    void tickle() override;
    bool stopping() override;
    void idle() override;
    void onTimerInsertedAtFront() override;

    void contextResize(size_t size);

    /**
     * @brief  判断是否可以停止 
     *
     * @param   timeout 最近要触发的定时器事件间隔
     */
    bool stopping(uint64_t& timeout);

private:
    int m_epfd;
    /// 用于tickele的pipe文件描述符，fd[0]是读端，fd[1]是写端
    int m_tickleFds[2];
    /// 正在等待执行的IO事件数量
    std::atomic<size_t> m_pendingEventCount = {0};
    RWMutexType m_mutex;
    /// socket事件上下文的容器
    std::vector<FdContext*> m_fdContexts;
};

}

#endif


#include "iomanager.h"
#include "macro.h"

#include <fcntl.h>
#include <sys/epoll.h>
#include <errno.h>

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

enum EpollCtlOp
{};

static std::ostream& operator<<(std::ostream& os, const EpollCtlOp& op)
{
    switch((int)op)
    {
#define XX(ctl) \
        case ctl: \
            return os << #ctl;
        XX(EPOLL_CTL_ADD);
        XX(EPOLL_CTL_MOD);
        XX(EPOLL_CTL_DEL);
        default:
            return os << (int)op;
    }
#undef XX
}

static std::ostream& operator<<(std::ostream& os, EPOLL_EVENTS events)
{
    if(!events)
    {
        return os << "0";
    }

    bool first = true;
#define XX(E) \
    if(events & E) \
    { \
        if(!first) \
        { \
            os << "|"; \
        } \
        os << #E; \
        first = false; \
    }
    XX(EPOLLIN);
    XX(EPOLLPRI);
    XX(EPOLLOUT);
    XX(EPOLLRDNORM);
    XX(EPOLLRDBAND);
    XX(EPOLLWRNORM);
    XX(EPOLLWRBAND);
    XX(EPOLLMSG);
    XX(EPOLLERR);
    XX(EPOLLHUP);
    XX(EPOLLRDHUP);
    XX(EPOLLONESHOT);
    XX(EPOLLET);
#undef XX
    return os;
}

IOManager::FdContext::EventContext& IOManager::FdContext::getContext(Event event)
{
    switch(event)
    {
        case IOManager::READ:
            return read;
        case IOManager::WRITE:
            return write;
        default:
            SYLAR_ASSERT2(false, "getContext");
    }
    throw std::invalid_argument("getContext invalid event");
}

void IOManager::FdContext::resetContext(EventContext& ctx)
{
    ctx.scheduler = nullptr;
    ctx.fiber.reset();
    ctx.cb = nullptr;
}

void IOManager::FdContext::triggerEvent(Event event)
{
    SYLAR_ASSERT(events & event);           // 触发的事件必须是先存在的
    events = (Event)(events & ~event);      // 去掉要触发的事件
    EventContext& ctx = getContext(event);  // 拿到要触发的事件上下文，将它放入调度器里面去
    if(ctx.cb)
    {
        ctx.scheduler->schedule(&ctx.cb);
    }
    else
    {
        ctx.scheduler->schedule(&ctx.fiber);
    }
    ctx.scheduler = nullptr;
    return;
}

IOManager::IOManager(size_t threads, bool use_caller,  const std::string& name)
    : Scheduler(threads, use_caller, name)
{
    m_epfd = epoll_create(5000);
    SYLAR_ASSERT(m_epfd > 0);

    // 创建pipe，获取m_tickeleFds[2]，其中[0]是管道的读端，[1]是管道的写端
    int rt = pipe(m_tickleFds);
    SYLAR_ASSERT(!rt);

    // 注册pipe的读事件，ET模式。通过data.fd保存文件描述符
    epoll_event event;
    memset(&event, 0, sizeof(epoll_event));
    event.events = EPOLLIN | EPOLLET;
    event.data.fd = m_tickleFds[0];

    // 设置为非阻塞，配合ET模式使用
    rt = fcntl(m_tickleFds[0], F_SETFL, O_NONBLOCK);
    SYLAR_ASSERT(!rt);

    rt = epoll_ctl(m_epfd, EPOLL_CTL_ADD, m_tickleFds[0], &event);
    SYLAR_ASSERT(!rt);

    contextResize(32);

    // 创建即可调度协程
    start();
}

IOManager::~IOManager()
{
    stop();
    close(m_epfd);
    close(m_tickleFds[0]);
    close(m_tickleFds[1]);

    for(size_t i=0; i<m_fdContexts.size(); ++i)
    {
        if(m_fdContexts[i])
        {
            delete m_fdContexts[i];
            m_fdContexts[i] = nullptr;
        }
    }
}

int IOManager::addEvent(int fd, Event event, std::function<void()> cb)
{
    FdContext* fd_ctx = nullptr;
    RWMutexType::ReadLock lock(m_mutex);
    if((int)m_fdContexts.size() > fd)
    {
        fd_ctx = m_fdContexts[fd];
        lock.unlock();
    }
    else
    {
        lock.unlock();
        RWMutexType::WriteLock lock2(m_mutex);
        contextResize(fd * 1.5);
        fd_ctx = m_fdContexts[fd];
    }

    FdContext::MutexType::Lock lock2(fd_ctx->mutex);
    // 同一个fd不允许重复添加相同的事件
    if(SYLAR_UNLIKELY(fd_ctx->events & event))
    {
        SYLAR_LOG_ERROR(g_logger) << "addEvent assert fd=" << fd
            << " event=" << (EPOLL_EVENTS)event
            << " fd_ctx.event=" << (EPOLL_EVENTS)fd_ctx->events;
        SYLAR_ASSERT(!(fd_ctx->events & event));
    }

    int op = fd_ctx->events ? EPOLL_CTL_MOD : EPOLL_CTL_ADD;
    epoll_event epevent;
    epevent.events = EPOLLET | fd_ctx->events | event;
    epevent.data.ptr = fd_ctx;

    int rt = epoll_ctl(m_epfd, op, fd, &epevent);
    if(rt)
    {
        SYLAR_LOG_ERROR(g_logger) << "epoll_ctl(" << m_epfd << ", "
            << (EpollCtlOp)op << ", " << fd << ", " << (EPOLL_EVENTS)epevent.events << "):"
            << rt << " (" << errno << ") (" << strerror(errno) << ") fd_ctx->events="
            << (EPOLL_EVENTS)fd_ctx->events;
        return -1;
    }

    ++m_pendingEventCount;      // 待执行IO事件数加1

    fd_ctx->events = (Event)(fd_ctx->events | event);

    // 找到这个fd的event事件对应的EventContext，对其中的scheduler、cb或fiber进行赋值
    FdContext::EventContext& event_ctx = fd_ctx->getContext(event);
    SYLAR_ASSERT(!event_ctx.scheduler
              && !event_ctx.fiber
              && !event_ctx.cb);

    event_ctx.scheduler = Scheduler::GetThis();
    if(cb)
    {
        event_ctx.cb.swap(cb);
    }
    else
    {
        event_ctx.fiber = Fiber::GetThis();
        SYLAR_ASSERT2(event_ctx.fiber->getState() == Fiber::EXEC,
                "state=" << event_ctx.fiber->getState());
    }
    return 0;
}

bool IOManager::delEvent(int fd, Event event)
{
    RWMutexType::ReadLock lock(m_mutex);
    if((int)m_fdContexts.size() <= fd)
    {
        return false;
    }

    FdContext* fd_ctx = m_fdContexts[fd];
    lock.unlock();

    FdContext::MutexType::Lock lock2(fd_ctx->mutex);
    if(SYLAR_UNLIKELY(!(fd_ctx->events & event)))
    {
        return false;
    }

    Event new_events = (Event)(fd_ctx->events & ~event);
    int op = new_events ? EPOLL_CTL_MOD : EPOLL_CTL_DEL;
    epoll_event epevent;
    epevent.events = EPOLLET | new_events;
    epevent.data.ptr = fd_ctx;

    int rt = epoll_ctl(m_epfd, op, fd, &epevent);
    if(rt)
    {
        SYLAR_LOG_ERROR(g_logger) << "epoll_ctl(" << m_epfd << ", "
            << (EpollCtlOp)op << ", " << fd << ", " << (EPOLL_EVENTS)epevent.events << "):"
            << rt << " (" << errno << ") (" << strerror(errno) << ")";
        return false;
    }

    --m_pendingEventCount;      // 待执行事件减1

    // 重置该fd对应的event事件上下文
    fd_ctx->events = new_events;
    FdContext::EventContext& event_ctx = fd_ctx->getContext(event);
    fd_ctx->resetContext(event_ctx);
    return true;
}

/**
 * @brief  取消事件：如果该事件被注册过回调，那就触发一次回调事件 
 */
bool IOManager::cancelEvent(int fd, Event event)
{
    RWMutexType::ReadLock lock(m_mutex);
    if((int)m_fdContexts.size() <= fd)
    {
        return false;
    }

    FdContext* fd_ctx = m_fdContexts[fd];
    lock.unlock();

    FdContext::MutexType::Lock lock2(fd_ctx->mutex);
    if(SYLAR_UNLIKELY(!(fd_ctx->events & event)))
    {
        return false;
    }

    Event new_events = (Event)(fd_ctx->events & ~event);
    int op = new_events ? EPOLL_CTL_MOD : EPOLL_CTL_DEL;
    epoll_event epevent;
    epevent.events = EPOLLET | new_events;
    epevent.data.ptr = fd_ctx;

    int rt = epoll_ctl(m_epfd, op, fd, &epevent);
    if(rt)
    {
        SYLAR_LOG_ERROR(g_logger) << "epoll_ctl(" << m_epfd << ", "
            << (EpollCtlOp)op << ", " << fd << ", " << (EPOLL_EVENTS)epevent.events << "):"
            << rt << " (" << errno << ") (" << strerror(errno) << ")";
        return false;
    }

    // 触发一次事件
    fd_ctx->triggerEvent(event);
    --m_pendingEventCount;
    return true;
}

bool IOManager::cancelAll(int fd)
{
    RWMutexType::ReadLock lock(m_mutex);
    if((int)m_fdContexts.size() <= fd)
    {
        return false;
    }

    FdContext* fd_ctx = m_fdContexts[fd];
    lock.unlock();

    FdContext::MutexType::Lock lock2(fd_ctx->mutex);
    if(!fd_ctx->events)
    {
        return false;
    }

    int op = EPOLL_CTL_DEL;
    epoll_event epevent;
    epevent.events = 0;
    epevent.data.ptr = fd_ctx;

    int rt = epoll_ctl(m_epfd, op, fd, &epevent);
    if(rt)
    {
        SYLAR_LOG_ERROR(g_logger) << "epoll_ctl(" << m_epfd << ", "
            << (EpollCtlOp)op << ", " << fd << ", " << (EPOLL_EVENTS)epevent.events << "):"
            << rt << " (" << errno << ") (" << strerror(errno) << ")";
        return false;
    }

    // 触发全部已注册的事件
    if(fd_ctx->events & READ)
    {
        fd_ctx->triggerEvent(READ);
        --m_pendingEventCount;
    }
    if(fd_ctx->events & WRITE)
    {
        fd_ctx->triggerEvent(WRITE);
        --m_pendingEventCount;
    }

    SYLAR_ASSERT(fd_ctx->events == 0);
    return true;
}

IOManager* IOManager::GetThis()
{
    return dynamic_cast<IOManager*>(Scheduler::GetThis());
}

/**
 * @brief   通知调度器有任务要调度
 *          写pipe让idle协程从epoll_wait退出，
 *          等idle协程yield之后，Scheduler::run就可以调度其他任务了
 */
void IOManager::tickle()
{
    // 如果当前没有空闲的调度线程，则直接return掉
    if(!hasIdleThreads())
    {
        return;
    }

    int rt = write(m_tickleFds[1], "T", 1);
    SYLAR_ASSERT(rt == 1);
}

bool IOManager::stopping()
{
    uint64_t timeout = 0;
    return stopping(timeout);
}

/**
 * @brief   idle退出的时机是epoll_wait返回，对应的操作是tickle或注册的IO事件就绪
 *
 */
void IOManager::idle()
{
    SYLAR_LOG_DEBUG(g_logger) << "idle";
    // 一次epoll_wait最多检测256个就绪事件，如果就绪事件超过了这个数，那么会在下轮epoll_wait继续处理
    const uint64_t MAX_EVENTS = 256;
    epoll_event* events = new epoll_event[MAX_EVENTS]();
    std::shared_ptr<epoll_event> shared_events(events, [](epoll_event* ptr)    // 自定义删除函数
    {
        delete[] ptr;
    });

    while(true)
    {
        uint64_t next_timeout = 0;
        if(SYLAR_UNLIKELY(stopping(next_timeout)))
        {
            SYLAR_LOG_INFO(g_logger) << "name=" << getName()
                                     << " idle stopping exit";
            break;
        }

        int rt = 0;
        do
        {
            static const int MAX_TIMEOUT = 3000;
            // 选择小的那个时间
            if(next_timeout != ~0ull)
            {
                next_timeout = (int)next_timeout > MAX_TIMEOUT ? MAX_TIMEOUT : next_timeout;
            }
            else
            {
                next_timeout = MAX_TIMEOUT;
            }

            rt = epoll_wait(m_epfd, events, MAX_EVENTS, (int)next_timeout);
            if(rt < 0 && errno == EINTR)
            {}
            else
            {
                break;
            }
        }while(true);

        std::vector<std::function<void()> > cbs;
        listExpiredCb(cbs);     // 拿到所有到期的定时任务
        if(!cbs.empty())
        {
            schedule(cbs.begin(), cbs.end());
            cbs.clear();
        }

        for(int i=0; i<rt; ++i)
        {
            epoll_event& event = events[i];
            if(event.data.fd == m_tickleFds[0])
            {
                // tickeleFd[0]用于通知协程调度，
                // 这时只需要把管道里的内容读完即可，
                // 本轮idle结束后，Scheduler::run会重新执行协程调度
                uint8_t dummy[256];
                while(read(m_tickleFds[0], dummy, sizeof(dummy)) > 0);
                continue;
            }

            FdContext* fd_ctx = (FdContext*)event.data.ptr;
            FdContext::MutexType::Lock lk(fd_ctx->mutex);
            // EPOLLERR 出错，比如写读端以及关闭的pipe
            // EPOLLHUP 套接字对端关闭
            // 出现这两种事件时，应该同时触发fd的读和写事件，否则有可能出现注册的事件永远执行不到的情况
            if(event.events & (EPOLLERR | EPOLLHUP))
            {
                event.events |= (EPOLLIN | EPOLLOUT) & fd_ctx->events;
            }

            int real_events = NONE;
            if(event.events & EPOLLIN)
            {
                real_events |= READ;
            }
            if(event.events & EPOLLOUT)
            {
                real_events |= WRITE;
            }

            if((fd_ctx->events & real_events) == NONE)  // 没有发生所存的事件，则continue
            {
                continue;
            }

            // 剔除已经发生的事件，将剩下的事件重新加入epoll_wait
            int left_events = (fd_ctx->events & ~real_events);
            int op = left_events ? EPOLL_CTL_MOD : EPOLL_CTL_DEL;
            event.events = EPOLLET | left_events;
            
            int rt2 = epoll_ctl(m_epfd, op, fd_ctx->fd, &event);
            if(rt2)
            {
                SYLAR_LOG_ERROR(g_logger) << "epoll_ctl(" << m_epfd << ", "
                    << (EpollCtlOp)op << ", " << fd_ctx->fd << ", " << (EPOLL_EVENTS)event.events << "):"
                    << rt2 << " (" << errno << ") (" << strerror(errno) << ")";
                continue;
            }

            // 处理已经发送的事件，也就是让调度器调度指定的函数或协程
            if(real_events & READ)
            {
                fd_ctx->triggerEvent(READ);
                --m_pendingEventCount;
            }
            if(real_events & WRITE)
            {
                fd_ctx->triggerEvent(WRITE);
                --m_pendingEventCount;
            }
        }

        // 一旦处理完所有的事件，idle协程切出，这样就可以让调度协程Scheduler::run重新检查是否有新任务要调度
        Fiber::ptr cur = Fiber::GetThis();
        auto raw_ptr = cur.get();
        cur.reset();

        raw_ptr->swapOut();
    }
}

/**
 * @brief   有定时事件插入到定时器的最前面，则马上tickle一下
 */
void IOManager::onTimerInsertedAtFront()
{
    tickle();
}

void IOManager::contextResize(size_t size)
{
    m_fdContexts.resize(size);

    for(size_t i=0; i<m_fdContexts.size(); ++i)
    {
        if(!m_fdContexts[i])
        {
            m_fdContexts[i] = new FdContext;
            m_fdContexts[i]->fd = i;
        }
    }
}

bool IOManager::stopping(uint64_t& timeout)
{
    timeout = getNextTimer();           // 获取下一个定时器的超时时间
    return timeout == ~0ull
        && m_pendingEventCount == 0     // 所有IO事件都完成调度
        && Scheduler::stopping();
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
