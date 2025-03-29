# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- hook 模块 hook 底层和 socket 相关的 API、socket IO 相关的 API、sleep 相关的API。
- hook 的开启控制是线程粒度的，可以自由选择（通过线程局部变量实现）。
- 通过 hook 模块，可以使一些不具异步功能的 API，展现出异步的性能，如 MySQL。实现上是通过定时器、协程、IO协程调度等模块的配合。
- hook 实际上就是对系统调用 API 进行一次封装，将其封装成一个与原始的系统调用 API 同名的接口，应用在调用这个接口时，会先执行封装中的操作，再执行原始的系统调用 API。
- hook 的目的是在不重新编写代码的情况下，把老代码中的 socket IO 相关的 API 都转成异步，以提高性能。即以同步的方式编写代码，实现的效果却是异步执行的，效率很高。
- 在 IO 协程调度中对相关的系统调用进行 hook，可以让调度线程尽可能得把时间片都花在有意义的操作上，而不是浪费在阻塞等待中。
- hook 的实现机制就是通过动态库的全局符号介入功能，用自定义的接口来替换掉同名的系统调用接口。由于系统调用接口基本上是由 C 标准函数库 libc 提供的，所以这里要做的事情就是用自定义的动态库来覆盖掉 libc 中的同名符号。
	- 那些被覆盖的符号，它们只是被“雪藏”了而已，实际还是存在于程序的进程空间中的，通过一定的办法，可以把它们再找回来。在 Linux 中，这个方法就是 dslym，它的函数原型如下：
	```
	#define _GNU_SOURCE
	#include <dlfcn.h>
	void *dlsym(void *handle, const char *symbol);
	```
	- 关于 dlsym 的使用可参考 man 3 dlsym，在链接时需要指定 -ldl 参数。使用 dlsym 找回被覆盖的符号时，第一个参数固定为 RTLD_NEXT，第二个参数为符号的名称。
- 为了管理所有的 socket fd，sylar 设计了一个 FdManager 类来记录所有分配过的 fd 的上下文，这是一个单例类，每个 socket fd 上下文记录了当前 fd 的读写超时，是否设置非阻塞等信息。


# FdCtx

- 文件句柄上下文类。
- FdCtx 类在用户态记录了 fd 的读写超时和非阻塞信息，其中非阻塞包括用户显式设置的非阻塞和 hook 内部设置的非阻塞，区分这两种非阻塞可以有效应对用户对 fd 设置/获取 NONBLOCK 模式的情形。


# FdManager

- 文件局部上下文管理类。
- FdManager 类对 FdCtx 的寻址采用了直接用fd作为数组下标进行寻址的方式。


# hook
- hook 模块。
- 线程局部变量 t_hook_enable，用于表示当前线程是否启用 hook，使用线程局部变量表示 hook 模块是线程粒度的，各个线程可单独启用或关闭 hook。
- hook_init() 函数放在 _HookIniter 结构体里面，而 _HookIniter 类型的全局静态变量 s_hook_initer 会在 main() 函数运行之前初始化，即会在 main() 函数运行之前调用 hook_init()。
- 各个接口的 hook 实现要放在 extern "C" 中，以防止 C++ 编译器对符号名称添加修饰。


# 其他说明

- 在编译时给 gcc 增加一个 "-v" 参数，可以将整个编译流程详细地打印出来。
- 通过设置 LD_PRELOAD 环境变量，可以设置成优先加载相关动态库。
- 默认情况下，协程调度器的调度线程会开启 hook，而其他线程则不会开启。
- hook socket/fcntl/ioctl/close 等接口，这类接口主要处理的是边缘情况，比如分配 fd 上下文，处理超时及用户显式设置非阻塞问题。
- 所有参与协程调度的 fd 都会被设置成非阻塞模式，所以要在应用层维护好用户设置的非阻塞标志。
- 按 sylar hook 模块的实现，非调度线程不支持启用 hook。


# 部分相关代码
```C++
/**
 * @filename    hook.h
 * @brief   hook 模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-03
 */
#ifndef __SYLAR_HOOK_H__
#define __SYLAR_HOOK_H__

#include <fcntl.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <stdint.h>
#include <time.h>
#include <unistd.h>

namespace sylar
{
    bool is_hook_enable();

    void set_hook_enable(bool flag);
}

extern "C"      // 防止 C++ 编译器对符号名称添加修饰
{

/// sleep
typedef unsigned int (*sleep_fun)(unsigned int seconds);
extern sleep_fun sleep_f;

typedef int (*usleep_fun)(useconds_t usec);
extern usleep_fun usleep_f;

typedef int (*nanosleep_fun)(const struct timespec* req, struct timespec* rem);
extern nanosleep_fun nanosleep_f;


/// socket
typedef int (*socket_fun)(int domain, int type, int protocol);
extern socket_fun socket_f;

typedef int (*connect_fun)(int sockfd, const struct sockaddr* addr, socklen_t addrlen);
extern connect_fun connect_f;

typedef int (*accept_fun)(int s, struct sockaddr* addr, socklen_t* addrlen);
extern accept_fun accept_f;


/// read
typedef ssize_t (*read_fun)(int fd, void* buf, size_t count);
extern read_fun read_f;

typedef ssize_t (*readv_fun)(int fd, const struct iovec* iov, int iovcnt);
extern readv_fun readv_f;

typedef ssize_t (*recv_fun)(int sockfd, void* buf, size_t len, int flags);
extern recv_fun recv_f;

typedef ssize_t (*recvfrom_fun)(int sockfd, void* buf, size_t len, int flags, struct sockaddr* src_addr, socklen_t* addrlen);
extern recvfrom_fun recvfrom_f;

typedef ssize_t (*recvmsg_fun)(int sockfd, struct msghdr* msg, int flags);
extern recvmsg_fun recvmsg_f;


/// write
typedef ssize_t (*write_fun)(int fd, const void* buf, size_t count);
extern write_fun write_f;

typedef ssize_t (*writev_fun)(int fd, const struct iovec* iov, int iovcnt);
extern writev_fun writev_f;

typedef ssize_t (*send_fun)(int s, const void* msg, size_t len, int flags);
extern send_fun send_f;

typedef ssize_t (*sendto_fun)(int s, const void* msg, size_t len, int flags, const struct sockaddr* to, socklen_t tolen);
extern sendto_fun sendto_f;

typedef ssize_t (*sendmsg_fun)(int s, const struct msghdr* msg, int flags);
extern sendmsg_fun sendmsg_f;


typedef int (*close_fun)(int fd);
extern close_fun close_f;


typedef int (*fcntl_fun)(int fd, int cmd, ... /* arg */ );
extern fcntl_fun fcntl_f;

typedef int (*ioctl_fun)(int d, unsigned long int request, ...);
extern ioctl_fun ioctl_f;

typedef int (*getsockopt_fun)(int sockfd, int level, int optname, void* optval, socklen_t* optlen);
extern getsockopt_fun getsockopt_f;

typedef int (*setsockopt_fun)(int sockfd, int level, int optname, const void* optval, socklen_t optlen);
extern setsockopt_fun setsockopt_f;


extern int connect_with_timeout(int fd, const struct sockaddr* addr, socklen_t addrlen, uint64_t timeout_ms);

}

#endif

// hook.cc
#include "hook.h"
#include "log.h"
#include "config.h"
#include "macro.h"
#include "fiber.h"
#include "iomanager.h"
#include "fdmanager.h"
#include <dlfcn.h>

sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

namespace sylar
{

static sylar::ConfigVar<int>::ptr g_tcp_connect_timeout = 
    sylar::Config::Lookup("tcp.connect.timeout", 5000, "tcp connect timeout");

static thread_local bool t_hook_enable = false;     // 当前线程是否 hook 的标志

#define HOOK_FUN(XX) \
    XX(sleep) \
    XX(usleep) \
    XX(nanosleep) \
    XX(socket) \
    XX(connect) \
    XX(accept) \
    XX(read) \
    XX(readv) \
    XX(recv) \
    XX(recvfrom) \
    XX(recvmsg) \
    XX(write) \
    XX(writev) \
    XX(send) \
    XX(sendto) \
    XX(sendmsg) \
    XX(close) \
    XX(fcntl) \
    XX(ioctl) \
    XX(getsockopt) \
    XX(setsockopt)

void hook_init()
{
    static bool is_inited = false;
    if(is_inited)
    {
        return;
    }
#define XX(name) name ## _f = (name ## _fun)dlsym(RTLD_NEXT, #name);
    // 这里宏展开之后就是，以name为sleep举例：
    // sleep_f = (sleep_fun)dlsym(RTLD_NEXT, "sleep");
    HOOK_FUN(XX);
#undef XX
}

static uint64_t s_connect_timeout = -1;
struct _HookIniter
{
    _HookIniter()
    {
        hook_init();
        s_connect_timeout = g_tcp_connect_timeout->getValue();

        g_tcp_connect_timeout->addListener([](const int& old_value, const int& new_value)
        {
            SYLAR_LOG_INFO(g_logger) << "tcp connect timeout changed from "
                                     << old_value << " to " << new_value;
            s_connect_timeout = new_value;
        });
    }
};

static _HookIniter s_hook_initer;

bool is_hook_enable()
{
    return t_hook_enable;
}

void set_hook_enable(bool flag)
{
    t_hook_enable = flag;
}

}  // namespace sylar end

struct timer_info
{
    int cancelled = 0;
};

template<typename OriginFun, typename... Args>
static ssize_t do_io(int fd, OriginFun fun, const char* hook_fun_name, 
                     uint32_t event, int timeout_so, Args&&... args)
{
    if(!sylar::t_hook_enable)
    {
        return fun(fd, std::forward<Args>(args)...);
    }

    sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(fd);
    if(!ctx)
    {
        return fun(fd, std::forward<Args>(args)...);
    }

    if(ctx->isClose())
    {
        errno = EBADF;
        return -1;
    }

    // 如果不是 socket fd 或是用户显式设置过非阻塞模式，那么就不需要 hook 了。
    if(!ctx->isSocket() || ctx->getUserNonblock())
    {
        return fun(fd, std::forward<Args>(args)...);
    }

    uint64_t to = ctx->getTimeout(timeout_so); // 得到该类型的超时时间
    std::shared_ptr<timer_info> tinfo(new timer_info);

retry:
    ssize_t n = fun(fd, std::forward<Args>(args)...);
    while(n == -1 && errno == EINTR)    // 被中断
    {
        n = fun(fd, std::forward<Args>(args)...);
    }

    if(n == -1 && errno == EAGAIN)      // 再次尝试（fd是非阻塞的）
    {
        sylar::IOManager* iom = sylar::IOManager::GetThis();
        sylar::Timer::ptr timer;
        std::weak_ptr<timer_info> winfo(tinfo); // 用于条件定时器

        if(to != (uint64_t)-1)          // 超时时间有效
        {
            timer = iom->addConditionTimer(to, [winfo, fd, iom, event]()
            {
                auto t = winfo.lock();
                if(!t || t->cancelled)
                {
                    return;
                }
                // 设置超时标志，并触发一次对应的事件
                t->cancelled = ETIMEDOUT;
                iom->cancelEvent(fd, (sylar::IOManager::Event)(event));
            }, winfo);
        }

        int rt = iom->addEvent(fd, (sylar::IOManager::Event)(event));
        if(SYLAR_UNLIKELY(rt))
        {
            SYLAR_LOG_ERROR(g_logger) << hook_fun_name << " addEvent("
                << fd << ", " << event << ")";
            if(timer)
            {
                timer->cancel();
            }
            return -1;
        }
        else       // 添加事件成功后，当前协程让出CPU
        {
            sylar::Fiber::YieldToHold();
            if(timer)
            {
                timer->cancel();
            }

            // 协程从上面的 YieldHold 点返回，返回之后通过超时标志设置 errno 并返回 -1。
            if(tinfo->cancelled)
            {
                errno = tinfo->cancelled;
                return -1;
            }
            goto retry;     // 没有超时，则继续再去读写（EAGAIN才会进入这里面）
        }
    }
    
    return n;
}

extern "C"
{
#define XX(name) name ## _fun name ## _f = nullptr;
    HOOK_FUN(XX);
#undef XX

unsigned int sleep(unsigned int seconds)
{
    if(!sylar::t_hook_enable)
    {
        return sleep_f(seconds);
    }

    sylar::Fiber::ptr fiber = sylar::Fiber::GetThis();
    sylar::IOManager* iom = sylar::IOManager::GetThis();
    iom->addTimer(seconds * 1000, std::bind(
                (void(sylar::Scheduler::*)(sylar::Fiber::ptr, int thread))&sylar::IOManager::schedule, 
                iom, fiber, -1));
    sylar::Fiber::YieldToHold();    // 添加定时器后，当前协程让出CPU，等它从这个点返回的时候，就是超时到了的时候。
    return 0;
}

int usleep(useconds_t usec)
{
    if(!sylar::t_hook_enable)
    {
        return usleep_f(usec);
    }

    sylar::Fiber::ptr fiber = sylar::Fiber::GetThis();
    sylar::IOManager* iom = sylar::IOManager::GetThis();
    iom->addTimer(usec / 1000, std::bind(
                (void(sylar::Scheduler::*)(sylar::Fiber::ptr, int thread))&sylar::IOManager::schedule, 
                iom, fiber, -1));
    sylar::Fiber::YieldToHold();
    return 0;
}

int nanosleep(const struct timespec* req, struct timespec* rem)
{
    if(!sylar::t_hook_enable)
    {
        return nanosleep_f(req, rem);
    }

    int timeout_ms = req->tv_sec * 1000 + req->tv_nsec / 1000 / 1000;
    sylar::Fiber::ptr fiber = sylar::Fiber::GetThis();
    sylar::IOManager* iom = sylar::IOManager::GetThis();
    iom->addTimer(timeout_ms, std::bind(
                (void(sylar::Scheduler::*)(sylar::Fiber::ptr, int thread))&sylar::IOManager::schedule, 
                iom, fiber, -1));
    sylar::Fiber::YieldToHold();
    return 0;
}

int socket(int domain, int type, int protocol)
{
    if(!sylar::t_hook_enable) 
    {
        return socket_f(domain, type, protocol);
    }
    
    int fd = socket_f(domain, type, protocol);
    if(fd == -1)
    {
        return fd;
    }
    sylar::FdMgr::GetInstance()->get(fd, true); // 如果文件描述符管理器里面没有，则自动创建
    return fd;
}

int connect(int sockfd, const struct sockaddr* addr, socklen_t addrlen)
{
    return connect_with_timeout(sockfd, addr, addrlen, sylar::s_connect_timeout);
}

int accept(int s, struct sockaddr* addr, socklen_t* addrlen)
{
    int fd = do_io(s, accept_f, "accept", sylar::IOManager::READ, SO_RCVTIMEO, addr, addrlen);
    if(fd >= 0)
    {
        sylar::FdMgr::GetInstance()->get(fd, true);
    }
    return fd;
}

ssize_t read(int fd, void* buf, size_t count)
{
    return do_io(fd, read_f, "read", sylar::IOManager::READ, SO_RCVTIMEO, buf, count);
}

ssize_t readv(int fd, const struct iovec* iov, int iovcnt)
{
    return do_io(fd, readv_f, "readv", sylar::IOManager::READ, SO_RCVTIMEO, iov, iovcnt);
}

ssize_t recv(int sockfd, void* buf, size_t len, int flags)
{
    return do_io(sockfd, recv_f, "recv", sylar::IOManager::READ, SO_RCVTIMEO, buf, len, flags);
}

ssize_t recvfrom(int sockfd, void* buf, size_t len, int flags, struct sockaddr* src_addr, socklen_t* addrlen)
{
    return do_io(sockfd, recvfrom_f, "recvfrom", sylar::IOManager::READ, SO_RCVTIMEO, buf, len, flags, src_addr, addrlen);
}

ssize_t recvmsg(int sockfd, struct msghdr* msg, int flags)
{
    return do_io(sockfd, recvmsg_f, "recvmsg", sylar::IOManager::READ, SO_RCVTIMEO, msg, flags);
}

ssize_t write(int fd, const void* buf, size_t count)
{
    return do_io(fd, write_f, "write", sylar::IOManager::WRITE, SO_SNDTIMEO, buf, count);
}

ssize_t writev(int fd, const struct iovec* iov, int iovcnt)
{
    return do_io(fd, writev_f, "writev", sylar::IOManager::WRITE, SO_SNDTIMEO, iov, iovcnt);
}

ssize_t send(int s, const void* msg, size_t len, int flags)
{
    return do_io(s, send_f, "send", sylar::IOManager::WRITE, SO_SNDTIMEO, msg, len, flags);
}

ssize_t sendto(int s, const void* msg, size_t len, int flags, const struct sockaddr* to, socklen_t tolen)
{
    return do_io(s, sendto_f, "sendto", sylar::IOManager::WRITE, SO_SNDTIMEO, msg, len, flags, to, tolen);
}

ssize_t sendmsg(int s, const struct msghdr* msg, int flags)
{
    return do_io(s, sendmsg_f, "sendmsg", sylar::IOManager::WRITE, SO_SNDTIMEO, msg, flags);
}

int close(int fd)
{
    if(!sylar::t_hook_enable)
    {
        return close_f(fd);
    }

    sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(fd);
    if(ctx)
    {
        auto iom = sylar::IOManager::GetThis();
        if(iom)
        {
            iom->cancelAll(fd); // 此时会让 fd 的读写事件回调都执行一次
        }
        sylar::FdMgr::GetInstance()->del(fd);
    }
    return close_f(fd);
}

int fcntl(int fd, int cmd, ... /* arg */ )
{
    va_list va;
    va_start(va, cmd);
    switch(cmd)
    {
        case F_SETFL:
            {
                int arg = va_arg(va, int);
                va_end(va);
                sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(fd);
                if(!ctx || ctx->isClose() || !ctx->isSocket())
                {
                    return fcntl_f(fd, cmd, arg);
                }

                ctx->setUserNonblock(arg & O_NONBLOCK); // 用户主动设置非阻塞
                
                if(ctx->getSysNonblock())
                {
                    arg |= O_NONBLOCK;
                }
                else
                {
                    arg &= ~O_NONBLOCK;
                }
                return fcntl_f(fd, cmd, arg);
            }
            break;
        case F_GETFL:
            {
                va_end(va);
                int arg = fcntl_f(fd, cmd);
                sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(fd);
                if(!ctx || ctx->isClose() || !ctx->isSocket())
                {
                    return arg;
                }

                // 用户得到的是否阻塞，是它曾经主动设置过的
                if(ctx->getUserNonblock())
                {
                    return arg |= O_NONBLOCK;
                }
                else
                {
                    return arg &= ~O_NONBLOCK;
                }
            }
            break;
        case F_DUPFD:
        case F_DUPFD_CLOEXEC:
        case F_SETFD:
        case F_SETOWN:
        case F_SETSIG:
        case F_SETLEASE:
        case F_NOTIFY:
#ifdef F_SETPIPE_SZ
        case F_SETPIPE_SZ:
#endif
            {
                int arg = va_arg(va, int);
                va_end(va);
                return fcntl_f(fd, cmd, arg);
            }
            break;
        case F_GETFD:
        case F_GETOWN:
        case F_GETSIG:
        case F_GETLEASE:
#ifdef F_GETPIPE_SZ
        case F_GETPIPE_SZ:
#endif
            {
                va_end(va);
                return fcntl_f(fd, cmd);
            }
            break;
        case F_SETLK:
        case F_SETLKW:
        case F_GETLK:
            {
                struct flock* arg = va_arg(va, struct flock*);
                va_end(va);
                return fcntl_f(fd, cmd, arg);
            }
            break;
        case F_GETOWN_EX:
        case F_SETOWN_EX:
            {
                struct f_owner_exlock* arg = va_arg(va, struct f_owner_exlock*);
                va_end(va);
                return fcntl_f(fd, cmd, arg);
            }
            break;
        default:
            va_end(va);
            return fcntl_f(fd, cmd);
    }
}

int ioctl(int d, unsigned long int request, ...)
{
    va_list va;
    va_start(va, request);
    void* arg = va_arg(va, void*);
    va_end(va);

    if(FIONBIO == request)      // 用户主动设置是否非阻塞
    {
        bool user_nonblock = !!*(int*)arg;
        sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(d);
        if(!ctx || ctx->isClose() || !ctx->isSocket())
        {
            return ioctl_f(d, request, arg);
        }
        ctx->setUserNonblock(user_nonblock);
    }
    return ioctl_f(d, request, arg);
}

int getsockopt(int sockfd, int level, int optname, void* optval, socklen_t* optlen)
{
    return getsockopt_f(sockfd, level, optname, optval, optlen);
}

int setsockopt(int sockfd, int level, int optname, const void* optval, socklen_t optlen)
{
    if(!sylar::t_hook_enable)
    {
        return setsockopt_f(sockfd, level, optname, optval, optlen);
    }

    if(level == SOL_SOCKET)
    {
        // 用户设置的超时时间要同步给文件描述符管理器
        if(optname == SO_RCVTIMEO || optname == SO_SNDTIMEO)
        {
            sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(sockfd);
            if(ctx)
            {
                const timeval* v = (const timeval*)optval;
                ctx->setTimeout(optname, v->tv_sec * 1000 + v->tv_usec / 1000);
            }
        }
    }
    return setsockopt_f(sockfd, level, optname, optval, optlen);
}

int connect_with_timeout(int fd, const struct sockaddr* addr, socklen_t addrlen, uint64_t timeout_ms)
{
    if(!sylar::t_hook_enable)
    {
        return connect_f(fd, addr, addrlen);
    }

    sylar::FdCtx::ptr ctx = sylar::FdMgr::GetInstance()->get(fd);
    if(!ctx || ctx->isClose())
    {
        errno = EBADF;
        return -1;
    }

    if(!ctx->isSocket())        // 判断是否为套接字
    {
        return connect_f(fd, addr, addrlen);
    }

    if(ctx->getUserNonblock())  // 是否被用户显式设置了非阻塞模式
    {
        return connect_f(fd, addr, addrlen);
    }

    int n = connect_f(fd, addr, addrlen);   // 套接字是非阻塞的，因此会直接返回EINPROGRESS
    if(n == 0)
    {
        return 0;
    }
    else if(n != -1 || errno != EINPROGRESS)
    {
        return n;
    }

    sylar::IOManager* iom = sylar::IOManager::GetThis();
    sylar::Timer::ptr timer;
    std::shared_ptr<timer_info> tinfo(new timer_info);
    std::weak_ptr<timer_info> winfo(tinfo);

    if(timeout_ms != (uint64_t)-1)  // 传入的超时参数有效
    {
        timer = iom->addConditionTimer(timeout_ms, [winfo, fd, iom]()
                {
                    auto t = winfo.lock();
                    if(!t || t->cancelled)
                    {
                        return;
                    }
                    // 设置超时标志，并触发一次写事件
                    t->cancelled = ETIMEDOUT;
                    iom->cancelEvent(fd, sylar::IOManager::WRITE);
                }, winfo);
    }

    int rt = iom->addEvent(fd, sylar::IOManager::WRITE);
    if(rt == 0)
    {
        sylar::Fiber::YieldToHold();    // 成功添加写事件之后就让出，等待写事件触发再往下执行
        // 等待超时或套接字可写，这里是未超时，套接字就可写了，那么直接取消定时器。
        // 就算超时了，也是取消定时器。
        if(timer)
        {
            timer->cancel();
        }

        // 等待超时或套接字可写，如果先超时，则条件变量 winfo 仍然有效，
        // 通过 winfo 拿到 t 来设置超时标志并触发WRITE事件（上面的条件定时器），
        // 协程从上面的 YieldHold 点返回，返回之后通过超时标志设置 errno 并返回 -1。
        if(tinfo->cancelled)
        {
            errno = tinfo->cancelled;
            return -1;
        }
    }
    else
    {
        if(timer)
        {
            timer->cancel();
        }
        SYLAR_LOG_ERROR(g_logger) << "connect addEvent (" << fd << ", WRITE) error";
    }

    int error = 0;
    socklen_t len = sizeof(int);
    if(-1 == getsockopt(fd, SOL_SOCKET, SO_ERROR, &error, &len)) // 获取socket的错误信息
    {
        return -1;
    }
    if(!error)
    {
        return 0;
    }
    else
    {
        errno = error;
        return -1;
    }
}

}

/**
 * @filename    fdmanager.h
 * @brief   文件描述符管理类
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-03
 */
#ifndef __SYLAR_FDMANAGER_H__
#define __SYLAR_FDMANAGER_H__

#include <memory>
#include <vector>
#include "mutex.h"
#include "singleton.h"

namespace sylar
{

/**
 * @brief   文件描述符上下文类(其实就是对文件描述符一些属性的封装)
 */
class FdCtx : public std::enable_shared_from_this<FdCtx>
{
public:
    typedef std::shared_ptr<FdCtx> ptr;

    FdCtx(int fd);
    ~FdCtx();

    bool isInit() const { return m_isInit; }
    bool isSocket() const { return m_isSocket; }
    bool isClose() const { return m_isClosed; }

    void setUserNonblock(bool v) { m_userNonblock = v; }
    bool getUserNonblock() { return m_userNonblock; }
    void setSysNonblock(bool v) { m_sysNonblock = v; }
    bool getSysNonblock() { return m_sysNonblock; }

    /**
     * @brief  设置超时时间 
     *
     * @param   type    类型：SO_RCVTIMEO(读超时)、SO_SNDTIMEO(写超时)
     * @param   v
     */
    void setTimeout(int type, uint64_t v);
    
    /**
     * @brief   获取超时时间
     *
     * @param   type    类型：SO_RCVTIMEO(读超时)、SO_SNDTIMEO(写超时)
     */
    uint64_t getTimeout(int type);

private:
    bool init();

private:
    /// 是否初始化
    bool m_isInit:1;
    /// 是否是socket
    bool m_isSocket:1;
    /// 是否 hook 非阻塞
    bool m_sysNonblock:1;
    /// 是否用户主动设置非阻塞
    bool m_userNonblock:1;
    /// 是否关闭
    bool m_isClosed:1;
    /// 文件描述符
    int m_fd;
    /// 读超时时间
    uint64_t m_recvTimeout;
    /// 写超时时间
    uint64_t m_sendTimeout;
};

/**
 * @brief   文件描述符管理类
 */
class FdManager
{
public:
    typedef RWMutex RWMutexType;

    FdManager();

    /**
     * @brief   获取/创建文件描述符类 FdCtx
     *
     * @param   fd  文件描述符
     * @param   auto_create 是否自动创建
     *
     * @return  返回对应文件描述符类 FdCtx::ptr
     */
    FdCtx::ptr get(int fd, bool auto_create = false);
    void del(int fd);

private:
    RWMutexType m_mutex;
    std::vector<FdCtx::ptr> m_datas;
};

typedef Singleton<FdManager> FdMgr;     // 文件描述符管理器单例

}

#endif

// fdmanager.cc
#include "fdmanager.h"
#include "hook.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

namespace sylar
{

FdCtx::FdCtx(int fd)
    : m_isInit(false)
    , m_isSocket(false)
    , m_sysNonblock(false)
    , m_userNonblock(false)
    , m_isClosed(false)
    , m_fd(fd)
    , m_recvTimeout(-1)
    , m_sendTimeout(-1)
{
    init();
}

FdCtx::~FdCtx()
{
}

void FdCtx::setTimeout(int type, uint64_t v)
{
    if(type == SO_RCVTIMEO)
    {
        m_recvTimeout = v;
    }
    else
    {
        m_sendTimeout = v;
    }
}

uint64_t FdCtx::getTimeout(int type)
{
    if(type == SO_RCVTIMEO)
    {
        return m_recvTimeout;
    }
    else
    {
        return m_sendTimeout;
    }
}

bool FdCtx::init()
{
    if(m_isInit)
    {
        return true;
    }

    m_recvTimeout = -1;
    m_sendTimeout = -1;

    struct stat fd_stat;
    if(-1 == fstat(m_fd, &fd_stat))
    {
        m_isInit = false;
        m_isSocket = false;
    }
    else
    {
        m_isInit = true;
        m_isSocket = S_ISSOCK(fd_stat.st_mode);
    }

    // 如果是socket，设置为非阻塞
    if(m_isSocket)
    {
        int flags = fcntl_f(m_fd, F_GETFL, 0);
        if(!(flags & O_NONBLOCK))
        {
            fcntl_f(m_fd, F_SETFL, flags | O_NONBLOCK);
        }
        m_sysNonblock = true;
    }
    else
    {
        m_sysNonblock = false;
    }

    m_userNonblock = false;
    m_isClosed = false;
    return m_isInit;
}

FdManager::FdManager()
{
    m_datas.resize(64);
}

FdCtx::ptr FdManager::get(int fd, bool auto_create)
{
    if(fd == -1)
    {
        return nullptr;
    }

    RWMutexType::ReadLock lock(m_mutex);
    if((int)m_datas.size() <= fd)
    {
        if(auto_create == false)
        {
            return nullptr;
        }
    }
    else
    {
        if(m_datas[fd] || !auto_create)
        {
            return m_datas[fd];
        }
    }
    lock.unlock();

    RWMutexType::WriteLock lock2(m_mutex);
    FdCtx::ptr ctx(new FdCtx(fd));
    if(fd >= (int)m_datas.size())
    {
        m_datas.resize(fd * 1.5);
    }
    
    m_datas[fd] = ctx;
    return ctx;
}

void FdManager::del(int fd)
{
    RWMutexType::WriteLock lk(m_mutex);
    if((int)m_datas.size() <= fd)
    {
        return;
    }
    m_datas[fd].reset();
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
