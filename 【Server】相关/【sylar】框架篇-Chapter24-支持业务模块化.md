# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- sylar 框架如何做一些小项目？直接从 main 函数开始说起。
- sylar 的做法是将业务都封装成一个一个的动态库，从框架的去加载。


# sylar启动流程

```
		main函数
			↓
Application的init(读配置文件，读到业务动态库的存放路径)
			↓
ModuleManager的init(每个so库依次initModule)
			↓
ModuleManager::initModule(调用Library的GetModule，并缓存起来)
			↓
Library的GetModule(dlopen加载动态库，并CreateModule)
			↓
回到main函数，Application开始run(其实最终是Application::run_fiber)
			↓
Application的run_fiber(根据配置，初始化每一个server)
			↓
	最后就是等待连接
```


# 业务模块的启动

```
外部通过导出的"CreateModule"接口实例化该模块
			↓
Application的init函数的最后会调用每个Module的onServerReady函数
			↓
Module的onServerReady注册好业务模块自己的Servlet(实例化并缓存起来)
			↓
Application初始化完成后，就开始监听socket连接，如果有连接进来，最后是调用对应Servlet的handle方法。
			↓
在业务模块自己Servlet的handle方法里面实现对应的业务逻辑即可
```


# 其他说明

- 每个业务动态库都有"CreateModule"和"DestoryModule"两个接口，extern "C"导出。
	```
	// 可以用如下命令查看接口：
	nm -D libparamquery.so

	// 如果没有extern "C"，导出的函数名例如：
	_ZTIN5sylar12SocketStreamE

	// 如果有extern "C"，导出的函数名例如：
	testFunc
	```
- main函数是在框架那里，并不是在业务模块。


# 部分相关代码
```C++
// main.cc
#include "sylar/application.h"
#include <stdlib.h>
#include <time.h>

int main(int argc, char** argv) 
{
    setenv("TZ", ":/etc/localtime", 1);
    tzset();
    srand(time(0));
    sylar::Application app;
    if(app.init(argc, argv))
    {
        return app.run();
    }
    return 0;
}


// module.h
#ifndef __SYLAR_MODULE_H__
#define __SYLAR_MODULE_H__

#include "sylar/stream.h"
#include "sylar/singleton.h"
#include "sylar/mutex.h"
#include "sylar/protocol.h"
//#include "sylar/rock/rock_stream.h"
#include <map>
#include <unordered_map>

namespace sylar 
{

class Module {
public:
    enum Type {
        MODULE = 0,
    //    ROCK = 1,
    };

    typedef std::shared_ptr<Module> ptr;
    Module(const std::string& name
         , const std::string& version
         , const std::string& filename
         , uint32_t type = MODULE);
    virtual ~Module() {}

    virtual void onBeforeArgsParse(int argc, char** argv);
    virtual void onAfterArgsParse(int argc, char** argv);

    virtual bool onLoad();
    virtual bool onUnload();

    virtual bool onConnect(sylar::Stream::ptr stream);
    virtual bool onDisconnect(sylar::Stream::ptr stream);
    
    virtual bool onServerReady();
    virtual bool onServerUp();

    virtual bool handleRequest(sylar::Message::ptr req
                             , sylar::Message::ptr rsp
                             , sylar::Stream::ptr stream);
    virtual bool handleNotify(sylar::Message::ptr notify
                            , sylar::Stream::ptr stream);

    virtual std::string statusString();

    const std::string& getName() const { return m_name; }
    const std::string& getVersion() const { return m_version; }
    const std::string& getFilename() const { return m_filename; }
    const std::string& getId() const { return m_id; }

    void setFilename(const std::string& v) { m_filename = v; }

    uint32_t getType() const { return m_type; }

    //void registerService(const std::string& server_type
    //                   , const std::string& domain
    //                   , const std::string& service);

protected:
    std::string m_name;
    std::string m_version;
    std::string m_filename;
    std::string m_id;
    uint32_t m_type;
};

//class RockModule : public Module {
//public:
//    typedef std::shared_ptr<RockModule> ptr;
//    RockModule(const std::string& name
//               ,const std::string& version
//               ,const std::string& filename);
//
//    virtual bool handleRockRequest(sylar::RockRequest::ptr request
//                        ,sylar::RockResponse::ptr response
//                        ,sylar::RockStream::ptr stream) = 0;
//    virtual bool handleRockNotify(sylar::RockNotify::ptr notify
//                        ,sylar::RockStream::ptr stream) = 0;
//
//    virtual bool handleRequest(sylar::Message::ptr req
//                               ,sylar::Message::ptr rsp
//                               ,sylar::Stream::ptr stream);
//    virtual bool handleNotify(sylar::Message::ptr notify
//                              ,sylar::Stream::ptr stream);
//
//};

class ModuleManager 
{
public:
    typedef RWMutex RWMutexType;

    ModuleManager();

    void add(Module::ptr m);
    void del(const std::string& name);
    void delAll();

    void init();

    Module::ptr get(const std::string& name);

    void onConnect(Stream::ptr stream);
    void onDisconnect(Stream::ptr stream);

    void listAll(std::vector<Module::ptr>& ms);
    void listByType(uint32_t type, std::vector<Module::ptr>& ms);
    void foreach(uint32_t type, std::function<void(Module::ptr)> cb);

private:
    void initModule(const std::string& path);

private:
    RWMutexType m_mutex;
    std::unordered_map<std::string, Module::ptr> m_modules;
    std::unordered_map<uint32_t
        , std::unordered_map<std::string, Module::ptr> > m_type2Modules;
};

typedef sylar::Singleton<ModuleManager> ModuleMgr;

}

#endif


// module.cc
#include "module.h"
#include "config.h"
#include "env.h"
#include "library.h"
#include "util.h"
#include "log.h"
#include "application.h"

namespace sylar {

static sylar::ConfigVar<std::string>::ptr g_module_path
    = Config::Lookup("module.path", std::string("module"), "module path");

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

Module::Module(const std::string& name
            ,const std::string& version
            ,const std::string& filename
            ,uint32_t type)
    :m_name(name)
    ,m_version(version)
    ,m_filename(filename)
    ,m_id(name + "/" + version)
    ,m_type(type) 
{
}

void Module::onBeforeArgsParse(int argc, char** argv)
{
}

void Module::onAfterArgsParse(int argc, char** argv) 
{
}

bool Module::handleRequest(sylar::Message::ptr req
                           ,sylar::Message::ptr rsp
                           ,sylar::Stream::ptr stream)
{
    SYLAR_LOG_DEBUG(g_logger) << "handleRequest req=" << req->toString()
            << " rsp=" << rsp->toString() << " stream=" << stream;
    return true;
}

bool Module::handleNotify(sylar::Message::ptr notify
                          ,sylar::Stream::ptr stream)
{
    SYLAR_LOG_DEBUG(g_logger) << "handleNotify nty=" << notify->toString()
            << " stream=" << stream;
    return true;
}

bool Module::onLoad()
{
    return true;
}

bool Module::onUnload() 
{
    return true;
}

bool Module::onConnect(sylar::Stream::ptr stream) 
{
    return true;
}

bool Module::onDisconnect(sylar::Stream::ptr stream)
{
    return true;
}

bool Module::onServerReady() 
{
    return true;
}

bool Module::onServerUp() 
{
    return true;
}

//void Module::registerService(const std::string& server_type
//                           , const std::string& domain
//                           , const std::string& service)
//{
//    auto sd = Application::GetInstance()->getServiceDiscovery();
//    if(!sd) 
//    {
//        return;
//    }
//    std::vector<TcpServer::ptr> svrs;
//    if(!Application::GetInstance()->getServer(server_type, svrs))
//    {
//        return;
//    }
//    for(auto& i : svrs)
//    {
//        auto socks = i->getSocks();
//        for(auto& s : socks)
//        {
//            auto addr = std::dynamic_pointer_cast<IPv4Address>(s->getLocalAddress());
//            if(!addr)
//            {
//                continue;
//            }
//            auto str = addr->toString();
//            if(str.find("127.0.0.1") == 0)
//            {
//                continue;
//            }
//            std::string ip_and_port;
//            if(str.find("0.0.0.0") == 0)
//            {
//                ip_and_port = sylar::GetIPv4() + ":" + std::to_string(addr->getPort());
//            }
//            else 
//            {
//                ip_and_port = addr->toString();
//            }
//            sd->registerServer(domain, service, ip_and_port, server_type);
//        }
//    }
//}

std::string Module::statusString()
{
    std::stringstream ss;
    ss << "Module name=" << getName()
       << " version=" << getVersion()
       << " filename=" << getFilename()
       << std::endl;
    return ss.str();
}

//RockModule::RockModule(const std::string& name
//                       ,const std::string& version
//                       ,const std::string& filename)
//    :Module(name, version, filename, ROCK) {
//}
//
//bool RockModule::handleRequest(sylar::Message::ptr req
//                               ,sylar::Message::ptr rsp
//                               ,sylar::Stream::ptr stream) {
//    auto rock_req = std::dynamic_pointer_cast<sylar::RockRequest>(req);
//    auto rock_rsp = std::dynamic_pointer_cast<sylar::RockResponse>(rsp);
//    auto rock_stream = std::dynamic_pointer_cast<sylar::RockStream>(stream);
//    return handleRockRequest(rock_req, rock_rsp, rock_stream);
//}
//
//bool RockModule::handleNotify(sylar::Message::ptr notify
//                              ,sylar::Stream::ptr stream) {
//    auto rock_nty = std::dynamic_pointer_cast<sylar::RockNotify>(notify);
//    auto rock_stream = std::dynamic_pointer_cast<sylar::RockStream>(stream);
//    return handleRockNotify(rock_nty, rock_stream);
//}

ModuleManager::ModuleManager()
{
}

Module::ptr ModuleManager::get(const std::string& name)
{
    RWMutexType::ReadLock lock(m_mutex);
    auto it = m_modules.find(name);
    return it == m_modules.end() ? nullptr : it->second;
}

void ModuleManager::add(Module::ptr m)
{
    del(m->getId());
    RWMutexType::WriteLock lock(m_mutex);
    m_modules[m->getId()] = m;
    m_type2Modules[m->getType()][m->getId()] = m;
}

void ModuleManager::del(const std::string& name) 
{
    Module::ptr module;
    RWMutexType::WriteLock lock(m_mutex);
    auto it = m_modules.find(name);
    if(it == m_modules.end()) 
    {
        return;
    }
    module = it->second;
    m_modules.erase(it);
    m_type2Modules[module->getType()].erase(module->getId());
    if(m_type2Modules[module->getType()].empty())
    {
        m_type2Modules.erase(module->getType());
    }
    lock.unlock();
    module->onUnload();
}

void ModuleManager::delAll() 
{
    RWMutexType::ReadLock lock(m_mutex);
    auto tmp = m_modules;
    lock.unlock();

    for(auto& i : tmp)
    {
        del(i.first);
    }
}

void ModuleManager::init()
{
    auto path = EnvMgr::GetInstance()->getAbsolutePath(g_module_path->getValue());
   
    SYLAR_LOG_DEBUG(g_logger) << "module path:" << path;

    std::vector<std::string> files;
    sylar::FSUtil::ListAllFile(files, path, ".so");

    std::sort(files.begin(), files.end());
    for(auto& i : files)
    {
        initModule(i);
    }
}

void ModuleManager::listByType(uint32_t type, std::vector<Module::ptr>& ms) 
{
    RWMutexType::ReadLock lock(m_mutex);
    auto it = m_type2Modules.find(type);
    if(it == m_type2Modules.end()) 
    {
        return;
    }
    for(auto& i : it->second)
    {
        ms.push_back(i.second);
    }
}

void ModuleManager::foreach(uint32_t type, std::function<void(Module::ptr)> cb)
{
    std::vector<Module::ptr> ms;
    listByType(type, ms);
    for(auto& i : ms) 
    {
        cb(i);
    }
}

void ModuleManager::onConnect(Stream::ptr stream)
{
    std::vector<Module::ptr> ms;
    listAll(ms);

    for(auto& m : ms)
    {
        m->onConnect(stream);
    }
}

void ModuleManager::onDisconnect(Stream::ptr stream) 
{
    std::vector<Module::ptr> ms;
    listAll(ms);

    for(auto& m : ms)
    {
        m->onDisconnect(stream);
    }
}

void ModuleManager::listAll(std::vector<Module::ptr>& ms)
{
    RWMutexType::ReadLock lock(m_mutex);
    for(auto& i : m_modules) 
    {
        ms.push_back(i.second);
    }
}

void ModuleManager::initModule(const std::string& path)
{
    Module::ptr m = Library::GetModule(path);
    if(m) 
    {
        add(m);
    }
}

}


//library.h
#ifndef __SYLAR_LIBRARY_H__
#define __SYLAR_LIBRARY_H__

#include <memory>
#include "module.h"

namespace sylar
{

class Library
{
public:
    static Module::ptr GetModule(const std::string& path);
};

}

#endif


// library.cc
#include "library.h"

#include <dlfcn.h>
#include "sylar/config.h"
#include "sylar/env.h"
#include "sylar/log.h"

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

typedef Module* (*create_module)();
typedef void (*destory_module)(Module*);

class ModuleCloser 
{
public:
    ModuleCloser(void* handle, destory_module d)
        : m_handle(handle)
        , m_destory(d) 
    {
    }

    void operator()(Module* module) 
    {
        std::string name = module->getName();
        std::string version = module->getVersion();
        std::string path = module->getFilename();
        m_destory(module);
        int rt = dlclose(m_handle);
        if(rt) 
        {
            SYLAR_LOG_ERROR(g_logger) << "dlclose handle fail handle="
                << m_handle << " name=" << name
                << " version=" << version
                << " path=" << path
                << " error=" << dlerror();
        }
        else 
        {
            SYLAR_LOG_INFO(g_logger) << "destory module=" << name
                << " version=" << version
                << " path=" << path
                << " handle=" << m_handle
                << " success";
        }
    }

private:
    void* m_handle;
    destory_module m_destory;
};

Module::ptr Library::GetModule(const std::string& path)
{
    void* handle = dlopen(path.c_str(), RTLD_NOW);
    if(!handle)
    {
        SYLAR_LOG_ERROR(g_logger) << "cannot load library path="
            << path << " error=" << dlerror();
        return nullptr;
    }

    create_module create = (create_module)dlsym(handle, "CreateModule");
    if(!create) 
    {
        SYLAR_LOG_ERROR(g_logger) << "cannot load symbol CreateModule in "
            << path << " error=" << dlerror();
        dlclose(handle);
        return nullptr;
    }

    destory_module destory = (destory_module)dlsym(handle, "DestoryModule");
    if(!destory) 
    {
        SYLAR_LOG_ERROR(g_logger) << "cannot load symbol DestoryModule in "
            << path << " error=" << dlerror();
        dlclose(handle);
        return nullptr;
    }

    Module::ptr module(create(), ModuleCloser(handle, destory));
    module->setFilename(path);
    SYLAR_LOG_INFO(g_logger) << "load module name=" << module->getName()
        << " version=" << module->getVersion()
        << " path=" << module->getFilename()
        << " success";
    Config::LoadFromConfDir(sylar::EnvMgr::GetInstance()->getConfigPath(), true);
    return module;
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
