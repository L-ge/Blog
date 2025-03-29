# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 仿照 Log4pp 写的日志模块。
- 提供宏定义给外界使用。


# LogLevel

- 日志级别封装类。
- 提供“从日志级别枚举值转换到字符串”、“从字符串转换相应的日志级别枚举值”等方法。


# LogEvent

- 日志事件类。
- 封装日志事件的属性，例如时间、线程id、日志等级、内容等等，并对外提供访问方法。
- 日志事件的构造在使用上会通过宏定义来简化。


# LogEventWrap

- 日志事件包装类，内含日志事件 LogEvent。
- 该类的作用主要是便于编写宏，方便外界使用。
- 日志事件在析构时由日志器进行输出。


# LogFormatter

- 日志格式类。
- 通过传递日志样式字符串给该类，该类对传入的字符串进行解析，例如 %d%t%p%m%n 表示 时间、线程号、日志等级、内容、换行。
- 内含一个虚基类-日志内容格式化项 FormatItem，像时间、线程号、日志等级、内容、换行等这些日志格式项都有其对应的子类，其子类正是继承 FormatItem。有13个子类，
	- 消息-MessageFormatItem、
	- 日志级别-LevelFormatItem、
	- 累计毫秒数-ElapseFormatItem、
	- 日志名称-NameFormatItem、
	- 线程id-ThreadIdFormatItem、
	- 换行-NewLineFormatItem、
	- 时间-DateTimeFormatItem、
	- 文件名-FilenameFormatItem、
	- 行号-LineFormatItem、
	- Tab-TabFormatItem、
	- 协程id-FiberIdFormatItem、
	- 线程名称-ThreadNameFormatItem、
	- 直接打印字符串-StringFormatItem。
- 内含一个 vector，存放解析后的格式。vector 存放的是基类 FormatItem 的智能指针。
- 整个日志模块最复杂的逻辑就是该类解析日志样式的函数 init()。
- LogFormatter 提供 format 方法给外界访问，format 方法有两个形式（重载），传入参数 Logger、LogLevel、LogEvent，传出参数是格式化后的字符串。通过遍历存放在 vector 里面的日志样式，对日志内容进行格式化。


# LogAppender

- 日志输出目的地类。
- LogAppender 为虚基类，有纯虚函数，留给子类去各自实现。实现的子类如 StdoutLogAppender 和 FileLogAppender。
- Appender 自带一个默认的 LogFormatter，以默认方式输出。

1. StdoutLogAppender
- 标准化输出类。
- 日志输出到控制台。

2. FileLogAppender
- 文件输出类。
- 日志输出到相应的文件中。


# Logger

- 日志器类。
- 设置日志名称、设置日志等级 LogLevel、设置日志输出位置 LogAppender、设置日志格式、根据日志级别控制日志输出等。


# LoggerManager

- 日志管理器类。
- 利用 map 存放各个 Logger 实例，其中 key 是日志器的名称，value 是日志器的智能指针。
- 还内含一个主日志器 root。


# 其他说明

- 每个类都 typedef std::shared_ptr<T> ptr，方便外界使用其智能指针。
- 普遍使用 Spinlock 实现互斥，保证线程安全。Spinlock 比 普通的 Mutex 效率高，但耗CPU。


# 数据流转
```C++
// 以使用流式方式写日志举例：
通过宏SYLAR_LOG_NAME(name)从LoggerMgr取得对应的Logger，假设是g_logger
				|
				|
			   \|/
通过宏SYLAR_LOG_DEBUG(logger)->SYLAR_LOG_LEVEL(logger, level)
				|
				|
			   \|/
宏SYLAR_LOG_LEVEL new出一个LogEvent，然后再在该实例传给LogEventWrap，从而创建出LogEventWrap临时对象，从该临时对象拿到std::stringstream
				|
				|
			   \|/
通过上面拿到的std::stringstream，用<<操作符即可把日志内容存入std::stringstream
				|
				|
			   \|/
LogEventWrap临时对象析构时，将通过构造时传入的LogEvent拿到其中的Logger，在调用Logger的log方法
				|
				|
			   \|/
Logger的log方法是遍历自己的LogAppender，调用每个LogAppender的log方法（传入logger，level，event参数），假设这里是FileLogAppender
				|
				|
			   \|/
LogAppender的log方法是加上自己的std::ostream参数（如果是输出到控制台，则是std::cout）后，调用LogFormatter的format方法（传入std::ostream，logger，level，event参数）。
				|
				|
			   \|/
LogFormatter的format方法是遍历自己缓存的所有FormatItem(继承了FormatItem的各种子类智能指针)，将日志内容格式化（例如加上时间日期、线程id等）。调用的是每个FormatItem的format方法（传入std::ostream，logger，level，event参数）
				|
				|
			   \|/
每个FormatItem的format方法是格式化后的内容以流式方式存入std::ostream。如果是输出到控制台，那么这里就直接输出了。如果是文件，因为std::ostream关联了文件，因此会对文件进行缓存写（非实时写）。
```


# 待完善

- 支持日志文件按大小或日期分文件存储。


# 部分相关代码
```C++
/**
 * @filename    log.h
 * @brief   日志模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-11
 */
#ifndef __SYLAR_LOG_H__
#define __SYLAR_LOG_H__

#include <string>
#include <memory>
#include <sstream>
#include <fstream>
#include <list>
#include <vector>
#include <map>
#include <stdarg.h>
#include <iostream>

#include "mutex.h"
#include "util.h"
#include "singleton.h"
#include "thread.h"

/**
 * @brief   使用流式方式写日志
 */
#define SYLAR_LOG_LEVEL(logger, level) \
    if(logger->getLevel() <= level) \
        sylar::LogEventWrap(sylar::LogEvent::ptr(new sylar::LogEvent(logger, level, \
                        __FILE__, __LINE__, 0, sylar::GetThreadId(),\
                sylar::GetFiberId(), time(0), sylar::Thread::GetName()))).getSS()

#define SYLAR_LOG_DEBUG(logger) SYLAR_LOG_LEVEL(logger, sylar::LogLevel::DEBUG)
#define SYLAR_LOG_INFO(logger) SYLAR_LOG_LEVEL(logger, sylar::LogLevel::INFO)
#define SYLAR_LOG_WARN(logger) SYLAR_LOG_LEVEL(logger, sylar::LogLevel::WARN)
#define SYLAR_LOG_ERROR(logger) SYLAR_LOG_LEVEL(logger, sylar::LogLevel::ERROR)
#define SYLAR_LOG_FATAL(logger) SYLAR_LOG_LEVEL(logger, sylar::LogLevel::FATAL)

/**
 * @brief   使用格式化方式写日志
 */
#define SYLAR_LOG_FMT_LEVEL(logger, level, fmt, ...) \
    if(logger->getLevel() <= level) \
        sylar::LogEventWrap(sylar::LogEvent::ptr(new sylar::LogEvent(logger, level, \
                        __FILE__, __LINE__, 0, sylar::GetThreadId(),\
                sylar::GetFiberId(), time(0), sylar::Thread::GetName()))).getEvent()->format(fmt, __VA_ARGS__)

#define SYLAR_LOG_FMT_DEBUG(logger, fmt, ...) SYLAR_LOG_FMT_LEVEL(logger, sylar::LogLevel::DEBUG, fmt, __VA_ARGS__)
#define SYLAR_LOG_FMT_INFO(logger, fmt, ...)  SYLAR_LOG_FMT_LEVEL(logger, sylar::LogLevel::INFO, fmt, __VA_ARGS__)
#define SYLAR_LOG_FMT_WARN(logger, fmt, ...)  SYLAR_LOG_FMT_LEVEL(logger, sylar::LogLevel::WARN, fmt, __VA_ARGS__)
#define SYLAR_LOG_FMT_ERROR(logger, fmt, ...) SYLAR_LOG_FMT_LEVEL(logger, sylar::LogLevel::ERROR, fmt, __VA_ARGS__)
#define SYLAR_LOG_FMT_FATAL(logger, fmt, ...) SYLAR_LOG_FMT_LEVEL(logger, sylar::LogLevel::FATAL, fmt, __VA_ARGS__)


/**
 * @brief   获取主日志器
 */
#define SYLAR_LOG_ROOT() sylar::LoggerMgr::GetInstance()->getRoot()


/**
 * @brief   获取name的日志器
 */
#define SYLAR_LOG_NAME(name) sylar::LoggerMgr::GetInstance()->getLogger(name)

namespace sylar
{
class Logger;

/**
 * @brief   日志级别封装类
 */
class LogLevel
{
public:
    enum Level
    {
        UNKNOW = 0,
        DEBUG = 1,
        INFO = 2,
        WARN = 3,
        ERROR = 4,
        FATAL = 5,
    };
    
    static const char* ToString(LogLevel::Level level);

    static LogLevel::Level FromString(const std::string& str);
};

/**
 * @brief  日志事件类 
 */
class LogEvent
{
public:
    typedef std::shared_ptr<LogEvent> ptr;

    LogEvent(std::shared_ptr<Logger> logger, 
             LogLevel::Level level,
             const char* file,
             int32_t line,
             uint32_t elapse,
             uint32_t threadId,
             uint32_t fiberId,
             uint64_t time,
             const std::string& threadName);

    const char* getFile() const { return m_file; }

    int32_t getLine() const { return m_line; } 

    uint32_t getElapse() const { return m_elapse; }

    uint32_t getThreadId() const { return m_threadId; }

    uint32_t getFiberId() const { return m_fiberId; }

    uint64_t getTime() const { return m_time; }

    const std::string& getThreadName() const { return m_threadName; }

    std::string getContent() const { return m_ss.str(); }

    std::shared_ptr<Logger> getLogger() const { return m_logger; }

    LogLevel::Level getLevel() const { return m_level; }

    std::stringstream& getSS() { return m_ss; }

    void format(const char* fmt, ...);

    void format(const char* fmt, va_list al);

private:
    /// 日志器
    std::shared_ptr<Logger> m_logger;
    /// 日志等级
    LogLevel::Level m_level;
    /// 文件名
    const char* m_file = nullptr;
    /// 行号
    int32_t m_line = 0;
    /// 程序运行的毫秒数
    uint32_t m_elapse = 0;
    /// 线程ID
    uint32_t m_threadId = 0;
    /// 协程ID
    uint32_t m_fiberId = 0;
    /// 时间戳
    uint64_t m_time = 0;
    /// 线程名称
    std::string m_threadName;
    /// 日志内容流
    std::stringstream m_ss;
};

/**
 * @brief   日志事件包装类
 */
class LogEventWrap
{
public:
    LogEventWrap(LogEvent::ptr e);

    ~LogEventWrap();

    LogEvent::ptr getEvent() const { return m_event; }

    std::stringstream& getSS();

private:
    LogEvent::ptr m_event;
};

/**
 * @brief   日志格式类
 */
class LogFormatter
{
public:
    typedef std::shared_ptr<LogFormatter> ptr;

    LogFormatter(const std::string& pattern);

    std::string format(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event);

std::ostream& format(std::ostream& ofs, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event);

public:
    /**
    * @brief   日志内容格式化项
    */
    class FormatItem
    {
    public:
        typedef std::shared_ptr<FormatItem> ptr;

        virtual ~FormatItem() {}

        virtual void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event) = 0;
    };

    void init();

    bool isError() const { return m_error; }

    const std::string getPattern() const { return m_pattern; }

private:
    std::string m_pattern;
    std::vector<FormatItem::ptr> m_items;
    bool m_error;
};

/**
 * @brief   日志输出目的地类
 */
class LogAppender
{
    friend class Logger;
    
public:
    typedef std::shared_ptr<LogAppender> ptr;
    typedef Spinlock MutexType;

    virtual ~LogAppender() {}

    virtual void log(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event) = 0;
    virtual std::string toYamlString() = 0;

    void setFormatter(LogFormatter::ptr val);
    LogFormatter::ptr getFormatter();
    LogLevel::Level getLevel() const { return m_level; }
    void setLevel(LogLevel::Level val) { m_level = val; }

protected:
    LogLevel::Level m_level = LogLevel::DEBUG;
    /// 是否有自己的日志格式器
    bool m_hasFormatter = false;
    MutexType m_mutex;
    LogFormatter::ptr m_formatter;
};

/**
 * @brief   标准化输出类
 */
class StdoutLogAppender : public LogAppender
{
public:
    typedef std::shared_ptr<StdoutLogAppender> ptr;

    void log(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event) override;
    std::string toYamlString() override;
};

/**
 * @brief   文件输出类
 */
class FileLogAppender : public LogAppender
{
public:
    typedef std::shared_ptr<FileLogAppender> ptr;

    FileLogAppender(const std::string& filename);

    void log(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event) override;
    std::string toYamlString() override;

    bool reopen();

private:
    std::string m_filename;
    std::ofstream m_filestream;
    /// 上次重新打开的时间
    uint64_t m_lastTime;
};

/**
 * @brief   日志器类
 */
class Logger : public std::enable_shared_from_this<Logger>
{
    friend class LoggerManager;
public:
    typedef std::shared_ptr<Logger> ptr;
    typedef Spinlock MutexType;

    Logger(const std::string& name = "root"); 
    
    void log(LogLevel::Level level, LogEvent::ptr event); 
    void debug(LogEvent::ptr event); 
    void info(LogEvent::ptr event); 
    void warn(LogEvent::ptr event); 
    void error(LogEvent::ptr event);
    void fatal(LogEvent::ptr event);
    
    void addAppender(LogAppender::ptr appender);
    void delAppender(LogAppender::ptr appender);
    void clearAppenders();

    void setLevel(LogLevel::Level val) { m_level = val; }
    LogLevel::Level getLevel() const { return m_level; }
    void setFormatter(LogFormatter::ptr val);
    void setFormatter(const std::string& val);
    LogFormatter::ptr getFormatter();
    const std::string& getName() const { return m_name; }
    
    std::string toYamlString();

private:
    std::string m_name;
    LogLevel::Level m_level;
    MutexType m_mutex;
    std::list<LogAppender::ptr> m_appenders;
    LogFormatter::ptr m_formatter;
    Logger::ptr m_root;
};

/**
 * @brief   日志管理器类
 */
class LoggerManager
{
public:
    typedef Spinlock MutexType;

    LoggerManager();

    void init();

    Logger::ptr getLogger(const std::string& name);
    Logger::ptr getRoot() const { return m_root; }

    std::string toYamlString();

private:
    MutexType m_mutex;
    std::map<std::string, Logger::ptr> m_loggers;
    Logger::ptr m_root;
};

typedef sylar::Singleton<LoggerManager> LoggerMgr;

};

#endif


#include "log.h"
#include "config.h"

namespace sylar
{

const char* LogLevel::ToString(LogLevel::Level level)
{
    switch(level)
    {
#define XX(name) \
        case LogLevel::name: \
            return #name; \
            break;

        XX(DEBUG);
        XX(INFO);
        XX(WARN);
        XX(ERROR);
        XX(FATAL);
#undef XX
        default:
            return "UNKNOW";
    }
    return "UNKNOW";
}

LogLevel::Level LogLevel::FromString(const std::string& str)
{
#define XX(level, v) \
    if(str == #v) \
    { \
        return LogLevel::level; \
    }

    XX(DEBUG, debug);
    XX(INFO, info);
    XX(WARN, warn);
    XX(ERROR, error);
    XX(FATAL, fatal);

    XX(DEBUG, DEBUG);
    XX(INFO, INFO);
    XX(WARN, WARN);
    XX(ERROR, ERROR);
    XX(FATAL, FATAL);

    return LogLevel::UNKNOW;
#undef XX
}

LogEvent::LogEvent(std::shared_ptr<Logger> logger, 
        LogLevel::Level level,
        const char* file,
        int32_t line,
        uint32_t elapse,
        uint32_t threadId,
        uint32_t fiberId,
        uint64_t time,
        const std::string& threadName)
    : m_logger(logger)
    , m_level(level)
    , m_file(file)
    , m_line(line)
    , m_elapse(elapse)
    , m_threadId(threadId)
    , m_fiberId(fiberId)
    , m_time(time)
    , m_threadName(threadName)
{
}

void LogEvent::format(const char* fmt, ...)
{
    va_list al;
    va_start(al, fmt);
    format(fmt, al);
    va_end(al);
}

void LogEvent::format(const char* fmt, va_list al)
{
    char* buf = nullptr;
    int len = vasprintf(&buf, fmt, al);
    if(len != -1)
    {
        m_ss << std::string(buf, len);
        free(buf);
    }
}

LogEventWrap::LogEventWrap(LogEvent::ptr e)
    : m_event(e)
{

}

LogEventWrap::~LogEventWrap()
{
    m_event->getLogger()->log(m_event->getLevel(), m_event);
}

std::stringstream& LogEventWrap::getSS()
{
    return m_event->getSS();
}

/**
 * @brief   m:消息
 */
class  MessageFormatItem : public LogFormatter::FormatItem
{
public:
    MessageFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getContent();
    }
};

/**
 * @brief   p:日志级别
 */
class LevelFormatItem : public LogFormatter::FormatItem
{
public:
    LevelFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << LogLevel::ToString(level);
    }
};

/**
 * @brief   r:累计毫秒数
 */
class ElapseFormatItem : public LogFormatter::FormatItem
{
public:
    ElapseFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getElapse();
    }
};

/**
 * @brief   c:日志名称
 */
class NameFormatItem : public LogFormatter::FormatItem
{
public:
    NameFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getLogger()->getName();
    }
};

/**
 * @brief   t:线程id
 */
class ThreadIdFormatItem : public LogFormatter::FormatItem
{
public:
    ThreadIdFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getThreadId();
    }
};

/**
 * @brief   n:换行
 */
class NewLineFormatItem : public LogFormatter::FormatItem
{
public:
    NewLineFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << std::endl;
    }
};

/**
 * @brief   d:时间
 */
class DateTimeFormatItem : public LogFormatter::FormatItem
{
public:
    DateTimeFormatItem(const std::string& format = "%Y-%m-%d %H:%M:%S")
        : m_format(format)
    {
        if(m_format.empty())
        {
            m_format = "%Y-%m-%d %H:%M:%S";
        }
    }

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        struct tm tm;
        time_t time = event->getTime();
        localtime_r(&time, &tm);
        char buf[64];
        strftime(buf, sizeof(buf), m_format.c_str(), &tm);
        os << buf;
    }

private:
    std::string m_format;
};

/**
 * @brief   f:文件名
 */
class FilenameFormatItem : public LogFormatter::FormatItem
{
public:
    FilenameFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getFile();
    }
};

/**
 * @brief   l:行号
 */
class LineFormatItem : public LogFormatter::FormatItem
{
public:
    LineFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getLine();
    }
};

/**
 * @brief   T:Tab
 */
class TabFormatItem : public LogFormatter::FormatItem
{
public:
    TabFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << "\t";
    }
};

/**
 * @brief   F:协程id
 */
class FiberIdFormatItem : public LogFormatter::FormatItem
{
public:
    FiberIdFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getFiberId();
    }
};

/**
 * @brief   N:线程名称
 */
class ThreadNameFormatItem : public LogFormatter::FormatItem
{
public:
    ThreadNameFormatItem(const std::string& str = "") {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << event->getThreadName();
    }
};

/**
 * @brief   直接打印字符串
 */
class StringFormatItem : public LogFormatter::FormatItem
{
public:
    StringFormatItem(const std::string& str)
        : m_string(str)
    {}

    void format(std::ostream& os, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
    {
        os << m_string;
    }

private:
    std::string m_string;
};

LogFormatter::LogFormatter(const std::string& pattern)
    : m_pattern(pattern)
{
    init();
}

std::string LogFormatter::format(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
{
    std::stringstream ss;
    for(auto& i : m_items)
    {
        i->format(ss, logger, level, event);
    }
    return ss.str();
}

std::ostream& LogFormatter::format(std::ostream& ofs, std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
{
    for(auto& i : m_items)
    {
        i->format(ofs, logger, level, event);
    }
    return ofs;
}

/**
 * @brief   该日志器逻辑最复杂的函数：对日志样式进行解析。有如下四种格式：
 *          %xxx        是对xxx进行解析
 *          %xxx{yyy}   是对xxx进行解析，它是yyy这种格式的
 *          %%          是对%进行转义
 *          还有一种是直接要打印的字符串，比如[]这种。
 */
void LogFormatter::init()  //%xxx %xxx{xxx} %%
{
    //str, format, type
    std::vector<std::tuple<std::string, std::string, int> > vec;    // 存放解析后的样式
    std::string nstr;                                               // 存放实实在在的、并不是样式的字符串
    for(size_t i = 0; i < m_pattern.size(); ++i)    // 循环解析每一个字符
    {
        if(m_pattern[i] != '%')         // 当前字符不是%，就把它缓存到nstr
        {
            nstr.append(1, m_pattern[i]);
            continue;
        }
        // 下面是等于%的情况


        // 第i个和第i+1个都是%，那么就是转义，压一个%进nstr 
        if((i + 1) < m_pattern.size()) 
        {
            if(m_pattern[i + 1] == '%') 
            {
                nstr.append(1, '%');
                continue;
            }
        }

        size_t n = i + 1;               // 第i个是%，那么从i+1个字符开始解析
        int fmt_status = 0;             // 解析方式：0是不需要解析格式，1是解析格式
        size_t fmt_begin = 0;

        std::string str;                // %xxx{yyy}中的xxx
        std::string fmt;                // %xxx{yyy}中的yyy
        while(n < m_pattern.size())     // 保证不越界
        {
            if(!fmt_status
                    && (!isalpha(m_pattern[n])
                        && m_pattern[n] != '{'
                        && m_pattern[n] != '}'))    // 表示解析的就是%xxx格式，于是结束这一段样式的解析
            {
                str = m_pattern.substr(i + 1, n - i - 1);
                break;
            }

            if(fmt_status == 0) 
            {
                if(m_pattern[n] == '{') // 需要解析格式
                {
                    str = m_pattern.substr(i + 1, n - i - 1);  // str即为xxx{yyy}中xxx
                    fmt_status = 1;     // 解析格式
                    fmt_begin = n;      // 记录需要解析格式的起始位置，即xxx{yyy}中的{的位置
                    ++n;
                    continue;
                }
            }
            else if(fmt_status == 1)    // 需要解析格式
            {
                if(m_pattern[n] == '}') // 结束yyy的查找了
                {
                    fmt = m_pattern.substr(fmt_begin + 1, n - fmt_begin - 1);   // fmt即为yyy
                    fmt_status = 0;     // 重置状态为不需要解析格式
                    ++n;
                    break;              // 结束这一段样式的解析了
                }
            }
            ++n;
            if(n == m_pattern.size())   // 已经到样式结尾了
            {
                if(str.empty())         // 如果样式为空的话，则样式即为第i+1个字符一直到结束，也就是只有一个样式或已经到了最后一个样式才有这种情况 
                {
                    str = m_pattern.substr(i + 1);
                }
            }
        }

        if(fmt_status == 0)
        {
            if(!nstr.empty())   // 如果nstr不为空，则证明是nstr是要打印的字符串，所以以string()的样式存到vector里面去
            {
                vec.push_back(std::make_tuple(nstr, std::string(), 0));
                nstr.clear();
            }
            vec.push_back(std::make_tuple(str, fmt, 1));
            i = n - 1;          // n-1再进入下一轮循环
        }
        else if(fmt_status == 1) // 因为上面会重置，因此此时还是1，证明解析样式有错误
        {
            std::cout << "pattern parse error: " << m_pattern << " - " << m_pattern.substr(i) << std::endl;
            m_error = true;
            vec.push_back(std::make_tuple("<<pattern_error>>", fmt, 0));
        }
    } // 这里已经是解析完成

    if(!nstr.empty())   // 解析完成后，nstr有值，证明其是要打印的字符串，则以string的形式存到vector里面去
    {
        vec.push_back(std::make_tuple(nstr, "", 0));
    }
    static std::map<std::string, std::function<FormatItem::ptr(const std::string& str)> > s_format_items = {
#define XX(str, C) \
        {#str, [](const std::string& fmt) { return FormatItem::ptr(new C(fmt));}}

        XX(m, MessageFormatItem),           // m:消息 
        XX(p, LevelFormatItem),             // p:日志级别
        XX(r, ElapseFormatItem),            // r:累计毫秒数
        XX(c, NameFormatItem),              // c:日志名称
        XX(t, ThreadIdFormatItem),          // t:线程id
        XX(n, NewLineFormatItem),           // n:换行
        XX(d, DateTimeFormatItem),          // d:时间
        XX(f, FilenameFormatItem),          // f:文件名
        XX(l, LineFormatItem),              // l:行号
        XX(T, TabFormatItem),               // T:Tab
        XX(F, FiberIdFormatItem),           // F:协程id
        XX(N, ThreadNameFormatItem),        // N:线程名称
#undef XX
    };

    for(auto& i : vec) 
    {
        if(std::get<2>(i) == 0)         // 也就是type为0，证明要打印的是字符串，并不是样式，因此解析为StringFormatItem，直接打印即可。
        {
            m_items.push_back(FormatItem::ptr(new StringFormatItem(std::get<0>(i))));
        } 
        else 
        {
            auto it = s_format_items.find(std::get<0>(i));  // 也就是xxx{yyy}中的xxx，这正式要解析的
            if(it == s_format_items.end()) 
            {
                m_items.push_back(FormatItem::ptr(new StringFormatItem("<<error_format %" + std::get<0>(i) + ">>")));
                m_error = true;
            }
            else // 一般来说，fmt是空的，只有xxx{yyy}这种情况才不是空，fmt即yyy，比如时间，xxx就是d，用来new出DateTimeFormatItem，而yyy可能是hh::mm:ss这种。
            {
                m_items.push_back(it->second(std::get<1>(i)));
            }
        }
    }
}

void LogAppender::setFormatter(LogFormatter::ptr val)
{
    MutexType::Lock lk(m_mutex);
    m_formatter = val;
    if(m_formatter = val)
    {
        m_hasFormatter = true;
    }
    else
    {
        m_hasFormatter = false;
    }
}

LogFormatter::ptr LogAppender::getFormatter()
{
    MutexType::Lock lk(m_mutex);
    return m_formatter;
}

void StdoutLogAppender::log(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
{
    if(level >= m_level)
    {
        MutexType::Lock lk(m_mutex);
        m_formatter->format(std::cout, logger, level, event);
    }
}

std::string StdoutLogAppender::toYamlString()
{
    MutexType::Lock lk(m_mutex);
    YAML::Node node;
    node["type"] = "StdoutLogAppender";
    if(m_level != LogLevel::UNKNOW)
    {
        node["level"] = LogLevel::ToString(m_level);
    }
    if(m_hasFormatter && m_formatter)
    {
        node["formatter"] = m_formatter->getPattern();
    }
    std::stringstream ss;
    ss << node;
    return ss.str();
}

FileLogAppender::FileLogAppender(const std::string& filename)
    : m_filename(filename)
{
    reopen();
}

void FileLogAppender::log(std::shared_ptr<Logger> logger, LogLevel::Level level, LogEvent::ptr event)
{
    if(level >= m_level)
    {
        /// 超过3ms就重新打开一次
        uint64_t now = event->getTime();
        if(now >= (m_lastTime + 3))
        {
            reopen();
            m_lastTime = now;
        }

        MutexType::Lock lk(m_mutex);
        if(!m_formatter->format(m_filestream, logger, level, event))
        {
            std::cout << "error" << std::endl;
        }
    }
}

std::string FileLogAppender::toYamlString()
{
    MutexType::Lock lk(m_mutex);
    YAML::Node node;
    node["type"] = "FileLogAppender";
    node["file"] = m_filename;
    if(m_level != LogLevel::UNKNOW)
    {
        node["level"] = LogLevel::ToString(m_level);
    }
    if(m_hasFormatter && m_formatter)
    {
        node["formatter"] = m_formatter->getPattern();
    }
    std::stringstream ss;
    ss << node;
    return ss.str();
}

bool FileLogAppender::reopen()
{
    MutexType::Lock lk(m_mutex);
    if(m_filestream)
    {
        m_filestream.close();
    }
    return FSUtil::OpenForWrite(m_filestream, m_filename, std::ios::app);
}

Logger::Logger(const std::string& name)
    : m_name(name)
    , m_level(LogLevel::DEBUG) 
{
    //m_formatter.reset(new LogFormatter("%d{%Y-%m-%d %H:%M:%S}%T%t%T%N%T%F%T[%p]%T[%c]%T%f:%l%T%m%n"));
    m_formatter.reset(new LogFormatter("%d{%Y-%m-%d %H:%M:%S}%T%m%n"));
}
    
void Logger::log(LogLevel::Level level, LogEvent::ptr event)
{
    if(level >= m_level)
    {
        auto self = shared_from_this();
        MutexType::Lock lk(m_mutex);
        if(!m_appenders.empty())
        {
            for(auto& i : m_appenders)
            {
                i->log(self, level, event);
            }
        }
        else if(m_root)
        {
            m_root->log(level, event);
        }
    }
}
    
void Logger::debug(LogEvent::ptr event)
{
    log(LogLevel::DEBUG, event);
}

void Logger::info(LogEvent::ptr event)
{
    log(LogLevel::INFO, event);
}
    
void Logger::warn(LogEvent::ptr event)
{
    log(LogLevel::WARN, event);
}

void Logger::error(LogEvent::ptr event)
{
    log(LogLevel::ERROR, event);
}
    
void Logger::fatal(LogEvent::ptr event)
{
    log(LogLevel::FATAL, event);
}
    
void Logger::addAppender(LogAppender::ptr appender)
{
    MutexType::Lock lk(m_mutex);
    if(!appender->getFormatter())
    {
        MutexType::Lock ll(appender->m_mutex);
        appender->m_formatter = m_formatter;
    }
    m_appenders.push_back(appender);
}

void Logger::delAppender(LogAppender::ptr appender)
{
    MutexType::Lock lk(m_mutex);
    for(auto it = m_appenders.begin();
            it != m_appenders.end();
            ++it)
    {
        if(*it == appender)
        {
            m_appenders.erase(it);
            break;
        }
    }
}

void Logger::clearAppenders()
{
    MutexType::Lock lk(m_mutex);
    m_appenders.clear();
}

void Logger::setFormatter(LogFormatter::ptr val)
{
    MutexType::Lock lk(m_mutex);
    m_formatter = val;
    for(auto& i : m_appenders)
    {
        MutexType::Lock ll(i->m_mutex);
        if(!i->m_hasFormatter)
        {
            i->m_formatter = m_formatter;
        }
    }
}

void Logger::setFormatter(const std::string& val)
{
    LogFormatter::ptr new_val(new LogFormatter(val));
    if(new_val->isError())
    {
        std::cout << "Logger setFormatter name=" << m_name
                  << " value=" << val 
                  << " invalid formatter\n";
        return;
    }
    setFormatter(new_val);
}

LogFormatter::ptr Logger::getFormatter()
{
    MutexType::Lock lk(m_mutex);
    return m_formatter;
}
    
std::string Logger::toYamlString()
{
    MutexType::Lock lk(m_mutex);
    YAML::Node node;
    node["name"] = m_name;
    if(m_level != LogLevel::UNKNOW)
    {
        node["level"] = LogLevel::ToString(m_level);
    }
    if(m_formatter)
    {
        node["formatter"] = m_formatter->getPattern();
    }
    for(auto& i : m_appenders)
    {
        node["appenders"].push_back(YAML::Load(i->toYamlString()));
    }
    std::stringstream ss;
    ss << node;
    return ss.str();
}

LoggerManager::LoggerManager()
{
    m_root.reset(new Logger);
    m_root->addAppender(LogAppender::ptr(new StdoutLogAppender));
    m_loggers[m_root->m_name] = m_root;

    init();
}

void LoggerManager::init()
{
}

Logger::ptr LoggerManager::getLogger(const std::string& name)
{
    MutexType::Lock lk(m_mutex);
    auto it = m_loggers.find(name);
    if(it != m_loggers.end())
    {
        return it->second;
    }

    Logger::ptr logger(new Logger(name));
    logger->m_root = m_root;
    m_loggers[name] = logger;
    return logger;
}

std::string LoggerManager::toYamlString()
{
    MutexType::Lock lk(m_mutex);
    YAML::Node node;
    for(auto& i : m_loggers)
    {
        node.push_back(YAML::Load(i.second->toYamlString()));
    }
    std::stringstream ss;
    ss << node;
    return ss.str();
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
