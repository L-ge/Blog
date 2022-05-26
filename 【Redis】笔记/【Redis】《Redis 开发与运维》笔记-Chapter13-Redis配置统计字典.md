###### 十三、Redis配置统计字典

1、info系统状态说明

- info命令的使用方法有以下三种：
    1. info：部分Redis系统状态统计信息。
    2. info all：全部Redis系统状态统计信息。
    3. info section：某一块的系统状态统计信息，其中section可以忽略大小写。
- 例如，只对Redis的内存相关统计比较感兴趣，可以执行info memory，此时section=memory。
- info all命令包含Redis最全的系统状态信息。
- info Server模块的统计信息，包含了Redis服务本身的一些信息，例如版本号、运行模式、操作系统的版本、TCP端口等。
- info Clients模块的统计信息，包含了连接数、阻塞命令连接数、输入输出缓冲区等相关统计信息。
- info Memory模块的统计信息，包含了Redis内存使用、系统内存使用、碎片率、内存分配器等相关统计信息。
- info Persistence模块的统计信息，包含了RDB和AOF两种持久化的一些统计信息。
- info Stats模块的统计信息，是Redis的基础统计信息，包含了：连接、命令、网络、过期、同步等很多统计信息。
- info Replication模块的统计信息，包含了Redis主从复制的一些统计信息，根据主从节点，统计信息也略有不同。
- info CPU模块的统计信息，包含了Redis进程和子进程对于CPU消耗的一些统计信息。
- info Commandstats模块的统计信息，是Redis命令统计信息，包含各个命令的命令名、总次数、总耗时、平均耗时。
- info Cluster模块的统计信息，目前只有一个统计信息，标识当前Redis是否为Cluster模式。
- info Keyspace模块的统计信息，包含了每个数据库的键值统计信息。


info命令所有的section(info all命令涉及的所有section，其中每个模块名就是我们上面提到的section)：
模块名 | 模块含义
---|---
Server | 服务器信息
Clients | 客户端信息
Memory | 内存信息
Persistence | 持久化信息
Stats | 全局统计信息
Replication | 复制信息
CPU | CPU消耗信息
Commandstats | 命令统计信息
Cluster | 集群信息
Keyspace | 数据库键统计信息


1）info Server模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
redis_version | 3.0.7 | Redis服务版本
redis_git_sha1 | 00000000 | Git SHA1
redis_git_dirty | 0 | Git dirty flag
redis_build_id | 186eba9451cf9390 | Redis build id
redis_mode | cluster | 运行模式，分为：Cluster、Sentinel、Standalone
os | Linux 2.6.18-274.el5 x86_64 | Redis所在机器的操作系统
arch_bits | 64 | 架构(32或64位)
multiplexing_api | epoll | Redis所使用的事件处理机制
gcc_version | 4.1.2 | 编译Redis时所使用的GCC版本
process_id | 31524 | Redis服务进程的PID
run_id | fd8b97739c469526f640b8895a5084d669ed151f | Redis服务的标识符
tcp_port | 6384 | 监听端口
uptime_in_seconds | 9753347 | 自Redis服务启动以来，运行的秒数
uptime_in_days | 112 | 自Redis服务启动以来，运行的天数
hz | 10 | serverCron每秒运行次数
lru_clock | 16388503 | 以分钟为单位进行自增的时钟，用于LRU管理
config_file | /opt/cachecloud/conf/redis-cluster-6384.conf | Redis的配置文件


2）info Clients模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
connected_clients | 262 | 当前客户端连接数
client_longest_output_list | 0 | 当前所有输出缓冲区中队列对象个数的最大值
client_biggest_input_buf | 0 | 当前所有输入缓冲区中占有的最大容量
blocked_clients | 0 | 正在等待阻塞命令(例如BLPOP等)的客户端数量


3）info Memory模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
used_memory | 183150904 | Redis分配器分配的内存总量，也就是内部存储的所有数据内存占用量
used_memory_human | 174.67M | 以可读的格式返回used_memory
used_memory_rss | 428621824 | 从操作系统的角度，Redis进程占用的物理内存总量
used_memory_peak | 522768352 | 内存使用的最大值，表示used_memory的峰值
used_memory_peak_human | 498.55M | 以可读的格式返回used_memory_peak
used_memory_lua | 35840 | Lua引擎所消耗的内存大小
mem_fragmentation_ratio | 2.34 | used_memory_rss/used_memory比值，表示内存碎片率
mem_allocator | jemalloc-3.6.0 | Redis所使用的内存分配器。默认为：jemalloc


4）info Persistence模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
loading | 0 | 是否在加载持久化文件。0否，1是
rdb_changes_since_last_save | 53308858 | 自上次RDB后，Redis数据改动条数
rdb_bgsave_in_progress | 0 | 标识RDB的bgsave操作是否进行中。0否，1是
rdb_last_save_time | 1456376460 | 上次bgsave操作的时间戳
rdb_last_bgsave | ok | 上次bgsave操作状态
rdb_last_bgsave_time_sec | 3 | 上次bgsave操作使用的时间(单位是秒)
rdb_current_bgsave_time_sec | -1 | 如果bgsva操作正在进行，则记录当前bgsave操作使用的时间(单位是秒)
aof_enabled | 1 | 是否开启了AOF功能。0否，1是
aof_rewrite_in_progress | 0 | 标识AOF的rewrite操作是否在进行中。0否，1是
aof_rewrite_scheduled | 0 | 标识是否将要在RDB的bgsave操作结束后执行AOF rewrite操作
aof_last_rewrite_time_sec | 0 | 上次AOF rewrite操作使用的时间(单位是秒)
aof_current_rewrite_time_sec | -1 | 如果rewrite操作正在进行，则记录当前AOF rewrite所使用的时间(单位是秒)
aof_last_bgrewrite_status | ok | 上次AOF重写操作的状态
aof_last_write_status | ok | 上次AOF写磁盘的结果
aof_current_size | 186702421 | AOF当前尺寸(单位是字节)
aof_base_size | 134279710 | AOF上次启动或rewrite的尺寸(单位是字节)
aof_buffer_length | 0 | AOF buffer的大小
aof_rewrite_buffer_length | 0 | AOF rewrite buffer的大小
aof_pending_bio_fsync | 0 | 后台IO队列中等待fsync任务的个数
aof_delayed_fsync | 64 | 延迟的fsync计数器


5）info Stats模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
total_connections_received | 495967 | 连接过的客户端总数
total_commands_processed | 5139857171 | 执行过的命令总数
instantaneous_ops_per_sec | 511 | 每秒处理命令总数
total_net_input_bytes | 282961395316 | 输入总网络流量(以字节为单位)
total_net_output_bytes | 1760503612586 | 输出总网络流量(以字节为单位)
instantaneous_input_kbps | 28.24 | 每秒输入的字节数
instantaneous_output_kbps | 234.90 | 每秒输出的字节数
rejected_connections | 0 | 拒绝的连接个数
sync_full | 4 | 主从完全同步成功次数
sync_partial_ok | 0 | 主从部分同步成功次数
sync_partial_err | 0 | 主从部分同步失败次数
expired_keys | 45534039 | 过期的key数量
evicted_keys | 0 | 剔除(超过了maxmemory后)的key数量
keyspace_hits | 3923837939 | 命中次数
keyspace_misses | 1078922155 | 不命中次数
pubsub_channels | 0 | 当前使用中的频道数量
pubsub_patterns | 0 | 当前使用中的模式数量
latest_fork_usec | 16194 | 最近一次fork操作消耗的时间(微秒)
migrate_cached_sockets| 0 | 记录当前Redis正在进行migrate操作的目标Redis个数。例如Redis A分别向Redis B和C执行migrate操作，那么这个值就是2


6）info Replication模块统计信息
角色 | 属性名 | 属性值 | 属性描述
---|---|---|---
通用配置 | role | master|slave | 节点的角色
主节点 | connected_slaves | 1 | 连接的从节点个数
主节点 | slave0 | slabe0:op=10.10.xx.169,port=6382,state=online,offset=426978948465,lag=1 | 连接的从节点信息
主节点 | master_repl_offset | 426978955146 | 主节点偏移量
从节点 | master_host | 10.10.xx.64 | 主节点IP
从节点 | master_port | 6387 | 主节点端口
从节点 | master_link_status | up | 与主节点的连接状态
从节点 | master_last_io_seconds_ago | 0 | 主节点最后与从节点的通信时间间隔，单位为秒
从节点 | master_sync_in_progress | 0 | 从节点是否正在全量同步主节点RDB文件
从节点 | slave_repl_offset | 426978956171 | 复制偏移量
从节点 | slave_priority | 100 | 从节点优先级
从节点 | slave_read_only | 1 | 从节点是否只读
从节点 | connected_slaves | 0 | 连接从节点个数
从节点 | master_repl_offset | 0 | 当前从节点作为其他节点的主节点时的复制偏移量
通用配置 | repl_backlog_active | 1 | 复制缓冲区状态
通用配置 | repl_backlog_size | 10000000 | 复制缓冲区尺寸(单位:字节)
通用配置 | repl_backlog_first_byte_offset | 426968955147 | 复制缓冲区起始偏移量，标识当前缓冲区可用范围
通用配置 | repl_backlog_histlen | 10000000 | 标识复制缓冲区已存有效数据长度


7）info CPU模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
used_cpu_sys | 31957.30 | Redis主进程在内核态所占用的CPU时钟总和
used_cpu_user | 72484.27 | Redis主进程在用户态所占用的CPU时钟总和
used_cpu_sys_children | 121.49 | Redis子进程在内核态所占用的CPU时钟总和
used_cput_user_children | 195.13 | Redis子进程在用户态所占用的CPU时钟总和


8）info Commandstats模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
cmdstat_get | calls=3738730699,usec=11054972404,usec_per_call=2.96 | get命令调用总次数、总耗时、平均耗时(单位:微秒)
cmdstat_set | calls=50174458,usec=323143686,usec_per_call=6.44 | set命令调用总次数、总耗时、平均耗时(单位:微秒)


9）info Cluster模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
cluster_enabled | 1 | 节点是否为cluster模式。1是，0否


10）info Keyspace模块统计信息
属性名 | 属性值 | 属性描述
---|---|---
db0 | db0:keys=106430,expires=56107,avg_ttl=60283952 | 当前数据库key总数，带有过期时间的key总数，平均存活时间
 

2、standalone配置说明和分析(Redis单机模式)

1）Redis 的一些总体配置，例如端口、 日志、数据库等。

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
daemonize | 是否是守护进程 | no | yes\|no | 不可以
port | 端口号 | 6379 | 整数 | 不可以
loglevel | 日志级别 | notice | debug\|verbose\|notice\|warning | 可以
logfile | 日志文件名 | 空 | 自定义，建议以端口号为名 | 不可以
databases | 可用的数据库数 | 16 | 整数 | 不可以
unixsocket | unix套接字 | 空(不通过unix套接字来监听) | 指定套接字文件 | 不可以 
unixsocketperm | unix套接字权限 | 0 | Linux三位数权限 | 不可以
pidfile | Redis运行的进程pid文件 | /var/run/redis.pid | /var/run/redis-{port}.pid | 不可以
lua-time-limit | Lua脚本"超时时间"(单位:毫秒) | 5000 | 整数，但是此超时不会真正停止脚本运行 | 可以
tcp-backlog | tcp-backlog | 511 | 整数 | 不可以
watchdog-period | 看门狗，用于诊断Redis的延迟问题，此参数是检查周期。(此参数需要在运行时配置才能生效) | 0 | 整数 | 可以
activerehashing | 指定是否激活重置哈希 | yes | yes\|no | 可以
dir | 工作目录(aof、rdb、日志文件都存放在此目录) | ./(当前目录) | 自定义 | 可以


2）Redis内存相关配置。

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
maxmemory | 最大可用内存(单位字节) | 0(没有限制) | 整数 | 可以
maxmemory-policy | 内存不够时，淘汰策略 | noeviction | volatile-lru -> 用lru算法删除过期的键值<br> allkeys-lru -> 用lru算法删除所有键值<br> volatile-random -> 随机删除过期的键值<br> allkeys-random -> 随机删除任何键值<br> volatile-ttl -> 删除最近要到期的键值<br> noeviction -> 不删除键 | 可以
maxmemory-samples | 检测LRU采样数 | 5 | 整数 | 可以

3）AOF方式持久化相关配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
appendonly | 是否开启AOF持久化模式 | no | no\|yes | 可以
appendfsync | AOF同步磁盘频率 | everysec | always\|everysec\|no | 可以
appendfilename | AOF文件名 | appendonly.aof | appendonly-{port}.aof | 不可以
aof-load-truncated | 加载AOF文件时，是否忽略AOF文件不完整的情况，让Redis正常启动 | yes|no | 可以
no-appendfsync-on-rewrite | 设置为yes表示rewrite期间对新写操作不fsync，暂时存在缓冲区中，等rewrite完成后再写入 | no | no\|yes | 可以
auto-aof-rewrite-min-size | 触发rewrite的AOF文件最小阈值(单位:兆) | 64m | 整数+m(代表兆) | 可以
auto-aof-rewrite-percentage | 触发rewrite的AOF文件的增长比例条件 | 100 | 整数 | 可以
aof-rewrite-incremental-fsync | AOF重写过程中，是否采取增量文件同步策略 | yes | yes\|no | 可以

4）RDB方式持久化相关配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
save | RDB保存条件 | save 900 1<br> save 300 10<br> save 60 10000 | 如果没有该配置，代表不使用自动RDB策略 | 可以
dbfilename | RDB文件名 | dump.rdb | dump-{port}.rdb | 可以
rdbcompression | RDB文件是否压缩 | yes | yes\|no | 可以
rdbchecksum | RDB文件是否使用校验和 | yes | yes\|no | 可以
stop-write-on-bgsave-error | bgsave执行错误，是否停止Redis接受写请求 | yes | yes\|no | 可以

5）Redis慢查询相关配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
slow-log-slower-than | 慢查询被记录的阀值(单位微妙) | 10000 | 整数 | 可以
slowlog-max-len | 最多记录慢查询的条数 | 128 | 整数 | 可以
latency-monitor-threshold | Redis服务内存延迟监控 | 0(关闭) | 整数 | 可以

6）Redis数据结构优化的相关配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
hash-max-ziplist-entries | hash数据结构优化参数 | 512 | 整数 | 可以
hash-max-ziplist-value | hash数据结构优化参数 | 64 | 整数 | 可以
list-max-ziplist-entries | list数据结构优化参数 | 512 | 整数 | 可以
list-max-ziplist-value | list数据结构优化参数 | 64 | 整数 | 可以
set-max-intset-entries | set数据结构优化参数 | 512 | 整数 | 可以
zset-max-ziplist-entries | zset数据结构优化参数 | 128 | 整数 | 可以
zset-max-ziplist-value | zset数据结构优化参数 | 64 | 整数 | 可以
hll-sparse-max-bytes | HyperLogLog数据结构优化参数 | 3000 | 整数 | 可以

7）Redis复制相关的配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
slaveof | 指定当前从节点复制哪个主节点，参数：主节点的ip和port | 空 | ip和端口 | 不可以，但可以用slaveof命令设置 
repl-ping-slave-period | 主节点定期向从节点发送ping命令的周期，用于判定从节点是否存活。(单位:秒) | 10 | 整数 | 可以
repl-timeout | 主从节点复制超时时间(单位:秒) | 60 | 整数 | 可以 
repl-backlog-size | 复制积压缓存区大小 | 1M | 整数 | 可以
repl-backlog-ttl | 主节点在没有从节点的情况下多长时间后释放复制积压缓存区空间 | 3600 | 整数 | 可以
slave-priority | 从节点的优先级 | 100 | 0-100 | 可以
min-slaves-to-write | 当主节点发现从节点数量小于min-slaves-to-write且延迟小于等于min-slaves-max-lag时，master停止写入操作 | 0 | 整数 | 可以
min-slaves-max-lag | 当主节点发现从节点数量小于min-slaves-to-write且延迟小于等于min-slaves-max-lag时，master停止写入操作 | 10 | 整数 | 可以
slave-serve-stale-data | 当从节点与主节点连接中断时，如果此参数值设置为"yes"，从节点可以继续处理客户端的请求。否则除info和slaveof命令之外，拒绝的所有请求并统一回复"SYNC with master in process" | yes | yes\|no | 可以
slave-read-only | 从节点是否开启只读模式，集群架构下从节点默认读写都不可用，需要调用readonly命令开启只读模式 | yes | yes\|no | 可以
repl-disble-tcp-nodelay | 是否开启主从复制socket的NO_DELAY选项：<br> yes:Redis会合并小的TCP包来节省带宽，但是这样增加同步延迟，造成主从数据不一致<br> no:主节点会立即发送同步数据，没有延迟 | no | yes\|no | 可以
repl-diskless-sync | 是否开启无盘复制 | no | yes\|no | 可以
repl-diskless-sync-delay | 开启无盘复制后，需要延迟多少秒后进行创建RDB操作，一般用于同时加入多个从节点时，保证多个从节点可共享RDB | 5 | 整数 | 可以

8）Redis客户端的相关配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
maxclients | 最大客户端连接数 | 10000 | 整数 | 可以
client-output-buffer-limit | 客户端输出缓冲区限制 | normal 0 0 0 slave 268435456 67108864 60 pubsub 33554432 8388608 60 | 整数 | 可以
timeout | 客户端闲置多少秒后关闭连接(单位:秒) | 0(永不关闭) | 整数 | 可以
tcp-keepalive | 检测TCP连接活性的周期(单位:秒) | 0(不检测) | 整数 | 可以

9）Redis安全的相关配置

配置名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
requirepass | 密码 | 空 | 自定义 | 可以
bind | 绑定IP | 空 | 自定义 | 不可以
masterauth | 从节点需要配置的主节点密码 | 空 | 主节点的密码 | 可以


3、Sentinel配置说明和分析

1）Sentinel节点是特殊的Redis节点，有几个特殊的配置

Redis Sentinel节点配置说明
参数名 | 含义 | 默认值 | 可选值 | 可否支持sentinel set配置热生效
---|---|---|---|---
sentinel monitor <master-name> <ip> <port> <quorum> | 定义监控的主节点名、ip、port、主观下线票数 | sentinel monitor mymaster 127.0.0.1 6379 2 | 自定义masterName、实际的ip:port、票数 | 支持<quorum>
sentinel down-after-milliseconds <master-name> <times> | Sentinel判定节点不可达的毫秒数 | sentinel down-after-milliseconds mymaster 30000 | 整数 | 支持
sentinel parallel-sync <master-name> <nums> | 在执行故障转移时，最多有多少个从服务器同时对新的主服务器进行同步 | sentinel parallel-sync mymaster 1 | 大于0，不超过从服务器个数 | 支持
sentinel failover-timeout <master-name> <times> | 故障迁移超时时间 | sentinel failover-timeout mymaster 180000 | 整数 | 支持
sentinel auth-pass <master-name> <password> | 主节点密码 | 空 | 主节点密码| 支持
sentinel notification-script <master-name> <script-path> | 故障转移期间脚本通知 | 空 | 脚本文件路径 | 支持
sentinel client-reconfig-script <master-name> <script-path> | 故障转移成功后脚本通知 | 空 | 脚本文件路径 | 支持


4、Cluster配置说明和分析

1）Cluster节点是特殊的Redis节点，有几个特殊的配置

Redis Cluster配置说明
参数名 | 含义 | 默认值 | 可选值 | 可否支持config set配置热生效
---|---|---|---|---
cluster-node-timeout | 集群节点超时时间(单位:毫秒) | 15000 | 整数 | 可以
cluster-migration-barrier | 主从节点切换需要的从节点数最小个数 | 1 | 整数 | 可以
cluster-slave-validity-factor | 从节点有效性判断因子，当从节点与主节点最后通行时间超过(cluster-node-timeout*slave-validity-factor)+repl-ping-slave-period时，对应从节点不具备故障转移资格，防止断线时间过长的从节点进行故障转移。设置为0标叔从节点永不过期。 | 10 | 整数 | 可以
cluster-require-full-converage | 集群是否需要所有的slot都分配给在线节点，才能正常访问 | yes | yes\|no | 可以
cluster-enabled | 是否开启集群模式 | yes | yes\|no | 不可以
cluster-config-file | 集群配置文件名称 | node.conf | nodes-{port}.conf | 不可以
