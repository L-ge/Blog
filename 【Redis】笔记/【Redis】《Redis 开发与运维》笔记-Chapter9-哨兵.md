###### 九、哨兵

1、Redis Sentinel相关名词解释

Redis从2.8开始正式提供了Redis Sentinel（哨兵）架构。

名词 |逻辑结构 | 物理结构
---|---|---
主节点(master) | Redis主服务/数据库 | 一个独立的Redis进程
从节点(slave) | Redis从服务/数据库 | 一个独立的Redis进程
Redis数据节点 | 主节点和从节点 | 主节点和从节点的进程
Sentinel节点 | 监控Redis数据节点 | 一个独立的Sentinel进程
Sentinel节点集合 | 若干Sentinel节点的抽象组合 | 若干Sentinel节点进程
Redis Sentinel | Redis高可用实现方案 | Sentinel节点集合和Redis数据节点进程
应用方 | 泛指一个或多个客户端 | 一个或者多个客户端进程或者线程

2、主从复制的问题

- (高可用问题)一旦主节点出现故障，需要手动将一个从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点，整个过程都需要人工干预。
- (分布式问题)主节点的写能力受到单机的限制。
- (分布式问题)主节点的存储能力受到单机的限制。

3、Redis Sentinel的高可用性

- 当主节点出现故障时，Redis Sentinel能自动完成故障发现和故障转移，并通知应用方，从而实现真正的高可用。
- Redis2.6版本提供Redis Sentinel v1版本，但是功能性和健壮性都有一些问题，如果想使用Redis Sentinel的话，建议使用2.8以上版本，也就是v2版本的Redis Sentinel。
- Redis Sentinel是一个分布式架构，其中包含若干个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和其余Sentinel节点进行监控，当它发现节点不可达时，会对节点做下线标识。如果被标识的是主节点，它还会和其他Sentinel节点进行“协商”，当大多数Sentinel节点都认为主节点不可达时，它们会选举出一个Sentinel节点来完成自动故障转移的工作，同时会将这个变化实时通知给Redis应用方。整个过程完全是自动的，不需要人工来介入，所以这套方案很有效地解决了Redis的高可用问题。

*这里的分布式是指：Redis数据节点、Sentinel节点集合、客户端分布在多个物理节点的架构，不要与Redis Cluster分布式混淆。*

Redis Sentinel与Redis主从复制模式只是多了若干Sentinel节点，所以Redis Sentinel并没有针对Redis节点做了特殊处理。从逻辑架构上看，Sentinel节点集合会定期对所有节点进行监控，特别是对主节点的故障实现自动转移。

Redis Sentinel具有以下几个功能：
- 监控：Sentinel节点会定期检测Redis数据节点、其余Sentinel节点是否可达。
- 通知：Sentinel节点会将故障转移的结果通知给应用方。
- 主节点故障转移：实现从节点晋升为主节点并维护后续正确的主从关系。
- 配置提供者：在Redis Sentinel结构中，客户端在初始化的时候连接的是Sentinel节点集合，从中获取主节点信息。

Redis Sentinel包含了若个Sentinel节点，这样做也带来了两个好处：
- 对于节点的故障判断是由多个Sentinel节点共同完成，这样可以有效地防止误判。
- Sentinel节点集合是由若干个Sentinel节点组成的，这样即使个别Sentinel节点不可用，整个Sentinel节点集合依然是健壮的。

但是Sentinel节点本身就是独立的Redis节点，只不过它们有一些特殊，它们不存储数据，只支持部分命令。

4、部署Redis数据节点

```
1、启动主节点
配置：
redis-6379.conf
port 6379
daemonize yes
logfile "6379.log"
dbfilename "dump-6379.rdb"
dir "/opt/soft/redis/data/"

启动主节点：
redis-server redis-6379.conf

确认是否启动：
$ redis-cli -h 127.0.0.1 -p 6379 ping
PONG


2、启动两个从节点
配置：
redis-6380.conf
port 6380
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"
dir "/opt/soft/redis/data/"
slaveof 127.0.0.1 6379

启动两个从节点：
redis-server redis-6380.conf
redis-server redis-6381.conf

验证：
$ redis-cli -h 127.0.0.1 -p 6380 ping
PONG
$ redis-cli -h 127.0.0.1 -p 6381 ping
PONG


3、确认主从关系
主节点的视角：
$ redis-cli -h 127.0.0.1 -p 6379 info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state-online,offset=281,lag=1
slave1:ip=127.0.0.1,port=6381,state-online,offset=281,lag=1

从节点的视角：
info replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
. . . . . . . . . . . . . . . . .

```

5、部署Sentinel节点
```
1、配置Sentinel节点(以配置sentinel-1节点为例)
redis-sentinel-26379.conf
port 26379
daemonize yes
logfile "26379.log"
dir /opt/soft/redis/data
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000

2、启动Sentinel节点
Sentinel节点的启动方法有两种(两种方法本质上是一样的)：
方法一，使用redis-sentinel命令：
redis-sentinel redis-sentinel-26379.conf

方法二，使用redis-server命令加--sentinel参数：
redis-server redis-sentinel-26379.conf --sentinel

3、确认
// 从下面info的Sentinel片段来看，Sentinel节点找到了主节点127.0.0.1:6379，发现了它的两个从节点，同时发现Redis Sentinel一共有3个Sentinel节点。
$ redis-cli -h 127.0.0.1 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```
- Sentinel节点的默认端口是26379。
- sentinel monitor mymaster 127.0.0.1 6379 2配置代表sentinel- 1节点需要监控127.0.0.1:6379这个主节点，2代表判断主节点失败至少需要2个Sentinel节点同意，mymaster是主节点的别名。
- Sentinel节点本质上是一个特殊的Redis节点，所以也可以通过info命令来查询它的相关信息.
- Sentinel节点能够彼此感知到对方，同时能够感知到Redis数据节点。
- 生产环境中建议Redis Sentinel的所有节点应该分布在不同的物理机上。
- Redis Sentinel中的数据节点和普通的Redis数据节点在配置上没有任何区别，只不过是添加了一些Sentinel节点对它们进行监控。

6、配置优化

Redis安装目录下有一个sentinel.conf，是默认的Sentinel节点配置文件。

1）配置说明和优化
```
port 26379
dir /opt/soft/redis/data
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
#sentinel auth-pass <master-name> <password>
#sentinel notification-script <master-name> <script-path>
#sentinel client-reconfig-script <master-name> <script-path>

1、sentinel monitor
// 本配置说明Sentinel节点要监控的是一个名字叫做<master-name>，ip地址和端口为<ip><port>的主节点。<quorum>代表要判定主节点最终不可达所需要的票数。
sentinel monitor <master-name> <ip> <port> <quorum>

2、sentinel down-after-milliseconds
sentinel down-after-milliseconds <master-name> <times>

3、sentinel parallel-syncs
sentinel parallel-syncs <master-name> <nums>

4、sentinel failover-timeout
sentinel failover-timeout <master-name> <times>

5、sentinel auth-pass
sentinel auth-pass <master-name> <password>

6、sentinel notification-script
sentinel notification-script <master-name> <script-path>

7、sentinel client-reconfig-script
sentinel client-reconfig-script <master-name> <script-path>

当故障转移结束，每个Sentinel节点会将故障转移的结果发送给对应的脚本，具体参数如下：
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
- <master-name>：主节点名。
- <role>：Sentinel节点的角色，分别是leader和observer，leader代表当前Sentinel节点是领导者，是它进行的故障转移； observer是其余Sentinel节点。
- <from-ip>：原主节点的ip地址。
- <from-port>：原主节点的端口。
- <to-ip>：新主节点的ip地址。
- <to-port>：新主节点的端口。
```
- port和dir分别代表Sentinel节点的端口和工作目录。
- 实际上Sentinel节点会对所有节点进行监控，但是在Sentinel节点的配置中没有看到有关从节点和其余Sentinel节点的配置，那是因为Sentinel节点会从主节点中获取有关从节点以及其余Sentinel节点的相关信息。
- 当所有节点启动后，配置文件中的内容发生了变化，体现在三个方面：
    1. Sentinel节点自动发现了从节点、其余Sentinel节点。
    2. 去掉了默认配置，例如parallel-syncs、failover-timeout参数。
    3. 添加了配置纪元相关参数。
- <quorum>参数用于故障发现和判定，例如将quorum配置为2，代表至少有2个Sentinel节点认为主节点不可达，那么这个不可达的判定才是客观的。对于<quorum>设置的越小，那么达到下线的条件越宽松，反之越严格。一般建议将其设置为Sentinel节点的一半加1。
- 同时<quorum>还与Sentinel节点的领导者选举有关，至少要有max（quorum，num（sentinels）/2+1）个Sentinel节点参与选举，才能选出领导者Sentinel ，从而完成故障转移。
- 每个Sentinel节点都要通过定期发送ping命令来判断Redis数据节点和其余Sentinel节点是否可达，如果超过了down-after-milliseconds配置的时间且没有有效的回复，则判定节点不可达，<times>（单位为毫秒）就是超时时间。这个配置是对节点失败判定的重要依据。
- down-after-milliseconds越大，代表Sentinel节点对于节点不可达的条件越宽松，反之越严格。条件宽松有可能带来的问题是节点确实不可达了，那么应用方需要等待故障转移的时间越长，也就意味着应用方故障时间可能越长。条件严格虽然可以及时发现故障完成故障转移，但是也存在一定的误判率。
- down-after-milliseconds虽然以<master-name>为参数，但实际上对Sentinel节点、主节点、从节点的失败判定同时有效。
- 当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，原来的从节点会向新的主节点发起复制操作，parallel-syncs就是用来限制在一次故障转移之后，每次向新的主节点发起复制操作的从节点个数。如果这个参数配置的比较大，那么多个从节点会向新的主节点同时发起复制操作，尽管复制操作通常不会阻塞主节点，但是同时向主节点发起复制，必然会对主节点所在的机器造成一定的网络和磁盘IO开销。
- failover-timeout通常被解释成故障转移超时时间，但实际上它作用于故障转移的各个阶段：
    1. 选出合适从节点。
    2. 晋升选出的从节点为主节点。
    3. 命令其余从节点复制新的主节点。
    4. 等待原主节点恢复后命令它去复制新的主节点。
- failover-timeout的作用具体体现在四个方面：
    1. 如果Redis Sentinel对一个主节点故障转移失败，那么下次再对该主节点做故障转移的起始时间是failover-timeout的2倍。
    2. 在上述第二阶段时，如果Sentinel节点向上述第一阶段选出来的从节点执行slaveof no one一直失败（例如该从节点此时出现故障），当此过程超过failover-timeout时，则故障转移失败。
    3. 在上述第二阶段如果执行成功，Sentinel节点还会执行info命令来确认上述第一阶段选出来的节点确实晋升为主节点，如果此过程执行时间超过failover-timeout时，则故障转移失败。
    4. 如果上述第三阶段执行时间超过了failover-timeout（不包含复制时间），则故障转移失败。注意即使超过了这个时间，Sentinel节点也会最终配置从节点去同步最新的主节点。
- 如果Sentinel监控的主节点配置了密码，sentinel auth-pass配置通过添加主节点的密码，防止Sentinel节点对主节点无法监控。
- sentinel notification-script的作用是在故障转移期间，当一些警告级别的Sentinel事件发生（指重要事件，例如-sdown：客观下线、-odown：主观下线）时，会触发对应路径的脚本，并向脚本发送相应的事件参数。
- sentinel client-reconfig-script的作用是在故障转移结束后，会触发对应路径的脚本，并向脚本发送故障转移结果的相关参数。
- 有关sentinel notification-script和sentinel client-reconfig-script有几点需要 注意：
    1. <script-path>必须有可执行权限。
    2. <script-path>开头必须包含shell脚本头（例如#！/bin/sh），否则事件发生时Redis将无法执行脚本产生错误。
    3. Redis规定脚本的最大执行时间不能超过60秒，超过后脚本将被杀掉。
    4. 如果shell脚本以exit 1结束，那么脚本稍后重试执行。如果以exit 2或者更高的值结束，那么脚本不会重试。正常返回值是exit 0。
    5. 如果需要运维的Redis Sentinel比较多，建议不要使用这种脚本的形式来进行通知，这样会增加部署的成本。

2）如何监控多个主节点

Redis Sentinel可以同时监控多个主节点。只需要指定多个masterName来区分不同的主节点即可。

3）调整配置

和普通的Redis数据节点一样，Sentinel节点也支持动态地设置参数，而且和普通的Redis数据节点一样并不是支持所有的参数：
`sentinel set <param> <value>`

sentinel set命令支持的参数如下表：

参数 | 使用方法
---|---
quorum | sentinel set mymaster quorum 2
down-after-milliseconds | sentinel set mymaster down-after-milliseconds 30000
failover-timout | sentinel set mymaster failover-timout 360000
parellel-syncs | sentinel set mymaster parellel-syncs 2
notification-script | sentinel set mymaster notification-script /opt/xx.sh
client-reconfig-script | sentinel set mymaster client-reconfig-script /opt/yy.sh
auth-pass | sentinel set mymaster auth-pass masterPassword

- sentinel set命令只对当前Sentinel节点有效。
- sentinel set命令如果执行成功会立即刷新配置文件，这点和Redis普通数据节点设置配置需要执行config rewrite刷新到配置文件不同。
- 建议所有Sentinel节点的配置尽可能一致，这样在故障发现和转移时比较容易达成一致。
- 上表中为sentinel set支持的参数，具体可以参考源码中的sentinel.c的sentinelSetCommand函数。
- Sentinel对外不支持config命令。

7、部署技巧

- Sentinel节点不应该部署在一台物理“机器”上。
- 部署至少三个且奇数个的Sentinel节点。
    1. 3个以上是通过增加Sentinel节点的个数提高对于故障判定的准确性，因为领导者选举需要至少一半加1个节点，奇数个节点可以在满足该条件的基础上节省一个节点。
- 只有一套Sentinel，还是每个主节点配置一套Sentinel？
    1. 方案一：一套Sentinel，很明显这种方案在一定程度上降低了维护成本，因为只需要维护固定个数的Sentinel节点，集中对多个Redis数据节点进行管理就可以了。但是这同时也是它的缺点，如果这套Sentinel节点集合出现异常，可能会对多个Redis数据节点造成影响。还有如果监控的Redis数据节点较多，会造成Sentinel节点产生过多的网络连接，也会有一定的影响。
    2. 方案二：多套Sentinel，显然这种方案的优点和缺点和上面是相反的，每个Redis主节点都有自己的Sentinel节点集合，会造成资源浪费。但是优点也很明显，每套Redis Sentinel都是彼此隔离的。
- 如果Sentinel节点集合监控的是同一个业务的多个主节点集合，那么使用方案一、否则一般建议采用方案二。

8、API
```
下面以Sentinel节点集合监控着两组主从模式的Redis数据节点为例进行说明：

1、sentinel masters：展示所有被监控的主节点状态以及相关的统计信息。
127.0.0.1:26379> sentinel masters
1)  1) "name"
    2) "mymaster-2"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6382"
. . . . . . . . .忽略 . . . . . . . . . . . .
2)  1) "name"
    2) "mymaster-1"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6379"
. . . . . . . . .忽略 . . . . . . . . . . . .


2、sentinel master<master name>：展示指定<master name>的主节点状态以及相关的统计信息。
127.0.0.1:26379> sentinel master mymaster-1
1) "name"
2) "mymaster-1"
3) "ip"
4) "127.0.0.1"
5) "port"
6) "6379"
. . . . . . . . .忽略 . . . . . . . . . . . .


3、sentinel slaves<master name>：展示指定<master name>的从节点状态以及相关的统计信息。
127.0.0.1:26379> sentinel slaves mymaster-1
1)  1) "name"
    2) "127.0.0.1:6380"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6380"
. . . . . . . . .忽略 . . . . . . . . . . . .
2)  1) "name"
    2) "127.0.0.1:6381"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6381"
. . . . . . . . .忽略 . . . . . . . . . . . .


4、sentinel sentinels<master name>：展示指定<master name>的Sentinel节点集合（不包含当前Sentinel节点）
127.0.0.1:26379> sentinel sentinels mymaster-1
1)  1) "name"
    2) "127.0.0.1:26380"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "26380"
. . . . . . . . .忽略 . . . . . . . . . . . .
2)  1) "name"
    2) "127.0.0.1:26381"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "26381"
. . . . . . . . .忽略 . . . . . . . . . . . .


5、sentinel get-master-addr-by-name <master name>：返回指定<master name>主节点的IP地址和端口
127.0.0.1:26379> sentinel get-master-addr-by-name mymaster-1
1) "127.0.0.1"
2) "6379"

6、sentinel reset <pattern>：当前Sentinel节点对符合<pattern>（通配符风格）主节点的配置进行重置，包含清除主节点的相关状态（例如故障转移），重新发现从节点和Sentinel节点。
127.0.0.1:26379> sentinel reset mymaster-1      // sentinel-1节点对mymaster-1节点重置状态
(integer) 1

7、sentinel failover <master name>：对指定<master name>主节点进行强制故障转移（没有和其他Sentinel节点“协商”），当故障转移完成后，其他Sentinel节点按照故障转移的结果更新自身配置。

8、sentinel ckquorum <master name>：检测当前可达的Sentinel节点总数是否达到<quorum>的个数
127.0.0.1:26379> sentinel ckquorum mymaster-1
OK 3 usable Sentinels. Quorum and failover authorization can be reached

9、sentinel flushconfig：将Sentinel节点的配置强制刷到磁盘上


10、sentinel remove <master name>：取消当前Sentinel节点对于指定<master name>主节点的监控。这个命令仅仅对当前Sentinel节点有效。


11、sentinel monitor <master name> <ip> <port> <quorum>：这个命令和配置文件中的含义是完全一样的，只不过是通过命令的形式来完成Sentinel节点对主节点的监控。


12、sentinel set <master name>：动态修改Sentinel节点配置选项

13、sentinel is-master-down-by-addr：Sentinel节点之间用来交换对主节点是否下线的判断，根据参数的不同，还可以作为Sentinel领导者选举的通信方式。

```

9、Redis Sentinel的客户端

Sentinel节点集合具备了监控、通知、自动故障转移、配置提供者若干功能，也就是说实际上最了解主节点信息的就是Sentinel节点集合，而各个主节点可以通过<master-name>进行标识的，所以，无论是哪种编程语言的客户端，如果需要正确地连接Redis Sentinel，必须有Sentinel节点集合和masterName两个参数。

10、Redis Sentinel客户端基本实现原理

实现一个Redis Sentinel客户端的基本步骤如下：
- 遍历Sentinel节点集合获取一个可用的Sentinel节点，后面会介绍Sentinel节点之间可以共享数据，所以从任意一个Sentinel节点获取主节点信息都是可以的。
- 通过sentinel get-master-addr-by-name master-name这个API来获取对应主节点的相关信息。
- 验证当前获取的“主节点”是真正的主节点，这样做的目的是为了防止故障转移期间主节点的变化。
- 保持和Sentinel节点集合的“联系” ，时刻获取关于主节点的相关“信息” 。

从上面的模型可以看出，Redis Sentinel客户端只有在初始化和切换主节点时需要和Sentinel节点集合进行交互来获取主节点信息，所以在设计客户端时需要将Sentinel节点集合考虑成配置（相关节点信息和变化）发现服务。

11、Java操作Redis Sentinel

略。

12、Redis Sentinel的基本实现原理之一——Redis Sentinel的三个定时任务

Redis Sentinel通过三个定时监控任务完成对各个节点发现和监控：
- 每隔10秒，每个Sentinel节点会向主节点和从节点发送info命令获取最新的拓扑结构。这个定时任务的作用具体可以表现在三个方面：
    1. 通过向主节点执行info命令，获取从节点的信息，这也是为什么Sentinel节点不需要显式配置监控从节点。
    2. 当有新的从节点加入时都可以立刻感知出来。
    3. 节点不可达或者故障转移后，可以通过info命令实时更新节点拓扑信息。
- 每隔2秒，每个Sentinel节点会向Redis数据节点的__sentinel__:hello频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息，同时每个Sentinel节点也会订阅该频道，来了解其他Sentinel节点以及它们对主节点的判断，所以这个定时任务可以完成以下两个工作：
    1. 发现新的Sentinel节点：通过订阅主节点的__sentinel__:hello了解其他的Sentinel节点信息，如果是新加入的Sentinel节点，将该Sentinel节点信息保存起来，并与该Sentinel节点创建连接。
    2. Sentinel节点之间交换主节点的状态，作为后面客观下线以及领导者选举的依据。
- 每隔1秒，每个Sentinel节点会向主节点、从节点、其余Sentinel节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达。这个定时任务是节点失败判定的重要依据。


13、Redis Sentinel的基本实现原理之二——主观下线和客观下线

1）主观下线
- 上面第三个定时任务，每个Sentinel节点会每隔1秒对主节点、从节点、其他Sentinel节点发送ping命令做心跳检测，当这些节点超过down-after-milliseconds没有进行有效回复，Sentinel节点就会对该节点做失败判定，这个行为叫做主观下线。


2）客观下线
- 当Sentinel主观下线的节点是主节点时，该Sentinel节点会通过sentinel is-master-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过<quorum>个数，Sentinel节点认为主节点确实有问题，这时该Sentinel节点会做出客观下线的决定，这样客观下线的含义是比较明显了，也就是大部分Sentinel节点都对主节点的下线做了同意的判定，那么这个判定就是客观的。
- 从节点、Sentinel节点在主观下线后，没有后续的故障转移操作。


sentinel is-master-down-by-addr命令的使用方法如下：
`sentinel is-master-down-by-addr <ip> <port> <current_epoch> <runid>`
- 请求参数：
    - ip：主节点IP。
    - port：主节点端口。
    - current_epoch：当前配置纪元。
    - runid：此参数有两种类型，不同类型决定了此API作用的不同。
        1. 当runid等于“*”时，作用是Sentinel节点直接交换对主节点下线的判定。
        2. 当runid等于当前Sentinel节点的runid时，作用是当前Sentinel节点希望目标Sentinel节点同意自己成为领导者的请求。
- 返回结果：
    - down state：目标Sentinel节点对于主节点的下线判断，1是下线，0是在线。
    - leader_runid：当leader_runid等于“*”时，代表返回结果是用来做主节点是否不可达，当leader_runid等于具体的runid，代表目标节点同意runid成为领导者。
    -  leader_epoch：领导者纪元。

14、Redis Sentinel的基本实现原理之三——领导者Sentinel节点选举

Redis使用了Raft算法实现领导者选举，因为Raft算法相对比较抽象和复杂，所以这里给出一个Redis Sentinel进行领导者选举的大致思路：
- 每个在线的Sentinel节点都有资格成为领导者，当它确认主节点主观下线时候，会向其他Sentinel节点发送sentinel is-master-down-by-addr命令，要求将自己设置为领导者。
- 收到命令的Sentinel节点，如果没有同意过其他Sentinel节点的sentinel is-master-down-by-addr命令，将同意该请求，否则拒绝。
- 如果该Sentinel节点发现自己的票数已经大于等于max(quorum，num(sentinels)/2+1)，那么它将成为领导者。
- 如果此过程没有选举出领导者，将进入下一次选举。

15、故障转移

领导者选举出的Sentinel节点负责故障转移，具体步骤如下：
1. 在从节点列表中选出一个节点作为新的主节点，选择方法如下：
    1. 过滤：“ 不健康” （主观下线、断线）、5秒内没有回复过Sentinel节点ping响应、与主节点失联超过down-after-milliseconds*10秒。
    2. 选择slave-priority（从节点优先级）最高的从节点列表，如果存在则返回，不存在则继续。
    3. 选择复制偏移量最大的从节点（复制的最完整），如果存在则返回，不存在则继续。
    4. 选择runid最小的从节点。
2. Sentinel领导者节点会对第一步选出来的从节点执行slaveof no one命令让其成为主节点。
3. Sentinel领导者节点会向剩余的从节点发送命令，让它们成为新主节点的从节点，复制规则和parallel-syncs参数有关。
4. Sentinel节点集合会将原来的主节点更新为从节点，并保持着对其关注，当其恢复后命令它去复制新的主节点。

```
graph TB

A[从节点列表]-->B[过滤]
B[过滤]-->C{slave-priority最大节点}

C{slave-priority最大节点}-->|yes| D1[选择完毕]
C{slave-priority最大节点}-->|no| D2[继续选择]

D2[继续选择]-->E{复制偏移量最大节点}
E{复制偏移量最大节点}-->|yes| F1[选择完毕]
E{复制偏移量最大节点}-->|no| F2[选择runid最小的节点]
```

16、故障转移日志分析

模拟故障的方法有很多，比较典型的方法有以下几种：
- 方法一，强制杀掉对应节点的进程号，这样可以模拟出宕机的效果。
- 方法二，使用Redis的debug sleep命令，让节点进入睡眠状态，这样可以模拟阻塞的效果。
- 方法三，使用Redis的shutdown命令，模拟正常的停掉Redis。

下表记录了Redis Sentinel在故障转移一些重要的事件消息对应的频道。

状态 | 说明
---|---
+reset-master <instance details> | 主节点被重置
+slave <instance details> | 一个新的从节点被发现并关联
+failover-state-reconf-slaves <instance details> | 故障转移进入reconf-slaves状态
+slave-reconf-sent <instance details> | 领导者Sentinel节点命令其他从节点复制新的主节点
+slave-reconf-inprog <instance details> | 从节点正在重新配置主节点的slave，但是同步过程尚未完成
+slave-reconf-done <instance details> | 其余从节点完成了和新节点的同步
+sentinel <instance details> | 一个新的sentinel节点被发现并关联
+sdown <instance details> | 添加对某个节点被主观下线
-sdown <instance details> | 撤销对某个节点被主观下线
+odown <instance details> | 添加对某个节点被客观下线
-odown <instance details> | 撤销对某个节点被客观下线
+new-epoch <instance details> | 当前纪元被更新
+try-failover <instance details> | 故障转移开始
+elected-leader <instance details> | 选出了故障转移的Sentinel节点
+failover-state-select-slave <instance details> | 故障转移进入select-slave状态(寻找合适的从节点)
no-good-slave <instance details> | 没有找到合适的从节点
selected-slave <instance details> | 找到了合适的从节点
failover-state-send-slaveof-noone <instance details> | 故障转移进入failover-state-send-slaveof-noone状态(对找到的从节点执行slaveof no one)
failover-end-for-timeout <instance details> | 故障转移由于超时而终止
failover-end <instance details> | 故障转移顺利完成
switch-master <master name><oldop><oldport><newip><newport> | 更新主节点信息，这个是许多客户端重点关注的

<instance details>格式如下：
`<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>`

部署各个节点的机器时间尽量要同步，否则日志的时序性会混乱，例如可以给机器添加NTP服务来同步时间。

17、节点运维

1）节点下线
- 临时下线： 暂时将节点关掉，之后还会重新启动，继续提供服务。
- 永久下线： 将节点关掉后不再使用，需要做一些清理工作，如删除配置文件、持久化文件、日志文件。

- Redis Sentinel存在多个从节点时，如果想将指定从节点晋升为主节点，可以将其他从节点的slavepriority配置为0 ，但是需要注意failover后，将slave-priority调回原值。

- 需要注意的是，Sentinel节点依然会对这些下线节点进行定期监控，这是由Redis Sentinel的设计思路所决定的。

2）节点上线
- 添加从节点的方法：添加slaveof{masterIp}{masterPort}的配置，使用redis-server启动即可，它将被Sentinel节点自动发现。
- 添加Sentinel节点的方法：添加sentinel monitor主节点的配置，使用redis-sentinel启动即可，它将被其余Sentinel节点自动发现。
- 添加主节点：因为Redis Sentinel中只能有一个主节点，所以不需要添加主节点，如果需要替换主节点，可以使用Sentinel failover手动故障转移。

3）节点配置
- Sentinel节点配置尽可能一致，这样在判断节点故障时会更加准确。
- Sentinel节点支持的命令非常有限，例如config命令是不支持的，而Sentinel节点也需要dir、loglevel之类的配置，所以尽量在一开始规划好，不过所幸Sentinel节点不存储数据，如果需要修改配置，重新启动即可。
- Sentinel节点只支持如下命令：ping、sentinel、subscribe、unsubscribe、 psubscribe、punsubscribe、publish、info、role 、client、shutdown。


18、Redis Sentinel读写分离设计思路

Redis Sentinel在对各个节点的监控中，如果有对应事件的发生，都会发出相应的事件消息，其中和从节点变动的事件有以下几个：
- +switch-master：切换主节点（原来的从节点晋升为主节点），说明减少了某个从节点。
- +convert-to-slave：切换从节点（原来的主节点降级为从节点），说明添加了某个从节点。
- +sdown：主观下线，说明可能某个从节点可能不可用（因为对从节点不会做客观下线），所以在实现客户端时可以采用自身策略来实现类似主观下线的功能。
- +reboot：重新启动了某个节点，如果它的角色是slave，那么说明添加了某个从节点。

所以在设计Redis Sentinel的从节点高可用时，只要能够实时掌握所有从节点的状态，把所有从节点看做一个资源池，无论是上线还是下线从节点，客户端都能及时感知到（将其从资源池中添加或者删除），这样从节点的高可用目标就达到了。

19、汇总
- Redis Sentinel从Redis2.8版本开始才正式生产可用，之前版本生产不可用。
- 尽可能在不同物理机上部署Redis Sentinel所有节点。
- Redis Sentinel中的Sentinel节点个数应该为大于等于3且最好为奇数。
- Redis Sentinel中的数据节点与普通数据节点没有区别。
- 客户端初始化时连接的是Sentinel节点集合，不再是具体的Redis节点，但Sentinel只是配置中心不是代理。
- Redis Sentinel实现读写分离高可用可以依赖Sentinel节点的消息通知，获取Redis数据节点的状态变化。
