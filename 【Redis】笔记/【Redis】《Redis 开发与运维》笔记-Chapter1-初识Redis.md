###### 一、初识Redis

1、Redis提供了两种持久化方式：RDB和AOF，即可以用这两种策略将内存的数据保存到硬盘中。

2、复制功能是分布式Redis的基础。

3、一般推荐使用的安装方式：源码的方式进行安装。下面以3.0.7版本为例(只需6步)：
```
$ wget http://download.redis.io/releases/redis-3.0.7.tar.gz
$ tar xzf redis-3.0.7.tar.gz
$ ln -s redis-3.0.7 redis           // 建立一个redis目录的软连接，指向redis-3.0.7。
$ cd redis
$ make                              // 编译(编译前确保操作系统已经安装gcc)
$ make install

$ redis-cli -v          // 查看Redis的版本
```
- 建立一个redis目录的软链接是为了不把redis目录固定在指定版本上，有利于Redis未来版本升级，算是安装软件的一个好习惯。
- 第6步的安装是将Redis的相关运行文件放到/usr/local/bin/下，这样就可以在任意目录下执行Redis的命令。

4、Redis可执行文件说明

可执行文件 | 作用
---|---
redis-server | 启动Redis
redis-cli | Redis命令行客户端
redis-benchmark | Redis基准测试工具
redis-check-aof | Redis AOF持久化文件检测和修复工具
redis-check-dump | Redis RDB持久化文件检测和修复工具
redis-sentinel | 启动Redis Sentinel

5、启动Redis

有三种方法启动Redis：默认配置、运行配置、配置文件启动。

1) 默认配置
- 这种方法会使用Redis的默认配置来启动。
```
$ redis-server
```

2) 运行启动
- redis-server加上要修改的配置名和值(可以是多对)，没有设置的配置将使用默认配置。
```
# redis-server --configKey1 configValue1 --configKey2 configValue2
# redis-server --port 6380
```

3）配置文件启动
- 将配置写到指定文件里，例如我们将配置写到了/opt/redis/redis.conf中，那么只需要执行如下命令即可启动Redis：
```
# redis-server /opt/redis/redis.conf
```

6、Redis的基础配置(Redis有60多个配置)

配置名 | 配置说明
---|---
port | 端口
logfile | 日志文件
dir | Redis工作目录(存放持久化文件和日志文件)
daemonize | 是否以守护进程的方式启动Redis

7、Redis命令行客户端

1）第一种是交互式方式：通过redis-cli -h {host} -p {port}的方式连接到Redis服务，之后所有的操作都是通过交互的方式实现，不需要再执行redis-cli了。
```
redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
```

2）第二种是命令方式：通过redis-cli -h {host} -p {port} {command}就可以直接得到命令的返回结果。
```
redis-cli -h 127.0.0.1 -p 6379 get hello
"world"
```
- 如果没有-h参数，那么默认连接127.0.0.1；如果没有-p，那么默认6379端口。

8、 停止Redis服务

Redis提供了shutdown命令来停止Redis服务。
```
$ redis-cli shutdown        // 停掉127.0.0.1上6379端口上的Redis服务
```
- Redis关闭的过程：断开与客户端的连接、持久化文件生成，是一种相对优雅的关闭方式。
- 除了可以通过shutdown命令关闭Redis服务以外，还可以通过kill进程号的方式关闭掉Redis，但是不要粗暴地使用kill -9强制杀死Redis服务，不但不会做持久化操作，还会造成缓冲区等资源不能被优雅关闭，极端情况会造成AOF和复制丢失数据的情况。
- shutdown还有一个参数，代表是否在关闭Redis前，生成持久化文件。
```
$ redis-cli shutdown nosave|save
```
