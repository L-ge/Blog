# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 基于 epoll_wait 超时实现定时器功能，精度毫秒级，支持在指定超时时间结束之后执行回调函数。
- sylar 使用时间堆的方式管理定时器。
   - 设计定时器的另一种实现思路是直接将超时时间当作 tick 周期，具体操作是每次都取出所有定时器中超时时间最小的超时值作为一个 tick，这样，一旦 tick 触发，超时时间最小的定时器必然到期。处理完已超时的定时器后，再从剩余的定时器中找出超时时间最小的一个，并将这个最小时间作为下一个 tick，如此反复，就可以实现较为精确的定时。
    - 最小堆很适合处理这种定时方案，将所有定时器按最小堆来组织，可以很方便地获取到当前的最小超时时间，sylar 采取的即是这种方案。
    - sylar 实现最小堆的方法是利用了自定义比较函数的 std::set。
- 所有定时器根据绝对的超时时间点进行排序，每次取出离当前时间最近的一个超时时间点（getNextTimer 函数），计算出超时需要等待的时间，然后等待超时。超时时间到后，获取当前的绝对时间点，然后把最小堆里超时时间点小于这个时间点的定时器都收集起来（listExpiredCb 函数），执行它们的回调函数。


# Timer

- 定时器类。
- 内含绝对时间点、是否循环执行的标识、回调函数等。
- 内含定时器比较仿函数 Comparator，比较的依据是两个定时器的绝对时间。


# TimerManager

- 定时器管理器。
- 内含存放定时器的 std::set。


# 其他说明

- 在注册定时事件时，提供的是相对时间。sylar 会根据传入的相对时间和当前的绝对时间计算出定时器超时时的绝对时间点（在 Timer 的构造函数里面计算出绝对时间），然后根据这个绝对时间点对定时器进行最小堆排序。因为依赖的是系统绝对时间，所以需要考虑校时因素。
- sylar 支持创建条件定时器，也就是在创建定时器时绑定一个变量，在定时器触发时判断一下该变量是否仍然有效（通过 std::weak_ptr 实现），如果变量无效，那就取消触发。
- 关于 onTimerInsertedAtFront() 方法的作用。这个方法是 IOManager 提供给 TimerManager 使用的，当 TimerManager 检测到新添加的定时器的超时时间比当前最小的定时器还要小时，TimerManager 通过这个方法来通知 IOManager 立刻更新当前的 epoll_wait 超时，否则新添加的定时器的执行时间将不准确。实际实现时，只需要在 onTimerInsertedAtFront() 方法内执行一次 tickle 就行了，tickle 之后，epoll_wait 会立即退出，并重新从 TimerManager 中获取最近的超时时间，这时拿到的超时时间就是新添加的最小定时器的超时时间了。
- sylar 的定时器以 gettimeofday() 来获取绝对时间点并判断超时，所以依赖于系统时间，如果系统进行了校时，比如 NTP 时间同步，那这套定时机制就失效了。sylar 的解决办法是设置一个较小的超时步长，比如 3 秒钟，也就是 epoll_wait 最多 3 秒超时，假设最近一个定时器的超时时间是 10 秒以后，那 epoll_wait 需要超时 3 次才会触发。每次超时之后除了要检查有没有要触发的定时器，还顺便检查一下系统时间有没有被往回调。如果系统时间往回调了1个小时以上，那就触发全部定时器。


# 部分相关代码
```C++
/**
 * @filename    timer.h
 * @brief   定时器模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-02
 */
#ifndef __SYLAR_TIMER_H__
#define __SYLAR_TIMER_H__

#include <memory>
#include <vector>
#include <set>

#include "thread.h"

namespace sylar
{

class TimerManager;
class Timer : public std::enable_shared_from_this<Timer>
{
friend class TimerManager;
public:
    typedef std::shared_ptr<Timer> ptr;

    bool cancel();
    bool refresh();
    bool reset(uint64_t ms, bool from_now);

private:
    /**
     * @brief   通过相对时间点构造定时器
     *
     * @param   ms  定时器的执行时间间隔（相对时间点）
     * @param   cb  回调函数
     * @param   recurring   是否循环执行
     * @param   manager 定时器管理器
     */
    Timer(uint64_t ms, std::function<void()> cb, bool recurring, TimerManager* manager);

    /**
     * @brief   通过绝对时间点构造定时器
     *
     * @param   next    执行的时间戳（绝对时间点）
     */
    Timer(uint64_t next);

private:
    /// 是否循环定时器
    bool m_recurring = false;
    /// 执行周期
    uint64_t m_ms = 0;
    /// 精确的执行时间
    uint64_t m_next = 0;
    /// 回调函数
    std::function<void()> m_cb;
    /// 定时器管理器
    TimerManager* m_manager = nullptr;

private:
    /**
     * @brief   定时器比较仿函数
     */
    struct Comparator
    {
        bool operator()(const Timer::ptr& lhs, const Timer::ptr& rhs) const;
    };
};

class TimerManager
{
friend class Timer;
public:
    typedef RWMutex RWMutexType;

    TimerManager();
    virtual ~TimerManager();

    /**
     * @brief  添加定时器 
     *
     * @param   ms  定时器执行的时间间隔
     * @param   cb  定时器回调函数
     * @param   recurring   是否是循环执行的定时器
     */
    Timer::ptr addTimer(uint64_t ms, std::function<void()> cb, bool recurring = false);

    /**
     * @brief   添加条件定时器
     *
     * @param   ms  定时器执行的时间间隔
     * @param   cb  定时器回调函数
     * @param   weak_cond   条件
     * @param   recurring   是否是循环执行的定时器
     */
    Timer::ptr addConditionTimer(uint64_t ms, std::function<void()> cb, 
            std::weak_ptr<void> weak_cond, bool recurring = false);

    /**
     * @brief   获取最近一个定时器执行的时间间隔
     */
    uint64_t getNextTimer();

    /**
     * @brief   获取需要执行的定时器的回调函数的集合
     */
    void listExpiredCb(std::vector<std::function<void()> >& cbs);

    /**
     * @brief   是否有定时器
     */
    bool hasTimer();

protected:
    /**
     * @brief  当有新的定时器插入到定时器的最前面时，执行该函数 
     */
    virtual void onTimerInsertedAtFront() = 0;
    
    /**
     * @brief   将定时器添加到定时器管理器中
     */
    void addTimer(Timer::ptr val, RWMutexType::WriteLock& lock);

private:
    /**
     * @brief   检测服务器时间是否被调后了
     */
    bool detectClockRollover(uint64_t now_ms);

private:
    RWMutexType m_mutex;
    /// 定时器集合（自定义比较函数）
    std::set<Timer::ptr, Timer::Comparator> m_timers;
    /// 是否触发 onTimerInsertedAtFront
    bool m_tickled = false;
    /// 上次执行时间
    uint64_t m_previousTime = 0;
};

}

#endif


#include "timer.h"
#include "util.h"

namespace sylar
{

Timer::Timer(uint64_t ms, std::function<void()> cb, bool recurring, TimerManager* manager)
    : m_recurring(recurring)
    , m_ms(ms)
    , m_cb(cb)
    , m_manager(manager)
{
    m_next = sylar::GetCurrentMS() + m_ms;      // 计算绝对时间点
}

Timer::Timer(uint64_t next)
    : m_next(next)          // 参数直接就是绝对时间点
{

}

bool Timer::cancel()
{
    TimerManager::RWMutexType::WriteLock lk(m_manager->m_mutex);
    if(m_cb)    // m_cb 为空，则表示该定时器已经超时了，执行过了
    {
        m_cb = nullptr;
        auto it = m_manager->m_timers.find(shared_from_this());
        m_manager->m_timers.erase(it);
        return true;
    }
    return false;
}

bool Timer::refresh()
{
    TimerManager::RWMutexType::WriteLock lk(m_manager->m_mutex);
    if(!m_cb)  // m_cb 为空，则表示该定时器已经超时了，执行过了
    {
        return false;
    }
    auto it = m_manager->m_timers.find(shared_from_this());
    if(it == m_manager->m_timers.end())
    {
        return false;
    }
    m_manager->m_timers.erase(it);
    m_next = sylar::GetCurrentMS() + m_ms;
    m_manager->m_timers.insert(shared_from_this());
    return true;
}

bool Timer::reset(uint64_t ms, bool from_now)
{
    if(ms == m_ms && !from_now)
    {
        return true;
    }

    TimerManager::RWMutexType::WriteLock lk(m_manager->m_mutex);
    if(!m_cb)   // m_cb 为空，则表示该定时器已经超时了，执行过了
    {
        return false;
    }
    auto it = m_manager->m_timers.find(shared_from_this());
    if(it == m_manager->m_timers.end())
    {
        return false;
    }
    m_manager->m_timers.erase(it);
    uint64_t start = 0;
    if(from_now)
    {
        start = sylar::GetCurrentMS();
    }
    else
    {
        start = m_next - m_ms;
    }
    m_ms = ms;
    m_next = start + m_ms;
    m_manager->addTimer(shared_from_this(), lk);
    return true;
}

bool Timer::Comparator::operator()(const Timer::ptr& lhs, const Timer::ptr& rhs) const
{
    if(!lhs && !rhs)                // 都是空，认为不小于
    {
        return false;
    }
    if(!lhs)                        // 自己是空，认为小于
    {
        return true;
    }
    if(!rhs)                        // 对方是空，认为不小于
    {
        return false;
    }
    if(lhs->m_next < rhs->m_next)   // 自己的绝对时间小，认为小于
    {
        return true;
    }
    if(rhs->m_next < lhs->m_next)   // 对方的绝对时间小，认为不小于
    {
        return false;
    }
    return lhs.get() < rhs.get();   // 其他情况，比较裸指针的大小
}

TimerManager::TimerManager()
{
    m_previousTime = sylar::GetCurrentMS();
}

TimerManager::~TimerManager()
{

}

Timer::ptr TimerManager::addTimer(uint64_t ms, std::function<void()> cb, bool recurring)
{
    Timer::ptr timer(new Timer(ms, cb, recurring, this));
    RWMutexType::WriteLock lk(m_mutex);
    addTimer(timer, lk);
    return timer;
}

static void OnTimer(std::weak_ptr<void> weak_cond, std::function<void()> cb)
{
    std::shared_ptr<void> tmp = weak_cond.lock();
    if(tmp)
    {
        cb();
    }
}

Timer::ptr TimerManager::addConditionTimer(uint64_t ms, std::function<void()> cb, 
                                           std::weak_ptr<void> weak_cond, bool recurring)
{
    // 通过std::bind，包装出一个新的回调函数
    return addTimer(ms, std::bind(&OnTimer, weak_cond, cb), recurring);
}

uint64_t TimerManager::getNextTimer()
{
    RWMutexType::ReadLock lk(m_mutex);
    m_tickled = false;
    if(m_timers.empty())
    {
        return ~0ull;
    }
    
    const Timer::ptr& next = *m_timers.begin();
    uint64_t now_ms = sylar::GetCurrentMS();
    if(now_ms >= next->m_next)  // 已经超时了，要马上执行
    {
        return 0;
    }
    else
    {
        return next->m_next - now_ms;
    }
}

void TimerManager::listExpiredCb(std::vector<std::function<void()> >& cbs)
{
    uint64_t now_ms = sylar::GetCurrentMS();
    std::vector<Timer::ptr> expired;
    {
        RWMutexType::ReadLock lk(m_mutex);
        if(m_timers.empty())
        {
            return;
        }
    }

    RWMutexType::WriteLock lk(m_mutex);
    if(m_timers.empty())
    {
        return;
    }

    bool rollover = detectClockRollover(now_ms);
    if(!rollover && ((*m_timers.begin())->m_next > now_ms)) // 没有时间倒退且第一个定时器还没超时
    {
        return;
    }

    Timer::ptr now_timer(new Timer(now_ms));
    // it 为第一个不小于 nowtimer 的定时器的迭代器
    auto it = rollover ? m_timers.end() : m_timers.lower_bound(now_timer);
    while(it != m_timers.end() && (*it)->m_next == now_ms)  // 处理多个定时器时间相等的情况
    {
        ++it;
    }
    expired.insert(expired.begin(), m_timers.begin(), it);
    m_timers.erase(m_timers.begin(), it);
    cbs.reserve(expired.size());

    for(auto& timer : expired)
    {
        cbs.push_back(timer->m_cb);
        if(timer->m_recurring)
        {
            timer->m_next = now_ms + timer->m_ms;
            m_timers.insert(timer);
        }
        else
        {
            timer->m_cb = nullptr;
        }
    }
}

bool TimerManager::hasTimer()
{
    RWMutexType::ReadLock lk(m_mutex);
    return !m_timers.empty();
}

void TimerManager::addTimer(Timer::ptr val, RWMutexType::WriteLock& lock)
{
    auto it = m_timers.insert(val).first;
    bool at_front = (it == m_timers.begin()) && !m_tickled;
    if(at_front)
    {
        m_tickled = true;
    }
    lock.unlock();

    if(at_front)
    {
        onTimerInsertedAtFront();
    }
}

bool TimerManager::detectClockRollover(uint64_t now_ms)
{
    bool rollover = false;
    if(now_ms < m_previousTime
            && now_ms < (m_previousTime - 60*60*1000)) // 小一个小时
    {
        rollover = true;
    }
    m_previousTime = now_ms;
    return rollover;
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
