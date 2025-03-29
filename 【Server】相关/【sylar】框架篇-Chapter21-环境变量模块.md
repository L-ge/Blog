# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 提供程序运行时的环境变量管理功能。这里的环境变量不仅包括系统环境变量，还包括程序自定义环境变量，命令行参数，帮助选项与描述，以及程序运行路径相关的信息。
- 所谓环境变量就是程序运行时可直接获取和设置的一组变量，它们往往代表一些特定的含义。所有的环境变量都以 key-value 的形式存储，key 和 value 都是字符串形式。这里可以参考系统环境变量来理解，在程序运行时，可以通过调用 getenv()/setenv() 接口来获取/设置系统环境变量，比如 getenv("PWD") 来获取当前路径。
- 在 shell 中可以通过 printenv 命令来打印当前所有的环境变量，并且在当前 shell 中运行的所有程序都共享这组环境变量值。
- 其他类型的环境变量也可以类比系统环境变量，只不过系统环境变量由 shell 来保存，而其他类型的环境变量由程序自己内部存储，但两者效果是一样的。具体地，sylar定义了以下几类环境变量：
	1. 系统环境变量，由 shell 保存，sylar 环境变量模块提供 getEnv()/setEnv() 方法用于操作系统环境变量。
	2. 程序自定义环境变量，对应 get()/add()/has()/del() 接口，自定义环境变量保存在程序自己的内存空间中，在内部实现为一个 std::map<std::string, std::string> 结构。
	3. 命令行参数，通过解析 main 函数的参数得到。所有参数都被解析成“选项-选项值”的形式，选项只能以-开头，后面跟选项值。如果一个参数只有选项没有值，那么值为空字符串。命令行参数也保存在程序自定义环境变量中。
	4. 帮助选项与描述。这里是为了统一生成程序的命令行帮助信息，在执行程序时如果指定了 -h 选项，那么就打印这些帮助信息。帮助选项与描述也是存储在程序自己的内存空间中，在内部实现为一个 std::vector<std::pair<std::string, std::string> > 结构。
	5. 与程序运行路径相关的信息，包括记录程序名，程序路径，当前路径，这些由单独的成员变量来存储。


# Env

- 封装环境变量类。
- 单例模式。通过单例可以保证程序的环境变量是全局唯一的，便于统一管理。
- Env 类提供以下方法：
	1. init: 环境变量模块初始化，需要将 main 函数的参数原样传入 init 接口中，以便于从 main 函数参数中提取命令行选项与值，以及通过 argv[0] 参数获取命令行程序名称。
	2. add/get/has/del：用于操作程序自定义环境变量，参数为 key-value，get 操作支持传入默认值，在对应的环境变量不存在时，返回这个默认值。
	3. setEnv/getEnv: 用于操作系统环境变量，对应标准库的 setenv/getenv 操作。
	4. addHelp/removeHelp/printHelp: 用于操作帮助选项和描述信息。
	5. getExe/getCwd/getAbsolutePath: 用于获取程序名称、程序路径、绝对路径。
	6. getConfigPath: 获取配置文件夹路径，配置文件夹路径由命令行 -c 选项传入。


# Application

- Server 主体框架：
	1. 防止重复启动多次（通过把 pid 写入文件里面去实现）
	2. 初始化日志文件路径（/path/to/log）
	3. 工作目录的路径（/path/to/work）
	4. 解析 httpserver 配置，通过配置，启动 httpserver。


# 其他说明

- 环境变量环境的一些实现细节：
	1. 获取程序的 bin 文件绝对路径是通过 /proc/$pid/ 目录下 exe 软链接文件指向的路径来确定的，用到了 readlink(2) 系统调用。
	2. 通过 bin 文件绝对路径可以得到 bin 文件所在的目录，只需要将最后的文件名部分去掉即可。
	3. 通过 argv[0] 获得命令行输入的程序路径，注意这里的路径可能是以 ./ 开头的相对路径。
	4. 通过 setenv/getenv 操作系统环境变量，参考 setenv(3), getenv(3)。
	5. 提供 getAbsolutePath 方法，传入一个相对于 bin 文件的路径，返回这个路径的绝对路径。比如默认的配置文件路径就是通过 getAbsolutePath(get("c", "conf")) 来获取的，也就是配置文件夹默认在 bin 文件所在目录的 conf 文件夹。
	6. 按使用惯例，main 函数执行的第一条语句应该就是调用 Env 的 init 方法初始化命令行参数。
- /proc/$pid/目录下的一些信息
	```shell
	/proc/$pid/cmd	启动的路径
	/proc/$pid/cmdline	exe(这个所带的路径不一定是绝对路径，它是执行程序的命令的路径)、启动参数（即传给 main 函数的参数）
	/proc/$pid/exe	exe(带上绝对路径)
	```
- 可以利用 /proc/$pid/cmdline 和全局变量构造函数，实现在进入 main 函数前解析启动参数。


# 待完善

- sylar 在解析命令行参数时，没有使用 getopt()/getopt_long() 接口，而是使用了自己编写的解析代码，这就导致 sylar 的命令行参数不支持长选项和选项合并，像 ps -aux 这样的多个选项组合在一起的命令行参数以及 ps --help 这样的长选项是不支持的。
- Application 有较多未完成的功能点，例如，ZKServiceDiscovery 和 RockSDLoadBalance。


# 部分相关代码
```C++
/**
 * @filename    env.h
 * @brief   环境变量模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-19
 */
#ifndef __SYLAR_ENV_H__
#define __SYLAR_ENV_H__
              
#include "sylar/singleton.h"
#include "sylar/mutex.h"
#include <iostream>
#include <map>
#include <vector>
              
namespace sylar
{             
              
class Env     
{             
public:       
    typedef RWMutex RWMutexType;
              
    bool init(int argc, char** argv);
              
    void add(const std::string& key, const std::string& val);
    bool has(const std::string& key);
    void del(const std::string& key);
    std::string get(const std::string& key, const std::string& default_value = "");
              
    void addHelp(const std::string& key, const std::string& desc);
    void removeHelp(const std::string& key);
    void printHelp();
              
    const std::string& getExe() const { return m_exe; }
    const std::string& getCwd() const { return m_cwd; }
              
    bool setEnv(const std::string& key, const std::string& val);
    std::string getEnv(const std::string& key, const std::string& default_value = "");
              
    std::string getAbsolutePath(const std::string& path) const;
    std::string getAbsoluteWorkPath(const std::string& path) const;
    std::string getConfigPath();

private:
    RWMutexType m_mutex;
    /// 参数信息
    std::map<std::string, std::string> m_args;
    /// 帮助信息
    std::vector<std::pair<std::string, std::string> > m_helps;
    /// 进程名(其实是进程信息，因为是argv[0]的值)
    std::string m_program;
    /// 即/proc/$pid/exe的值
    std::string m_exe;
    /// 根据/proc/$pid/exe的值设置的
    std::string m_cwd;
};            
              
typedef sylar::Singleton<Env> EnvMgr;

}
#endif

// env.cc
#include "env.h"
#include "sylar/config.h"
#include <iomanip>

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

bool Env::init(int argc, char** argv)
{
    char link[1024] = {0};
    char path[1024] = {0};
    sprintf(link, "/proc/%d/exe", getpid());
    readlink(link, path, sizeof(path)); // 从软链接获取真正的路径
    m_exe = path;

    auto pos = m_exe.find_last_of("/");
    m_cwd = m_exe.substr(0, pos) + "/";

    m_program = argv[0];

    const char* now_key = nullptr;
    for(int i=1; i<argc; ++i)
    {
        if(argv[i][0] == '-')
        {
            if(strlen(argv[i]) > 1)
            {
                if(now_key)
                {
                    add(now_key, "");
                }
                now_key = argv[i] + 1;
            }
            else
            {
                SYLAR_LOG_ERROR(g_logger) << "invalid arg idx=" << i
                                          << " val=" << argv[i];
                return false;
            }
        }
        else
        {
            if(now_key)
            {
                add(now_key, argv[i]);
                now_key = nullptr;
            }
            else
            {
                SYLAR_LOG_ERROR(g_logger) << "invalid arg idx=" << i
                                          << " val" << argv[i];
                return false;
            }
        }
    }
    if(now_key)
    {
        add(now_key, "");
    }
    return true;
}

void Env::add(const std::string& key, const std::string& val)
{
    RWMutexType::WriteLock lk(m_mutex);
    m_args[key] = val;
}

bool Env::has(const std::string& key)
{
    RWMutexType::ReadLock lk(m_mutex);
    auto it = m_args.find(key);
    return it != m_args.end();
}

void Env::del(const std::string& key)
{
    RWMutexType::WriteLock lk(m_mutex);
    m_args.erase(key);
}
    
std::string Env::get(const std::string& key, const std::string& default_value)
{
    RWMutexType::ReadLock lk(m_mutex);
    auto it = m_args.find(key);
    return it != m_args.end() ? it->second : default_value;
}
              
void Env::addHelp(const std::string& key, const std::string& desc)
{
    removeHelp(key);
    RWMutexType::WriteLock lk(m_mutex);
    m_helps.push_back(std::make_pair(key, desc));
}

void Env::removeHelp(const std::string& key)
{
    RWMutexType::WriteLock lk(m_mutex);
    for(auto it=m_helps.begin(); it!=m_helps.end();)
    {
        if(it->first == key)
        {
            it = m_helps.erase(it);
        }
        else
        {
            ++it;
        }
    }
}

void Env::printHelp()
{
    RWMutexType::ReadLock lk(m_mutex);
    std::cout << "Usage: " << m_program << " [options]" << std::endl;
    for(auto& i : m_helps)
    {
        std::cout << std::setw(5) << "-" << i.first << " : " << i.second << std::endl;
    }
}

bool Env::setEnv(const std::string& key, const std::string& val)
{
    //int setenv(const char* name, const char* value, int overwrite);
    //setenv()用来改变或增加环境变量的内容。
    //参数name为环境变量名称字符串。
    //参数value则为变量内容。
    //参数overwrite用来决定是否要改变已存在的环境变量。
    //如果没有此环境变量则无论overwrite为何值均添加此环境变量。
    //若环境变量存在，当overwrite不为0时，原内容会被改为参数value所指的变量内容；
    //当overwrite为0时，则参数value会被忽略。返回值执行成功则返回0，有错误发生时返回-1。
    return !setenv(key.c_str(), val.c_str(), 1);
}
    
std::string Env::getEnv(const std::string& key, const std::string& default_value)
{
    const char* v = getenv(key.c_str());
    if(v == nullptr)
    {
        return default_value;
    }
    return v;
}
              
std::string Env::getAbsolutePath(const std::string& path) const
{
    if(path.empty())
    {
        return "/";
    }
    if(path[0] == '/')  // 如果第一个字符已经是/，那这已经是绝对路径了。
    {
        return path;
    }
    return m_cwd + path;
}

std::string Env::getAbsoluteWorkPath(const std::string& path) const
{
    if(path.empty())
    {
        return "/";
    }
    if(path[0] == '/')
    {
        return path;
    }
    static sylar::ConfigVar<std::string>::ptr g_server_work_path = 
        sylar::Config::Lookup<std::string>("server.work_path");
    return g_server_work_path->getValue() + "/" + path;
}

std::string Env::getConfigPath()
{
    return getAbsolutePath(get("c", "conf"));
}

}

/**
 * @filename    application.h
 * @brief   Application 模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-19
 */
#ifndef __SYLAR_APPLICATION_H__
#define __SYLAR_APPLICATION_H__

#include "sylar/http/http_server.h"
//#include "sylar/streams/service_discovery.h"
//#include "sylar/rock/rock_stream.h"

namespace sylar 
{

class Application 
{
public:
    Application();

    static Application* GetInstance() { return s_instance; }
    bool init(int argc, char** argv);
    bool run();

    bool getServer(const std::string& type, std::vector<TcpServer::ptr>& svrs);
    void listAllServer(std::map<std::string, std::vector<TcpServer::ptr> >& servers);

    //ZKServiceDiscovery::ptr getServiceDiscovery() const { return m_serviceDiscovery; }
    //RockSDLoadBalance::ptr getRockSDLoadBalance() const { return m_rockSDLoadBalance; }

private:
    int main(int argc, char** argv);
    int run_fiber();

private:
    int m_argc = 0;
    char** m_argv = nullptr;

    std::map<std::string, std::vector<TcpServer::ptr> > m_servers;
    IOManager::ptr m_mainIOManager;
    static Application* s_instance;

    //ZKServiceDiscovery::ptr m_serviceDiscovery;
    //RockSDLoadBalance::ptr m_rockSDLoadBalance;
};

}

#endif

// application.cc
#include "application.h"

#include <unistd.h>
#include <signal.h>

#include "sylar/tcp_server.h"
#include "sylar/daemon.h"
#include "sylar/config.h"
#include "sylar/env.h"
#include "sylar/log.h"
#include "sylar/module.h"
//#include "sylar/rock/rock_stream.h"
//#include "sylar/worker.h"
//#include "sylar/http/ws_server.h"
//#include "sylar/rock/rock_server.h"
//#include "sylar/ns/name_server_module.h"
//#include "sylar/db/fox_thread.h"
//#include "sylar/db/redis.h"

namespace sylar 
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

static sylar::ConfigVar<std::string>::ptr g_server_work_path =
    sylar::Config::Lookup("server.work_path"
            ,std::string("/apps/work/sylar")
            , "server work path");

static sylar::ConfigVar<std::string>::ptr g_server_pid_file =
    sylar::Config::Lookup("server.pid_file"
            ,std::string("sylar.pid")
            , "server pid file");

//static sylar::ConfigVar<std::string>::ptr g_service_discovery_zk =
//    sylar::Config::Lookup("service_discovery.zk"
//            ,std::string("")
//            , "service discovery zookeeper");

static sylar::ConfigVar<std::vector<TcpServerConf> >::ptr g_servers_conf
    = sylar::Config::Lookup("servers", std::vector<TcpServerConf>(), "http server config");

Application* Application::s_instance = nullptr;

Application::Application()
{
    s_instance = this;
}

bool Application::init(int argc, char** argv)
{
    m_argc = argc;
    m_argv = argv;

    sylar::EnvMgr::GetInstance()->addHelp("s", "start with the terminal");
    sylar::EnvMgr::GetInstance()->addHelp("d", "run as daemon");
    sylar::EnvMgr::GetInstance()->addHelp("c", "conf path default: ./conf");
    sylar::EnvMgr::GetInstance()->addHelp("p", "print help");

    bool is_print_help = false;
    if(!sylar::EnvMgr::GetInstance()->init(argc, argv)) 
    {
        is_print_help = true;
    }

    if(sylar::EnvMgr::GetInstance()->has("p")) 
    {
        is_print_help = true;
    }

    // 加载配置文件
    std::string conf_path = sylar::EnvMgr::GetInstance()->getConfigPath();
    SYLAR_LOG_INFO(g_logger) << "load conf path:" << conf_path;
    sylar::Config::LoadFromConfDir(conf_path);

    ModuleMgr::GetInstance()->init();
    std::vector<Module::ptr> modules;
    ModuleMgr::GetInstance()->listAll(modules);
    for(auto i : modules) 
    {
        i->onBeforeArgsParse(argc, argv);
    }

    if(is_print_help) 
    {
        sylar::EnvMgr::GetInstance()->printHelp();
        return false;
    }

    for(auto i : modules) 
    {
        i->onAfterArgsParse(argc, argv);
    }
    modules.clear();

    //是否以守护进程的方式运行
    int run_type = 0;
    if(sylar::EnvMgr::GetInstance()->has("s"))
    {
        run_type = 1;
    }
    if(sylar::EnvMgr::GetInstance()->has("d"))
    {
        run_type = 2;
    }

    if(run_type == 0) 
    {
        sylar::EnvMgr::GetInstance()->printHelp();
        return false;
    }

    std::string pidfile = g_server_work_path->getValue()
                                + "/" + g_server_pid_file->getValue();
    if(sylar::FSUtil::IsRunningPidfile(pidfile)) 
    {
        SYLAR_LOG_ERROR(g_logger) << "server is running:" << pidfile;
        return false;
    }

    if(!sylar::FSUtil::Mkdir(g_server_work_path->getValue()))
    {
        SYLAR_LOG_FATAL(g_logger) << "create work path [" << g_server_work_path->getValue()
            << " errno=" << errno << " errstr=" << strerror(errno);
        return false;
    }
    return true;
}

bool Application::run() 
{
    bool is_daemon = sylar::EnvMgr::GetInstance()->has("d");
    return start_daemon(m_argc, m_argv,
            std::bind(&Application::main, this, std::placeholders::_1,
                std::placeholders::_2), is_daemon); // 两个形参，两个占位符
}

int Application::main(int argc, char** argv)
{
    signal(SIGPIPE, SIG_IGN);
    SYLAR_LOG_INFO(g_logger) << "main";
    std::string conf_path = sylar::EnvMgr::GetInstance()->getConfigPath();
    sylar::Config::LoadFromConfDir(conf_path, true);
    {
        std::string pidfile = g_server_work_path->getValue()
                                    + "/" + g_server_pid_file->getValue();
        std::ofstream ofs(pidfile);
        if(!ofs)
        {
            SYLAR_LOG_ERROR(g_logger) << "open pidfile " << pidfile << " failed";
            return false;
        }
        ofs << getpid();    // 把pid写入文件里面去
    }

    m_mainIOManager.reset(new sylar::IOManager(1, true, "main"));
    m_mainIOManager->schedule(std::bind(&Application::run_fiber, this));
    m_mainIOManager->addTimer(2000, [](){
            //SYLAR_LOG_INFO(g_logger) << "hello";
    }, true);
    m_mainIOManager->stop();
    return 0;
}

int Application::run_fiber() 
{
    SYLAR_LOG_INFO(g_logger) << "run_fiber";
    std::vector<Module::ptr> modules;
    ModuleMgr::GetInstance()->listAll(modules);
    bool has_error = false;
    for(auto& i : modules) 
    {
        if(!i->onLoad()) 
        {
            SYLAR_LOG_ERROR(g_logger) << "module name="
                << i->getName() << " version=" << i->getVersion()
                << " filename=" << i->getFilename();
            has_error = true;
        }
    }
    if(has_error) 
    {
        _exit(0);
    }

    //sylar::WorkerMgr::GetInstance()->init();
    //FoxThreadMgr::GetInstance()->init();
    //FoxThreadMgr::GetInstance()->start();
    //RedisMgr::GetInstance();

    auto http_confs = g_servers_conf->getValue();
    std::vector<TcpServer::ptr> svrs;
    for(auto& i : http_confs) 
    {
        SYLAR_LOG_DEBUG(g_logger) << std::endl << LexicalCast<TcpServerConf, std::string>()(i);

        std::vector<Address::ptr> address;
        for(auto& a : i.address)
        {
            size_t pos = a.find(":");
            if(pos == std::string::npos)
            {
                address.push_back(UnixAddress::ptr(new UnixAddress(a)));
                continue;
            }
            int32_t port = atoi(a.substr(pos + 1).c_str());
            
            //127.0.0.1，ip形式
            auto addr = sylar::IPAddress::Create(a.substr(0, pos).c_str(), port);
            if(addr)
            {
                address.push_back(addr);
                continue;
            }

            // 网卡形式
            std::vector<std::pair<Address::ptr, uint32_t> > result;
            if(sylar::Address::GetInterfaceAddresses(result, a.substr(0, pos)))
            {
                for(auto& x : result)
                {
                    auto ipaddr = std::dynamic_pointer_cast<IPAddress>(x.first);
                    if(ipaddr) 
                    {
                        ipaddr->setPort(atoi(a.substr(pos + 1).c_str()));
                    }
                    address.push_back(ipaddr);
                }
                continue;
            }
            
            // 网址形式
            auto aaddr = sylar::Address::LookupAny(a);
            if(aaddr)
            {
                address.push_back(aaddr);
                continue;
            }
            SYLAR_LOG_ERROR(g_logger) << "invalid address: " << a;
            _exit(0);
        }
    
        IOManager* accept_worker = sylar::IOManager::GetThis();
        IOManager* io_worker = sylar::IOManager::GetThis();
        IOManager* process_worker = sylar::IOManager::GetThis();
        //if(!i.accept_worker.empty())
        //{
        //    accept_worker = sylar::WorkerMgr::GetInstance()->getAsIOManager(i.accept_worker).get();
        //    if(!accept_worker)
        //    {
        //        SYLAR_LOG_ERROR(g_logger) << "accept_worker: " << i.accept_worker
        //            << " not exists";
        //        _exit(0);
        //    }
        //}
        //if(!i.io_worker.empty()) 
        //{
        //    io_worker = sylar::WorkerMgr::GetInstance()->getAsIOManager(i.io_worker).get();
        //    if(!io_worker) 
        //    {
        //        SYLAR_LOG_ERROR(g_logger) << "io_worker: " << i.io_worker
        //            << " not exists";
        //        _exit(0);
        //    }
        //}
        //if(!i.process_worker.empty()) 
        //{
        //    process_worker = sylar::WorkerMgr::GetInstance()->getAsIOManager(i.process_worker).get();
        //    if(!process_worker) 
        //    {
        //        SYLAR_LOG_ERROR(g_logger) << "process_worker: " << i.process_worker
        //            << " not exists";
        //        _exit(0);
        //    }
        //}

        TcpServer::ptr server;
        if(i.type == "http") 
        {
            server.reset(new sylar::http::HttpServer(i.keepalive,
                            process_worker, io_worker, accept_worker));
        }
        //else if(i.type == "ws") 
        //{
        //    server.reset(new sylar::http::WSServer(
        //                    process_worker, io_worker, accept_worker));
        //}
        //else if(i.type == "rock") 
        //{
        //    server.reset(new sylar::RockServer("rock", process_worker, io_worker, accept_worker));
        //} 
        //else if(i.type == "nameserver") 
        //{
        //    server.reset(new sylar::RockServer("nameserver",
        //                    process_worker, io_worker, accept_worker));
        //    ModuleMgr::GetInstance()->add(std::make_shared<sylar::ns::NameServerModule>());
        //}
        else
        {
            SYLAR_LOG_ERROR(g_logger) << "invalid server type=" << i.type
                << LexicalCast<TcpServerConf, std::string>()(i);
            _exit(0);
        }
        
        if(!i.name.empty()) 
        {
            server->setName(i.name);
        }
        std::vector<Address::ptr> fails;
        if(!server->bind(address, fails, i.ssl)) 
        {
            for(auto& x : fails)
            {
                SYLAR_LOG_ERROR(g_logger) << "bind address fail:"
                    << *x;
            }
            _exit(0);
        }
        //if(i.ssl)
        //{
        //    if(!server->loadCertificates(i.cert_file, i.key_file))
        //    {
        //        SYLAR_LOG_ERROR(g_logger) << "loadCertificates fail, cert_file="
        //            << i.cert_file << " key_file=" << i.key_file;
        //    }
        //}
        
        SYLAR_LOG_INFO(g_logger) << "server push:" << i.type;

        server->setConf(i);
        m_servers[i.type].push_back(server);
        svrs.push_back(server);
    }

    //if(!g_service_discovery_zk->getValue().empty()) 
    //{
    //    m_serviceDiscovery.reset(new ZKServiceDiscovery(g_service_discovery_zk->getValue()));
    //    m_rockSDLoadBalance.reset(new RockSDLoadBalance(m_serviceDiscovery));

    //    std::vector<TcpServer::ptr> svrs;
    //    if(!getServer("http", svrs))
    //    {
    //        m_serviceDiscovery->setSelfInfo(sylar::GetIPv4() + ":0:" + sylar::GetHostName());
    //    } 
    //    else 
    //    {
    //        std::string ip_and_port;
    //        for(auto& i : svrs) 
    //        {
    //            auto socks = i->getSocks();
    //            for(auto& s : socks)
    //            {
    //                auto addr = std::dynamic_pointer_cast<IPv4Address>(s->getLocalAddress());
    //                if(!addr)
    //                {
    //                    continue;
    //                }
    //                auto str = addr->toString();
    //                if(str.find("127.0.0.1") == 0) 
    //                {
    //                    continue;
    //                }
    //                if(str.find("0.0.0.0") == 0) 
    //                {
    //                    ip_and_port = sylar::GetIPv4() + ":" + std::to_string(addr->getPort());
    //                    break;
    //                }
    //                else
    //                {
    //                    ip_and_port = addr->toString();
    //                }
    //            }
    //            if(!ip_and_port.empty()) 
    //            {
    //                break;
    //            }
    //        }
    //        m_serviceDiscovery->setSelfInfo(ip_and_port + ":" + sylar::GetHostName());
    //    }
    //}

    for(auto& i : modules) 
    {
        i->onServerReady();
    }

    for(auto& i : svrs) 
    {
        i->start();
    }

    //if(m_rockSDLoadBalance)
    //{
    //    m_rockSDLoadBalance->start();
    //}

    for(auto& i : modules)
    {
        i->onServerUp();
    }
    return 0;
}

bool Application::getServer(const std::string& type, std::vector<TcpServer::ptr>& svrs) 
{
    auto it = m_servers.find(type);
    if(it == m_servers.end())
    {
        return false;
    }
    svrs = it->second;
    return true;
}

void Application::listAllServer(std::map<std::string, std::vector<TcpServer::ptr> >& servers)
{
    servers = m_servers;
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
