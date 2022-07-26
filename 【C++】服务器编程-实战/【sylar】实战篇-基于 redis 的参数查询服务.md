# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)

# 需求
- 设计上考虑一亿条记录。
- Http Get请求，json格式返回。
- 要求内存越小越好、要求在网络正常的情况下查询耗时不超过20ms。
- 参数表格式如下：
	```
	cardId	type	status
	44016758092746254872	7	2
	45018537070476554829	6	2
	...
	```
	- 其中，cardId 为字符串；type 为不超过 10 的纯数字；status 为 1 或 2。

# 设计思想
- 一亿条记录，对耗时有要求，因此用 redis 实现。
- 对内存使用有要求，因此利用 redis 中 hash 数据结构，控制数据使其内部编码为 ziplist。
- 指令组成：hset key field value
	- key 的计算方法为：
		```
		方式一：
		std::hash<std::string> str_hash;
		key = std::to_string(str_hash(sKey) % 300000);
		```
		- 哈希类型的内部编码有两种：
			- ziplist(压缩列表)：当哈希类型元素个数小于hash-max-ziplist-entries配置(默认512个)、同时所有值都小于hash-max-ziplist-value配置(默认64字节)时，Redis会使用ziplist作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀。
			- hashtable(哈希表)：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)。
		- 因此，1亿条记录，每个桶大概装500对键值对，大概需要200000个桶，为避免hash冲突，取300000个桶，因此取余300000。
	- field 的计算方法为：
		```
		unsigned int BKDRHash(const char *str) {
			unsigned int seed = 131; // 31 131 1313 13131 131313 etc..
			unsigned int hash = 0;
			while (*str) {
				hash = hash * seed + (*str++);
			}
			return (hash & 0x7FFFFFFF);
		}
		unsigned int nFieldKey = BKDRHash(sKey.c_str());
		```
		- 落到同一个桶的记录都是第一次求hash值取余后碰撞上的，因此这一步求field，使用另一个hash函数求值。
	- value 的计算方法为：
		```
		void encodeValue(const std::string& sStatus, const std::string& sType, std::string& sResult)
		{
			unsigned long nStatus = std::stoul(sStatus);
			unsigned long nType = std::stoul(sType);
			unsigned long nHashValue = 0x00;
			nHashValue = (nStatus << 6) & 0xC0;
			nHashValue |= (nType & 0x3F);
			sResult = std::to_string(nHashValue);
		}
		
		void decodeValue(std::string& sStatus, std::string& sType, const std::string& sResult)
		{
			unsigned long nHashValue = std::stoul(sResult);
			unsigned long nStatus = (nHashValue & 0xC0) >> 6;
			unsigned long nType = nHashValue & 0x3F;
			sStatus = std::to_string(nStatus);
			sType = std::to_string(nType);
		}
		```
		- type和status合并成value，且仅取一个字节（最低位字节）。
		- value的最低位字节的低6位用来表示type，最低位字节的高2位用来表示status，冗余空间用做日后扩展。
- 参数下发后放到指定目录，独立线程定时检查是否有参数更新，如果有参数更新，则预处理成redis协议的参数格式，例如：
	```
	// 预处理前
	cardId	type	status
	44016758092746254872	7	2
	
	// 预处理后
	*4
	$4
	hset
	$6
	252724
	$9
	143964783
	$3
	135
	```
	- 这里有个小坑，预处理的时候：windows下用\n，Linux下用\r\n，否则有如下报错：
		`ERR Protocol error: invalid multibulk length`
- 数据量较大，因此预处理后redis协议格式的参数文件通过如下命令进行导入到redis-server中：
	`cat xxx.dat | redis-cli --pipe`
- 请求和回复样式：
	```
	url:http://192.168.60.138:8020/paramquery?cardid=2030230000000002&cardnet=4401&cmdtype=blacklistquery&serialno=1005&tbtype=0&version=201708031
	ret:{"cmdtype":"blacklistqueryresult","errorcode":"0","recordcnt":"0","serialno":"1005","version":"201708031"}


	url:http://192.168.60.138:8020/paramquery?cardid=5186100181001473&cardnet=4401&cmdtype=blacklistquery&serialno=1011&tbtype=0&version=201708031
	ret:{"cmdtype":"blacklistqueryresult","darkstatus0":"1","darktype0":"7","darkver0":"1970-01-01 08:00:00","errorcode":"0","recordcnt":"1","serialno":"1011","status":"1","type":"7","version":"201708031"}
	```

# 测试情况
- 虚拟机概况：CentOs7、4核处理器(2×2)、4GB内存、固态硬盘。
- 数据量1亿，预处理耗时：251622ms ≈ 4.19分钟。
- 预处理前参数文件大小2.32GB，预处理后参数文件大小4.7GB。
- 每个桶大概300+个键值对，ziplist编码。
- 预处理后参数导入redis耗时：387242ms ≈ 6.45分钟。
- 重新启动后参数导入redis耗时：408096ms = 6.8分钟。
- 查询耗时：2-6ms之间。
- 对预处理后的hash结果与预处理前参数文件进行数据检验，发现误差情况如下：
	- 第一次和第二次hash都一样的记录有 16 条。
	- **可以选用更好的hash函数使其碰撞率降低。(待完善、待测试)**
		- 例如，由于数据的离散度较高，直接取cardId的后6位作为field。
- 具体运行情况：
	```
	# 加载参数前 Memory
	used_memory:595720				// Redis分配器分配的内存总量，也就是内部存储的所有数据内存占用量
	used_memory_human:581.76K		// 以可读的格式返回used_memory
	used_memory_rss:10936320
	used_memory_rss_human:10.43M	// 从操作系统的角度，Redis进程占用的物理内存总量
	used_memory_peak:595720
	used_memory_peak_human:581.76K	// 内存使用的最大值，表示used_memory的峰值
	used_memory_peak_perc:100.01%
	used_memory_overhead:579522
	used_memory_startup:512904
	used_memory_dataset:16198
	used_memory_dataset_perc:19.56%
	allocator_allocated:1219304
	allocator_active:1552384
	allocator_resident:8634368
	total_system_memory:3953926144
	total_system_memory_human:3.68G
	used_memory_lua:37888
	used_memory_lua_human:37.00K
	used_memory_scripts:0
	used_memory_scripts_human:0B
	number_of_cached_scripts:0
	maxmemory:0
	maxmemory_human:0B
	maxmemory_policy:noeviction
	allocator_frag_ratio:1.27
	allocator_frag_bytes:333080
	allocator_rss_ratio:5.56
	allocator_rss_bytes:7081984
	rss_overhead_ratio:1.27
	rss_overhead_bytes:2301952
	mem_fragmentation_ratio:19.72		// used_memory_rss/used_memory比值，表示内存碎片率
	mem_fragmentation_bytes:10381624
	mem_not_counted_for_evict:0
	mem_replication_backlog:0
	mem_clients_slaves:0
	mem_clients_normal:66618
	mem_aof_buffer:0
	mem_allocator:jemalloc-5.1.0
	active_defrag_running:0
	lazyfree_pending_objects:0

	
	# 更新参数后 Memory
	used_memory:1053488904
	used_memory_human:1004.69M
	used_memory_rss:1356754944
	used_memory_rss_human:1.26G
	used_memory_peak:1053488904
	used_memory_peak_human:1004.69M
	used_memory_peak_perc:100.00%
	used_memory_overhead:16773826
	used_memory_startup:512904
	used_memory_dataset:1036715078
	used_memory_dataset_perc:98.46%
	allocator_allocated:1053561192
	allocator_active:1305292800
	allocator_resident:1357246464
	total_system_memory:3953926144
	total_system_memory_human:3.68G
	used_memory_lua:37888
	used_memory_lua_human:37.00K
	used_memory_scripts:0
	used_memory_scripts_human:0B
	number_of_cached_scripts:0
	maxmemory:0
	maxmemory_human:0B
	maxmemory_policy:noeviction
	allocator_frag_ratio:1.24
	allocator_frag_bytes:251731608
	allocator_rss_ratio:1.04
	allocator_rss_bytes:51953664
	rss_overhead_ratio:1.00
	rss_overhead_bytes:-491520
	mem_fragmentation_ratio:1.29
	mem_fragmentation_bytes:303307056
	mem_not_counted_for_evict:0
	mem_replication_backlog:0
	mem_clients_slaves:0
	mem_clients_normal:66618
	mem_aof_buffer:0
	mem_allocator:jemalloc-5.1.0
	active_defrag_running:0
	lazyfree_pending_objects:0

	
	# top指令查看结果
	 PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
	3168 bread     20   0 1869256   1.3g   1284 S   0.3 34.3   6:27.41 redis-server
	```

# 其他事项

- 避免持久化阻塞redis，redis配置为不持久化(redis.conf)：
	```
	save 900 1	--->	# save 900 1
	save 300 10	--->	# save 300 10
	save 60 10000	--->	# save 60 10000
	```
- 为能远程到redis-server，修改如下配置(redis.conf)：
	```
	bind 127.0.0.1	--->	# bind 127.0.0.1
	protected-mode yes	--->	protected-mode no	
	```
	
# 框架相关代码
```C++
// my_module.h
#include "sylar/module.h"

namespace paramquery {

class MyModule : public sylar::Module
{
public:
    typedef std::shared_ptr<MyModule> ptr;
    
    MyModule();
    
    bool onLoad() override;
    bool onUnload() override;
    bool onServerReady() override;
    bool onServerUp() override;
};

}


// my_module.cc
#include "my_module.h"
#include "sylar/config.h"
#include "sylar/log.h"
#include "sylar/application.h"
#include "sylar/env.h"

#include "paramquery_servlet.h"

namespace paramquery 
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_ROOT();

MyModule::MyModule()
    : sylar::Module("paramquery", "1.0", "") 
{
}

bool MyModule::onLoad() 
{
    SYLAR_LOG_INFO(g_logger) << "onLoad";
    return true;
}

bool MyModule::onUnload() 
{
    SYLAR_LOG_INFO(g_logger) << "onUnload";
    return true;
}


bool MyModule::onServerReady() 
{
    SYLAR_LOG_INFO(g_logger) << "onServerReady";
    
    std::vector<sylar::TcpServer::ptr> svrs;
    if(!sylar::Application::GetInstance()->getServer("http", svrs)) 
    {
        SYLAR_LOG_INFO(g_logger) << "no httpserver alive";
        return false;
    }

    for(auto& i : svrs)
    {
        sylar::http::HttpServer::ptr http_server = std::dynamic_pointer_cast<sylar::http::HttpServer>(i);
        if(!i) 
        {
            continue;
        }
        
        auto slt_dispatch = http_server->getServletDispatch();

        sylar::http::ParamQueryServlet::ptr slt(new sylar::http::ParamQueryServlet(
                    sylar::EnvMgr::GetInstance()->getCwd()
        ));
        slt_dispatch->addGlobServlet("/paramquery", slt);
        SYLAR_LOG_INFO(g_logger) << "addServlet";
    }

    return true;
}

bool MyModule::onServerUp()
{
    SYLAR_LOG_INFO(g_logger) << "onServerUp";
    return true;
}

}

extern "C" {

sylar::Module* CreateModule()
{
    sylar::Module* module = new paramquery::MyModule;
    SYLAR_LOG_INFO(paramquery::g_logger) << "CreateModule " << module;
    return module;
}

void DestoryModule(sylar::Module* module) {
    SYLAR_LOG_INFO(paramquery::g_logger) << "DestoryModule " << module;
    delete module;
}

}


// paramquery_servlet.h
#ifndef __PARAMQUERY_SERVLET_H__
#define __PARAMQUERY_SERVLET_H__

#include "sylar/http/servlet.h"
#include "ahandleparam.h"

namespace sylar 
{

namespace http 
{

class ParamQueryServlet : public sylar::http::Servlet
{
public:
    typedef std::shared_ptr<ParamQueryServlet> ptr;
    
    ParamQueryServlet(const std::string& sPath);

    virtual int32_t handle(sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session) override;

private:
    std::string m_sPath;
    paramquery::AHandleParam::ptr m_pHandleParam;
};

}

}

#endif


// paramquery_servlet.cc
#include "paramquery_servlet.h"
#include "sylar/log.h"
#include <iostream>
#include <fstream>

namespace sylar 
{

namespace http 
{

static sylar::Logger::ptr g_logger = SYLAR_LOG_ROOT();

ParamQueryServlet::ParamQueryServlet(const std::string& sPath)
    : Servlet("ParamQueryServlet")
    , m_sPath(sPath)
{
    m_pHandleParam.reset(new paramquery::AHandleParam);
}

int32_t ParamQueryServlet::handle(sylar::http::HttpRequest::ptr request
                         , sylar::http::HttpResponse::ptr response
                         , sylar::http::HttpSession::ptr session)
{
    auto path = m_sPath + "/" + request->getPath();
    SYLAR_LOG_INFO(g_logger) << "handle path=" << path;
    if(path.find("..") != std::string::npos) 
    {
        response->setBody("invalid path");
        response->setStatus(sylar::http::HttpStatus::NOT_FOUND);
        return 0;
    }

    std::string sQuery = request->getQuery();
    SYLAR_LOG_INFO(g_logger) << "handle query=" << sQuery;
    
    // 查询参数
    std::string sResult = "";
    m_pHandleParam->queryParam(sQuery, sResult);
    
    // 回复结果
    std::stringstream ss;
    ss << sResult;
    response->setBody(ss.str());
    response->setHeader("content-type", "text/html;charset=utf-8");
    return 0;
}

}

}
```

# 业务关键代码
- 主要是 ahandleparam.h 和 ahandleparam.cc
	```
	// ahandleparam.h
	#ifndef AHANDLEPARAM_H
	#define AHANDLEPARAM_H

	#include <memory>
	#include <thread>
	#include <map>
	#include "aredisclient.h"

	namespace paramquery
	{

	class AHandleParam
	{
	public:
		typedef std::shared_ptr<AHandleParam> ptr;
		
		AHandleParam();
		~AHandleParam();

		int queryParam(const std::string& sQueryCond, std::string& sQueryResult);

	private:
		// 启动redis-server
		int startRedisServer();
		// 检查指定目录下是否有参数要更新
		void checkUpdateParam();
		// 对参数进行预处理
		int prefixParam(const std::string& sAbsoluteFilePath);
		// 对预处理后的参数加载到redis
		int loadParamToRedis(const std::string& sAbsoluteFilePath);
		int startLoadParamToRedis();

		std::map<std::string, std::string> getQueryCond(const std::string& sQueryCond);
		std::map<std::string, std::string> queryCardId(const std::string& sCardId);

	private:
		ARedisClient* m_pRedisClient;

		std::thread m_threadCheckUpdateParam;
		bool m_bStopThreadCheckUpdateParam;
	};

	#endif // AHANDLEPARAM_H

	}

	// ahandleparam.cc
	#include "ahandleparam.h"
	#include "sylar/log.h"
	#include "sylar/env.h"
	#include "sylar/util.h"
	#include <chrono>
	#include <fstream>
	#include <algorithm>

	namespace paramquery
	{

	static sylar::Logger::ptr g_logger = SYLAR_LOG_ROOT();

	std::vector<std::string> split(std::string str,std::string pattern)
	{
		std::string::size_type pos;
		std::vector<std::string> result;
		str += pattern;//扩展字符串以方便操作
		int size = str.size();
		for(int i=0; i<size; ++i)
		{
			pos = str.find(pattern,i);
			if((int)pos < size)
			{
				std::string s = str.substr(i, pos-i);
				result.push_back(s);
				i = pos + pattern.size() - 1;
			}
		}

		return result;
	}

	void encodeValue(const std::string& sStatus, const std::string& sType, std::string& sResult)
	{
		unsigned long nStatus = std::stoul(sStatus);
		unsigned long nType = std::stoul(sType);
		unsigned long nHashValue = 0x00;
		nHashValue = (nStatus << 6) & 0xC0;
		nHashValue |= (nType & 0x3F);
		sResult = std::to_string(nHashValue);
	}

	void decodeValue(std::string& sStatus, std::string& sType, const std::string& sResult)
	{
		unsigned long nHashValue = std::stoul(sResult);
		unsigned long nStatus = (nHashValue & 0xC0) >> 6;
		unsigned long nType = nHashValue & 0x3F;
		sStatus = std::to_string(nStatus);
		sType = std::to_string(nType);
	}

	// BKDR Hash Function
	unsigned int BKDRHash(const char *str)
	{
		unsigned int seed = 131; // 31 131 1313 13131 131313 etc..
		unsigned int hash = 0;

		while (*str)
		{
			hash = hash * seed + (*str++);
		}

		return (hash & 0x7FFFFFFF);
	}

	AHandleParam::AHandleParam()
	{
		if(0 != startRedisServer())
		{
			SYLAR_LOG_ERROR(g_logger) << "start redis server fail";
			// TODO 抛异常？
		}

		m_pRedisClient = new ARedisClient("127.0.0.1", 6379);

		// 等redis初始化完成，等100*10ms
		for(int i=0; i<100; ++i)
		{
			if(false == m_pRedisClient->connectToServer())
			{
				SYLAR_LOG_ERROR(g_logger) << "connect redis server fail";
				// TODO 抛异常？
				std::this_thread::sleep_for(std::chrono::milliseconds(10));
			}
			else
			{
				if(0 != startLoadParamToRedis())
					SYLAR_LOG_ERROR(g_logger) << "load param to redis fail";
				break;
			}
		}

		m_bStopThreadCheckUpdateParam = false;
		m_threadCheckUpdateParam = std::thread(&AHandleParam::checkUpdateParam, this);
	}

	AHandleParam::~AHandleParam()
	{
		m_bStopThreadCheckUpdateParam = true;
		if(m_threadCheckUpdateParam.joinable())
		{
			m_threadCheckUpdateParam.join();
		}

		if(nullptr != m_pRedisClient)
		{
			delete m_pRedisClient;
			m_pRedisClient = nullptr;
		}
	}

	int AHandleParam::queryParam(const std::string& sQueryCond, std::string& sQueryResult)
	{
		auto start = std::chrono::steady_clock::now();
		
		// 解析查询条件
		std::map<std::string, std::string> mpQueryCond = getQueryCond(sQueryCond);    
		if(mpQueryCond.count("cardnet") <= 0 || mpQueryCond.count("cardid") <= 0)
		{
			SYLAR_LOG_INFO(g_logger) << "param miss";
			return -1;
		}
		// 查询参数
		std::string sQueryCardId = mpQueryCond["cardnet"] + mpQueryCond["cardid"];
		SYLAR_LOG_INFO(g_logger) << "QueryCardId:" << sQueryCardId;
		std::map<std::string, std::string> mpResult = queryCardId(sQueryCardId);

		// 组装查询结果
		Json::Value json;
		for(auto& i : mpResult)     // 用于测试
		{
			json[i.first] = i.second;
		}
		
		// 根据协议的返回
		json["cmdtype"] = "blacklistqueryresult";
		json["errorcode"] = "0";
		json["serialno"] = mpQueryCond["serialno"];
		json["version"] = mpQueryCond["version"];
		if(mpResult.empty() || "2" == mpResult["status"])
		{
			json["recordcnt"] = "0";
		}
		else
		{
			json["recordcnt"] = "1";
			json["darkstatus0"] = mpResult["status"];
			json["darktype0"] = mpResult["type"];
			json["darkver0"] = "1970-01-01 08:00:00";
		}
		sQueryResult = sylar::JsonUtil::ToString(json);
		
		int nCostMs = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now()- start).count();
		SYLAR_LOG_INFO(g_logger) << "queryParam costMs:" << nCostMs;
		return 0;
	}

	int AHandleParam::startRedisServer()
	{
		auto start = std::chrono::steady_clock::now();

		pid_t pid = fork();
		if(0 == pid)
		{
			std::string sRedisServerExe = sylar::EnvMgr::GetInstance()->getCwd() + "redis/redis-server"; 
			std::string sRedisServerConf = sylar::EnvMgr::GetInstance()->getCwd() + "redis/redis.conf";
			SYLAR_LOG_INFO(g_logger) << "start:" << sRedisServerExe
				<< " " << sRedisServerConf;

			if(-1 == execl(sRedisServerExe.c_str(), "redis-server", sRedisServerConf.c_str(), NULL))
			{
				SYLAR_LOG_ERROR(g_logger) << "execl fail return=" << pid
					<< " errno=" << errno << " errstr=" << strerror(errno);
				return -1;
			}
			return 0;
		}
		else if(pid < 0)
		{
			SYLAR_LOG_ERROR(g_logger) << "fork fail return=" << pid
				<< " errno=" << errno << " errstr=" << strerror(errno);
			return -1;
		}
		else
		{
			int nCostMs = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now()- start).count();
			SYLAR_LOG_INFO(g_logger) << "startRedisServer costMs:" << nCostMs;
		}
		return 0;
	}

	void AHandleParam::checkUpdateParam()
	{
		while(false == m_bStopThreadCheckUpdateParam)
		{
			// 检查指定目录是否有参数要更新
			std::string sDataPath = sylar::EnvMgr::GetInstance()->getCwd() + "data/";
			//SYLAR_LOG_INFO(g_logger) << "DataPath:" << sDataPath;
			std::vector<std::string> files;
			sylar::FSUtil::ListAllFile(files, sDataPath, ".txt");
			std::sort(files.begin(), files.end());
			for(auto& i : files)
			{
				SYLAR_LOG_INFO(g_logger) << "DataParam:" << i;
				prefixParam(i);
				sylar::FSUtil::Rm(i);
				std::string sPrefixParamFileName = i.substr(0, i.length()-3) + "dat";
				loadParamToRedis(sPrefixParamFileName);
			}

			for(int i=0; i<50; ++i)     // 休眠50*100ms
			{
				if(false == m_bStopThreadCheckUpdateParam)
					std::this_thread::sleep_for(std::chrono::milliseconds(100));
			}
		}
	}

	int AHandleParam::prefixParam(const std::string& sAbsoluteFilePath)
	{
		std::ifstream ifParamFile(sAbsoluteFilePath.c_str());
		if(false == ifParamFile.is_open())
		{
			SYLAR_LOG_ERROR(g_logger) << "Open Param Fail, ParamName:" << sAbsoluteFilePath;
			return -1;
		}

		std::string sPrefixParamFileName = sAbsoluteFilePath.substr(0, sAbsoluteFilePath.length()-3) + "dat";
		std::ofstream ofPrefixParamFile;
		ofPrefixParamFile.open(sPrefixParamFileName, std::ios::app);
		
		std::hash<std::string> str_hash;
		
		auto start = std::chrono::steady_clock::now();
		std::string sLine;
		getline(ifParamFile, sLine);    // 去掉表头
		while(getline(ifParamFile, sLine))
		{
			std::vector<std::string> vecLine = split(sLine, "\t");
			std::string sKey = vecLine.at(0);
			std::string sBucketId = std::to_string(str_hash(sKey) % 300000);
			std::string sValue = "";
			encodeValue(vecLine.at(2), vecLine.at(1), sValue);
		
			unsigned int nFieldKey = BKDRHash(sKey.c_str());
			// windows下用\n，Linux下用\r\n，否则有如下报错：
			// ERR Protocol error: invalid multibulk length
			std::string sRedisProtocolCmd = std::string("*4\r\n")
				+ "$4\r\n"
				+ "hset\r\n"
				+ "$" + std::to_string(sBucketId.length()) + "\r\n"
				+ sBucketId + "\r\n"
				+ "$" + std::to_string(std::to_string(nFieldKey).length()) + "\r\n"
				+ std::to_string(nFieldKey) +"\r\n"
				+ "$" + std::to_string(sValue.length()) + "\r\n"
				+ sValue +"\r\n";
		
			ofPrefixParamFile << sRedisProtocolCmd;
		}
		ofPrefixParamFile.close();
		ifParamFile.close();
		
		int nCostMs = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now()- start).count();
		SYLAR_LOG_INFO(g_logger) << "prefixParam costMs:" << nCostMs;
		return 0;
	}

	int AHandleParam::loadParamToRedis(const std::string& sAbsoluteFilePath)
	{
		auto start = std::chrono::steady_clock::now();
		SYLAR_LOG_INFO(g_logger) << "PrefixFile:" << sAbsoluteFilePath;
		std::string sPipeCmd = "cat " + sAbsoluteFilePath + " | redis-cli --pipe";
		FILE* fp = popen(sPipeCmd.c_str(), "r");
		if(NULL == fp)
		{
			SYLAR_LOG_ERROR(g_logger) << "loadParamToRedis Error";
			return -1;
		}
		char buffer[1024];
		// TODO 处理返回的结果是否error非零
		while(NULL != fgets(buffer, sizeof(buffer), fp))
			SYLAR_LOG_INFO(g_logger) << "pipe to redis result:" << buffer;
		pclose(fp);
		
		int nCostMs = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now()- start).count();
		SYLAR_LOG_INFO(g_logger) << "loadParamToRedis costMs:" << nCostMs;
		return 0;
	}

	int AHandleParam::startLoadParamToRedis()
	{
		std::string sDataPath = sylar::EnvMgr::GetInstance()->getCwd() + "data/";
		SYLAR_LOG_INFO(g_logger) << "DataPath:" << sDataPath;
		std::vector<std::string> files;
		sylar::FSUtil::ListAllFile(files, sDataPath, ".dat");
		std::sort(files.begin(), files.end());
		for(auto& i : files)
		{
			if(0 != loadParamToRedis(i))
			{
				return -1;
			}
		}
		return 0;
	}

	std::map<std::string, std::string> AHandleParam::queryCardId(const std::string& sCardId)
	{
		std::hash<std::string> str_hash;
		std::string sRedisCmd = "hget " 
							  + std::to_string(str_hash(sCardId) % 300000)
							  + " "
							  + std::to_string(BKDRHash(sCardId.c_str()));
		ARedisClient::ReplyPtr pReplyPtr = m_pRedisClient->execRedisCommand(sRedisCmd);
		if(nullptr == pReplyPtr)
		{
			SYLAR_LOG_ERROR(g_logger) << "queryParam error:" << sRedisCmd;
			return {};
		}

		if(REDIS_REPLY_STRING == pReplyPtr->type)
		{
			SYLAR_LOG_INFO(g_logger) << "queryParam result:" << pReplyPtr->str;
			std::string sStatus;
			std::string sType;
			decodeValue(sStatus, sType, pReplyPtr->str);

			std::map<std::string, std::string> mpResult;
			mpResult["status"] = sStatus;
			mpResult["type"] = sType;
			return mpResult;
		}

		return {};
	}

	std::map<std::string, std::string> AHandleParam::getQueryCond(const std::string& sQueryCond)
	{
		std::map<std::string, std::string> mpQueryCond;
		std::vector<std::string> vecQueryCond = split(sQueryCond, "&");
		for(auto& i : vecQueryCond)
		{
			auto nPos = i.find("=");
			mpQueryCond[i.substr(0, nPos)] = i.substr(nPos + 1, i.length());
		}
		return mpQueryCond;
	}

	}
	```

# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
