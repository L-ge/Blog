# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- HttpSession 的封装。


# HttpSession

- 继承自 SocketStream。
- 主要是接受 http 请求 recvRequest() 和 发送 http 响应 sendResponse() 两个方法。


# 部分相关代码
```C++
/**
 * @filename    http_session.h
 * @brief   HttpSession的封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-15
 */
#ifndef __SYLAR_HTTP_SESSION_H__
#define __SYLAR_HTTP_SESSION_H__

#include "sylar/streams/socket_stream.h"
#include "http.h"

namespace sylar
{

namespace http
{

class HttpSession : public SocketStream
{
public:
    typedef std::shared_ptr<HttpSession> ptr;

    /**
     * @brief   构造函数 
     *
     * @param   sock    Socket类型
     * @param   owner   是否托管
     */
    HttpSession(Socket::ptr sock, bool owner = true);

    /**
     * @brief   接受http请求
     */
    HttpRequest::ptr recvRequest();
    
    /**
     * @brief  发送http响应 
     *
     * @param   rsp http响应
     *
     * @return  >0 发送成功
     *          =0 对方关闭
     *          <0 Socket异常
     */
    int sendResponse(HttpResponse::ptr rsp);
};

}

}

#endif


#include "http_session.h"
#include "http_parser.h"

namespace sylar
{

namespace http
{

HttpSession::HttpSession(Socket::ptr sock, bool owner)
    : SocketStream(sock, owner)
{
}

HttpRequest::ptr HttpSession::recvRequest()
{
    HttpRequestParser::ptr parser(new HttpRequestParser);
    uint64_t buff_size = HttpRequestParser::GetHttpRequestBufferSize();
    std::shared_ptr<char> buffer(new char[buff_size], [](char* ptr){ delete[] ptr; });  // 自定义删除函数
    char* data = buffer.get();      // 虽然用了智能指针，但是使用的时候还是直接用裸指针
    int offset = 0;     // 偏移量，尚未解析完的数据
    // 先去理解parser->execute里面的memmove方法，再看这个do-while循环会比较容易理解
    do
    {
        int len = read(data + offset, buff_size - offset);  // 有offset个字节的数据尚未解析，因此偏移量是offset
        if(len <= 0)
        {
            close();
            return nullptr;
        }
        len += offset;                  // 加上上一次没解析完的数据，这次再一起解析
        size_t nparse = parser->execute(data, len); // 这里面有memmove的动作，所以是从data开始解析就行
        if(parser->hasError())
        {
            close();
            return nullptr;
        }
        offset = len - nparse;          // 尚未解析完成的数据
        if(offset == (int)buff_size)    // 尚未解析完的数据等于原始数据，相当于没解析成功一点东西，即失败
        {
            close();
            return nullptr;
        }
        if(parser->isFinished())
        {
            break;
        }
    } while(true);

    int64_t length = parser->getContentLength();    // 消息主体的长度(不包括http首部)
    if(length > 0)
    {
        std::string body;
        body.resize(length);
        
        int len = 0;
        if(length >= offset)
        {
            memcpy(&body[0], data, offset);
            len = offset;
        }
        else
        {
            memcpy(&body[0], data, length);
            len = length;
        }

        length -= offset;   // 减掉已经copy到body的长度
        if(length > 0)
        {
            if(readFixSize(&body[len], length) <= 0)
            {
                close();
                return nullptr;
            }
        }
        parser->getData()->setBody(body);
    }

    parser->getData()->init();  // 里面主要是初始化是否长连接
    return parser->getData();
}

int HttpSession::sendResponse(HttpResponse::ptr rsp)
{
    std::stringstream ss;
    ss << *rsp;             // 重载了<<运算符，输出的其实就是一个http报文了
    std::string data = ss.str();
    return writeFixSize(data.c_str(), data.size());
}

}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
