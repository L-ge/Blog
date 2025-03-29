# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 线程相关的封装。


# Thread

- 线程类。
- 私有继承自 Noncopyable 类，不可拷贝和赋值。
- 基于 pthread_t 实现。
- 构造时传入线程入口函数 std::function<void()> 和线程名称。其中，线程入口函数如果需要带参数，用 std::bind 绑定即可。构造函数通过信号量机制，使得构造函数在线程入口函数真正执行前（此时 pthread_create 传入的线程执行函数已经是执行中的状态）才返回。
- 内含两个线程局部变量 t_thread 和 t_thread_name，其中 t_thread 是当前线程的 Thread 指针，t_thread_name 是当前线程的线程名称。


# 部分相关代码
```C++
/**
 * @filename    thread.h
 * @brief   线程模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-26
 */
#ifndef __SYLAR_THREAD_H__
#define __SYLAR_THREAD_H__

#include "mutex.h"

namespace sylar
{

/**
 * @brief   线程类，不可拷贝和赋值
 */
class Thread : Noncopyable
{
public:
    typedef std::shared_ptr<Thread> ptr;

    Thread(std::function<void()> cb, const std::string& name);
    ~Thread();

    pid_t getId() const { return m_id; }
    const std::string& getName() const { return m_name; }
    void join();
    
    static Thread* GetThis();
    static const std::string& GetName();
    static void SetName(const std::string& name);

private:
    static void* run(void* arg);

private:
    pid_t m_id = -1;
    pthread_t m_thread = 0;
    std::function<void()> m_cb;
    std::string m_name;
    Semaphore m_semaphore;
};

}

#endif


#include "thread.h"
#include "log.h"
#include "util.h"

namespace sylar
{

static thread_local Thread* t_thread = nullptr;
static thread_local std::string t_thread_name = "UNKNOW";

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

Thread::Thread(std::function<void()> cb, const std::string& name)
    : m_cb(cb)
    , m_name(name)
{
    if(name.empty())
    {
        m_name = "UNKNOW";
    }

    int rt = pthread_create(&m_thread, nullptr, &Thread::run, this);
    if(rt)
    {
        SYLAR_LOG_ERROR(g_logger) << "pthread_create thread fail, rt=" << rt
                                  << " name=" << name;
        throw std::logic_error("pthread_create error");
    }

    // 通过信号量机制，使得构造函数在 cb 真正执行前（此时 run 已经是执行中的状态）才返回
    m_semaphore.wait();
}
    
Thread::~Thread()
{
    if(m_thread)
    {
        pthread_detach(m_thread);
    }
}

void Thread::join()
{
    if(m_thread)
    {
        int rt = pthread_join(m_thread, nullptr);
        if(rt)
        {
            SYLAR_LOG_ERROR(g_logger) << "pthread_join thread fail, rt=" << rt
                                      << " name=" << m_name;
            throw std::logic_error("pthread_join error");
        }
        m_thread = 0;
    }
}

Thread* Thread::GetThis()
{
    return t_thread;
}
    
const std::string& Thread::GetName()
{
    return t_thread_name;
}
    
void Thread::SetName(const std::string& name)
{
    if(name.empty())
    {
        return;
    }

    if(t_thread)
    {
        t_thread->m_name = name;
    }

    t_thread_name = name;
}

void* Thread::run(void* arg)
{
    Thread* thread = (Thread*)arg;
    t_thread = thread;
    t_thread_name = thread->m_name;
    thread->m_id = sylar::GetThreadId();
    pthread_setname_np(pthread_self(), thread->m_name.substr(0, 15).c_str());

    std::function<void()> cb;
    cb.swap(thread->m_cb);

    thread->m_semaphore.notify();

    cb();

    return 0;
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
