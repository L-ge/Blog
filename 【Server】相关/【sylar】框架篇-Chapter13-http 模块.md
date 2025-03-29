# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 对 http 协议的封装。


# http.h

- http 定义结构体封装模块。
- 一些方法描述和错误码定义是直接拷贝自以下地址：
	` https://github.com/nodejs/http-parser/blob/main/http_parser.h`
- HttpMethod
	- http 方法枚举类。
	- 利用了拷贝回来的宏定义。
- HttpStatus
	- http 状态枚举类。
	- 利用了拷贝回来的宏定义。
- HttpRequest
	- 请求的结构体的封装。
	- 例如 请求方法、请求的 http 版本号、请求路径、请求参数、请求消息体、cookie 等等。
- HttpResponse
	- 响应的结构体的封装。
	- 例如 响应状态、响应的 http 版本、响应消息体、响应原因 等等。


# http_parser.h

- HttpRequestParser
	- http请求解析类。
- HttpResponseParser
	- http响应解析类。
- 这两个类主要是设置一些回调函数，然后通过这些回调函数来设置某些属性。


# 其他说明

- http_parser.h 模块利用了 Ragel 来解析 http 协议报文。
	- Ragel 是个有限状态机编译器，它将基于正则表达式的状态机编译成传统语言（C，C++，D，Java，Ruby 等）的解析器。Ragel 不仅仅可以用来解析字节流，它实际上可以解析任何可以用正则表达式表达出来的内容。而且可以很方便的将解析代码嵌入到传统语言中。
- http_parser.h 模块利用了别人写好的按照 ragel 提供的语法与关键字编写的 .rl 文件，从中拷贝出 3 个头文件，2 个 rl 文件。其中 .c 文件是通过 rl 文件生成的。
	- 具体地址如下：
		`https://github.com/mongrel2/mongrel2/tree/master/src/http11`
	- 具体生成 .cc 文件的指令如下：
		```shell
		$ ragel -v			// 获取版本号
		$ ragel -help		// 获取帮助文档
		
		$ ragel -G2 -C http11_parser.rl -o http11_parser.cc
		$ ragel -G2 -C httpclient_parser.rl -o httpclient_parser.cc
		$ mv http11_parser.cc http11_parser.rl.cc			  // 与自己写的代码区分开
		$ mv httpclient_parser.cc httpclient_parser.rl.cc     // 与自己写的代码区分开
		```
	- 注意，sylar 对拷贝回来的几个文件做了一些修改，使其编译通过、支持“分段解析协议”的功能、支持中文的功能。
- http.h 中 HttpRequest 和 HttpResponse 的 dump 方法里面组装出来的字符串其实就是 http 报文了。
- 关于 HTTP 请求和响应的格式可参考如下地址：
	`https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages`


# 部分相关代码
```C++
/**
 * @filename    http.h
 * @brief   http 定义结构体封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-11
 */
#ifndef __SYLAR_HTTP_HTTP_H__
#define __SYLAR_HTTP_HTTP_H__

#include <memory>
#include <string>
#include <map>
#include <vector>
#include <iostream>
#include <sstream>
#include <boost/lexical_cast.hpp>

namespace sylar
{

namespace http
{
// copy from https://github.com/nodejs/http-parser/blob/main/http_parser.h
/* Request Methods */
#define HTTP_METHOD_MAP(XX)         \
  XX(0,  DELETE,      DELETE)       \
  XX(1,  GET,         GET)          \
  XX(2,  HEAD,        HEAD)         \
  XX(3,  POST,        POST)         \
  XX(4,  PUT,         PUT)          \
  /* pathological */                \
  XX(5,  CONNECT,     CONNECT)      \
  XX(6,  OPTIONS,     OPTIONS)      \
  XX(7,  TRACE,       TRACE)        \
  /* WebDAV */                      \
  XX(8,  COPY,        COPY)         \
  XX(9,  LOCK,        LOCK)         \
  XX(10, MKCOL,       MKCOL)        \
  XX(11, MOVE,        MOVE)         \
  XX(12, PROPFIND,    PROPFIND)     \
  XX(13, PROPPATCH,   PROPPATCH)    \
  XX(14, SEARCH,      SEARCH)       \
  XX(15, UNLOCK,      UNLOCK)       \
  XX(16, BIND,        BIND)         \
  XX(17, REBIND,      REBIND)       \
  XX(18, UNBIND,      UNBIND)       \
  XX(19, ACL,         ACL)          \
  /* subversion */                  \
  XX(20, REPORT,      REPORT)       \
  XX(21, MKACTIVITY,  MKACTIVITY)   \
  XX(22, CHECKOUT,    CHECKOUT)     \
  XX(23, MERGE,       MERGE)        \
  /* upnp */                        \
  XX(24, MSEARCH,     M-SEARCH)     \
  XX(25, NOTIFY,      NOTIFY)       \
  XX(26, SUBSCRIBE,   SUBSCRIBE)    \
  XX(27, UNSUBSCRIBE, UNSUBSCRIBE)  \
  /* RFC-5789 */                    \
  XX(28, PATCH,       PATCH)        \
  XX(29, PURGE,       PURGE)        \
  /* CalDAV */                      \
  XX(30, MKCALENDAR,  MKCALENDAR)   \
  /* RFC-2068, section 19.6.1.2 */  \
  XX(31, LINK,        LINK)         \
  XX(32, UNLINK,      UNLINK)       \
  /* icecast */                     \
  XX(33, SOURCE,      SOURCE)       \

/* Status Codes */
#define HTTP_STATUS_MAP(XX)                                                 \
  XX(100, CONTINUE,                        Continue)                        \
  XX(101, SWITCHING_PROTOCOLS,             Switching Protocols)             \
  XX(102, PROCESSING,                      Processing)                      \
  XX(200, OK,                              OK)                              \
  XX(201, CREATED,                         Created)                         \
  XX(202, ACCEPTED,                        Accepted)                        \
  XX(203, NON_AUTHORITATIVE_INFORMATION,   Non-Authoritative Information)   \
  XX(204, NO_CONTENT,                      No Content)                      \
  XX(205, RESET_CONTENT,                   Reset Content)                   \
  XX(206, PARTIAL_CONTENT,                 Partial Content)                 \
  XX(207, MULTI_STATUS,                    Multi-Status)                    \
  XX(208, ALREADY_REPORTED,                Already Reported)                \
  XX(226, IM_USED,                         IM Used)                         \
  XX(300, MULTIPLE_CHOICES,                Multiple Choices)                \
  XX(301, MOVED_PERMANENTLY,               Moved Permanently)               \
  XX(302, FOUND,                           Found)                           \
  XX(303, SEE_OTHER,                       See Other)                       \
  XX(304, NOT_MODIFIED,                    Not Modified)                    \
  XX(305, USE_PROXY,                       Use Proxy)                       \
  XX(307, TEMPORARY_REDIRECT,              Temporary Redirect)              \
  XX(308, PERMANENT_REDIRECT,              Permanent Redirect)              \
  XX(400, BAD_REQUEST,                     Bad Request)                     \
  XX(401, UNAUTHORIZED,                    Unauthorized)                    \
  XX(402, PAYMENT_REQUIRED,                Payment Required)                \
  XX(403, FORBIDDEN,                       Forbidden)                       \
  XX(404, NOT_FOUND,                       Not Found)                       \
  XX(405, METHOD_NOT_ALLOWED,              Method Not Allowed)              \
  XX(406, NOT_ACCEPTABLE,                  Not Acceptable)                  \
  XX(407, PROXY_AUTHENTICATION_REQUIRED,   Proxy Authentication Required)   \
  XX(408, REQUEST_TIMEOUT,                 Request Timeout)                 \
  XX(409, CONFLICT,                        Conflict)                        \
  XX(410, GONE,                            Gone)                            \
  XX(411, LENGTH_REQUIRED,                 Length Required)                 \
  XX(412, PRECONDITION_FAILED,             Precondition Failed)             \
  XX(413, PAYLOAD_TOO_LARGE,               Payload Too Large)               \
  XX(414, URI_TOO_LONG,                    URI Too Long)                    \
  XX(415, UNSUPPORTED_MEDIA_TYPE,          Unsupported Media Type)          \
  XX(416, RANGE_NOT_SATISFIABLE,           Range Not Satisfiable)           \
  XX(417, EXPECTATION_FAILED,              Expectation Failed)              \
  XX(421, MISDIRECTED_REQUEST,             Misdirected Request)             \
  XX(422, UNPROCESSABLE_ENTITY,            Unprocessable Entity)            \
  XX(423, LOCKED,                          Locked)                          \
  XX(424, FAILED_DEPENDENCY,               Failed Dependency)               \
  XX(426, UPGRADE_REQUIRED,                Upgrade Required)                \
  XX(428, PRECONDITION_REQUIRED,           Precondition Required)           \
  XX(429, TOO_MANY_REQUESTS,               Too Many Requests)               \
  XX(431, REQUEST_HEADER_FIELDS_TOO_LARGE, Request Header Fields Too Large) \
  XX(451, UNAVAILABLE_FOR_LEGAL_REASONS,   Unavailable For Legal Reasons)   \
  XX(500, INTERNAL_SERVER_ERROR,           Internal Server Error)           \
  XX(501, NOT_IMPLEMENTED,                 Not Implemented)                 \
  XX(502, BAD_GATEWAY,                     Bad Gateway)                     \
  XX(503, SERVICE_UNAVAILABLE,             Service Unavailable)             \
  XX(504, GATEWAY_TIMEOUT,                 Gateway Timeout)                 \
  XX(505, HTTP_VERSION_NOT_SUPPORTED,      HTTP Version Not Supported)      \
  XX(506, VARIANT_ALSO_NEGOTIATES,         Variant Also Negotiates)         \
  XX(507, INSUFFICIENT_STORAGE,            Insufficient Storage)            \
  XX(508, LOOP_DETECTED,                   Loop Detected)                   \
  XX(510, NOT_EXTENDED,                    Not Extended)                    \
  XX(511, NETWORK_AUTHENTICATION_REQUIRED, Network Authentication Required) \

/**
 * @brief   http 方法枚举类
 */
enum class HttpMethod
{
#define XX(num, name, string) name = num,
    HTTP_METHOD_MAP(XX)
#undef XX
    INVALID_METHOD
};

/**
 * @brief   http 状态枚举类
 */
enum class HttpStatus
{
#define XX(code, name, desc) name = code,
    HTTP_STATUS_MAP(XX)
#undef XX
};

/**
 * @brief   将字符串方法转成http方法枚举
 */
HttpMethod StringToHttpMethod(const std::string& m);

/**
 * @brief   将字符串方法转成http方法枚举
 */
HttpMethod CharsToHttpMethod(const char* m);

/**
 * @brief   将http方法枚举转换成字符串
 */
const char* HttpMethodToString(const HttpMethod& m);

/**
 * @brief   将http状态枚举转换成字符串
 */
const char* HttpStatusTostring(const HttpStatus& s);

/**
 * @brief   忽略大小写比较的仿函数
 */
struct CaseInsensitiveLess
{
    bool operator()(const std::string& lhs, const std::string& rhs) const;
};

/**
 * @brief  获取MapType中的key值，并转换成对应的类型，返回是否成功 
 */
template<class MapType, class T>
bool checkGetAs(const MapType& m, const std::string& key, T& val, const T& def = T())
{
    auto it = m.find(key);
    if(it == m.end())
    {
        val = def;
        return false;
    }
    try
    {
        val = boost::lexical_cast<T>(it->second);
        return true;
    }
    catch(...)
    {
        val = def;
    }
    return false;
}

/**
 * @brief  获取MapType中的key值，并转换成对应的类型返回 
 */
template<class MapType, class T>
T getAs(const MapType& m, const std::string& key, const T& def = T())
{
    auto it = m.find(key);
    if(it == m.end())
    {
        return def;
    }
    try
    {
        return boost::lexical_cast<T>(it->second);
    }
    catch(...)
    {
    }
    return def;
}

class HttpResponse;

/**
 * @brief   http 请求结构
 */
class HttpRequest
{
public:
    typedef std::shared_ptr<HttpRequest> ptr;
    typedef std::map<std::string, std::string, CaseInsensitiveLess> MapType;

    /**
     * @brief   构造函数
     *
     * @param   version http版本号. 0x11是1.1版本，0x10是1.0版本
     * @param   close   是否keepalive
     */
    HttpRequest(uint8_t version = 0x11, bool close = true);
    std::shared_ptr<HttpResponse> createResponse();
    
    HttpMethod getMethod() const { return m_method; }
    uint8_t getVersion() const { return m_version; }
    const std::string& getPath() const { return m_path; }
    const std::string& getQuery() const { return m_query; }
    const std::string& getBody() const { return m_body; }
    const MapType& getHeaders() const { return m_headers; }
    const MapType& getParams() const { return m_params; }
    const MapType& getCookies() const { return m_cookies; }

    void setMethod(HttpMethod v) { m_method = v; }
    void setVersion(uint8_t v) { m_version = v; }
    void setPath(const std::string& v) { m_path = v; }
    void setQuery(const std::string& v) { m_query = v; }
    void setFragment(const std::string& v) { m_fragment = v; }
    void setBody(const std::string& v) { m_body = v; }

    bool isClose() const { return m_close; }
    void setClose(bool v) { m_close = v; }
    bool isWebsocket() const { return m_websocket; }
    void setWebsocket(bool v) { m_websocket = v; }
    
    void setHeaders(const MapType& v) { m_headers = v; }
    void setParams(const MapType& v) { m_params = v; }
    void setCookies(const MapType& v) { m_cookies = v; }
    
    std::string getHeader(const std::string& key, const std::string& def = "") const;
    std::string getParam(const std::string& key, const std::string& def = "");
    std::string getCookie(const std::string& key, const std::string& def = "");
    
    void setHeader(const std::string& key, const std::string& val);
    void setParam(const std::string& key, const std::string& val);
    void setCookie(const std::string& key, const std::string& val);
    
    void delHeader(const std::string& key);
    void delParam(const std::string& key);
    void delCookie(const std::string& key);

    bool hasHeader(const std::string& key, std::string* val = nullptr);
    bool hasParam(const std::string& key, std::string* val = nullptr);
    bool hasCookie(const std::string& key, std::string* val = nullptr);

    template<class T>
    bool checkGetHeadersAs(const std::string& key, T& val, const T& def = T())
    {
        return checkGetAs(m_headers, key, val, def);
    }

    template<class T>
    T getHeaderAs(const std::string& key, const T& def = T())
    {
        return getAs(m_headers, key, def);
    }

    template<class T>
    bool checkGetParamAs(const std::string& key, T& val, const T& def = T())
    {
        initQueryParam();
        initBodyParam();
        return checkGetAs(m_params, key, val, def);
    }

    template<class T>
    T getParamAs(const std::string& key, const T& def = T())
    {
        initQueryParam();
        initBodyParam();
        return getAs(m_params, key, def);
    }
    
    template<class T>
    bool checkGetCookieAs(const std::string& key, T& val, const T& def = T())
    {
        initCookies();
        return checkGetAs(m_cookies, key, val, def);
    }

    template<class T>
    T getCookieAs(const std::string& key, const T& def = T())
    {
        return getAs(m_cookies, key, def);
    }

    std::ostream& dump(std::ostream& os) const;
    std::string toString() const;

    void init();
    void initParam();
    void initQueryParam();
    void initBodyParam();
    void initCookies();

private:
    /// http方法
    HttpMethod m_method;
    /// http版本
    uint8_t m_version;
    /// 是否自动关闭
    bool m_close;
    /// 是否为websocket
    bool m_websocket;
    
    /// 是否解析完报文的标识，0001b是请求参数、0010b是请求消息体、0100b是请求cookie
    uint8_t m_parserParamFlag;
    /// 请求路径
    std::string m_path;
    /// 请求参数
    std::string m_query;
    /// 请求fragment
    std::string m_fragment;
    /// 请求消息体
    std::string m_body;
    /// 请求头部map
    MapType m_headers;
    /// 请求参数map
    MapType m_params;
    /// 请求cookie map
    MapType m_cookies;
};

/**
 * @brief   http 响应结构
 */
class HttpResponse
{
public:
    typedef std::shared_ptr<HttpResponse> ptr;
    typedef std::map<std::string, std::string, CaseInsensitiveLess> MapType;

    /**
     * @brief   构造函数
     *
     * @param   version http版本号
     * @param   close   是否自动关闭
     */
    HttpResponse(uint8_t version = 0x11, bool close = true);
   
    HttpStatus getStatus() const { return m_status; }
    uint8_t getVersion() const { return m_version; }
    const std::string& getBody() const { return m_body; }
    const std::string& getReason() const { return m_reason; }
    const MapType& getHeaders() const { return m_headers; }
    void setStatus(HttpStatus v) { m_status = v; }
    void setVersion(uint8_t v) { m_version = v; }
    void setBody(const std::string& v) { m_body = v; }
    void setReason(const std::string& v) { m_reason = v; }
    void setHeaders(const MapType& v) { m_headers = v; }

    bool isClose() const { return m_close; }
    void setClose(bool v) { m_close = v; }
    bool isWebsocket() const { return m_websocket; }
    void setWebsocket(bool v) { m_websocket = v; }

    std::string getHeader(const std::string& key, const std::string& def = "") const;
    void setHeader(const std::string& key, const std::string& val);
    void delHeader(const std::string& key);

    template<class T>
    bool checkGetHeadersAs(const std::string& key, T& val, const T& def = T())
    {
        return checkGetAs(m_headers, key, val, def);
    }

    template<class T>
    T getHeaderAs(const std::string& key, const T& def = T())
    {
        return getAs(m_headers, key, def);
    }

    std::ostream& dump(std::ostream& os) const;
    std::string toString() const;

    void setRedirect(const std::string& uri);
    void setCookie(const std::string& key, const std::string& val, 
                   time_t expired = 0, const std::string& path = "",
                   const std::string& domain = "", bool secure = false);

private:
    /// 响应状态
    HttpStatus m_status;
    /// http版本
    uint8_t m_version;
    /// 是否自动关闭
    bool m_close;
    /// 是否为websocket
    bool m_websocket;
    
    /// 响应消息体
    std::string m_body;
    /// 响应原因
    std::string m_reason;
    /// 响应头部map
    MapType m_headers;
    /// 响应cookie 
    std::vector<std::string> m_cookies;
};

std::ostream& operator<<(std::ostream& os, const HttpRequest& req);

std::ostream& operator<<(std::ostream& os, const HttpResponse& rsp);

}   // end namespace http

}   // end namespace sylar

#endif

// http.cc
#include "http.h"
#include "sylar/util.h"

namespace sylar
{

namespace http
{

HttpMethod StringToHttpMethod(const std::string& m)
{
#define XX(num, name, string) \
    if(strcmp(#string, m.c_str()) == 0) \
    { \
        return HttpMethod::name; \
    } \
    HTTP_METHOD_MAP(XX);
#undef XX
    return HttpMethod::INVALID_METHOD;
}

HttpMethod CharsToHttpMethod(const char* m)
{
#define XX(num, name, string) \
    if(strncmp(#string, m, strlen(#string)) == 0) \
    { \
        return HttpMethod::name; \
    } \
    
    HTTP_METHOD_MAP(XX);
#undef XX
    return HttpMethod::INVALID_METHOD;
}

static const char* s_method_string[] =
{
#define XX(num, name, string) #string,
    HTTP_METHOD_MAP(XX)
#undef XX
};

const char* HttpMethodToString(const HttpMethod& m)
{
    uint32_t idx = (uint32_t)m;
    if(idx >= (sizeof(s_method_string) / sizeof(s_method_string[0])))
    {
        return "<unknown>";
    }
    return s_method_string[idx];
}

const char* HttpStatusTostring(const HttpStatus& s)
{
    switch(s)
    {
#define XX(code, name, msg) \
        case HttpStatus::name: \
            return #msg;
        HTTP_STATUS_MAP(XX);
#undef XX
        default:
            return "<unknown>";
    }
}

bool CaseInsensitiveLess::operator()(const std::string& lhs, const std::string& rhs) const
{
    return strcasecmp(lhs.c_str(), rhs.c_str()) < 0;
}

HttpRequest::HttpRequest(uint8_t version, bool close)
    : m_method(HttpMethod::GET)
    , m_version(version)
    , m_close(close)
    , m_websocket(false)
    , m_parserParamFlag(0)
    , m_path("/")
{
}

std::shared_ptr<HttpResponse> HttpRequest::createResponse()
{
    HttpResponse::ptr rsp(new HttpResponse(getVersion(), isClose()));
    return rsp;
}

std::string HttpRequest::getHeader(const std::string& key, const std::string& def) const
{
    auto it = m_headers.find(key);
    return it == m_headers.end() ? def : it->second;
}

std::string HttpRequest::getParam(const std::string& key, const std::string& def)
{
    initQueryParam();
    initBodyParam();
    auto it = m_params.find(key);
    return it == m_params.end() ? def : it->second;
}

std::string HttpRequest::getCookie(const std::string& key, const std::string& def)
{
    initCookies();
    auto it = m_cookies.find(key);
    return it == m_cookies.end() ? def : it->second;
}

void HttpRequest::setHeader(const std::string& key, const std::string& val)
{
    m_headers[key] = val;
}

void HttpRequest::setParam(const std::string& key, const std::string& val)
{
    m_params[key] = val;
}

void HttpRequest::setCookie(const std::string& key, const std::string& val)
{
    m_cookies[key] = val;
}

void HttpRequest::delHeader(const std::string& key)
{
    m_headers.erase(key);
}

void HttpRequest::delParam(const std::string& key)
{
    m_params.erase(key);
}

void HttpRequest::delCookie(const std::string& key)
{
    m_cookies.erase(key);
}

bool HttpRequest::hasHeader(const std::string& key, std::string* val)
{
    auto it = m_headers.find(key);
    if(it == m_headers.end())
    {
        return false;
    }
    if(val)
    {
        *val = it->second;
    }
    return true;
}

bool HttpRequest::hasParam(const std::string& key, std::string* val)
{
    initQueryParam();
    initBodyParam();
    auto it = m_params.find(key);
    if(it == m_params.end())
    {
        return false;
    }
    if(val)
    {
        *val = it->second;
    }
    return true;
}

bool HttpRequest::hasCookie(const std::string& key, std::string* val)
{
    initCookies();
    auto it = m_cookies.find(key);
    if(it == m_cookies.end())
    {
        return false;
    }
    if(val)
    {
        *val = it->second;
    }
    return true;
}

std::ostream& HttpRequest::dump(std::ostream& os) const
{
    os << HttpMethodToString(m_method) << " "
       << m_path
       << (m_query.empty() ? "" : "?")
       << m_query
       << (m_fragment.empty() ? "" : "#")
       << " HTTP/"
       << ((uint32_t)(m_version >> 4))
       << "."
       << ((uint32_t)(m_version & 0x0F))
       << "\r\n";
    if(!m_websocket)
    {
        os << "connection: " << (m_close ? "close" : "keep-alive") << "\r\n";
    }

    for(auto& i : m_headers)
    {
        if(!m_websocket && strcasecmp(i.first.c_str(), "connection") == 0)
        {
            continue;
        }
        os << i.first << ": " << i.second << "\r\n";
    }

    if(!m_body.empty())
    {
        os << "content-length: " << m_body.size() << "\r\n\r\n" << m_body;
    }
    else
    {
        os << "\r\n";
    }
    return os;
}

std::string HttpRequest::toString() const
{
    std::stringstream ss;
    dump(ss);
    return ss.str();
}

void HttpRequest::init()
{
    std::string conn = getHeader("connection");
    if(!conn.empty())
    {
        if(strcasecmp(conn.c_str(), "keep-alive") == 0)
        {
            m_close = false;
        }
        else
        {
            m_close = true;
        }
    }
}

void HttpRequest::initParam()
{
    initQueryParam();
    initBodyParam();
    initCookies();
}

void HttpRequest::initQueryParam()
{
    if(m_parserParamFlag & 0x1)     // 已经解析过了，直接return
    {
        return;
    }

#define PARSE_PARAM(str, m, flag, trim) \
    size_t pos = 0; \
    do \
    { \
        size_t last = pos; \
        pos = str.find('=', pos); \
        if(pos == std::string::npos) \
        { \
            break; \
        } \
        size_t key = pos; \
        pos = str.find(flag, pos); \
        m.insert(std::make_pair(trim(str.substr(last, key-last)), \
                    sylar::StringUtil::UrlDecode(str.substr(key+1, pos-key-1)))); \
        if(pos == std::string::npos) \
        { \
            break; \
        } \
        ++pos; \
    } while(true);

    PARSE_PARAM(m_query, m_params, '&',);
    m_parserParamFlag |= 0x1;
}

void HttpRequest::initBodyParam()
{
    if(m_parserParamFlag & 0x2)
    {
        return;
    }

    std::string content_type = getHeader("content-type");

    // form的enctype属性为编码方式，常用有两种：application/x-www-form-urlencoded和multipart/form-data，默认为application/x-www-form-urlencoded。
    // 1.x-www-form-urlencoded
    // 当action为get时候，浏览器用x-www-form-urlencoded的编码方式把form数据转换成一个字串（name1=value1&name2=value2…），然后把这个字串append到url后面，用?分割，加载这个新的url。
    //2.multipart/form-data
    //当action为post时候，浏览器把form数据封装到http body中，然后发送到server。 如果没有type=file的控件，用默认的application/x-www-form-urlencoded就可以了。 但是如果有type=file的话，就要用到multipart/form-data了。浏览器会把整个表单以控件为单位分割，并为每个部分加上Content-Disposition(form-data或者file),Content-Type(默认为text/plain),name(控件name)等信息，并加上分割符(boundary)。
    
    // 请求头Content-Type为application/x-www-form-urlencoded，意思是这个url是被编码过的。
    // 如果请求端未执行编码操作，那么响应端将获取不到传过来的参数
    if(strcasestr(content_type.c_str(), "application/x-www-form-urlencoded") == nullptr)
    {
        m_parserParamFlag |= 0x2;
        return;
    }
    PARSE_PARAM(m_body, m_params, '&',);
    m_parserParamFlag |= 0x2;
}

void HttpRequest::initCookies()
{
    if(m_parserParamFlag & 0x4)
    {
        return;
    }
    std::string cookie = getHeader("cookie");
    if(cookie.empty())
    {
        m_parserParamFlag |= 0x4;
        return;
    }
    PARSE_PARAM(cookie, m_cookies, ';', sylar::StringUtil::Trim);
    m_parserParamFlag |= 0x4;
}

HttpResponse::HttpResponse(uint8_t version, bool close)
    : m_status(HttpStatus::OK)
    , m_version(version)
    , m_close(close)
    , m_websocket(false)
{
}

std::string HttpResponse::getHeader(const std::string& key, const std::string& def) const
{
    auto it = m_headers.find(key);
    return it == m_headers.end() ? def : it->second;
}

void HttpResponse::setHeader(const std::string& key, const std::string& val)
{
    m_headers[key] = val;
}

void HttpResponse::delHeader(const std::string& key)
{
    m_headers.erase(key);
}

std::ostream& HttpResponse::dump(std::ostream& os) const
{
    os << "HTTP/"
       << ((uint32_t)(m_version >> 4))
       << "."
       << ((uint32_t)(m_version & 0x0F))
       << " "
       << (uint32_t)m_status
       << " "
       << (m_reason.empty() ? HttpStatusTostring(m_status) : m_reason)
       << "\r\n";

    for(auto& i : m_headers)
    {
        if(!m_websocket && strcasecmp(i.first.c_str(), "connection") == 0)
        {
            continue;
        }
        os << i.first << ": " << i.second << "\r\n";
    }

    for(auto& i : m_cookies)
    {
        os << "Set-Cookie: " << i << "\r\n";
    }

    if(!m_websocket)
    {
        os << "connection: " << (m_close ? "close" : "keep-alive") << "\r\n";
    }
    if(!m_body.empty())
    {
        os << "content-length: " << m_body.size() << "\r\n\r\n" << m_body;
    }
    else
    {
        os << "\r\n";
    }
    return os;
}

std::string HttpResponse::toString() const
{
    std::stringstream ss;
    dump(ss);
    return ss.str();
}

void HttpResponse::setRedirect(const std::string& uri)
{
    // Location 首部指定的是需要将页面重新定向至的地址。一般在响应码为 3xx 的响应中才会有意义。
    // location属性表示该网页的跳转网址，也被称为重定向网址。当网页中包含该属性后，浏览器会自动从当前网页跳转到location中指定的网址。
    m_status = HttpStatus::FOUND;       // FOUND是302
    setHeader("Location", uri);
}

void HttpResponse::setCookie(const std::string& key, const std::string& val, 
                   time_t expired, const std::string& path,
                   const std::string& domain, bool secure)
{
    std::stringstream ss;
    ss << key << "=" << val;
    if(expired > 0)     // 设置Cookie的生存期
    {
        ss << ";expires=" 
           << sylar::Time2Str(expired, "%a, %d %b %Y:%M:%S")
           << " GMT";
    }
    if(!domain.empty()) // 指定了可以访问该Cookie的Web站点或域
    {
        ss << ";domain=" << domain;
    }
    if(!path.empty())   // 定义了Web站点上可以访问该Cookie的目录
    {
        ss << ";path=" << path;
    }
    if(secure)          // 指定是否使用HTTPS安全协议发送Cookie
    {
        ss << ";secure";
    }
    m_cookies.push_back(ss.str());
}

std::ostream& operator<<(std::ostream& os, const HttpRequest& req)
{
    return req.dump(os);
}

std::ostream& operator<<(std::ostream& os, const HttpResponse& rsp)
{
    return rsp.dump(os);
}

}

}


/**
 * @filename    http_parser.h
 * @brief   http 协议解析封装
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-11
 */
#ifndef __SYLAR_HTTP_PARSER_H__
#define __SYLAR_HTTP_PARSER_H__

#include "http.h"
#include "http11_parser.h"
#include "httpclient_parser.h"

namespace sylar
{

namespace http
{

/**
 * @brief   http请求解析类
 */
class HttpRequestParser
{
public:
    typedef std::shared_ptr<HttpRequestParser> ptr;

    HttpRequestParser();

    /**
     * @brief  解析协议 
     *
     * @param   data    协议文本缓存
     * @param   len     协议文本缓存长度
     *
     * @return  返回实际解析的长度，并且将已解析的数据移除
     */
    size_t execute(char* data, size_t len);
    
    /**
     * @brief   是否解析完成
     */
    int isFinished();
    
    /**
     * @brief   是否有错误
     */
    int hasError();
    
    HttpRequest::ptr getData() const { return m_data; }
    void setError(int v) { m_error = v; }
    uint64_t getContentLength();
    const http_parser& getParser() const { return m_parser; }

public:
    /**
     * @brief   获取HttpRequest协议解析的缓存大小
     */
    static uint64_t GetHttpRequestBufferSize();
    
    /**
     * @brief   获取HTTPRequest协议的最大消息体大小
     */
    static uint64_t GetHttpRequestMaxBodySize();

private:
    http_parser m_parser;
    HttpRequest::ptr m_data;
    /// 错误码
    /// 1000: invalid method
    /// 1001: invalid version
    /// 1002: invalid field
    int m_error;
};

/**
 * @brief   http响应解析类
 */
class HttpResponseParser
{
public:
    typedef std::shared_ptr<HttpResponseParser> ptr;

    HttpResponseParser();

    /**
     * @brief   解析http响应协议
     *
     * @param   data    协议数据缓存
     * @param   len     协议数据缓存大小
     * @param   chunck  是否在解析chunck
     *          Chunk就是将大文件分成块，一个块对应着一个Http请求，然后会对每个Http进行编号，然后在接收方重组。
     *          正常的Http请求都是客户端请求，服务器返回然后就结束了。
     *          而Chunk不会，是会一直等待服务器多次发送数据，发送数据完成后才会结束。
     *          通过Header中的Transfer-Encoding = Chunked判断一个http是不是chunk。
     * @return  返回实际解析的长度，并且移除已解析的数据
     */
    size_t execute(char* data, size_t len, bool chunck);
    int isFinished();
    int hasError();
    HttpResponse::ptr getData() const { return m_data; }
    void setError(int v) { m_error = v; }
    uint64_t getContentLength();
    const httpclient_parser& getParser() const { return m_parser; }

public:
    static uint64_t GetHttpResponseBufferSize();
    static uint64_t GetHttpResponseMaxBodySize();

private:
    httpclient_parser m_parser;
    HttpResponse::ptr m_data;
    /// 错误码
    /// 1001: invalid version
    /// 1002: invalid field
    int m_error;
};

}   // end namespace http

}   // end namespace sylar


#endif

// http_parser.cc
#include "http_parser.h"
#include "sylar/log.h"
#include "sylar/config.h"
#include <string.h>

namespace sylar
{

namespace http
{
static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

static sylar::ConfigVar<uint64_t>::ptr g_http_request_buffer_size = 
    sylar::Config::Lookup("http.request.buffer_size", (uint64_t)(4*1024), "http request buffer size");

static sylar::ConfigVar<uint64_t>::ptr g_http_request_max_body_size = 
    sylar::Config::Lookup("http.request.max_body_size", (uint64_t)(64*1024*1024), "http request max body size");

static sylar::ConfigVar<uint64_t>::ptr g_http_response_buffer_size = 
    sylar::Config::Lookup("http.response.buffer_size", (uint64_t)(4*1024), "http response buffer size");

static sylar::ConfigVar<uint64_t>::ptr g_http_response_max_body_size = 
    sylar::Config::Lookup("http.response.max_body_size", (uint64_t)(64*1024*1024), "http response max body size");

static uint64_t s_http_request_buffer_size = 0;
static uint64_t s_http_request_max_body_size = 0;
static uint64_t s_http_response_buffer_size = 0;
static uint64_t s_http_response_max_body_size = 0;

namespace
{
struct _RequestSizeIniter
{
    _RequestSizeIniter()
    {
        s_http_request_buffer_size = g_http_request_buffer_size->getValue();
        s_http_request_max_body_size = g_http_request_max_body_size->getValue();
        s_http_response_buffer_size = g_http_response_buffer_size->getValue();
        s_http_response_max_body_size = g_http_response_max_body_size->getValue();

        g_http_request_buffer_size->addListener(
                [](const uint64_t& ov, const uint64_t& nv){
                s_http_request_buffer_size = nv;
        });

        g_http_request_max_body_size->addListener(
                [](const uint64_t& ov, const uint64_t& nv){
                s_http_request_max_body_size = nv;
        });

        g_http_response_buffer_size->addListener(
                [](const uint64_t& ov, const uint64_t& nv){
                s_http_response_buffer_size = nv;
        });

        g_http_response_max_body_size->addListener(
                [](const uint64_t& ov, const uint64_t& nv){
                s_http_response_max_body_size = nv;
        });
    }
};

static _RequestSizeIniter _init;
}

void on_request_method(void* data, const char* at, size_t length)
{
    HttpRequestParser* parser = static_cast<HttpRequestParser*>(data);
    HttpMethod m = CharsToHttpMethod(at);

    if(m == HttpMethod::INVALID_METHOD)
    {
        SYLAR_LOG_WARN(g_logger) << "invalid http request method: "
            << std::string(at, length);
        parser->setError(1000);
        return;
    }
    parser->getData()->setMethod(m);
}

void on_request_uri(void* data, const char* at, size_t length)
{
}

void on_request_fragment(void* data, const char* at, size_t length)
{
    HttpRequestParser* parser = static_cast<HttpRequestParser*>(data);
    parser->getData()->setFragment(std::string(at, length));
}

void on_request_path(void* data, const char* at, size_t length)
{
    HttpRequestParser* parser = static_cast<HttpRequestParser*>(data);
    parser->getData()->setPath(std::string(at, length));
}

void on_request_query(void* data, const char* at, size_t length)
{
    HttpRequestParser* parser = static_cast<HttpRequestParser*>(data);
    parser->getData()->setQuery(std::string(at, length));
}

void on_request_version(void* data, const char* at, size_t length)
{
    HttpRequestParser* parser = static_cast<HttpRequestParser*>(data);
    uint8_t v = 0;
    if(strncmp(at, "HTTP/1.1", length) == 0)
    {
        v = 0x11;
    }
    else if(strncmp(at, "HTTP/1.0", length) == 0)
    {
        v = 0x10;
    }
    else
    {
        SYLAR_LOG_WARN(g_logger) << "invalid http request version: "
            << std::string(at, length);
        parser->setError(1001);
        return;
    }
    parser->getData()->setVersion(v);
}

void on_request_header_done(void* data, const char* at, size_t length)
{
}

void on_request_http_field(void* data, const char* field, size_t flen, const char* value, size_t vlen)
{
    HttpRequestParser* parser = static_cast<HttpRequestParser*>(data);
    if(flen == 0)
    {
        SYLAR_LOG_WARN(g_logger) << "invalid http request field length == 0";
        return;
    }
    parser->getData()->setHeader(std::string(field, flen), std::string(value, vlen));
}

HttpRequestParser::HttpRequestParser()
    : m_error(0)
{
    m_data.reset(new sylar::http::HttpRequest);
    
    http_parser_init(&m_parser);
    m_parser.request_method = on_request_method;
    m_parser.request_uri = on_request_uri;
    m_parser.fragment = on_request_fragment;
    m_parser.request_path = on_request_path;
    m_parser.query_string = on_request_query;
    m_parser.http_version = on_request_version;
    m_parser.header_done = on_request_header_done;
    m_parser.http_field = on_request_http_field;
    m_parser.data = this;
}

size_t HttpRequestParser::execute(char* data, size_t len)
{
    size_t offset = http_parser_execute(&m_parser, data, len, 0);
    
    // void *memmove(void *str1, const void *str2, size_t n) 从 str2 复制 n 个字符到 str1，
    // 但是在重叠内存块这方面，memmove() 是比 memcpy() 更安全的方法。
    // 如果目标区域和源区域有重叠的话，memmove() 能够保证源串在被覆盖之前将重叠区域的字节拷贝到目标区域中，
    // 复制后源区域的内容会被更改。如果目标区域与源区域没有重叠，则和 memcpy() 函数功能相同。
    
    // 比如len=10字节，offset=3字节，data指向0字节，
    // 则意思就是将后面10-3=7个字节移动到前面去，实现所谓的“移除已解析数据”的功能
    memmove(data, data + offset, (len - offset));
    return offset;
}

int HttpRequestParser::isFinished()
{
    return http_parser_finish(&m_parser);
}

int HttpRequestParser::hasError()
{
    return m_error || http_parser_has_error(&m_parser);
}

uint64_t HttpRequestParser::getContentLength()
{
    return m_data->getHeaderAs<uint64_t>("content-length", 0);
}

uint64_t HttpRequestParser::GetHttpRequestBufferSize()
{
    return s_http_request_buffer_size;
}

uint64_t HttpRequestParser::GetHttpRequestMaxBodySize()
{
    return s_http_request_max_body_size;
}

void on_response_reason(void* data, const char* at, size_t length)
{
    HttpResponseParser* parser = static_cast<HttpResponseParser*>(data);
    parser->getData()->setReason(std::string(at, length));
}

void on_response_status(void* data, const char* at, size_t length)
{
    HttpResponseParser* parser = static_cast<HttpResponseParser*>(data);
    HttpStatus status = (HttpStatus)(atoi(at));
    parser->getData()->setStatus(status);
}

void on_response_chunk(void* data, const char* at, size_t length)
{
}

void on_response_version(void* data, const char* at, size_t length)
{
    HttpResponseParser* parser = static_cast<HttpResponseParser*>(data);
    uint8_t v = 0;
    if(strncmp(at, "HTTP/1.1", length) == 0)
    {
        v = 0x11;
    }
    else if(strncmp(at, "HTTP/1.0", length) == 0)
    {
        v = 0x10;
    }
    else
    {
        SYLAR_LOG_WARN(g_logger) << "invalid http response version: "
            << std::string(at, length);
        parser->setError(1001);
        return;
    }
    parser->getData()->setVersion(v);
}

void on_response_header_done(void* data, const char* at, size_t length)
{
}

void on_response_last_chunk(void* data, const char* at, size_t length)
{
}

void on_response_http_field(void* data, const char* field, size_t flen, const char* value, size_t vlen)
{
    HttpResponseParser* parser = static_cast<HttpResponseParser*>(data);
    if(flen == 0)
    {
        SYLAR_LOG_WARN(g_logger) << "invalid http response field length == 0";
        return;
    }
    parser->getData()->setHeader(std::string(field, flen), std::string(value, vlen));
}

HttpResponseParser::HttpResponseParser()
    : m_error(0)
{
    m_data.reset(new sylar::http::HttpResponse);
    
    httpclient_parser_init(&m_parser);
    m_parser.reason_phrase = on_response_reason;
    m_parser.status_code = on_response_status;
    m_parser.chunk_size = on_response_chunk;
    m_parser.http_version = on_response_version;
    m_parser.header_done = on_response_header_done;
    m_parser.last_chunk = on_response_last_chunk;
    m_parser.http_field = on_response_http_field;
    m_parser.data = this;
}

size_t HttpResponseParser::execute(char* data, size_t len, bool chunck)
{
    // 如果是分段的，就重新初始化一下它里面的数据，把它数据置空(清掉解析的上下文)再往下执行
    if(chunck)
    {
        httpclient_parser_init(&m_parser);
    }
    size_t offset = httpclient_parser_execute(&m_parser, data, len, 0);
    memmove(data, data + offset, (len - offset));
    return offset;
}

int HttpResponseParser::isFinished()
{
    return httpclient_parser_finish(&m_parser);
}

int HttpResponseParser::hasError()
{
    return m_error || httpclient_parser_has_error(&m_parser);
}

uint64_t HttpResponseParser::getContentLength()
{
    return m_data->getHeaderAs<uint64_t>("content-length", 0);
}

uint64_t HttpResponseParser::GetHttpResponseBufferSize()
{
    return s_http_response_buffer_size;
}

uint64_t HttpResponseParser::GetHttpResponseMaxBodySize()
{
    return s_http_response_max_body_size;
}

}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
