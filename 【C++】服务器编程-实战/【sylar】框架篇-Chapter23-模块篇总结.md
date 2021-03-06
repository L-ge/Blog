# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述
- 基础模块篇约14个模块。
- 系统篇约2个模块。
- 框架性能测试。

# 日志模块
- Logger、LogAppender、LogFormatter、LogEvent

# 配置模块
- ConfigBase、ConfigVar、Config
- 约定优于配置。

# 线程模块
- Thread、RWMutex、Mutex、Semaphore、SpinLock

# 协程模块
- Fiber

# 协程调度模块
- Scheduler

# IO 协程调度模块
- IOManager(epoll)、TimerManager(epoll_wait)
- 最重要最复杂的模块。       

# HOOK 模块
- FdManager、socket、connect
- 例如，配合 IO 协程调度模块一起使用，当 socket 同步读的时候阻塞了，所在协程让出执行权（注意这里如果没有协程，那阻塞的就是线程，线程的数量是有限的），从而实现异步 IO 的效果。

# Socket 模块
- Address、Socket

# ByteArray 模块
- ByteArray
- 解决网络传输时字节序问题、压缩问题。

# Stream 模块
- Stream、SocketStream

# TcpServer 模块 
- TcpServer

# HttpServer 模块
- HttpServer

# Servlet 模块
- Servlet、NotFoundServlet、FunctionServlet
- 抽象服务端的逻辑，某个路径映射到某个Servlet。

# HttpConnection 模块
- HttpConnection、HttpConnectionPool
- HttpConnectionPool 针对的是长连接的场景。

# 系统篇
- 系统篇包含了以下内容，但总结起来就是两模块（守护进程模块和环境变量模块）：
	- 守护进程
	- 启动参数解析
	- 环境变量
	- 配置加载
	- Application

# 压力测试
- 压测工具AB
	```
	yum install -y httpd-tools
	```
- 编译选项选 -O3，性能最好。
- 留意有没有限制socket句柄等
	```
	ulimit -a
	```
- 压测 sylar 框架的小程序：examples/my_http_server.cc
- 注意关闭防火墙
	```
	service firewalld stop
	```
- ab的使用
	```
	ab --help

	ab -n 1000000 -c 200 "http://192.168.60.138:8020/sylar"
	ab -n 1000000 -c 200 -k "http://192.168.60.138:8020/sylar"
	-n 是多少个请求
	-c 是多少个连接
	-k 是长连接。
	```
	- 测试结果：
		- 短链接：Requests per second:    31714.17 [#/sec] (mean)
		- 长链接：Requests per second:    62924.43 [#/sec] (mean)
- 和 nginx 比较
	```
	sudo yum install epel-release
	sudo yum update
	sudo yum install -y nginx		
	sudo nginx							// 启动nginx，默认是80端口
	sudo netstat -anp | grep nginx		// nginx在80端口

	ab -n 1000000 -c 200 "http://192.168.60.138:80/sylar"
	ab -n 1000000 -c 200 -k "http://192.168.60.138:80/sylar"
	```
	- 测试结果：
		- 短链接：Requests per second:    12548.61 [#/sec] (mean)
		- 长链接：Requests per second:    92893.75 [#/sec] (mean)
- 和 libevent 比较（sylar测试过，我没有测试）
	```
	wget xxx.com/libtool.tar.xz		// libtool是automake常用的工具
	安装到自己的路径/apps/..

	git clone https://github.com/libevent/libevent.git
	cd libevent
	./autogen.sh
	安装到自己的路径/apps/..
	cd sample		// 里面有例子

	./http-server -p 8030 .
	```
- 其他小工具
	```
	top -H -p 5066(pid)			// 看线程的运行情况

	yum install -y net-tools	// 安装netstat
	
	netstat -ntlap | grep 5066 | wc -l	// 连接数
	```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
