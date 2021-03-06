# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- Http 客户端的封装。


# http_connection.h

- HttpResult
	- 内含枚举类——错误码定义 Error。
	- 对响应结果的封装。
- HttpConnection
	- 继承自 SocketStream。
- HttpConnectionPool
	- HttpConnectionPool 针对的是长连接的场景。
	- 留意里面的 ReleasePtr 方法，算是一种优化。


# uri.h 和 uri.rl

- url 是 uri 的一种实例。
- uri 定义的文档参考如下：
	`https://www.ietf.org/rfc/rfc3986.txt`
- uri.rl 仿照上面的文档写的，最后的 uri 用 ragel 进行解析。
- 在 uri.rl 中，在那些解析部分，其实就是通过像下面这样去解析并赋值到 uri 对象中。
	```
	action save_scheme
	{
		uri->setScheme(std::string(mark, fpc - mark));
		mark = NULL;
	}
	```
- 命令去生成 .cc 文件：
	```
	$ ragel -G2 -C uri.rl -o uri.cc
	mv uri.cc uri.rl.cc
	```


# 部分相关代码
```C++
/**
 * @filename    http_connection.h
 * @brief   http客户端类
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-18
 */
#ifndef __SYLAR_HTTP_CONNECTION_H__
#define __SYLAR_HTTP_CONNECTION_H__

#include <list>
#include "sylar/streams/socket_stream.h"
#include "http.h"
#include "sylar/uri.h"
#include "sylar/mutex.h"

namespace sylar
{

namespace http
{

/**
 * @brief   http响应结果
 */
struct HttpResult 
{
    typedef std::shared_ptr<HttpResult> ptr;
   
    /**
     * @brief   错误码定义
     */
    enum class Error 
    {
        /// 正常
        OK                      = 0,
        /// 非法URL
        INVALID_URL             = 1,
        /// 无法解析HOST
        INVALID_HOST            = 2,
        /// 连接失败
        CONNECT_FAIL            = 3,
        /// 连接被对端关闭
        SEND_CLOSE_BY_PEER      = 4,
        /// 发送请求产生Socket错误
        SEND_SOCKET_ERROR       = 5,
        /// 超时
        TIMEOUT                 = 6,
        /// 创建Socket失败
        CREATE_SOCKET_ERROR     = 7,
        /// 从连接池中取连接失败
        POOL_GET_CONNECTION     = 8,
        /// 无效的连接
        POOL_INVALID_CONNECTION = 9,
    };

    /**
     * @brief   构造函数  
     *
     * @param   _result 错误码
     * @param   _response   http响应结构体
     * @param   _error      错误描述
     */
    HttpResult(int _result
             , HttpResponse::ptr _response
             , const std::string& _error)
        : result(_result)
        , response(_response)
        , error(_error)
    {}

    std::string toString() const;
    
    /// 错误码
    int result;
    /// HTTP响应结构体
    HttpResponse::ptr response;
    /// 错误描述
    std::string error;
};

class HttpConnectionPool;

/**
 * @brief   http客户端类
 */
class HttpConnection : public SocketStream
{

friend class HttpConnectionPool;

public:
    typedef std::shared_ptr<HttpConnection> ptr;

    static HttpResult::ptr DoGet(const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    static HttpResult::ptr DoGet(Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");
    
    static HttpResult::ptr DoPost(const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    static HttpResult::ptr DoPost(Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    /**
     * @brief   发送http请求
     *
     * @param   method      请求类型
     * @param   url         请求的url
     * @param   timeout_ms  超时时间(毫秒)
     * @param   headers     http请求头部参数
     * @param   body        请求消息体
     *
     * @return  返回http结果结构体
     */
    static HttpResult::ptr DoRequest(HttpMethod method
                            , const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    /**
     * @brief   发送http请求
     *
     * @param   method      请求类型
     * @param   uri         URI 结构体
     * @param   timeout_ms  超时时间(毫秒)
     * @param   headers     http请求头部参数
     * @param   body        请求消息体
     *
     * @return  返回http结果结构体
     */
    static HttpResult::ptr DoRequest(HttpMethod method
                            , Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    static HttpResult::ptr DoRequest(HttpRequest::ptr req
                            , Uri::ptr uri
                            , uint64_t timeout_ms);

    HttpConnection(Socket::ptr sock, bool owner = true);
    ~HttpConnection();

    HttpResponse::ptr recvResponse();
    int sendRequest(HttpRequest::ptr req);

private:
    uint64_t m_createTime = 0;
    uint64_t m_request = 0;
};

class HttpConnectionPool 
{
public:
    typedef std::shared_ptr<HttpConnectionPool> ptr;
    typedef Mutex MutexType;

    static HttpConnectionPool::ptr Create(const std::string& uri
                                   , const std::string& vhost
                                   , uint32_t max_size
                                   , uint32_t max_alive_time
                                   , uint32_t max_request);

    HttpConnectionPool(const std::string& host
                       , const std::string& vhost
                       , uint32_t port
                       , bool is_https
                       , uint32_t max_size
                       , uint32_t max_alive_time
                       , uint32_t max_request);

    HttpConnection::ptr getConnection();

    HttpResult::ptr doGet(const std::string& url
                          , uint64_t timeout_ms
                          , const std::map<std::string, std::string>& headers = {}
                          , const std::string& body = "");

    HttpResult::ptr doGet(Uri::ptr uri
                           , uint64_t timeout_ms
                           , const std::map<std::string, std::string>& headers = {}
                           , const std::string& body = "");

    HttpResult::ptr doPost(const std::string& url
                           , uint64_t timeout_ms
                           , const std::map<std::string, std::string>& headers = {}
                           , const std::string& body = "");

    HttpResult::ptr doPost(Uri::ptr uri
                           , uint64_t timeout_ms
                           , const std::map<std::string, std::string>& headers = {}
                           , const std::string& body = "");

    HttpResult::ptr doRequest(HttpMethod method
                            , const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    HttpResult::ptr doRequest(HttpMethod method
                            , Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers = {}
                            , const std::string& body = "");

    HttpResult::ptr doRequest(HttpRequest::ptr req
                            , uint64_t timeout_ms);

private:
    static void ReleasePtr(HttpConnection* ptr, HttpConnectionPool* pool);

private:
    std::string m_host;
    std::string m_vhost;
    uint32_t m_port;
    uint32_t m_maxSize;
    uint32_t m_maxAliveTime;
    uint32_t m_maxRequest;
    bool m_isHttps;

    MutexType m_mutex;
    std::list<HttpConnection*> m_conns;
    std::atomic<int32_t> m_total = {0};
};

}

}

#endif

// http_connection.cc
#include "http_connection.h"
#include "http_parser.h"
#include "sylar/log.h"
#include "sylar/streams/zlib_stream.h"

namespace sylar
{

namespace http
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

std::string HttpResult::toString() const 
{
    std::stringstream ss;
    ss << "[HttpResult result=" << result
       << " error=" << error
       << " response=" << (response ? response->toString() : "nullptr")
       << "]";
    return ss.str();
}

HttpResult::ptr HttpConnection::DoGet(const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers
                            , const std::string& body) 
{
    Uri::ptr uri = Uri::Create(url);
    if(!uri) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::INVALID_URL
                , nullptr, "invalid url: " + url);
    }
    return DoGet(uri, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnection::DoGet(Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers
                            , const std::string& body)
{
    return DoRequest(HttpMethod::GET, uri, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnection::DoPost(const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers
                            , const std::string& body) 
{
    Uri::ptr uri = Uri::Create(url);
    if(!uri) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::INVALID_URL
                , nullptr, "invalid url: " + url);
    }
    return DoPost(uri, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnection::DoPost(Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers
                            , const std::string& body)
{
    return DoRequest(HttpMethod::POST, uri, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnection::DoRequest(HttpMethod method
                            , const std::string& url
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers
                            , const std::string& body)
{
    Uri::ptr uri = Uri::Create(url);
    if(!uri) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::INVALID_URL
                , nullptr, "invalid url: " + url);
    }
    return DoRequest(method, uri, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnection::DoRequest(HttpMethod method
                            , Uri::ptr uri
                            , uint64_t timeout_ms
                            , const std::map<std::string, std::string>& headers
                            , const std::string& body)
{
    HttpRequest::ptr req = std::make_shared<HttpRequest>();
    req->setPath(uri->getPath());
    req->setQuery(uri->getQuery());
    req->setFragment(uri->getFragment());
    req->setMethod(method);
    
    bool has_host = false;
    for(auto& i : headers) 
    {
        if(strcasecmp(i.first.c_str(), "connection") == 0) 
        {
            if(strcasecmp(i.second.c_str(), "keep-alive") == 0)
            {
                req->setClose(false);
            }
            continue;
        }

        if(!has_host && strcasecmp(i.first.c_str(), "host") == 0) 
        {
            has_host = !i.second.empty();
        }

        req->setHeader(i.first, i.second);
    }
    if(!has_host) 
    {
        req->setHeader("Host", uri->getHost());
    }
    req->setBody(body);
    return DoRequest(req, uri, timeout_ms);
}

HttpResult::ptr HttpConnection::DoRequest(HttpRequest::ptr req
                            , Uri::ptr uri
                            , uint64_t timeout_ms) 
{
    // bool is_ssl = uri->getScheme() == "https";
    Address::ptr addr = uri->createAddress();
    if(!addr) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::INVALID_HOST
                , nullptr, "invalid host: " + uri->getHost());
    }
    
    //Socket::ptr sock = is_ssl ? SSLSocket::CreateTCP(addr) : Socket::CreateTCP(addr);
    Socket::ptr sock = Socket::CreateTCP(addr);
    if(!sock)
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::CREATE_SOCKET_ERROR
                , nullptr, "create socket fail: " + addr->toString()
                        + " errno=" + std::to_string(errno)
                        + " errstr=" + std::string(strerror(errno)));
    }
    
    if(!sock->connect(addr)) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::CONNECT_FAIL
                , nullptr, "connect fail: " + addr->toString());
    }
    
    sock->setRecvTimeout(timeout_ms);
    HttpConnection::ptr conn = std::make_shared<HttpConnection>(sock);
    int rt = conn->sendRequest(req);
    if(rt == 0)
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::SEND_CLOSE_BY_PEER
                , nullptr, "send request closed by peer: " + addr->toString());
    }
    if(rt < 0) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::SEND_SOCKET_ERROR
                    , nullptr, "send request socket error errno=" + std::to_string(errno)
                    + " errstr=" + std::string(strerror(errno)));
    }
    
    auto rsp = conn->recvResponse();
    if(!rsp)
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::TIMEOUT
                    , nullptr, "recv response timeout: " + addr->toString()
                    + " timeout_ms:" + std::to_string(timeout_ms));
    }
    return std::make_shared<HttpResult>((int)HttpResult::Error::OK, rsp, "ok");
}

HttpConnection::HttpConnection(Socket::ptr sock, bool owner)
    : SocketStream(sock, owner) 
{
}

HttpConnection::~HttpConnection() 
{
    SYLAR_LOG_DEBUG(g_logger) << "HttpConnection::~HttpConnection";
}

HttpResponse::ptr HttpConnection::recvResponse() 
{
    HttpResponseParser::ptr parser(new HttpResponseParser);
    uint64_t buff_size = HttpRequestParser::GetHttpRequestBufferSize();
    std::shared_ptr<char> buffer(new char[buff_size + 1], [](char* ptr){
                                 delete[] ptr;
                                });
    char* data = buffer.get();
    int offset = 0;
    // 和 HttpSession::recvRequest() 处的用法是类似的，可以看那里的注释
    do
    {
        int len = read(data + offset, buff_size - offset);
        if(len <= 0) 
        {
            close();
            return nullptr;
        }
        len += offset;
        data[len] = '\0';   // 是ragel那里解析要求的
        size_t nparse = parser->execute(data, len, false);
        if(parser->hasError()) 
        {
            close();
            return nullptr;
        }
        offset = len - nparse;
        if(offset == (int)buff_size) 
        {
            close();
            return nullptr;
        }
        if(parser->isFinished()) 
        {
            break;
        }
    } while(true);
    
    auto& client_parser = parser->getParser();
    std::string body;
    // 下面是分块的时候的解析
    if(client_parser.chunked) 
    {
        int len = offset;
        do 
        {
            bool begin = true;
            do
            {
                if(!begin || len == 0) 
                {
                    int rt = read(data + len, buff_size - len);
                    if(rt <= 0)
                    {
                        close();
                        return nullptr;
                    }
                    len += rt;
                }
                data[len] = '\0';
                size_t nparse = parser->execute(data, len, true);
                if(parser->hasError())
                {
                    close();
                    return nullptr;
                }
                len -= nparse;
                if(len == (int)buff_size) 
                {
                    close();
                    return nullptr;
                }
                begin = false;
            } while(!parser->isFinished());
            //len -= 2;
            
            SYLAR_LOG_DEBUG(g_logger) << "content_len=" << client_parser.content_len;
            if(client_parser.content_len + 2 <= len) 
            {
                body.append(data, client_parser.content_len);
                memmove(data, data + client_parser.content_len + 2
                        , len - client_parser.content_len - 2);
                len -= client_parser.content_len + 2;
            }
            else 
            {
                body.append(data, len);
                int left = client_parser.content_len - len + 2;
                while(left > 0) 
                {
                    int rt = read(data, left > (int)buff_size ? (int)buff_size : left);
                    if(rt <= 0) 
                    {
                        close();
                        return nullptr;
                    }
                    body.append(data, rt);
                    left -= rt;
                }
                body.resize(body.size() - 2);
                len = 0;
            }
        } while(!client_parser.chunks_done);
    }
    else
    {
        int64_t length = parser->getContentLength();
        if(length > 0) 
        {
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
            length -= offset;
            if(length > 0)
            {
                if(readFixSize(&body[len], length) <= 0) 
                {
                    close();
                    return nullptr;
                }
            }
        }
    }

    if(!body.empty())
    {
        auto content_encoding = parser->getData()->getHeader("content-encoding");
        SYLAR_LOG_DEBUG(g_logger) << "content_encoding: " << content_encoding
            << " size=" << body.size();
        // gzip，一种由文件压缩程序「Gzip，GUN zip」产生的编码格式，描述于 RFC 1952。
        // 这种编码格式是一种具有 32 位 CRC 的 Lempel-Ziv 编码（LZ77）
        if(strcasecmp(content_encoding.c_str(), "gzip") == 0)
        {
            auto zs = ZlibStream::CreateGzip(false);
            zs->write(body.c_str(), body.size());
            zs->flush();
            zs->getResult().swap(body);
        }
        // deflate，由定义于 RFC 1950 的「ZLIB」编码格式与 RFC 1951 中描述的「DEFLATE」压缩机制组合而成的产物
        else if(strcasecmp(content_encoding.c_str(), "deflate") == 0)
        {
            auto zs = ZlibStream::CreateDeflate(false);
            zs->write(body.c_str(), body.size());
            zs->flush();
            zs->getResult().swap(body);
        }
        parser->getData()->setBody(body);
    }
    return parser->getData();
}

int HttpConnection::sendRequest(HttpRequest::ptr rsp) 
{
    std::stringstream ss;
    ss << *rsp;
    std::string data = ss.str();
    return writeFixSize(data.c_str(), data.size());
}

HttpConnectionPool::ptr HttpConnectionPool::Create(const std::string& uri
                                                   , const std::string& vhost
                                                   , uint32_t max_size
                                                   , uint32_t max_alive_time
                                                   , uint32_t max_request)
{
    Uri::ptr turi = Uri::Create(uri);
    if(!turi)
    {
        SYLAR_LOG_ERROR(g_logger) << "invalid uri=" << uri;
    }
    return std::make_shared<HttpConnectionPool>(turi->getHost()
            , vhost, turi->getPort(), turi->getScheme() == "https"
            , max_size, max_alive_time, max_request);
}

HttpConnectionPool::HttpConnectionPool(const std::string& host
                                        , const std::string& vhost
                                        , uint32_t port
                                        , bool is_https
                                        , uint32_t max_size
                                        , uint32_t max_alive_time
                                        , uint32_t max_request)
    : m_host(host)
    , m_vhost(vhost)
    , m_port(port ? port : (is_https ? 443 : 80))
    , m_maxSize(max_size)
    , m_maxAliveTime(max_alive_time)
    , m_maxRequest(max_request)
    , m_isHttps(is_https)
{
}

HttpConnection::ptr HttpConnectionPool::getConnection()
{
    uint64_t now_ms = sylar::GetCurrentMS();
    std::vector<HttpConnection*> invalid_conns;
    HttpConnection* ptr = nullptr;
    MutexType::Lock lock(m_mutex);
    while(!m_conns.empty())
    {
        auto conn = *m_conns.begin();
        m_conns.pop_front();
        if(!conn->isConnected()) 
        {
            invalid_conns.push_back(conn);
            continue;
        }
        if((conn->m_createTime + m_maxAliveTime) > now_ms)
        {
            invalid_conns.push_back(conn);
            continue;
        }
        ptr = conn;
        break;
    }
    lock.unlock();
    for(auto i : invalid_conns) 
    {
        delete i;
    }
    m_total -= invalid_conns.size();

    if(!ptr)
    {
        IPAddress::ptr addr = Address::LookupAnyIPAddress(m_host);
        if(!addr)
        {
            SYLAR_LOG_ERROR(g_logger) << "get addr fail: " << m_host;
            return nullptr;
        }
        addr->setPort(m_port);
        //Socket::ptr sock = m_isHttps ? SSLSocket::CreateTCP(addr) : Socket::CreateTCP(addr);
        Socket::ptr sock = Socket::CreateTCP(addr);
        if(!sock)
        {
            SYLAR_LOG_ERROR(g_logger) << "create sock fail: " << *addr;
            return nullptr;
        }
        if(!sock->connect(addr))
        {
            SYLAR_LOG_ERROR(g_logger) << "sock connect fail: " << *addr;
            return nullptr;
        }

        ptr = new HttpConnection(sock);
        ++m_total;
    }
    return HttpConnection::ptr(ptr, std::bind(&HttpConnectionPool::ReleasePtr
                               , std::placeholders::_1, this));
}


HttpResult::ptr HttpConnectionPool::doGet(const std::string& url
                          , uint64_t timeout_ms
                          , const std::map<std::string, std::string>& headers
                          , const std::string& body) 
{
    return doRequest(HttpMethod::GET, url, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnectionPool::doGet(Uri::ptr uri
                                   , uint64_t timeout_ms
                                   , const std::map<std::string, std::string>& headers
                                   , const std::string& body) 
{
    std::stringstream ss;
    ss << uri->getPath()
       << (uri->getQuery().empty() ? "" : "?")
       << uri->getQuery()
       << (uri->getFragment().empty() ? "" : "#")
       << uri->getFragment();
    return doGet(ss.str(), timeout_ms, headers, body);
}

HttpResult::ptr HttpConnectionPool::doPost(const std::string& url
                                   , uint64_t timeout_ms
                                   , const std::map<std::string, std::string>& headers
                                   , const std::string& body)
{
    return doRequest(HttpMethod::POST, url, timeout_ms, headers, body);
}

HttpResult::ptr HttpConnectionPool::doPost(Uri::ptr uri
                                   , uint64_t timeout_ms
                                   , const std::map<std::string, std::string>& headers
                                   , const std::string& body) 
{
    std::stringstream ss;
    ss << uri->getPath()
       << (uri->getQuery().empty() ? "" : "?")
       << uri->getQuery()
       << (uri->getFragment().empty() ? "" : "#")
       << uri->getFragment();
    return doPost(ss.str(), timeout_ms, headers, body);
}

HttpResult::ptr HttpConnectionPool::doRequest(HttpMethod method
                                    , const std::string& url
                                    , uint64_t timeout_ms
                                    , const std::map<std::string, std::string>& headers
                                    , const std::string& body)
{
    HttpRequest::ptr req = std::make_shared<HttpRequest>();
    req->setPath(url);
    req->setMethod(method);
    req->setClose(false);
    
    bool has_host = false;
    for(auto& i : headers) {
        if(strcasecmp(i.first.c_str(), "connection") == 0) 
        {
            if(strcasecmp(i.second.c_str(), "keep-alive") == 0) 
            {
                req->setClose(false);
            }
            continue;
        }

        if(!has_host && strcasecmp(i.first.c_str(), "host") == 0) 
        {
            has_host = !i.second.empty();
        }

        req->setHeader(i.first, i.second);
    }

    if(!has_host) 
    {
        if(m_vhost.empty()) 
        {
            req->setHeader("Host", m_host);
        } 
        else 
        {
            req->setHeader("Host", m_vhost);
        }
    }
    req->setBody(body);
    return doRequest(req, timeout_ms);
}

HttpResult::ptr HttpConnectionPool::doRequest(HttpMethod method
                                    , Uri::ptr uri
                                    , uint64_t timeout_ms
                                    , const std::map<std::string, std::string>& headers
                                    , const std::string& body)
{
    std::stringstream ss;
    ss << uri->getPath()
       << (uri->getQuery().empty() ? "" : "?")
       << uri->getQuery()
       << (uri->getFragment().empty() ? "" : "#")
       << uri->getFragment();
    return doRequest(method, ss.str(), timeout_ms, headers, body);
}

HttpResult::ptr HttpConnectionPool::doRequest(HttpRequest::ptr req
                                        , uint64_t timeout_ms)
{
    auto conn = getConnection();
    if(!conn)
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::POOL_GET_CONNECTION
                , nullptr, "pool host:" + m_host + " port:" + std::to_string(m_port));
    }

    auto sock = conn->getSocket();
    if(!sock) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::POOL_INVALID_CONNECTION
                , nullptr, "pool host:" + m_host + " port:" + std::to_string(m_port));
    }
    sock->setRecvTimeout(timeout_ms);
    int rt = conn->sendRequest(req);
    if(rt == 0)
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::SEND_CLOSE_BY_PEER
                , nullptr, "send request closed by peer: " + sock->getRemoteAddress()->toString());
    }
    if(rt < 0)
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::SEND_SOCKET_ERROR
                    , nullptr, "send request socket error errno=" + std::to_string(errno)
                    + " errstr=" + std::string(strerror(errno)));
    }
    auto rsp = conn->recvResponse();
    if(!rsp) 
    {
        return std::make_shared<HttpResult>((int)HttpResult::Error::TIMEOUT
                    , nullptr, "recv response timeout: " + sock->getRemoteAddress()->toString()
                    + " timeout_ms:" + std::to_string(timeout_ms));
    }
    return std::make_shared<HttpResult>((int)HttpResult::Error::OK, rsp, "ok");
}

void HttpConnectionPool::ReleasePtr(HttpConnection* ptr, HttpConnectionPool* pool)
{
    ++ptr->m_request;
    if(!ptr->isConnected()
            || ((ptr->m_createTime + pool->m_maxAliveTime) >= sylar::GetCurrentMS())
            || (ptr->m_request >= pool->m_maxRequest)) 
    {
        delete ptr;
        --pool->m_total;
        return;
    }
    MutexType::Lock lock(pool->m_mutex);
    pool->m_conns.push_back(ptr);
}

}

}

/**
 * @filename    uri.h
 * @brief   URI 封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-18
 */
#ifndef __SYLAR_URI_H__
#define __SYLAR_URI_H__

#include <memory>
#include <string>
#include <stdint.h>
#include "address.h"

namespace sylar 
{

/*
    foo://user@example.com:8042/over/there?name=ferret#nose
    \_/   \___/\______________/\_________/ \_________/ \__/
     |      |        |            |            |        |
  scheme userinfo authority      path        query   fragment
*/

class Uri
{
public:
    typedef std::shared_ptr<Uri> ptr;

    /**
     * @brief   创建Uri对象
     *
     * @param   uristr  uri字符串
     *
     * @return  解析成功返回Uri对象，否则返回nullptr
     */
    static Uri::ptr Create(const std::string& uristr);

    Uri();

    const std::string& getScheme() const { return m_scheme; }
    const std::string& getUserinfo() const { return m_userinfo; }
    const std::string& getHost() const { return m_host; }
    const std::string& getPath() const;
    const std::string& getQuery() const { return m_query; }
    const std::string& getFragment() const { return m_fragment; }
    int32_t getPort() const;

    void setScheme(const std::string& v) { m_scheme = v; }
    void setUserinfo(const std::string& v) { m_userinfo = v; }
    void setHost(const std::string& v) { m_host = v; }
    void setPath(const std::string& v) { m_path = v; }
    void setQuery(const std::string& v) { m_query = v; }
    void setFragment(const std::string& v) { m_fragment = v; }
    void setPort(int32_t v) { m_port = v; }
    
    std::ostream& dump(std::ostream& os) const;
    std::string toString() const;
    
    Address::ptr createAddress() const;

private:
    bool isDefaultPort() const;

private:
    /// scheme
    std::string m_scheme;
    /// 用户信息
    std::string m_userinfo;
    /// host
    std::string m_host;
    /// 路径
    std::string m_path;
    /// 查询参数
    std::string m_query;
    /// fragment
    std::string m_fragment;
    /// 端口
    int32_t m_port;
};

}

#endif

// uri.rl
#include "uri.h"
#include <sstream>

namespace sylar {
%%{
    # See RFC 3986: http://www.ietf.org/rfc/rfc3986.txt

    machine uri_parser;

    gen_delims = ":" | "/" | "?" | "#" | "[" | "]" | "@";
    sub_delims = "!" | "$" | "&" | "'" | "(" | ")" | "*" | "+" | "," | ";" | "=";
    reserved = gen_delims | sub_delims;
    unreserved = alpha | digit | "-" | "." | "_" | "~";
    pct_encoded = "%" xdigit xdigit;

    action marku { mark = fpc; }
    action markh { mark = fpc; }

    action save_scheme
    {
        uri->setScheme(std::string(mark, fpc - mark));
        mark = NULL;
    }

    scheme = (alpha (alpha | digit | "+" | "-" | ".")*) >marku %save_scheme;

    action save_port
    {
        if(fpc != mark) 
        {
            uri->setPort(atoi(mark));
        }
        mark = NULL;
    }
    action save_userinfo
    {
        if(mark) 
        {
            //std::cout << std::string(mark, fpc - mark) << std::endl;
            uri->setUserinfo(std::string(mark, fpc - mark));
        }
        mark = NULL;
    }
    action save_host
    {
        if(mark != NULL)
        {
            //std::cout << std::string(mark, fpc - mark) << std::endl;
            uri->setHost(std::string(mark, fpc - mark));
        }
    }

    userinfo = (unreserved | pct_encoded | sub_delims | ":")*;
    dec_octet = digit | [1-9] digit | "1" digit{2} | 2 [0-4] digit | "25" [0-5];
    IPv4address = dec_octet "." dec_octet "." dec_octet "." dec_octet;
    h16 = xdigit{1,4};
    ls32 = (h16 ":" h16) | IPv4address;
    IPv6address = (                         (h16 ":"){6} ls32) |
                  (                    "::" (h16 ":"){5} ls32) |
                  ((             h16)? "::" (h16 ":"){4} ls32) |
                  (((h16 ":"){1} h16)? "::" (h16 ":"){3} ls32) |
                  (((h16 ":"){2} h16)? "::" (h16 ":"){2} ls32) |
                  (((h16 ":"){3} h16)? "::" (h16 ":"){1} ls32) |
                  (((h16 ":"){4} h16)? "::"              ls32) |
                  (((h16 ":"){5} h16)? "::"              h16 ) |
                  (((h16 ":"){6} h16)? "::"                  );
    IPvFuture = "v" xdigit+ "." (unreserved | sub_delims | ":")+;
    IP_literal = "[" (IPv6address | IPvFuture) "]";
    reg_name = (unreserved | pct_encoded | sub_delims)*;
    host = IP_literal | IPv4address | reg_name;
    port = digit*;

    authority = ( (userinfo %save_userinfo "@")? host >markh %save_host (":" port >markh %save_port)? ) >markh;

    action save_segment
    {
        mark = NULL;
    }

    action save_path
    {
        //std::cout << std::string(mark, fpc - mark) << std::endl;
        uri->setPath(std::string(mark, fpc - mark));
        mark = NULL;
    }


#    pchar = unreserved | pct_encoded | sub_delims | ":" | "@";
# add (any -- ascii) support chinese
    pchar         = ( (any -- ascii ) | unreserved | pct_encoded | sub_delims | ":" | "@" ) ;
    segment = pchar*;
    segment_nz = pchar+;
    segment_nz_nc = (pchar - ":")+;

    action clear_segments
    {
    }

    path_abempty = (("/" segment))? ("/" segment)*;
    path_absolute = ("/" (segment_nz ("/" segment)*)?);
    path_noscheme = segment_nz_nc ("/" segment)*;
    path_rootless = segment_nz ("/" segment)*;
    path_empty = "";
    path = (path_abempty | path_absolute | path_noscheme | path_rootless | path_empty);

    action save_query
    {
        //std::cout << std::string(mark, fpc - mark) << std::endl;
        uri->setQuery(std::string(mark, fpc - mark));
        mark = NULL;
    }
    action save_fragment
    {
        //std::cout << std::string(mark, fpc - mark) << std::endl;
        uri->setFragment(std::string(mark, fpc - mark));
        mark = NULL;
    }

    query = (pchar | "/" | "?")* >marku %save_query;
    fragment = (pchar | "/" | "?")* >marku %save_fragment;

    hier_part = ("//" authority path_abempty > markh %save_path) | path_absolute | path_rootless | path_empty;

    relative_part = ("//" authority path_abempty) | path_absolute | path_noscheme | path_empty;
    relative_ref = relative_part ( "?" query )? ( "#" fragment )?;

    absolute_URI = scheme ":" hier_part ( "?" query )? ;
    # Obsolete, but referenced from HTTP, so we translate
    relative_URI = relative_part ( "?" query )?;

    URI = scheme ":" hier_part ( "?" query )? ( "#" fragment )?;
    URI_reference = URI | relative_ref;
    main := URI_reference;
    write data;
}%%

Uri::ptr Uri::Create(const std::string& uristr) 
{
    Uri::ptr uri(new Uri);
    int cs = 0;
    const char* mark = 0;
    %% write init;
    const char *p = uristr.c_str();
    const char *pe = p + uristr.size();
    const char* eof = pe;
    %% write exec;
    if(cs == uri_parser_error) 
    {
        return nullptr;
    } 
    else if(cs >= uri_parser_first_final) 
    {
        return uri;
    }
    return nullptr;
}

Uri::Uri()
    : m_port(0) 
{
}


const std::string& Uri::getPath() const 
{
    static std::string s_default_path = "/";
    return m_path.empty() ? s_default_path : m_path;
}

int32_t Uri::getPort() const 
{
    if(m_port) 
    {
        return m_port;
    }
    if(m_scheme == "http"
        || m_scheme == "ws") 
    {
        return 80;
    } 
    else if(m_scheme == "https"
            || m_scheme == "wss") 
    {
        return 443;
    }
    return m_port;
}

std::ostream& Uri::dump(std::ostream& os) const 
{
    os << m_scheme << "://"
       << m_userinfo
       << (m_userinfo.empty() ? "" : "@")
       << m_host
       << (isDefaultPort() ? "" : ":" + std::to_string(m_port))
       << getPath()
       << (m_query.empty() ? "" : "?")
       << m_query
       << (m_fragment.empty() ? "" : "#")
       << m_fragment;
    return os;
}

std::string Uri::toString() const
{
    std::stringstream ss;
    dump(ss);
    return ss.str();
}

Address::ptr Uri::createAddress() const 
{
    auto addr = Address::LookupAnyIPAddress(m_host);
    if(addr) 
    {
        addr->setPort(getPort());
    }
    return addr;
}

bool Uri::isDefaultPort() const 
{
    if(m_port == 0) 
    {
        return true;
    }
    if(m_scheme == "http"
            || m_scheme == "ws") 
    {
        return m_port == 80;
    } 
    else if(m_scheme == "https"
            || m_scheme == "wss")
    {
        return m_port == 443;
    }
    return false;
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
