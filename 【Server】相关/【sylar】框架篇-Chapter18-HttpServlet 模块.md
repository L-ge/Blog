# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- HTTPServlet 模块。
- 提供 HTTP 请求路径到处理类的映射，用于规范化的 HTTP 消息处理流程。
- HTTP Servlet 包括两部分，第一部分是 Servlet 对象，每个 Servlet 对象表示一种处理 HTTP 消息的方法；第二部分是 ServletDispatch，它包含一个请求路径到 Servlet 对象的映射，用于指定一个请求路径该用哪个 Servlet 来处理。

# Servlet

- 纯虚基类，子类必须重写其虚方法 handleClient。


# FunctionServlet

- 函数式 Servlet。
- 继承自 Servlet。
- 内含回调函数，其 handleClient 方法其实是直接调用回调函数。


# NotFoundServlet

- ServletDispatch 默认的 Servlet，404 那种。
- 继承自 Servlet。


# IServletCreator

- 纯虚基类，子类必须重写其 get() 和 getName() 方法。


# HoldServletCreator

- 继承自 IServletCreator。


# ServletCreator

- 继承自 IServletCreator。
- 模板类，根据类型创建 Servlet。


# ServletDispatch

- Servlet 分发器。
- 继承自 Servlet。
- 内含精准匹配的 Servlet map 和 模糊匹配的 Servlet vector，并提供增、删、查方法。


# 部分相关代码
```C++
/**
 * @filename    servlet.h
 * @brief   Servlet 模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-16
 */
#ifndef __SYLAR_HTTP_SERVLET_H__
#define __SYLAR_HTTP_SERVLET_H__

#include <memory>
#include <functional>
#include <string>
#include <vector>
#include <unordered_map>
#include "http.h"
#include "http_session.h"
#include "sylar/mutex.h"
#include "sylar/util.h"

namespace sylar
{

namespace http
{

/**
 * @brief   Servlet封装
 */
class Servlet
{
public:
    typedef std::shared_ptr<Servlet> ptr;

    Servlet(const std::string& name)
        : m_name(name)
    {}
    virtual ~Servlet() {}

    /**
     * @brief   处理请求
     *
     * @param   request http请求
     * @param   rsponse http响应
     * @param   session http连接
     *
     * @return  是否处理成功
     */
    virtual int32_t handle(sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session) = 0;

    const std::string& getName() const { return m_name; }

protected:
    /// 名称(一般用于打印日志区分不同的Servlet)
    std::string m_name;
};

/**
 * @brief   函数式Servlet
 */
class FunctionServlet : public Servlet
{
public:
    typedef std::shared_ptr<FunctionServlet> ptr;

    /// 函数回调类型的定义
    typedef std::function< int32_t (sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session)> callback;

    /**
     * @brief   构造函数
     *
     * @param   cb  回调函数
     */
    FunctionServlet(callback cb);

    virtual int32_t handle(sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session) override;

private:
    /// 回调函数
    callback m_cb;
};

class IServletCreator
{
public:
    typedef std::shared_ptr<IServletCreator> ptr;
    virtual ~IServletCreator() {}
    virtual Servlet::ptr get() const = 0;
    virtual std::string getName() const = 0;
};

class HoldServletCreator : public IServletCreator 
{
public:
    typedef std::shared_ptr<HoldServletCreator> ptr;
    
    HoldServletCreator(Servlet::ptr slt)
        : m_servlet(slt) 
    {}

    Servlet::ptr get() const override { return m_servlet; }
    std::string getName() const override { return m_servlet->getName(); }

private:
    Servlet::ptr m_servlet;
};

template<class T>
class ServletCreator : public IServletCreator
{
public:
    typedef std::shared_ptr<ServletCreator> ptr;

    ServletCreator() {}

    Servlet::ptr get() const override { return Servlet::ptr(new T); }
    std::string getName() const override { return TypeToName<T>(); }
};

/**
 * @brief   Servlet分发器
 */
class ServletDispatch : public Servlet
{
public:
    typedef std::shared_ptr<ServletDispatch> ptr;
    typedef RWMutex RWMutexType;

    ServletDispatch();

    virtual int32_t handle(sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session) override;

    void addServlet(const std::string& uri, Servlet::ptr slt);
    void addServlet(const std::string& uri, FunctionServlet::callback cb);

    /**
     * @brief   添加模糊匹配Servlet
     *
     * @param   uri 模糊匹配 /sylar_*
     * @param   slt Servlet
     */
    void addGlobServlet(const std::string& uri, Servlet::ptr slt);
    
    /**
     * @brief   添加模糊匹配Servlet
     *
     * @param   uri 模糊匹配 /sylar_*
     * @param   cb  FunctionServlet回调函数
     */
    void addGlobServlet(const std::string& uri, FunctionServlet::callback cb);
    void addServletCreatot(const std::string& uri, IServletCreator::ptr creator);
    void addGlobServletCreatot(const std::string& uri, IServletCreator::ptr creator);

    template<class T>
    void addServletCreator(const std::string& uri)
    {
        addServletCreator(uri, std::make_shared<ServletCreator<T> >());
    }

    template<class T>
    void addGlobServletCreator(const std::string& uri)
    {
        addGlobServletCreator(uri, std::make_shared<ServletCreator<T> >());
    }

    void delServlet(const std::string& uri);
    void delGlobServlet(const std::string& uri);

    Servlet::ptr getDefault() const { return m_default; }
    void setDefault(Servlet::ptr v) { m_default = v; }

    Servlet::ptr getServlet(const std::string& uri);
    Servlet::ptr getGlobServlet(const std::string& uri);

    /**
     * @brief   通过uri获取Servlet
     *
     * @param   uri uri
     *
     * @return  优先精准匹配，其次模糊匹配，最后返回默认
     */
    Servlet::ptr getMatchedServlet(const std::string& uri);

    void listAllServletCreator(std::map<std::string, IServletCreator::ptr>& infos);
    void listAllGlobServletCreator(std::map<std::string, IServletCreator::ptr>& infos);

private:
    /// 读写互斥量
    RWMutexType m_mutex;
    /// 精准匹配Servlet map
    /// uri(/sylar/xxx) -> servlet
    std::unordered_map<std::string, IServletCreator::ptr> m_datas;
    /// 模糊匹配Servlet 数组
    /// uri(/sylar/*) -> servlet
    std::vector<std::pair<std::string, IServletCreator::ptr> > m_globs;
    /// 默认servlet，所有路径都没匹配到时使用 
    Servlet::ptr m_default;
};

/**
 * @brief   NotFoundServlet(默认返回404)
 */
class NotFoundServlet : public Servlet
{
public:
    typedef std::shared_ptr<NotFoundServlet> ptr;

    NotFoundServlet(const std::string& name);

    virtual int32_t handle(sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session) override;

private:
    std::string m_name;
    std::string m_content;
};

}

}

#endif


#include "servlet.h"
#include <fnmatch.h>

namespace sylar
{

namespace http
{

FunctionServlet::FunctionServlet(callback cb)
    : Servlet("FunctionServlet")
    , m_cb(cb)
{
}

int32_t FunctionServlet::handle(sylar::http::HttpRequest::ptr request
                     , sylar::http::HttpResponse::ptr response
                     , sylar::http::HttpSession::ptr session)
{
    return m_cb(request, response, session);
}

ServletDispatch::ServletDispatch()
    : Servlet("ServletDispatch")
{
    m_default.reset(new NotFoundServlet("sylar/1.0"));
}

int32_t ServletDispatch::handle(sylar::http::HttpRequest::ptr request
                     , sylar::http::HttpResponse::ptr response
                     , sylar::http::HttpSession::ptr session)
{
    // 通过请求的路径拿到Servlet进行处理
    auto slt = getMatchedServlet(request->getPath());
    if(slt)
    {
        slt->handle(request, response, session);
    }
    return 0;
}

void ServletDispatch::addServlet(const std::string& uri, Servlet::ptr slt)
{
    RWMutexType::WriteLock lk(m_mutex);
    m_datas[uri] = std::make_shared<HoldServletCreator>(slt);
}

void ServletDispatch::addServlet(const std::string& uri, FunctionServlet::callback cb)
{
    RWMutexType::WriteLock lk(m_mutex);
    m_datas[uri] = std::make_shared<HoldServletCreator>(std::make_shared<FunctionServlet>(cb));
}

void ServletDispatch::addGlobServlet(const std::string& uri, Servlet::ptr slt)
{
    RWMutexType::WriteLock lk(m_mutex);
    for(auto it = m_globs.begin(); it != m_globs.end(); ++it) 
    {
        // 把原来的删掉再把新的加进去
        if(it->first == uri) 
        {
            m_globs.erase(it);
            break;
        }
    }
    m_globs.push_back(std::make_pair(uri, std::make_shared<HoldServletCreator>(slt)));
}

void ServletDispatch::addGlobServlet(const std::string& uri, FunctionServlet::callback cb)
{
    return addGlobServlet(uri, std::make_shared<FunctionServlet>(cb));
}

void ServletDispatch::addServletCreatot(const std::string& uri, IServletCreator::ptr creator)
{
    RWMutexType::WriteLock lk(m_mutex);
    m_datas[uri] = creator;
}

void ServletDispatch::addGlobServletCreatot(const std::string& uri, IServletCreator::ptr creator)
{
    RWMutexType::WriteLock lk(m_mutex);
    for(auto it = m_globs.begin(); it != m_globs.end(); ++it)
    {
        if(it->first == uri) 
        {
            m_globs.erase(it);
            break;
        }
    }
    m_globs.push_back(std::make_pair(uri, creator));
}

void ServletDispatch::delServlet(const std::string& uri)
{
    RWMutexType::WriteLock lk(m_mutex);
    m_datas.erase(uri);
}

void ServletDispatch::delGlobServlet(const std::string& uri)
{
    RWMutexType::WriteLock lk(m_mutex);
    for(auto it = m_globs.begin(); it != m_globs.end(); ++it)
    {
        if(it->first == uri)
        {
            m_globs.erase(it);
            break;
        }
    }
}

Servlet::ptr ServletDispatch::getServlet(const std::string& uri)
{
    RWMutexType::ReadLock lk(m_mutex);
    auto it = m_datas.find(uri);
    return it == m_datas.end() ? nullptr : it->second->get();
}

Servlet::ptr ServletDispatch::getGlobServlet(const std::string& uri)
{
    RWMutexType::ReadLock lk(m_mutex);
    for(auto it = m_globs.begin(); it != m_globs.end(); ++it) 
    {
        if(it->first == uri)
        {
            return it->second->get();
        }
    }
    return nullptr;
}

Servlet::ptr ServletDispatch::getMatchedServlet(const std::string& uri)
{
    RWMutexType::ReadLock lk(m_mutex);
    auto mit = m_datas.find(uri);
    if(mit != m_datas.end())                // 先精准
    {
        return mit->second->get();
    }

    for(auto it = m_globs.begin(); it != m_globs.end(); ++it)   // 再模糊
    {
        // fnmatch(pattern, string, flags);
        // pattern: 要检索的模式；string: 要检查的字符串或文件；flags: 可选
        if(!fnmatch(it->first.c_str(), uri.c_str(), 0)) 
        {
            return it->second->get();
        }
    }
    return m_default;       // 最后默认
}

void ServletDispatch::listAllServletCreator(std::map<std::string, IServletCreator::ptr>& infos)
{
    RWMutexType::ReadLock lk(m_mutex);
    for(auto& i : m_datas) 
    {
        infos[i.first] = i.second;
    }
}

void ServletDispatch::listAllGlobServletCreator(std::map<std::string, IServletCreator::ptr>& infos)
{
    RWMutexType::ReadLock lk(m_mutex);
    for(auto& i : m_globs) 
    {
        infos[i.first] = i.second;
    }
}

NotFoundServlet::NotFoundServlet(const std::string& name)
    : Servlet("NotFoundServlet")
    , m_name(name)
{
    m_content = "<html><head><title>404 Not Found"
                "</title></head><body><center><h1>404 Not Found</h1></center>"
                "<hr><center>" + name + "</center></body></html>";
}

int32_t NotFoundServlet::handle(sylar::http::HttpRequest::ptr request
                     , sylar::http::HttpResponse::ptr response
                     , sylar::http::HttpSession::ptr session)
{
    response->setStatus(sylar::http::HttpStatus::NOT_FOUND);
    response->setHeader("Server", "sylar/1.0.0");
    response->setHeader("Content-Type", "text/html");
    response->setBody(m_content);
    return 0;
}

}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
