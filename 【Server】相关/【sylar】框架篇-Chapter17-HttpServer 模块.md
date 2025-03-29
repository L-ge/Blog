# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- Http 服务器的封装。


# HttpServer

- HttpServer 封装类。
- 继承自 TcpServer。
- 内含 Servlet 分发器（可设置、可获取）。
- 重写了 TcpServer 的 handleClient 方法。


# 接收请求和发送响应的数据流转

```C++
new sylar::http::HttpServer
		↓
HttpServer::bind (socket bind 和 listen)
		↓
HttpServer::start (放入调度器 accept)
		↓
收到请求：HttpServer::handleClient，创建 HttpSession
		↓
在HttpSession::recvRequest() 读取报文(其实是从 SocketStream 读的)
		↓
用 HttpRequestParser 解析报文并组装得到 HttpRequest
		↓
创建 HttpResponse，并把 HttpRequest、HttpResponse、HttpSession 一并交给 ServletDispatch 去处理

ServletDispatch 根据HttpRequest 的 path 匹配最佳的 Servlet 进行处理
		↓
Servlet 的处理是设置 HttpResponse 的一些内容(也就是响应报文))
		↓
ServletDispatch 处理完，HttpSession 将 HttpResponse 发出去(数据传给 Stream- > SocketStream，最后通过 socket send 出去，socket 的 send 是 hook 过的)
```


# 部分相关代码
```C++
/**
 * @filename    http_server.h
 * @brief   Http服务器的封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-16
 */
#ifndef __SYLAR_HTTP_HTTP_SERVER_H
#define __SYLAR_HTTP_HTTP_SERVER_H

#include "sylar/tcp_server.h"
#include "http_session.h"
#include "servlet.h"

namespace sylar
{

namespace http
{

class HttpServer : public TcpServer
{
public:
    typedef std::shared_ptr<HttpServer> ptr;

    /**
     * @brief   构造函数
     *
     * @param   keepalive   是否长连接
     * @param   worker      工作调度器
     * @param   io_worker
     * @param   accept_worker   接收连接的调度器
     */
    HttpServer(bool keepalive = false
             , sylar::IOManager* worker = sylar::IOManager::GetThis()
             , sylar::IOManager* io_worker = sylar::IOManager::GetThis()
             , sylar::IOManager* accept_worker = sylar::IOManager::GetThis());

    /**
     * @brief   获取ServletDispatch
     */
    ServletDispatch::ptr getServletDispatch() const { return m_dispatch; }

    /**
     * @brief   设置ServletDispatch 
     */
    void setServletDispatch(ServletDispatch::ptr v) { m_dispatch = v; }

    virtual void setName(const std::string& v) override;

protected:
    virtual void handleClient(Socket::ptr client) override;

private:
    /// 是否支持长连接
    bool m_isKeepalive;
    /// Servlet分发器
    ServletDispatch::ptr m_dispatch;
};

}

}

#endif


#include "http_server.h"
#include "sylar/log.h"
//#include "sylar/http/servlets/config_servlet.h"
//#include "sylar/http/servlets/status_servlet.h"

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

namespace http
{

HttpServer::HttpServer(bool keepalive
                     , sylar::IOManager* worker
                     , sylar::IOManager* io_worker
                     , sylar::IOManager* accept_worker)
    : TcpServer(worker, io_worker, accept_worker)
    , m_isKeepalive(keepalive)
{
    m_dispatch.reset(new ServletDispatch);

    m_type = "http";
    //m_dispatch->addServlet("/_/status", Servlet::ptr(new StatusServlet));
    //m_dispatch->addServlet("/_/config", Servlet::ptr(new ConfigServlet));
}

void HttpServer::setName(const std::string& v)
{
    TcpServer::setName(v);
    m_dispatch->setDefault(std::make_shared<NotFoundServlet>(v));
}

void HttpServer::handleClient(Socket::ptr client)
{
    SYLAR_LOG_DEBUG(g_logger) << "handleClient " << *client;
    HttpSession::ptr session(new HttpSession(client));
    do
    {
        auto req = session->recvRequest();
        if(!req)
        {
            SYLAR_LOG_DEBUG(g_logger) << "recv http request fail, errno="
                << errno << " errstr=" << strerror(errno)
                << " client:" << *client << " keep_alive=" << m_isKeepalive;
            break;
        }

        HttpResponse::ptr rsp(new HttpResponse(req->getVersion()
                    , req->isClose() || !m_isKeepalive));
        rsp->setHeader("Server", getName());
        m_dispatch->handle(req, rsp, session);  // 交给ServletDispatch处理
        session->sendResponse(rsp);

        if(!m_isKeepalive || req->isClose())
        {
            break;
        }
    } while(true);

    session->close();
}

}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
