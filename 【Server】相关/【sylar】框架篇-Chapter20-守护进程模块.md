# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 将进程与终端解绑，转到后台运行（通过 fork 出子进程作为主业务进程的方式实现）。
- sylar 实现了双进程唤醒功能，父进程作为守护进程的同时会检测子进程是否退出，如果子进程退出，则会定时重新拉起子进程。


# daemon.h

- ProcessInfo
	- 单例模式，存放所需的进程的一些信息。
- start_daemon
	- 对外提供的接口。


# 其他说明

- 以下是守护进程的实现步骤：
	1. 调用 daemon(1, 0) 将当前进程以守护进程的形式运行；
	2. 守护进程 fork 子进程，在子进程运行主业务；
	3. 父进程通过 waitpid() 检测子进程是否退出，如果子进程退出，则重新拉起子进程。
- daemon() 函数
	```C
	#include <unistd.h>
	int daemon(int nochdir, int noclose);
	```
	- 当 nochdir 为0时，daemon 将更改进程的工作目录为根目录 root("/")。
	- 当 noclose 为0时，daemon 将进程的 STDIN, STDOUT, STDERR 都重定向到 /dev/null，也就是不输出任何信息，否则照样输出。一般情况下，这个参数都是设为0的。
	- deamon() 调用了 fork()，如果fork成功，父进程在 daemon 函数运行完毕后自杀，子进程由 init 进程领养。此时的子进程，其实就是守护进程了。
	- 一般使用 daemon(1, 0);


# 部分相关代码
```C++
/**
 * @filename    daemon.h
 * @brief   守护进程模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-18
 */
#ifndef __SYLAR_DAEMON_H__
#define __SYLAR_DAEMON_H__

#include <unistd.h>
#include <functional>
#include "sylar/singleton.h"

namespace sylar 
{

struct ProcessInfo 
{
    /// 父进程id
    pid_t parent_id = 0;
    /// 主进程id
    pid_t main_id = 0;
    /// 父进程启动时间
    uint64_t parent_start_time = 0;
    /// 主进程启动时间
    uint64_t main_start_time = 0;
    /// 主进程重启的次数
    uint32_t restart_count = 0;

    std::string toString() const;
};

typedef sylar::Singleton<ProcessInfo> ProcessInfoMgr;

/**
 * @brief 启动程序可以选择用守护进程的方式
 * @param[in] argc 参数个数
 * @param[in] argv 参数值数组
 * @param[in] main_cb 启动函数
 * @param[in] is_daemon 是否守护进程的方式
 * @return 返回程序的执行结果
 */
int start_daemon(int argc, char** argv
                 , std::function<int(int argc, char** argv)> main_cb
                 , bool is_daemon);

}

#endif


#include "daemon.h"
#include "sylar/log.h"
#include "sylar/config.h"
#include <time.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

namespace sylar 
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");
static sylar::ConfigVar<uint32_t>::ptr g_daemon_restart_interval
    = sylar::Config::Lookup("daemon.restart_interval", (uint32_t)5, "daemon restart interval");

std::string ProcessInfo::toString() const 
{
    std::stringstream ss;
    ss << "[ProcessInfo parent_id=" << parent_id
       << " main_id=" << main_id
       << " parent_start_time=" << sylar::Time2Str(parent_start_time)
       << " main_start_time=" << sylar::Time2Str(main_start_time)
       << " restart_count=" << restart_count << "]";
    return ss.str();
}

static int real_start(int argc, char** argv,
                     std::function<int(int argc, char** argv)> main_cb)
{
    ProcessInfoMgr::GetInstance()->main_id = getpid();
    ProcessInfoMgr::GetInstance()->main_start_time = time(0);
    return main_cb(argc, argv);
}

static int real_daemon(int argc, char** argv,
                     std::function<int(int argc, char** argv)> main_cb) 
{
    // #include <unistd.h>
    // int daemon(int nochdir, int noclose);
    // 当 nochdir 为0时，daemon 将更改进程的工作目录为根目录root("/")。
    // 当noclose为0时，daemon将进程的STDIN, STDOUT, STDERR都重定向到/dev/null，也就是不输出任何信息，否则照样输出。一般情况下，这个参数都是设为0的。
    // deamon()调用了fork()，如果fork成功，父进程在daemon函数运行完毕后自杀，子进程由init进程领养。此时的子进程，其实就是守护进程了。
    daemon(1, 0);   // 创建守护进程
    ProcessInfoMgr::GetInstance()->parent_id = getpid();
    ProcessInfoMgr::GetInstance()->parent_start_time = time(0);
    while(true) 
    {
        pid_t pid = fork();
        if(pid == 0) 
        {
            //子进程返回
            ProcessInfoMgr::GetInstance()->main_id = getpid();
            ProcessInfoMgr::GetInstance()->main_start_time  = time(0);
            SYLAR_LOG_INFO(g_logger) << "process start pid=" << getpid();
            return real_start(argc, argv, main_cb);
        } 
        else if(pid < 0) 
        {
            SYLAR_LOG_ERROR(g_logger) << "fork fail return=" << pid
                << " errno=" << errno << " errstr=" << strerror(errno);
            return -1;
        }
        else
        {
            //父进程返回
            int status = 0;
            waitpid(pid, &status, 0);
            if(status)
            {
                if(status == 9)
                {
                    SYLAR_LOG_INFO(g_logger) << "killed";
                    break;
                }
                else 
                {
                    SYLAR_LOG_ERROR(g_logger) << "child crash pid=" << pid
                        << " status=" << status;
                }
            } 
            else 
            {
                SYLAR_LOG_INFO(g_logger) << "child finished pid=" << pid;
                break;
            }
            ProcessInfoMgr::GetInstance()->restart_count += 1;
            // 防止子进程死掉之后资源还没有被全部回收，
            // 此时又马上fork出子进程可能会有问题。
            sleep(g_daemon_restart_interval->getValue());
        }
    }
    return 0;
}

int start_daemon(int argc, char** argv
                 , std::function<int(int argc, char** argv)> main_cb
                 , bool is_daemon) 
{
    if(!is_daemon) 
    {
        ProcessInfoMgr::GetInstance()->parent_id = getpid();
        ProcessInfoMgr::GetInstance()->parent_start_time = time(0);
        return real_start(argc, argv, main_cb);
    }
    return real_daemon(argc, argv, main_cb);
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
