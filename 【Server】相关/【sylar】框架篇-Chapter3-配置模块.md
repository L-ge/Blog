# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 从配置文件中加载用户配置，灵活实现用户自己按需配置。
- 其实最简单的配置文件方式还是 .ini 那种，极其简单。YAML 的好处是支持复杂的数据类型。
- 这里使用 YAML 作为配置文件，配置名称大小写不敏感，并且支持级别格式的数据类型。
- 这里支持 STL 容器（vector, list, set, map 等等），支持自定义类型（但需要实现序列化和反序列化方法，也就是两个仿函数）。
- YAML 语法快速简单入门：https://www.runoob.com/w3cnote/yaml-intro.html


# ConfigVarBase

- 配置变量的基类。
- 虚基类。包含配置名称和配置描述两个属性。提供 toString() 和 fromString() 两个纯虚函数将参数值转换成 YAML String 和从 YAML String 转成参数的值。


# LexicalCast

- template<class F, class T>。
- 类型转换模板类(F 源类型, T 目标类型)。
- 有如下类型转换模块类(用于基本内置类型)：
	```C++
	template<class F, class T>
	class LexicalCast {
	public:
    	T operator()(const F& v) {
        	return boost::lexical_cast<T>(v);
    	}
	};
	```
- 有如下的偏特化：
	- template<class T> class LexicalCast<std::string, std::vector <T> >
	- template<class T> class LexicalCast<std::vector <T>, std::string>
	- template<class T> class LexicalCast<std::string, std::list<T> >
	- template<class T> class LexicalCast<std::list <T>, std::string>
	- template<class T> class LexicalCast<std::string, std::set <T> >
	- template<class T> class LexicalCast<std::set <T>, std::string> 
	- template<class T> class LexicalCast<std::string, std::unordered_set <T> >
	- template<class T> class LexicalCast<std::unordered_set <T>, std::string> 
	- template<class T> class LexicalCast<std::string, std::map <std::string, T> >
	- template<class T> class LexicalCast<std::map <std::string, T>, std::string> 
	- template<class T> class LexicalCast<std::string, std::unordered_map <std::string, T> >
	- template<class T> class LexicalCast<std::unordered_map <std::string, T>, std::string>
- 该模块最烦琐的部分，以上偏特化的实现均为仿函数。
- 该实现的精巧之处(模板编程的精巧之处)：比如 LexicalCast<std::string, std::vector<T> >，则先调用 LexicalCast<std::string, std::vector<T> > 版本，在处理每个元素 T 时(这里假设 T 是内置基本数据类型)，则调用 boost::lexical_cast<T>(v) 那个最基础的版本。就算 T 是 std::vector<T> 类型或 std::list<T> 等已经实现偏特化的类型，也可以做到层层嵌套去解析。


#  ConfigVar

- template<class T, class FromStr = LexicalCast<std::string, T>, class ToStr = LexicalCast<T, std::string> >。其中，T 是配置项参数，FromStr 是将 YAML String 转成参数的值的仿函数，ToStr 是将参数值转换成 YAML String 的仿函数。
- 对于每种类型的配置，在对应的 ConfigVar 模板类实例化时都要提供其 FromStr 和 ToStr 两个仿函数，用于实现该类型和 YAML 字符串的相互转换。
- 配置参数模板子类。继承自 ConfigVarBase。
- 保存对应类型 T 的参数值。
- 根据不同的非内置基本类型 T，FromStr 和 ToStr 具有不同的模板偏特化实现。
- 内含一个 std::map，存放配置变更的回调函数。
- 提供 setValue() 和 getValue() 函数，其中在 setValue() 函数中，会调用 std::map 中存放的所有回调函数进行通知配置变更。
- 关于回调函数，提供 addListener()、delListener()、getListener()、clearListener() 四个函数给外界使用。


# Config

- ConfigVar 的管理类。
- 管理所有的配置项，通过静态函数 GetDatas() 获取缓存的 std::unordered_map<std::string, ConfigVarBase::ptr>。
- 提供 Lookup(const std::string& name, const T& default_value, const std::string& description = "") 函数给外界获取/创建对应参数名的配置参数。
- 提供 Lookup(const std::string& name) 函数给外界查找配置参数。
- 提供 LoadFromYaml(const YAML::Node& root) 函数给外界使用YAML::Node初始化配置模块。
- 提供 LoadFromConfDir(const std::string& path, bool force = false) 给外界加载path文件夹里面的配置文件。
- 值得注意的是 Config 所有函数和变量均为静态的。


# 大概用法
```C++
sylar::ConfigVar<int>::ptr g_int_value_config = sylar::Config::Lookup("system.port", (int)8080, "system port");
  				|
				|
			   \|/
SYLAR_LOG_INFO(SYLAR_LOG_ROOT()) << "before: " << g_int_value_config->getValue();
  				|
				|
			   \|/
YAML::Node root = YAML::LoadFile("/home/sylar/workspace/sylar/bin/conf/test.yml");
sylar::Config::LoadFromYaml(root);
				|
				|
			   \|/
SYLAR_LOG_INFO(SYLAR_LOG_ROOT()) << "after: " << g_int_value_config->getValue();
```


# 部分相关代码

```C++
/**
 * @filename    config.h
 * @brief   配置模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-06-25
 */
#ifndef __SYLAR_CONFIG_H__
#define __SYLAR_CONFIG_H__

#include <boost/lexical_cast.hpp>
#include <yaml-cpp/yaml.h>
#include <memory>
#include <string>
#include <vector>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>

#include "log.h"

namespace sylar
{

/**
 * @brief   配置变量的基类
 */
class ConfigVarBase
{
public:
    typedef std::shared_ptr<ConfigVarBase> ptr;
    
    ConfigVarBase(const std::string& name, const std::string& description = "")
        : m_name(name)
        , m_description(description)
    {
        std::transform(m_name.begin(), m_name.end(), m_name.begin(), ::tolower);    // 转小写
    }

    virtual ~ConfigVarBase() {}

    const std::string& getName() const { return m_name; }
    const std::string& getDescription() const { return m_description; }
    
    virtual std::string toString() = 0;
    virtual bool fromString(const std::string& val) = 0;
    virtual std::string getTypeName() const = 0;

protected:
    std::string m_name;
    std::string m_description;
};

/**
 * @brief  类型转换模板类 
 *
 * @tparam F    源类型
 * @tparam T    目标类型
 */
template<class F, class T>
class LexicalCast
{
public:
    T operator()(const F& v)
    {
        return boost::lexical_cast<T>(v);
    }
};

template<class T>
class LexicalCast<std::string, std::vector<T> >
{
public:
    std::vector<T> operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        typename std::vector<T> vec;
        std::stringstream ss;
        for(size_t i=0; i<node.size(); ++i)
        {
            ss.str("");     // 清空流
            ss << node[i];
            vec.push_back(LexicalCast<std::string, T>()(ss.str()));
        }
        return vec;
    }
};

template<class T>
class LexicalCast<std::vector<T>, std::string>
{
public:
    std::string operator()(const std::vector<T>& v)
    {
        YAML::Node node(YAML::NodeType::Sequence);
        for(auto& i : v)
        {
            node.push_back(YAML::Load(LexicalCast<T, std::string>()(i)));
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

template<class T>
class LexicalCast<std::string, std::list<T> >
{
public:
    std::list<T> operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        typename std::list<T> vec;
        std::stringstream ss;
        for(size_t i=0; i<node.size(); ++i)
        {
            ss.str("");
            ss << node[i];
            vec.push_back(LexicalCast<std::string, T>()(ss.str()));
        }
        return vec;
    }
};

template<class T>
class LexicalCast<std::list<T>, std::string>
{
public:
    std::string operator()(const std::list<T>& v)
    {
        YAML::Node node(YAML::NodeType::Sequence);
        for(auto& i : v)
        {
            node.push_back(YAML::Load(LexicalCast<T, std::string>()(i)));
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

template<class T>
class LexicalCast<std::string, std::set<T> >
{
public:
    std::set<T> operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        typename std::set<T> vec;
        std::stringstream ss;
        for(size_t i=0; i<node.size(); ++i)
        {
            ss.str("");
            ss << node[i];
            vec.insert(LexicalCast<std::string, T>()(ss.str()));
        }
        return vec;
    }
};

template<class T>
class LexicalCast<std::set<T>, std::string>
{
public:
    std::string operator()(const std::set<T>& v)
    {
        YAML::Node node(YAML::NodeType::Sequence);
        for(auto& i : v)
        {
            node.push_back(YAML::Node(LexicalCast<T, std::string>()(i)));
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

template<class T>
class LexicalCast<std::string, std::unordered_set<T> >
{
public:
    std::unordered_set<T> operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        typename std::unordered_set<T> vec;
        std::stringstream ss;
        for(size_t i=0; i<node.size(); ++i)
        {
            ss.str("");
            ss << node[i];
            vec.insert(LexicalCast<std::string, T>()(ss.str()));
        }
        return vec;
    }
};

template<class T>
class LexicalCast<std::unordered_set<T>, std::string>
{
public:
    std::string operator()(const std::unordered_set<T>& v)
    {
        YAML::Node node(YAML::NodeType::Sequence);
        for(auto& i : v)
        {
            node.push_back(YAML::Load(LexicalCast<T, std::string>()(i)));
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

template<class T>
class LexicalCast<std::string, std::map<std::string, T> >
{
public:
    std::map<std::string, T> operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        typename std::map<std::string, T> vec;
        std::stringstream ss;
        for(auto it=node.begin(); it!=node.end(); ++it)
        {
            ss.str("");
            ss << it->second;
            vec.insert(std::make_pair(it->first.Scalar(), LexicalCast<std::string, T>()(ss.str())));
        }
        return vec;
    }
};

template<class T>
class LexicalCast<std::map<std::string, T>, std::string>
{
public:
    std::string operator()(const std::map<std::string, T>& v)
    {
        YAML::Node node(YAML::NodeType::Map);
        for(auto& i : v)
        {
            node[i.first] = YAML::Load(LexicalCast<T, std::string>()(i.second));
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

template<class T>
class LexicalCast<std::string, std::unordered_map<std::string, T> >
{
public:
    std::unordered_map<std::string, T> operator()(const std::string& v)
    {
        YAML::Node node = YAML::Load(v);
        typename std::unordered_map<std::string, T> vec;
        std::stringstream ss;
        for(auto it=node.begin(); it!=node.end(); ++it)
        {
            ss.str("");
            ss << it->second;
            vec.insert(std::make_pair(it->first.Scalar(), LexicalCast<std::string, T>()(ss.str())));
        }
        return vec;
    }
};

template<class T>
class LexicalCast<std::unordered_map<std::string, T>, std::string>
{
public:
    std::string operator()(const std::unordered_map<std::string, T>& v)
    {
        YAML::Node node(YAML::NodeType::Map);
        for(auto& i : v)
        {
            node[i.first] = YAML::Load(LexicalCast<T, std::string>()(i.second));
        }
        std::stringstream ss;
        ss << node;
        return ss.str();
    }
};

/**
 * @brief   配置参数模板子类，保存对应类型的参数值
 *
 * @tparam T    具体的配置参数类型
 * @tparam FromStr  从 std::string 转换成 T 类型的仿函数
 * @tparam T        从 T 类型转换成 std::string 的仿函数
 */
template<class T, 
         class FromStr = LexicalCast<std::string, T>,
         class ToStr = LexicalCast<T, std::string> >
class ConfigVar : public ConfigVarBase
{
public:
    typedef RWMutex RWMutexType;
    typedef std::shared_ptr<ConfigVar> ptr;
    typedef std::function<void (const T& old_value, const T& new_value)> on_change_cb;

    ConfigVar(const std::string& name, const T& default_value, const std::string& description = "")
        : ConfigVarBase(name, description)
        , m_val(default_value)
    {}


    /**
     * @brief   将配置值转换成 YAML String
     */
    std::string toString() override
    {
        try
        {
            RWMutexType::ReadLock lk(m_mutex);
            return ToStr()(m_val);
        }
        catch(std::exception& e)
        {
            SYLAR_LOG_ERROR(SYLAR_LOG_ROOT()) << "ConfigVar::toString exception "
                << e.what() << " convert: " << TypeToName<T>() << " to string"
                << " name=" << m_name;
        }
        return "";
    }

    /**
     * @brief  将 YAML String 转换成配置值 
     */
    bool fromString(const std::string& val) override
    {
        try
        {
            setValue(FromStr()(val));
        }
        catch(std::exception& e)
        {
            SYLAR_LOG_ERROR(SYLAR_LOG_ROOT()) << "ConfigVar::fromString exception " 
                << e.what() << " convert: string to " << TypeToName<T>()
                << " name=" << m_name
                << " - " << val;
        }
        return false;
    }

    const T getValue()
    {
        RWMutexType::ReadLock lk(m_mutex);
        return m_val;
    }

    void setValue(const T& v)
    {
        {
            RWMutexType::ReadLock lk(m_mutex);
            if(v == m_val)
            {
                return;
            }

            // 遍历调用回调函数，通知参数变更
            for(auto& i : m_cbs)
            {
                i.second(m_val, v);
            }
        }
        RWMutexType::WriteLock lk(m_mutex);
        m_val = v;
    }

    std::string getTypeName() const override
    {
        return TypeToName<T>();
    }

    uint64_t addListener(on_change_cb cb)
    {
        static uint64_t s_fun_id = 0;
        RWMutexType::WriteLock lk(m_mutex);
        ++s_fun_id;
        m_cbs[s_fun_id] = cb;
        return s_fun_id;
    }

    void delListener(uint64_t key)
    {
        RWMutexType::WriteLock lk(m_mutex);
        m_cbs.erase(key);
    }

    void clearListener()
    {
        RWMutexType::WriteLock lk(m_mutex);
        m_cbs.clear();
    }

    on_change_cb getListener(uint64_t key)
    {
        RWMutexType::ReadLock lk(m_mutex);
        auto it = m_cbs.find(key);
        return it == m_cbs.end() ? nullptr : it->second;
    }

private:
    RWMutexType m_mutex;
    T m_val;
    std::map<uint64_t, on_change_cb> m_cbs;
};

/**
 * @brief   ConfigVar 的管理类
 */
class Config
{
public:
    typedef std::unordered_map<std::string, ConfigVarBase::ptr> ConfigVarMap;
    typedef RWMutex RWMutexType;

    template<class T>
    static typename ConfigVar<T>::ptr Lookup(const std::string& name,
            const T& default_value, const std::string& description = "")
    {
        RWMutexType::WriteLock lk(GetMutex());
        auto it = GetDatas().find(name);
        if(it != GetDatas().end())
        {
            auto tmp = std::dynamic_pointer_cast<ConfigVar<T> >(it->second);
            if(tmp)
            {
                SYLAR_LOG_INFO(SYLAR_LOG_ROOT()) << "Lookup name=" << name << " exists";
                return tmp;
            }
            else
            {
                SYLAR_LOG_ERROR(SYLAR_LOG_ROOT()) << "Lookup name=" << name
                    << " exists but type not " << TypeToName<T>()
                    << " real_type=" << it->second->getTypeName()
                    << " " << it->second->toString();
                return nullptr;
            }
        }

        // 判断是否含有非法字符
        if(name.find_first_not_of("abcdefghijklmnopqrstuvwxyz._012345678")
                != std::string::npos)
        {
            SYLAR_LOG_ERROR(SYLAR_LOG_ROOT()) << "Lookup name invalid " << name;
            throw std::invalid_argument(name);
        }

        typename ConfigVar<T>::ptr v(new ConfigVar<T>(name, default_value, description));
        GetDatas()[name] = v;
        return v;
    }

    template<class T>
    static typename ConfigVar<T>::ptr Lookup(const std::string& name)
    {
        RWMutexType::ReadLock lk(GetMutex());
        auto it = GetDatas().end();
        if(it == GetDatas().end())
        {
            return nullptr;
        }
        return std::dynamic_pointer_cast<ConfigVar<T> >(it->second);
    }

    static void LoadFromYaml(const YAML::Node& root);

    /**
     * @brief   从配置文件加载配置到缓存
     *
     * @param   path    相对路径
     * @param   force   是否强制更新(不管配置文件是否有变化)，默认为 false
     */
    static void LoadFromConfDir(const std::string& path, bool force = false);
    
    static ConfigVarBase::ptr LookupBase(const std::string& name);
    
    /**
     * @brief   遍历缓存的所有配置项
     *
     * @param   cb  回调函数，形参为每个配置项
     */
    static void Visit(std::function<void(ConfigVarBase::ptr)> cb);

private:
    static ConfigVarMap& GetDatas()
    {
        static ConfigVarMap s_datas;
        return s_datas;
    }

    static RWMutexType& GetMutex()
    {
        static RWMutexType s_mutex;
        return s_mutex;
    }
};

}

#endif


#include "config.h"
#include "env.h"

namespace sylar
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

ConfigVarBase::ptr Config::LookupBase(const std::string& name)
{
    RWMutexType::ReadLock lk(GetMutex());
    auto it = GetDatas().find(name);
    return it == GetDatas().end() ? nullptr : it->second;
}

static void ListAllMember(const std::string& prefix,
                          const YAML::Node& node,
                          std::list<std::pair<std::string, const YAML::Node> >& output)
{
    if(prefix.find_first_not_of("abcdefghijklmnopqrstuvwxyz._012345678")
            != std::string::npos)
    {
        SYLAR_LOG_ERROR(g_logger) << "Config invalid name: " << prefix << " : " << node;
        SYLAR_LOG_ERROR(g_logger) << prefix << ":" << prefix.find_first_not_of("abcdefghijklmnopqrstuvwxyz._012345678");
        return;
    }

    output.push_back(std::make_pair(prefix, node));
    if(node.IsMap())
    {
        for(auto it=node.begin(); it!=node.end(); ++it)
        {
            ListAllMember(prefix.empty() ? it->first.Scalar() 
                    : prefix + "." + it->first.Scalar(), it->second, output);
        }
    }
}

void Config::LoadFromYaml(const YAML::Node& root)
{
    std::list<std::pair<std::string, const YAML::Node> > all_nodes;
    ListAllMember("", root, all_nodes);     // 平铺所有配置项到 all_nodes 中

    for(auto& i : all_nodes)    // 逐个更新配置
    {
        std::string key = i.first;
        if(key.empty())
        {
            continue;
        }

        std::transform(key.begin(), key.end(), key.begin(), ::tolower);
        ConfigVarBase::ptr var = LookupBase(key);

        if(var)
        {
            if(i.second.IsScalar())
            {
                var->fromString(i.second.Scalar());
            }
            else
            {
                std::stringstream ss;
                ss << i.second;
                var->fromString(ss.str());
            }
        }
    }
}

static std::map<std::string, uint64_t> s_file2modifytime;
static sylar::Mutex s_mutex;

void Config::LoadFromConfDir(const std::string& path, bool force)
{
    std::string absoulte_path = sylar::EnvMgr::GetInstance()->getAbsolutePath(path);
    std::vector<std::string> files;
    FSUtil::ListAllFile(files, absoulte_path, ".yml");  // 获取绝对路径下所有配置文件

    for(auto& i : files)
    {
        // 判断文件修改时间是否有变化，变化才更新
        {
            struct stat st;
            lstat(i.c_str(), &st);
            sylar::Mutex::Lock lk(s_mutex);
            if(!force && s_file2modifytime[i] == (uint64_t)st.st_mtime)
            {
                continue;
            }
            s_file2modifytime[i] = st.st_mtime;
        }

        try
        {
            YAML::Node root = YAML::LoadFile(i);
            LoadFromYaml(root);
            SYLAR_LOG_INFO(g_logger) << "LoadConfFile file=" << i << " ok";
        }
        catch(...)
        {
            SYLAR_LOG_ERROR(g_logger) << "LoadConfFile file=" << i << " failed";
        }
    }
}

void Config::Visit(std::function<void(ConfigVarBase::ptr)> cb)
{
    RWMutexType::ReadLock lk(GetMutex());
    ConfigVarMap& m = GetDatas();
    for(auto it=m.begin(); it!=m.end(); ++it)
    {
        cb(it->second);
    }
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
