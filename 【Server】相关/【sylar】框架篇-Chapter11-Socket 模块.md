# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 套接字类，表示一个套接字对象。
- 其实就是在操作系统的 API 上再封装了一层。


# Socket

- 继承自 std::enable_shared_from_this<Socket> 和 sylar::Noncopyable。
- 含有以下属性：
	- 文件描述符 m_sock。
	- 地址类型（AF_INET，AF_INET6 等）m_family。
	- 套接字类型（SOCK_STREAM，SOCK_DGRAM 等）m_tyoe。
	- 协议类型（这项其实可以忽略）m_protocol。
	- 是否连接（针对 TCP 套接字；如果是 UDP 套接字，则默认已连接）m_isConnected。
	- 本地地址 m_localAddress。
	- 远端地址 m_remoteAddress。
- 含有如下方法：
	- 创建各种类型的套接字对象的方法（TCP 套接字，UDP 套接字，Unix 域套接字）。
	- 设置套接字选项，比如超时参数。
	- bind / connect / listen 方法，实现绑定地址、发起连接、发起监听功能。
	- accept 方法，返回连入的套接字对象。
	- 发送和接收数据的方法。
	- 获取本地地址和远端地址的方法。
	- 获取套接字类型、地址类型、协议类型的方法。
	- 取消套接字读、写事件的方法。


# SSLSocket

- 继承自 Socket。
- 基于 openssl。


# 其他说明

- UnixTCPSocket 在 bind 函数里面 connect。


# 部分相关代码
```C++
/**
 * @filename    socket.h
 * @brief   Socket 模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-09
 */
#ifndef __SYLAR_SOCKET_H__
#define __SYLAR_SOCKET_H__

#include <memory>
#include <netinet/tcp.h>
#include <sys/types.h>
#include <sys/socket.h>
//#include <openssl/err.h>
//#include <openssl/ssl.h>
#include "address.h"
#include "noncopyable.h"

namespace sylar
{

/**
 * @brief   Socket封装类
 */
class Socket : public std::enable_shared_from_this<Socket>, Noncopyable
{
public:
    typedef std::shared_ptr<Socket> ptr;
    typedef std::weak_ptr<Socket> weak_ptr;

    /**
     * @brief   Socket类型枚举
     */
    enum Type
    {
        TCP = SOCK_STREAM,
        UDP = SOCK_DGRAM,
    };

    /**
     * @brief   Socket协议簇枚举
     */
    enum Family
    {
        IPv4 = AF_INET,
        IPv6 = AF_INET6,
        UNIX = AF_UNIX,
    };

    static Socket::ptr CreateTCP(sylar::Address::ptr address);
    static Socket::ptr CreateUDP(sylar::Address::ptr address);
    static Socket::ptr CreateTCPSocket();
    static Socket::ptr CreateUDPSocket();
    static Socket::ptr CreateTCPSocket6();
    static Socket::ptr CreateUDPSocket6();
    static Socket::ptr CreateUnixTCPSocket();
    static Socket::ptr CreateUnixUDPSocket();

    Socket(int family, int type, int protocol = 0);
    virtual ~Socket();
    
    /**
     * @brief   设置发送超时时间(毫秒) 
     */
    void setSendTimeout(int64_t v);
    int64_t getSendTimeout();
    
    /**
     * @brief   设置接受超时时间(毫秒)
     */
    void setRecvTimeout(int64_t v);
    int64_t getRecvTimeout();
    
    bool getOption(int level, int option, void* result, socklen_t* len);

    template<class T>
    bool getOption(int level, int option, T& result)
    {
        socklen_t length = sizeof(T);
        return getOption(level, option, &result, &length);
    }

    bool setOption(int level, int option, const void* result, socklen_t len);

    template<class T>
    bool setOption(int level, int option, const T& value)
    {
        return setOption(level, option, &value, sizeof(T));
    }

    virtual Socket::ptr accept();
    virtual bool bind(const Address::ptr addr);
    virtual bool connect(const Address::ptr addr, uint64_t timeout_ms = -1);
    virtual bool reconnet(uint64_t timeout_ms = -1);
    virtual bool listen(int backlog = SOMAXCONN);
    virtual bool close();

    virtual int send(const void* buffer, size_t length, int flags = 0);
    virtual int send(const iovec* buffers, size_t length, int flags = 0);
    virtual int sendTo(const void* buffer, size_t length, const Address::ptr to, int flags = 0);
    virtual int sendTo(const iovec* buffers, size_t length, const Address::ptr to, int flags = 0);
    virtual int recv(void* buffer, size_t length, int flags = 0);
    virtual int recv(iovec* buffer, size_t length, int flags = 0);
    virtual int recvFrom(void* buffer, size_t length, Address::ptr from, int flags = 0);
    virtual int recvFrom(iovec* buffers, size_t length, Address::ptr from, int flags = 0);

    /**
     * @brief  获取远端地址 
     */
    Address::ptr getRemoteAddress();
    
    /**
     * @brief  获取本地地址 
     */
    Address::ptr getLocalAddress();
    
    int getFamily() const { return m_family; }
    int getType() const { return m_type; }
    int getProtocol() const { return m_protocol; }
    bool isConnected() const { return m_isConnected; }
    
    /**
     * @brief  是否有效（m_sock != -1）
     */
    bool isValid() const;
    int getError();

    /**
     * @brief  输出信息到流中 
     */
    virtual std::ostream& dump(std::ostream& os) const;
    virtual std::string toString() const;

    int getSocket() const { return m_sock; }
    bool cancelRead();
    bool cancelWrite();
    bool cancelAccept();
    bool cancelAll();

protected:
    void initSock();

    /**
     * @brief   创建socket
     */
    void newSock();
    virtual bool init(int sock);

protected:
    /// socket句柄
    int m_sock;
    /// 协议簇
    int m_family;
    /// 类型
    int m_type;
    /// 协议
    int m_protocol;
    /// 是否连接
    bool m_isConnected;
    /// 本地地址
    Address::ptr m_localAddress;
    /// 远端地址
    Address::ptr m_remoteAddress;
};

/*
class SSLSocket : public Socket
{
public:
    typedef std::shared_ptr<SSLSocket> ptr;

m_isConnected    static SSLSocket::ptr CreateTCP(sylar::Address::ptr address);
    static SSLSocket::ptr CreateTCPSocket();
    static SSLSocket::ptr CreateTCPSocket6();

    SSLSocket(int family, int type, int protocol = 0);
    virtual Socket::ptr accept() override;
    virtual bool bind(const Address::ptr addr) override;
    virtual bool connect(const Address::ptr addr, uint64_t timeout_ms = -1) override;
    virtual bool listen(int backlog = SOMAXCONN) override;
    virtual bool close() override;
    
    virtual int send(const void* buffer, size_t length, int flags = 0) override;
    virtual int send(const iovec* buffers, size_t length, int flags = 0) override;
    virtual int sendTo(const void* buffer, size_t length, const Address::ptr to, int flags = 0) override;
    virtual int sendTo(const iovec* buffers, size_t length, const Address::ptr to, int flags = 0) override;
    virtual int recv(void* buffer, size_t length, int flags = 0) override;
    virtual int recv(iovec* buffer, size_t length, int flags = 0) override;
    virtual int recvFrom(void* buffer, size_t length, Address::ptr from, int flags = 0) override;
    virtual int recvFrom(iovec* buffers, size_t length, Address::ptr from, int flags = 0) override;

    bool loadCertificates(const std::string& cert_file, const std::string& key_file);
    virtual std::ostream& dump(std::ostream& os) const override;

protected:
    virtual bool init(int sock) override;

private:
    std::shared_ptr<SSL_CTX> m_ctx;
    std::shared_ptr<SSL> m_ssl;
};
*/

/**
 * @brief   流式输出socket   
 *
 * @param   os      输出流
 * @param   sock    Socket类
 */
std::ostream& operator<<(std::ostream& os, const Socket& sock);

}


#endif


#include "socket.h"
#include "log.h"
#include "iomanager.h"
#include "fdmanager.h"
#include "hook.h"
#include "macro.h"
#include "util.h"

namespace sylar
{
static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");
    
Socket::ptr Socket::CreateTCP(sylar::Address::ptr address)
{
    Socket::ptr sock(new Socket(address->getFamily(), TCP, 0));
    return sock;
}

Socket::ptr Socket::CreateUDP(sylar::Address::ptr address)
{
    Socket::ptr sock(new Socket(address->getFamily(), UDP, 0));
    sock->newSock();
    sock->m_isConnected = true;     // UDP默认已连接
    return sock;
}

Socket::ptr Socket::CreateTCPSocket()
{
    Socket::ptr sock(new Socket(IPv4, TCP, 0));
    return sock;
}

Socket::ptr Socket::CreateUDPSocket()
{
    Socket::ptr sock(new Socket(IPv4, UDP, 0));
    sock->newSock();
    sock->m_isConnected = true;
    return sock;
}

Socket::ptr Socket::CreateTCPSocket6()
{
    Socket::ptr sock(new Socket(IPv6, TCP, 0));
    return sock;
}

Socket::ptr Socket::CreateUDPSocket6()
{
    Socket::ptr sock(new Socket(IPv6, UDP, 0));
    sock->newSock();
    sock->m_isConnected = true;
    return sock;
}

Socket::ptr Socket::CreateUnixTCPSocket()
{
    Socket::ptr sock(new Socket(UNIX, TCP, 0));
    return sock;
}

Socket::ptr Socket::CreateUnixUDPSocket()
{
    Socket::ptr sock(new Socket(UNIX, UDP, 0));
    return sock;
}

Socket::Socket(int family, int type, int protocol)
    : m_sock(-1)
    , m_family(family)
    , m_type(type)
    , m_protocol(protocol)
    , m_isConnected(false)
{

}

Socket::~Socket()
{
    close();
}

void Socket::setSendTimeout(int64_t v)
{
    struct timeval tv{int(v / 1000), int(v % 1000 * 1000)};
    setOption(SOL_SOCKET, SO_SNDTIMEO, tv);
}

int64_t Socket::getSendTimeout()
{
    FdCtx::ptr ctx = FdMgr::GetInstance()->get(m_sock);
    if(ctx)
    {
        return ctx->getTimeout(SO_SNDTIMEO);
    }
    return -1;
}

void Socket::setRecvTimeout(int64_t v)
{
    struct timeval tv{int(v / 1000), int(v % 1000 * 1000)};
    setOption(SOL_SOCKET, SO_RCVTIMEO, tv);
}

int64_t Socket::getRecvTimeout()
{
    FdCtx::ptr ctx = FdMgr::GetInstance()->get(m_sock);
    if(ctx)
    {
        return ctx->getTimeout(SO_RCVTIMEO);
    }
    return -1;
}
    
bool Socket::getOption(int level, int option, void* result, socklen_t* len)
{
    int rt = getsockopt(m_sock, level, option, result, (socklen_t*)len);
    if(rt)
    {
        SYLAR_LOG_DEBUG(g_logger) << "getOption sock=" << m_sock
            << " level=" << level << " option=" << option
            << " errno=" << errno << " errstr=" << strerror(errno);
        return false;
    }
    return true;
}

bool Socket::setOption(int level, int option, const void* result, socklen_t len)
{
    int rt = setsockopt(m_sock, level, option, result, (socklen_t)len);
    if(rt)
    {
        SYLAR_LOG_DEBUG(g_logger) << "setOption sock=" << m_sock
            << " level=" << level << " option=" << option
            << " errno=" << errno << " errstr=" << strerror(errno);
        return false;
    }
    return true;
}

Socket::ptr Socket::accept()
{
    Socket::ptr sock(new Socket(m_family, m_type, m_protocol));
    int newsock = ::accept(m_sock, nullptr, nullptr);
    if(newsock == -1)
    {
        SYLAR_LOG_ERROR(g_logger) << "accept(" << m_sock << ") errno="
            << errno << " errstr=" << strerror(errno);
        return nullptr;
    }
    if(sock->init(newsock))
    {
        return sock;
    }
    return nullptr;
}

bool Socket::bind(const Address::ptr addr)
{
    if(!isValid())
    {
        newSock();
        if(SYLAR_UNLIKELY(!isValid()))
        {
            return false;
        }
    }

    if(SYLAR_UNLIKELY(addr->getFamily() != m_family))
    {
        SYLAR_LOG_ERROR(g_logger) << "bind sock.family("
            << m_family << ") addr.family(" << addr->getFamily()
            << ") not equal, addr=" << addr->toString();
        return false;
    }

    UnixAddress::ptr uaddr = std::dynamic_pointer_cast<UnixAddress>(addr);
    if(uaddr)
    {
        Socket::ptr sock = Socket::CreateUnixTCPSocket();
        if(sock->connect(uaddr))
        {
            return false;
        }
        else
        {
            // connect失败，则对该文件解除链接
            sylar::FSUtil::Unlink(uaddr->getPath(), true);
        }
    }

    if(::bind(m_sock, addr->getAddr(), addr->getAddrLen()))
    {
        SYLAR_LOG_ERROR(g_logger) << "bind error errno=" << errno
            << " errstr=" << strerror(errno);
        return false;
    }
    getLocalAddress();
    return true;
}

bool Socket::connect(const Address::ptr addr, uint64_t timeout_ms)
{
    m_remoteAddress = addr;     // 设置远端地址
    if(!isValid())
    {
        newSock();
        if(SYLAR_UNLIKELY(!isValid()))
        {
            return false;
        }
    }

    if(SYLAR_UNLIKELY(addr->getFamily() != m_family))
    {
        SYLAR_LOG_ERROR(g_logger) << "connect sock.family("
            << m_family << ") addr.family(" << addr->getFamily()
            << ") not equal, addr=" << addr->toString();
        return false;
    }

    if(timeout_ms == (uint64_t)-1)
    {
        if(::connect(m_sock, addr->getAddr(), addr->getAddrLen()))
        {
            SYLAR_LOG_ERROR(g_logger) << "sock=" << m_sock << " connect(" << addr->toString() 
                << ") error errno=" << errno << " errstr=" << strerror(errno);
            close();
            return false;
        }
    }
    else
    {
        if(::connect_with_timeout(m_sock, addr->getAddr(), addr->getAddrLen(), timeout_ms))
        {
            SYLAR_LOG_ERROR(g_logger) << "sock=" << m_sock << " connect(" << addr->toString() 
                << ") timeout=" << timeout_ms << ") error errno=" 
                << errno << " errstr=" << strerror(errno);
            close();
            return false;
        }
    }

    m_isConnected = true;
    getRemoteAddress();
    getLocalAddress();
    return true;
}

bool Socket::reconnet(uint64_t timeout_ms)
{
    if(!m_remoteAddress)
    {
        SYLAR_LOG_ERROR(g_logger) << "reconnect m_remoteAddress is null";
        return false;
    }
    m_localAddress.reset();
    return connect(m_remoteAddress, timeout_ms);
}

bool Socket::listen(int backlog)
{
    if(!isValid())
    {
        SYLAR_LOG_ERROR(g_logger) << "listen error sock=-1";
        return false;
    }
    if(::listen(m_sock, backlog))
    {
        SYLAR_LOG_ERROR(g_logger) << "listen error errno=" << errno
            << " errstr=" << strerror(errno);
        return false;
    }
    return true;
}

bool Socket::close()
{
    if(!m_isConnected && m_sock == -1)
    {
        return true;
    }
    m_isConnected = false;
    if(m_sock != -1)
    {
        ::close(m_sock);
        m_sock = -1;
    }
    return false;
}

int Socket::send(const void* buffer, size_t length, int flags)
{
    if(isConnected())
    {
        return ::send(m_sock, buffer, length, flags);
    }
    return -1;
}

int Socket::send(const iovec* buffers, size_t length, int flags)
{
    if(isConnected())
    {
        msghdr msg;
        memset(&msg, 0, sizeof(msg));
        msg.msg_iov = (iovec*)buffers;
        msg.msg_iovlen = length;
        return ::sendmsg(m_sock, &msg, flags);
    }
    return -1;
}

int Socket::sendTo(const void* buffer, size_t length, const Address::ptr to, int flags)
{
    if(isConnected())
    {
        return ::sendto(m_sock, buffer, length, flags, to->getAddr(), to->getAddrLen());
    }
    return -1;
}

int Socket::sendTo(const iovec* buffers, size_t length, const Address::ptr to, int flags)
{
    if(isConnected())
    {
        msghdr msg;
        memset(&msg, 0, sizeof(msg));
        msg.msg_iov = (iovec*)buffers;
        msg.msg_iovlen = length;
        msg.msg_name = to->getAddr();
        msg.msg_namelen = to->getAddrLen();
        return ::sendmsg(m_sock, &msg, flags);
    }
    return -1;
}

int Socket::recv(void* buffer, size_t length, int flags)
{
    if(isConnected())
    {
        return ::recv(m_sock, buffer, length, flags);
    }
    return -1;
}

int Socket::recv(iovec* buffer, size_t length, int flags)
{
    if(isConnected())
    {
        msghdr msg;
        memset(&msg, 0, sizeof(msg));
        msg.msg_iov = (iovec*)buffer;
        msg.msg_iovlen = length;
        return ::recvmsg(m_sock, &msg, flags);
    }
    return -1;
}

int Socket::recvFrom(void* buffer, size_t length, Address::ptr from, int flags)
{
    if(isConnected())
    {
        socklen_t len = from->getAddrLen();
        return ::recvfrom(m_sock, buffer, length, flags, from->getAddr(), &len);
    }
    return -1;
}

int Socket::recvFrom(iovec* buffers, size_t length, Address::ptr from, int flags)
{
    if(isConnected())
    {
        msghdr msg;
        memset(&msg, 0, sizeof(msg));
        msg.msg_iov = (iovec*)buffers;
        msg.msg_iovlen = length;
        msg.msg_name = from->getAddr();
        msg.msg_namelen = from->getAddrLen();
        return ::sendmsg(m_sock, &msg, flags);
    }
    return -1;
}

Address::ptr Socket::getRemoteAddress()
{
    if(m_remoteAddress)
    {
        return m_remoteAddress;
    }

    Address::ptr result;
    switch(m_family)
    {
        case AF_INET:
            result.reset(new IPv4Address());
            break;
        case AF_INET6:
            result.reset(new IPv6Address());
            break;
        case AF_UNIX:
            result.reset(new UnixAddress());
            break;
        default:
            result.reset(new UnknownAddress(m_family));
            break;
    }

    socklen_t addrlen = result->getAddrLen();
    // 获取与m_sock连接的对端的地址信息
    if(getpeername(m_sock, result->getAddr(), &addrlen))
    {
        return Address::ptr(new UnknownAddress(m_family));
    }

    if(m_family == AF_UNIX)
    {
        UnixAddress::ptr addr = std::dynamic_pointer_cast<UnixAddress>(result);
        addr->setAddrLen(addrlen);  // 重新设置一下addrlen
    }

    m_remoteAddress = result;
    return m_remoteAddress;
}

Address::ptr Socket::getLocalAddress()
{
    if(m_localAddress)
    {
        return m_localAddress;
    }

    Address::ptr result;
    switch(m_family)
    {
        case AF_INET:
            result.reset(new IPv4Address());
            break;
        case AF_INET6:
            result.reset(new IPv6Address());
            break;
        case AF_UNIX:
            result.reset(new UnixAddress());
            break;
        default:
            result.reset(new UnknownAddress(m_family));
            break;
    }

    socklen_t addrlen = result->getAddrLen();
    // 获取m_sock当前关联的地址
    if(getsockname(m_sock, result->getAddr(), &addrlen))
    {
        SYLAR_LOG_ERROR(g_logger) << "getsockname error sock=" << m_sock
            << " errno=" << errno << " errstr=" << strerror(errno);
        return Address::ptr(new UnknownAddress(m_family));
    }

    if(m_family == AF_UNIX)
    {
        UnixAddress::ptr addr = std::dynamic_pointer_cast<UnixAddress>(result);
        addr->setAddrLen(addrlen);
    }

    m_localAddress = result;
    return m_localAddress;
}
    
bool Socket::isValid() const
{
    return m_sock != -1;
}

int Socket::getError()
{
    int error = 0;
    socklen_t len = sizeof(error);
    if(!getOption(SOL_SOCKET, SO_ERROR, &error, &len))
    {
        error = errno;
    }
    return error;
}

std::ostream& Socket::dump(std::ostream& os) const
{
    os << "[Socket sock=" << m_sock
       << " is_connected=" << m_isConnected
       << " family=" << m_family
       << " type=" << m_type
       << " protocol=" << m_protocol;

    if(m_localAddress)
    {
        os << " local_address=" << m_localAddress->toString();
    }
    if(m_remoteAddress)
    {
        os << " remote_address=" << m_remoteAddress->toString();
    }
    os << "]";
    return os;
}

std::string Socket::toString() const
{
    std::stringstream ss;
    dump(ss);
    return ss.str();
}

bool Socket::cancelRead()
{
    return IOManager::GetThis()->cancelEvent(m_sock, sylar::IOManager::READ);
}

bool Socket::cancelWrite()
{  
    return IOManager::GetThis()->cancelEvent(m_sock, sylar::IOManager::WRITE);
}

bool Socket::cancelAccept()
{
    // Accept也即读事件
    return IOManager::GetThis()->cancelEvent(m_sock, sylar::IOManager::READ);
}

bool Socket::cancelAll()
{
    return IOManager::GetThis()->cancelAll(m_sock);
}

void Socket::initSock()
{
    int val = 1;
    setOption(SOL_SOCKET, SO_REUSEADDR, val);      // 设置端口复用属性
    if(m_type == SOCK_STREAM)
    {
        setOption(IPPROTO_TCP, TCP_NODELAY, val);  // TCP连接一般都是设置nodelay属性
    }
}

void Socket::newSock()
{
    m_sock = socket(m_family, m_type, m_protocol);
    if(SYLAR_LIKELY(m_sock != -1))
    {
        initSock();
    }
    else
    {
        SYLAR_LOG_ERROR(g_logger) << "socket(" << m_family
            << ", " << m_type << ", " << m_protocol << ") errno="
            << errno << " errstr=" << strerror(errno);
    }
}

bool Socket::init(int sock)
{
    FdCtx::ptr ctx = FdMgr::GetInstance()->get(sock);
    if(ctx && ctx->isSocket() && !ctx->isClose())
    {
        m_sock = sock;
        m_isConnected = true;
        initSock();
        getLocalAddress();
        getRemoteAddress();
        return true;
    }
    return false;
}
/*
namespace
{

struct _SSLInit
{
    _SSLInit()
    {
        SSL_library_init();
        SSL_load_error_strings();
        OpenSSL_add_all_algorithms();
    }
};

static _SSLInit s_init;

}

SSLSocket::ptr SSLSocket::CreateTCP(sylar::Address::ptr address)
{
    SSLSocket::ptr sock(new SSLSocket(address->getFamily(), TCP, 0));
    return sock;
}

SSLSocket::ptr SSLSocket::CreateTCPSocket()
{
    SSLSocket::ptr sock(new SSLSocket(IPv4, TCP, 0));
    return sock;
}

SSLSocket::ptr SSLSocket::CreateTCPSocket6()
{
    SSLSocket::ptr sock(new SSLSocket(IPv6, TCP, 0));
    return sock;
}

SSLSocket::SSLSocket(int family, int type, int protocol)
    : Socket(family, type, protocol)
{

}
    
Socket::ptr SSLSocket::accept()
{
    SSLSocket::ptr sock(new SSLSocket(m_family, m_type, m_protocol));
    int newsock = ::accept(m_sock, nullptr, nullptr);
    if(newsock == -1)
    {
        SYLAR_LOG_ERROR(g_logger) << "accept(" << m_sock << ") errno="
            << errno << " errstr=" << strerror(errno);
        return nullptr;
    }
    sock->m_ctx = m_ctx;
    if(sock->init(newsock))
    {
        return sock;
    }
    return nullptr;
}

bool SSLSocket::bind(const Address::ptr addr)
{
    return Socket::bind(addr);
}

bool SSLSocket::connect(const Address::ptr addr, uint64_t timeout_ms)
{
    bool v = Socket::connect(addr, timeout_ms);
    if(v)
    {
        m_ctx.reset(SSL_CTX_new(SSLv23_client_method()), SSL_CTX_free);
        m_ssl.reset(SSL_new(m_ctx.get()), SSL_free);
        SSL_set_fd(m_ssl.get(), m_sock);
        v = (SSL_connect(m_ssl.get()) == 1);
    }
    return v;
}

bool SSLSocket::listen(int backlog)
{
    return Socket::listen(backlog);
}

bool SSLSocket::close()
{
    return Socket::close();
}

int SSLSocket::send(const void* buffer, size_t length, int flags)
{
    if(m_ssl)
    {
        return SSL_write(m_ssl.get(), buffer, length);
    }
    return -1;
}

int SSLSocket::send(const iovec* buffers, size_t length, int flags)
{
    if(!m_ssl)
    {
        return -1;
    }
    
    int total = 0;
    for(size_t i=0; i<length; ++i)
    {
        int tmp = SSL_write(m_ssl.get(), buffers[i].iov_base, buffers[i].iov_len);
        if(tmp <= 0)
        {
            return tmp;
        }
        total += tmp;
        if(tmp != (int)buffers[i].iov_len)
        {
            break;
        }
    }
    return total;
}

int SSLSocket::sendTo(const void* buffer, size_t length, const Address::ptr to, int flags)
{
    SYLAR_ASSERT(false);
    return -1;
}

int SSLSocket::sendTo(const iovec* buffers, size_t length, const Address::ptr to, int flags)
{
    SYLAR_ASSERT(false);
    return -1;
}

int SSLSocket::recv(void* buffer, size_t length, int flags)
{
    if(m_ssl)
    {
        return SSL_read(m_ssl.get(), buffer, length);
    }

    return -1;
}

int SSLSocket::recv(iovec* buffer, size_t length, int flags)
{
    if(!m_ssl)
    {
        return -1;
    }
    
    int total = 0;
    for(size_t i=0; i<length; ++i)
    {
        int tmp = SSL_read(m_ssl.get(), buffer[i].iov_base, buffer[i].iov_len);
        if(tmp <= 0)
        {
            return tmp;
        }
        total += tmp;
        if(tmp != (int)buffer[i].iov_len)
        {
            break;
        }
    }
    return total;
}

int SSLSocket::recvFrom(void* buffer, size_t length, Address::ptr from, int flags)
{
    SYLAR_ASSERT(false);
    return -1;
}

int SSLSocket::recvFrom(iovec* buffers, size_t length, Address::ptr from, int flags)
{
    SYLAR_ASSERT(false);
    return -1;
}

bool SSLSocket::loadCertificates(const std::string& cert_file, const std::string& key_file)
{
    m_ctx.reset(SSL_CTX_new(SSLv23_server_method()), SSL_CTX_free);
    if(SSL_CTX_use_certificate_chain_file(m_ctx.get(), cert_file.c_str()) != 1)
    {
        SYLAR_LOG_ERROR(g_logger) << "SSL_CTX_use_certificate_chain_file("
            << cert_file << ") error";
        return false;
    }

    if(SSL_CTX_use_PrivateKey_file(m_ctx.get(), key_file.c_str(), SSL_FILETYPE_PEM) != 1)
    {
        SYLAR_LOG_ERROR(g_logger) << "SSL_CTX_use_PrivateKey_file("
            << key_file << ") error";
        return false;
    }

    if(SSL_CTX_check_private_key(m_ctx.get()) != 1)
    {
        SYLAR_LOG_ERROR(g_logger) << "SSL_CTX_check_private_key cert_file="
            << cert_file << " key_file=" << key_file;
        return false;
    }
    return true;
}

std::ostream& SSLSocket::dump(std::ostream& os) const
{
    os << "[SSLSocket sock=" << m_sock
       << " is_connected=" << m_isConnected
       << " family=" << m_family
       << " type=" << m_type
       << " protocol=" << m_protocol;

    if(m_localAddress)
    {
        os << " local_address=" << m_localAddress->toString();
    }
    if(m_remoteAddress)
    {
        os << " remote_address=" << m_remoteAddress->toString();
    }
    os << "]";
    return os;
}

bool SSLSocket::init(int sock)
{
    bool v = Socket::init(sock);
    if(v)
    {
        m_ssl.reset(SSL_new(m_ctx.get()), SSL_free);
        SSL_set_fd(m_ssl.get(), m_sock);
        v = (SSL_accept(m_ssl.get()) == 1);
    }
    return v;
}
*/
std::ostream& operator<<(std::ostream& os, const Socket& sock)
{
    return sock.dump(os);
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
