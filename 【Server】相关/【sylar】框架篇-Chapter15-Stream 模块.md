# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 流结构，提供字节流读写接口。
- 所有的流结构都继承自抽象类 Stream，Stream 类规定了一个流必须具备 read / write 接口和 readFixSize / writeFixSize 接口，继承自 Stream 的类必须实现这些接口。


# Stream

- 流接口的封装类。
- 虚基类。
- 含有纯虚方法 read / write 和 readFixSize / writeFixSize。


# SocketStream

- Socket 流式接口的封装。
- 继承自 Stream。
- 套接字流结构，将套接字封装成流结构，以支持 Stream 接口规范。
- 支持套接字关闭操作以及获取本地/远端地址的操作。


# 部分相关代码
```C++
/**
 * @filename    stream.h
 * @brief   流接口的封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-14
 */
#ifndef __SYLAR_STREAM_H__
#define __SYLAR_STREAM_H__

#include <memory>
#include "bytearray.h"

namespace sylar
{

class Stream
{
public:
    typedef std::shared_ptr<Stream> ptr;

    virtual ~Stream() {}

    /**
     * @brief   读数据
     *
     * @param   buffer  接收数据的内存
     * @param   length  接收数据的内存大小
     *
     * @return  返回接收到的数据的实际大小
     *      @retval >0  返回接收到的数据的实际大小 
     *      @retval =0  被关闭
     *      @retval <0  出现流错误
     */
    virtual int read(void* buffer, size_t length) = 0;
    
    /**
     * @brief  读数据 
     *
     * @param   ba  接收数据的ByteArray
     */
    virtual int read(ByteArray::ptr ba, size_t length) = 0;
    
    /**
     * @brief   读固定长度的数据
     */
    virtual int readFixSize(void* buffer, size_t length);
    virtual int readFixSize(ByteArray::ptr ba, size_t length);

    virtual int write(const void* buffer, size_t length) = 0;
    virtual int write(ByteArray::ptr ba, size_t length) = 0; 
    virtual int writeFixSize(const void* buffer, size_t length);
    virtual int wirteFixSize(ByteArray::ptr ba, size_t length);

    /**
     * @brief   关闭流
     */
    virtual void close() = 0;
};

}


#endif

// stream.cc
#include "stream.h"

namespace sylar
{

int Stream::readFixSize(void* buffer, size_t length)
{
    size_t offset = 0;
    int64_t left = length;
    while(left > 0)
    {
        int64_t len = read((char*)buffer + offset, left);
        if(len <= 0)
        {
            return len;
        }
        offset += len;
        left -= len;
    }
    return length;
}

int Stream::readFixSize(ByteArray::ptr ba, size_t length)
{
    int64_t left = length;
    while(left > 0)
    {
        // 会从ba拿到iovec，不用关注偏移量
        int64_t len = read(ba, left);
        if(len <= 0)
        {
            return len;
        }
        left -= len;
    }
    return length;
}

int Stream::writeFixSize(const void* buffer, size_t length)
{
    size_t offset = 0;
    int64_t left = length;
    while(left > 0)
    {
        int64_t len = write((const char*)buffer + offset, left);
        if(len <= 0)
        {
            return len;
        }
        offset += len;
        left -= len;
    }
    return length;
}

int Stream::wirteFixSize(ByteArray::ptr ba, size_t length)
{
    int64_t left = length;
    while(left > 0)
    {
        int64_t len = write(ba, left);
        if(len <= 0)
        {
            return len;
        }
        left -= len;
    }
    return length;
}

}


/**
 * @filename    socket_stream.h
 * @brief   Socket流式接口的封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-14
 */
#ifndef __SYLAR_SOCKET_STREAM_H__
#define __SYLAR_SOCKET_STREAM_H__

#include "sylar/stream.h"
#include "sylar/socket.h"
#include "sylar/mutex.h"
#include "sylar/iomanager.h"

namespace sylar
{

class SocketStream : public Stream
{
public:
    typedef std::shared_ptr<SocketStream> ptr;

    /**
     * @brief   构造函数
     *
     * @param   sock    Socket类
     * @param   owner   是否完全控制
     */
    SocketStream(Socket::ptr sock, bool owner = true);
    
    /**
     * @brief   析构函数（如果m_owner=true,则close）
     */
    ~SocketStream();

    virtual int read(void* buffer, size_t length) override;
    virtual int read(ByteArray::ptr ba, size_t length) override;

    virtual int write(const void* buffer, size_t length) override;
    virtual int write(ByteArray::ptr ba, size_t length) override;

    virtual void close() override;

    Socket::ptr getSocket() const { return m_socket; }
    bool isConnected() const;

    Address::ptr getRemoteAddress();
    Address::ptr getLocalAddress();
    std::string getRemoteAddressString();
    std::string getLocaloAddressString();

protected:
    /// Socket类
    Socket::ptr m_socket;
    /// 是否主控(主要是是否交由该类来控制关闭)
    bool m_owner;
};

}

#endif

// socket_stream.cc
#include "socket_stream.h"

namespace sylar
{

SocketStream::SocketStream(Socket::ptr sock, bool owner)
    : m_socket(sock)
    , m_owner(owner)
{
}

SocketStream::~SocketStream()
{
    if(m_owner && m_socket)
    {
        m_socket->close();
    }
}

int SocketStream::read(void* buffer, size_t length)
{
    if(!isConnected())
    {
        return -1;
    }
    return m_socket->recv(buffer, length);
}

int SocketStream::read(ByteArray::ptr ba, size_t length)
{
    if(!isConnected())
    {
        return -1;
    }
    
    std::vector<iovec> iovs;
    ba->getWriteBuffers(iovs, length);      // 拿到iovec写缓存
    int rt = m_socket->recv(&iovs[0], iovs.size());
    if(rt > 0)
    {
        ba->setPosition(ba->getPosition() + rt);    // 更新当前操作位置
    }
    return rt;
}

int SocketStream::write(const void* buffer, size_t length)
{
    if(!isConnected())
    {
        return -1;
    }
    return m_socket->send(buffer, length);
}

int SocketStream::write(ByteArray::ptr ba, size_t length)
{
    if(!isConnected())
    {
        return -1;
    }

    std::vector<iovec> iovs;
    ba->getReadBuffers(iovs, length);       // 拿到iovec读缓存
    int rt = m_socket->send(&iovs[0], iovs.size());
    if(rt > 0)
    {
        ba->setPosition(ba->getPosition() + rt);    // 更新当前操作位置
    }
    return rt;
}

void SocketStream::close()
{
    if(m_socket)
    {
        m_socket->close();
    }
}

bool SocketStream::isConnected() const
{
    return m_socket && m_socket->isConnected();
}

Address::ptr SocketStream::getRemoteAddress()
{
    if(m_socket)
    {
        return m_socket->getRemoteAddress();
    }
    return nullptr;
}

Address::ptr SocketStream::getLocalAddress()
{
    if(m_socket)
    {
        return m_socket->getLocalAddress();
    }
    return nullptr;
}

std::string SocketStream::getRemoteAddressString()
{
    auto addr = getRemoteAddress();
    if(addr)
    {
        return addr->toString();
    }
    return "";
}

std::string SocketStream::getLocaloAddressString()
{
    auto addr = getLocalAddress();
    if(addr)
    {
        return addr->toString();
    }
    return "";
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
