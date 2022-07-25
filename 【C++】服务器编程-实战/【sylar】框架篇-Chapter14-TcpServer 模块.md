# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- TCP 服务器的封装。


# TcpServerConf

- TcpServer 配置结构体。
- 两个模板偏特化：
	- template<> class LexicalCast<std::string, TcpServerConf>
	- template<> class LexicalCast<TcpServerConf, std::string>


# TcpServer

- 继承自 std::enable_shared_from_this<TcpServer> 和 sylar::Noncopyable。
-  TcpServer 类支持同时绑定多个地址进行监听，只需要在绑定时传入地址数组即可。有两个重载的 bind 方法。
-  TcpServer 还可以分别指定接收客户端和处理客户端的协程调度器。具体是 m_acceptWorker 和 m_ioWorker。


# 其他说明

- TcpServer 类采用了模板模式（Template Pattern）的设计模式，它的 HandleClient 是交由继承类来实现的。使用 TcpServer 时，必须从 TcpServer 派生一个新类，并重新实现子类的 handleClient 操作。
- 在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。这种类型的设计模式属于行为型模式。


# 待完善

- 暂未调试支持 SSL。


# 部分相关代码
```C++
/**
 * @filename    tcp_server.h
 * @brief   Tcp 服务器的封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-13
 */
#ifndef __SYLAR_TCP_SERVER_H__
#define __SYLAR_TCP_SERVER_H__

#include <memory>
#include <functional>
#include "address.h"
#include "iomanager.h"
#include "socket.h"
#include "noncopyable.h"
#include "config.h"

namespace sylar
{

/**
 * @brief   TcpServer配置结构体
 */
struct TcpServerConf
{
    typedef std::shared_ptr<TcpServerConf> ptr;

    std::vector<std::string> address;
    int keepalive = 0;
    int timeout = 1000 * 2 * 60;
    int ssl = 0;
    std::string id;

    // 服务器类型：http、ws、rock
    std::string type = "http";
    std::string name;
    std::string cert_file;
    std::string key_file;
    std::string accept_worker;
    std::string io_worker;
    std::string process_worker;
    std::map<std::string, std::string> args;

    bool isValid() const
    {
        return !address.empty();
    }

    bool operator==(const TcpServerConf& oth) const
    {
        return address == oth.address
            && keepalive == oth.keepalive
            && timeout == oth.timeout
            && name == oth.name
            && ssl == oth.ssl
            && cert_file == oth.cert_file
            && key_file == oth.key_file
            && accept_worker == oth.accept_worker
            && io_worker == oth.io_worker
            && process_worker == oth.process_worker
            && args == oth.args
            && id == oth.id
            && type == oth.type;
    }
};

template<>
class LexicalCast<std::string, TcpServerConf>
{
public:
    TcpServerConf operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        TcpServerConf conf;
        conf.id             = node["id"].as<std::string>(conf.id);
        conf.type           = node["type"].as<std::string>(conf.type);
        conf.keepalive      = node["keepalive"].as<int>(conf.keepalive);
        conf.timeout        = node["timeout"].as<int>(conf.timeout);
        conf.name           = node["name"].as<std::string>(conf.name);
        conf.ssl            = node["ssl"].as<int>(conf.ssl);
        conf.cert_file      = node["cert_file"].as<std::string>(conf.cert_file);
        conf.key_file       = node["key_file"].as<std::string>(conf.key_file);
        conf.accept_worker  = node["accept_worker"].as<std::string>();
        conf.io_worker      = node["io_worker"].as<std::string>();
        conf.process_worker = node["process_worker"].as<std::string>();
        conf.args           = LexicalCast<std::string
            , std::map<std::string, std::string> >()(node["args"].as<std::string>(""));
        if(node["address"].IsDefined())
        {
            for(size_t i = 0; i < node["address"].size(); ++i)
            {
                conf.address.push_back(node["address"][i].as<std::string>());
            }
        }
        return conf;
    }
};

template<>
class LexicalCast<TcpServerConf, std::string>
{
public:
    std::string operator()(const TcpServerConf& conf)
    {
        YAML::Node node;
        node["id"]              = conf.id;
        node["type"]            = conf.type;
        node["keepalive"]       = conf.keepalive;
        node["timeout"]         = conf.timeout;
        node["name"]            = conf.name;
        node["ssl"]             = conf.ssl;
        node["cert_file"]       = conf.cert_file;
        node["key_file"]        = conf.key_file;
        node["accept_worker"]  = conf.accept_worker;
        node["io_worker"]       = conf.io_worker;
        node["process_worker"]  = conf.process_worker;
        node["args"]            = YAML::Load(LexicalCast<
                std::map<std::string, std::string>, std::string>()(conf.args));
        for(auto& i : conf.address)
        {
            node["address"].push_back(i);
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

/**
 * @brief   Tcp服务器封装
 */
class TcpServer : public std::enable_shared_from_this<TcpServer>
                  , Noncopyable
{
public:
    typedef std::shared_ptr<TcpServer> ptr;

    /**
     * @brief   
     *
     * @param   worker          好像没啥用，不知道是干嘛的
     * @param   io_worker       socket客户端工作的协程调度器
     * @param   accpet_worker   服务器socket执行接收socket连接的协程调度器
     */
    TcpServer(sylar::IOManager* worker = sylar::IOManager::GetThis()
            , sylar::IOManager* io_worker = sylar::IOManager::GetThis()
            , sylar::IOManager* accpet_worker = sylar::IOManager::GetThis());
    virtual ~TcpServer();

    virtual bool bind(sylar::Address::ptr addr, bool ssl = false);
    
    /**
     * @brief  绑定地址数组 
     */
    virtual bool bind(const std::vector<Address::ptr>& addrs
                    , std::vector<Address::ptr>& fails
                    , bool ssl = false);

    //bool loadCertificates(const std::string& cert_file, const std::string& key_file);

    /**
     * @brief  启动服务 
     */
    virtual bool start();
    virtual void stop();
    
    uint64_t getRecvTimeout() const { return m_recvTimeout; }
    void setRecvTimeout(uint64_t v) { m_recvTimeout = v; }

    /**
     * @brief  返回服务器名称 
     */
    std::string getName() const { return m_name; }
    virtual void setName(const std::string& v) { m_name = v; }

    bool isStop() const { return m_isStop; }

    TcpServerConf::ptr getConf() const { return m_conf; }
    void setConf(TcpServerConf::ptr v) { m_conf = v; }
    void setConf(const TcpServerConf& v);

    virtual std::string toString(const std::string& prefix = "");
    std::vector<Socket::ptr> getSocks() const { return m_socks; }

protected:
    /**
     * @brief  处理新连接的Socket类 
     */
    virtual void handleClient(Socket::ptr client);
    
    /**
     * @brief   开始接受连接
     */
    virtual void startAccept(Socket::ptr sock);

protected:
    /// 监听Socket数组
    std::vector<Socket::ptr> m_socks;
    /// 不知道有什么用
    IOManager* m_worker;
    /// 新连接的Socket工作的调度器
    IOManager* m_ioWorker;
    /// 服务器Socket接收连接的调度器
    IOManager* m_acceptWorker;
    /// 接收超时时间（毫秒）
    uint64_t m_recvTimeout;
    /// 服务器名称
    std::string m_name;
    /// 服务器类型
    std::string m_type = "tcp";
    /// 服务是否停止
    bool m_isStop;

    bool m_ssl = false;
    TcpServerConf::ptr m_conf;
};

}

#endif


#include "tcp_server.h"
#include "log.h"

namespace sylar
{
static sylar::ConfigVar<uint64_t>::ptr g_tcp_server_read_timeout = 
    sylar::Config::Lookup("tcp_server.read_timeout", (uint64_t)(60 * 1000 * 2),
            "tcp server read timeout");

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

TcpServer::TcpServer(sylar::IOManager* worker
        , sylar::IOManager* io_worker
        , sylar::IOManager* accept_worker)
    : m_worker(worker)
    , m_ioWorker(io_worker)
    , m_acceptWorker(accept_worker)
    , m_recvTimeout(g_tcp_server_read_timeout->getValue())
    , m_name("sylar/1.0.0")
    , m_isStop(true)
{
}

TcpServer::~TcpServer()
{
    for(auto& i : m_socks)
    {
        i->close();
    }
    m_socks.clear();
}

bool TcpServer::bind(sylar::Address::ptr addr, bool ssl)
{
    std::vector<Address::ptr> addrs;
    std::vector<Address::ptr> fails;
    addrs.push_back(addr);
    return bind(addrs, fails, ssl);
}

bool TcpServer::bind(const std::vector<Address::ptr>& addrs
                , std::vector<Address::ptr>& fails
                , bool ssl)
{
    m_ssl = ssl;
    for(auto& addr : addrs)
    {
        //Socket::ptr sock = ssl ? SSLSocket::CreateTCP(addr) : Socket::CreateTCP(addr);
        Socket::ptr sock = Socket::CreateTCP(addr);
        if(!sock->bind(addr))
        {
            SYLAR_LOG_ERROR(g_logger) << "bind fail errno="
                << errno << " errstr=" << strerror(errno)
                << " addr=(" << addr->toString() << "]";
            fails.push_back(addr);
            continue;
        }
        if(!sock->listen())
        {
            SYLAR_LOG_ERROR(g_logger) << "listen fail errno="
                << errno << " errstr=" << strerror(errno)
                << " addr=" << addr->toString() << "]";
            fails.push_back(addr);
            continue;
        }
        m_socks.push_back(sock);
    }

    if(!fails.empty())
    {
        m_socks.clear();
        return false;
    }

    for(auto& i : m_socks)
    {
        SYLAR_LOG_INFO(g_logger) << "type=" << m_type
            << " name=" << m_name
            << " ssl=" << m_ssl
            << " server bind success: " << *i;
    }
    return true;
}

/*bool TcpServer::loadCertificates(const std::string& cert_file, const std::string& key_file)
{
    for(auto& i : m_socks)
    {
        auto ssl_socket = std::dynamic_pointer_cast<SSLSocket>(i);
        if(ssl_socket)
        {
            if(!ssl_socket->loadCertificates(cert_file, key_file))
            {
                return false;
            }
        }
    }
    return true;
}*/

bool TcpServer::start()
{
    if(!m_isStop)
    {
        return true;
    }
    m_isStop = false;
    for(auto& sock : m_socks)
    {
        m_acceptWorker->schedule(std::bind(&TcpServer::startAccept,
                        shared_from_this(), sock));
    }
    return true;
}

void TcpServer::stop()
{
    m_isStop = true;

    // 这里传多一个self，应该为了引用计数加1，防止被过早析构掉
    auto self = shared_from_this();
    m_acceptWorker->schedule([this, self]() {
        for(auto& sock : m_socks)
        {
            sock->cancelAll();
            sock->close();
        }
        m_socks.clear();
    });
}

void TcpServer::setConf(const TcpServerConf& v)
{
    m_conf.reset(new TcpServerConf(v));
}

std::string TcpServer::toString(const std::string& prefix)
{
    std::stringstream ss;
    ss << prefix << "[type=" << m_type
       << " name=" << m_name << " ssl=" << m_ssl
       << " worker=" << (m_worker ? m_worker->getName() : "")
       << " accept=" << (m_acceptWorker ? m_acceptWorker->getName() : "")
       << " recv_timeout=" << m_recvTimeout << "]" << std::endl;
    std::string pfx = prefix.empty() ? "    " : prefix;
    for(auto& i : m_socks)
    {
        ss << pfx << pfx << *i << std::endl;
    }
    return ss.str();
}

void TcpServer::handleClient(Socket::ptr client)
{
    SYLAR_LOG_INFO(g_logger) << "handleClient: " << *client;    
}

void TcpServer::startAccept(Socket::ptr sock)
{
    while(!m_isStop)
    {
        Socket::ptr client = sock->accept();
        if(client)
        {
            client->setRecvTimeout(m_recvTimeout);
            m_ioWorker->schedule(std::bind(&TcpServer::handleClient,
                        shared_from_this(), client));
        }
        else
        {
            SYLAR_LOG_ERROR(g_logger) << "accept errno=" << errno
                << " errstr=" << strerror(errno);
        }
    }
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
