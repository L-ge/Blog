###### 十二、开发运维的“ 陷阱”

1、Linux配置优化之内存分配控制

1）vm.overcommit_memory
- Linux操作系统对大部分申请内存的请求都回复yes，以便能运行更多的程序。因为申请内存后，并不会马上使用内存，这种技术叫做overcommit。
- 本节的可用内存代表物理内存与swap之和。
- 日志中的Background save代表的是bgsave和bgrewriteaof，如果当前可用内存不足，操作系统应该如何处理fork操作。如果vm.overcommit_memory=0，代表如果没有可用内存，就申请内存失败，对应到Redis就是执行fork失败，Redis建议把这个值设置为1，是为了让fork操作能够在低内存下也执行成功。

vm.overcommit_memory用来设置内存分配策略，有三个可选值，如下表：
值 | 含义
---|---
0 | 表示内核将检查是否有足够的可用内存，如果有足够的可用内存，内存申请通过，否则内存申请失败，并把错误返回给应用进程。
1 | 表示内核允许超量使用内存直到用完为止
2 | 表示内核决不过量的("never overcommit")使用内存，即系统整个内存地址空间不能超过swap+50%的RAM值，50%是overcommit_ratio默认值，此参数同样支持修改。

2）获取和设置
```
// 获取
# cat /proc/sys/vm/overcommit_memory
0

// 设置
echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
sysctl vm.overcommit_memory=1
```
3）最佳实践
- Redis设置合理的maxmemory，保证机器有20%~30%的闲置内存。
- 集中化管理AOF重写和RDB的bgsave。
- 设置vm.overcommit_memory= 1，防止极端情况下会造成fork失败。

2、Linux配置优化之swappiness

1）参数说明
- swap对于操作系统来比较重要，当物理内存不足时，可以将一部分内存页进行swap操作，已解燃眉之急。但世界上没有免费午餐，swap空间由硬盘提供，对于需要高并发、高吞吐的应用来说，磁盘IO通常会成为系统瓶颈。在Linux中，并不是要等到所有物理内存都使用完才会使用到swap，系统参数swppiness会决定操作系统使用swap的倾向程度。swappiness的取值范围是0~100，swappiness的值越大，说明操作系统可能使用swap的概率越高，swappiness值越低，表示操作系统更加倾向于使用物理内存。swap的默认值是60。
- OOM（Out Of Memory）killer机制是指Linux操作系统发现可用内存不足时，强制杀死一些用户进程（非内核进程），来保证系统有足够的可用内存进行分配。
- swappiness参数在Linux3.5版本前后的表现并不完全相同，Redis运维人员在设置这个值需要关注当前操作系统的内核版本。

swapniess重要值策略说明如下表：
值 | 策略
---|---
0 | Linux3.5以及以上：宁愿用OOM killer也不用swap<br>Linux3.4以及更早：宁愿swap也不用OOM killer
1 | Linux3.5以及以上：宁愿swap也不用OOM killer
60 | 默认值
100 | 操作系统会主动地使用swap

2）设置方法
```
echo {bestvalue} > /proc/sys/vm/swappiness

// 但是上述方法在系统重启后就会失效，为了让配置在重启Linux操作系统后立即生效，只需要在/etc/sysctl.conf追加vm.swappiness={bestvalue}即可。
echo vm.swappiness={bestvalue} >> /etc/sysctl.conf
```
- /proc/sys/vm/swappiness是设置操作，/etc/sysctl.conf是追加操作。

3）如何监控swap
- 查看swap的总体情况：Linux提供了free命令来查询操作系统的内存使用情况，其中也包含了swap的相关使用情况。
- 实时查看swap的使用：Linux提供了vmstat命令查询系统的相关性能指标，其中包含负载、CPU、内存、swap、IO的相关属性。但其中和swap有关的指标是si和so，它们分别代表操作系统的swap in和swap out。si和so都为0，代表当前没有使用swap。
- 查看指定进程的swap使用情况：Linux操作系统中，/proc/{pid}目录是存储指定进程的相关信息，其中/proc/{pid}/smaps记录了当前进程所对应的内存映像信息，这个信息对于查询指定进程的swap使用情况很有帮助。
```
// 通过info server获取Redis的进程号process_id：
redis-cli -h ip -p port info server | grep process_id
process_id:986

// 通过cat/proc/986/smaps查询Redis的smaps信息，会输出多个内存块信息。其中Swap字段代表该内存块存在swap分区的数据大小。

// 通过执行如下命令，就可以找到每个内存块镜像信息中，这个进程使用到的swap量，通过求和就可以算出总的swap用量：
cat /proc/986/smaps | grep Swap
```
- 如果Linux>3.5，vm.swapniess=1，否则vm.swapniess=0，从而实现如下两个目标：
    1. 物理内存充足时候，使Redis足够快。
    2. 物理内存不足时候，避免Redis死掉（如果当前Redis为高可用，死掉比阻塞更好） 。

3、Linux配置优化之THP
- Redis建议修改Transparent Huge Pages（THP）的相关配置，Linux kernel在2.6.38内核增加了THP特性，支持大内存页（2MB）分配，默认开启。当开启时可以降低fork子进程的速度，但fork操作之后，每个内存页从原来4KB变为2MB，会大幅增加重写期间父进程内存消耗。同时每次写命令引起的复制内存页单位放大了512倍，会拖慢写操作的执行时间，导致大量写操作慢查询。因此Redis日志中建议将此特性进行禁用。
- 在设置THP配置时需要注意：有些Linux的发行版本没有将THP放到/sys/kernel/mm/transparent_hugepage/enabled中，例如Red Hat6以上的THP配置放到/sys/kernel/mm/redhat_transparent_hugepage/enabled中。而Redis源码中 检查THP时，把THP位置写死/sys/kernel/mm/transparent_hugepage/enabled。所以在发行版中，虽然没有THP的日志提示，但是依然存在THP所带来的问题。
```
禁用方法如下：
echo never > /sys/kernel/mm/transparent_hugepage/enabled

为了使机器重启后THP配置依然生效，可以在/etc/rc.local中追加echo never>/sys/kernel/mm/transparent_hugepage/enabled。

echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
```

4、Linux配置优化之OOM killer
- OOM killer会在可用内存不足时选择性地杀掉用户进程。
- OOM killer进程会为每个用户进程设置一个权值，这个权值越高，被“下手”的概率就越高，反之概率越低。每个进程的权值存放在/proc/{progress_id}/oom_score中，这个值是受/proc/{progress_id}/oom_adj的控制，oom_adj在不同的Linux版本中最小值不同，可以参考Linux源码中oom.h（从-15到-17）。当oom_adj设置为最小值时，该进程将不会被OOM killer杀掉。
- 对于Redis所在的服务器来说，可以将所有Redis的oom_adj设置为最低值或者稍小的值，降低被OOM killer杀掉的概率。
- 笔者认为oom_adj参数只能起到辅助作用，合理地规划内存更为重要。
- 通常在高可用情况下，被杀掉比僵死更好，因此不要过多依赖oom_adj配置。
```
设置方法如下：
echo {value} > /proc/${process_id}/oom_adj

// 将所有Redis的oom_adj设置为最低值
for redis_pid in $ (pgrep -f "redis-server")
do
    echo -17 > /proc/${redis_pid}/oom_adj
done
```

5、Linux配置优化之使用NTP
- NTP（Network Time Protocol，网络时间协议）是一种保证不同机器时钟一致性的服务。
- 集群的时间不一致不会影响集群功能，集群节点依赖各自时钟。
- 可以每天定时去同步一次系统时间，从而使得集群中的时间保持统一。

6、Linux配置优化之ulimit
- 在Linux中，可以通过ulimit查看和设置系统当前用户进程的资源数。其中ulimit-a命令包含的open files参数，是单个用户同时打开的最大文件个数。
- Redis允许同时有多个客户端通过网络进行连接，可以通过配置maxclients来限制最大客户端连接数。对Linux操作系统来说，这些网络连接都是文件句柄。
- Redis建议把open files至少设置成10032，因为maxclients默认是10000，这些是用来处理客户端连接的，除此之外，Redis内部会使用最多32个文件描述符，所以这里的10032=10000+32。
- Redis不能将open files设置成10032，因为它没有权限设置。
- open files的限制优先级比maxclients大。
```
Open files的设置方法如下：
ulimit –Sn {max-open-files}
```

7、Linux配置优化之TCP backlog
- Redis默认的tcp-backlog值为511，可以通过修改配置tcp-backlog进行调整，如果Linux的tcp-backlog小于Redis设置的tcp-backlog，那么在Redis启动时会看到相关日志。
```
查看方法：
# cat /proc/sys/net/core/somaxconn
128

修改方法：
echo 511 > /proc/sys/net/core/somaxconn
```

8、flushall/flushdb误操作之缓存与存储
- 被误操作flush后，根据当前Redis是缓存还是存储使用策略有所不同：
    1. 缓存：对于业务数据的正确性可能造成损失还小一点，因为缓存中的数据可以从数据源重新进行构建。
    2. 存储：对业务方可能会造成巨大的影响，也许flush操作后的数据是重要配置，也可能是一些基础数据，也可能是业务上的重要一环，如果没有提前做业务降级操作，那么最终反馈到用户的应用可能就是报错或者空白页面等，其后果不堪设想。即使做了相应的降级或者容错处理，对于用户体验也有一定的影响。

9、flushall/flushdb误操作之借助AOF机制恢复
- Redis执行了flush操作后，对AOF持久化文件的影响如下：
    1. appendonly no：对AOF持久化没有任何影响，因为根本就不存在AOF文件。
    2. appendonly yes：只不过是在AOF文件中追加了一条记录(flush操作记录)。
    3. 虽然Redis中的数据被清除掉了，但是AOF文件还保存着flush操作之前完整的数据，这对恢复数据是很有帮助的。
- 如果发生了AOF重写，Redis遍历所有数据库重新生成AOF文件，并会覆盖之前的AOF文件。所以如果AOF重写发生了，也就意味着之前的数据就丢掉了，那么利用AOF文件来恢复的办法就失效了。所以当误操作后，需要考虑如下两件事。
    1. 调大AOF重写参数auto-aof-rewrite-percentage和auto-aof-rewrite-min-size ，让Redis不能产生AOF自动重写。
    2. 拒绝手动bgrewriteaof。
- 如果要用AOF文件进行数据恢复，那么必须要将AOF文件中的flushall相关操作去掉，为了更加安全，可以在去掉之后使用redis-check-aof这个工具去检验和修复一下AOF文件，确保AOF文件格式正确，保证数据恢复正常。

10、flushall/flushdb误操作之RDB变化
- 如果没有开启RDB的自动策略，那么除非手动执行过save、bgsave或者发生了主从的全量复制，否则RDB文件也会保存flush操作之前的数据，可以作为恢复数据的数据源。
- 防止手动执行save、bgsave ，如果此时执行save、bgsave，新的RDB文件就不会包含flush操作之前的数据，被老的RDB文件进行覆盖。
- RDB文件中的数据可能没有AOF实时性高，也就是说，RDB文件很可能很久以前主从全量复制生成的，或者之前用save、bgsave备份的。
- 如果开启了RDB的自动策略，由于flush涉及键值数量较多，RDB文件会被清除，意味着使用RDB恢复基本无望。
- 如果AOF已经开启了，那么用AOF来恢复是比较合理的方式，但是如果AOF关闭了，那么RDB虽然数据不是很实时，但是也能恢复部分数据，完全取决于RDB是什么时候备份的。当然RDB并不是一无是处，它的恢复速度要比AOF快很多，但是总体来说对于flush操作之后不是最好的恢复数据源。

11、flushall/flushdb误操作之从节点变化
- Redis从节点同步了主节点的flush命令，所以从节点的数据也是被清除了，从节点的RDB和AOF的变化与主节点没有任何区别。

12、flushall/flushdb误操作之快速恢复数据
```
下面使用AOF作为数据源进行恢复演练:
1、防止AOF重写。快速修改Redis主从的auto-aof-rewrite-percentage和auto-aof-rewrite-min-size变为一个很大的值，从而防止了AOF重写的发生，例如：
config set auto-aof-rewrite-percentage 1000
config set auto-aof-rewrite-min-size 100000000000

2、去掉主从AOF文件中的flush相关内容：
*1
$8
flushall

3、重启Redis主节点服务器，恢复数据。
```

13、安全的Redis概述
- 被攻击的Redis有如下特点：
    1. Redis所在的机器有外网IP。
    2. Redis以默认端口6379为启动端口，并且是对外网开放的。
    3. Redis是以root用户启动的。
    4. Redis没有设置密码。
    5. Redis的bind设置为0.0.0.0或者""。

14、安全的Redis之Redis密码机制

1）简单的密码机制
- Redis提供了requirepass配置为Redis提供密码功能，如果添加这个配置，客户端就不能通过redis-cli–h{ip}–p {port}来执行命令。
- Redis提供了两种方式访问配置了密码的Redis：
    1. redis-cli -a参数。使用redis-cli连接Redis时，添加-a加密码的参数，如果密码正确就可以正常访问Redis了。
    2. auth命令。通过redis-cli连接后，执行auth加密码命令，如果密码正确就可以正常访问访问Redis了。
- 这种密码机制能在一定程度上保护Redis的安全，但是在使用requirepass时候要注意一下几点：
    1. 密码要足够复杂（64个字节以上），因为Redis的性能很高，如果密码比较简单，完全是可以在一段时间内通过暴力破解来破译密码。
    2. 如果是主从结构的Redis，不要忘记在从节点的配置中加入masterauth（master的密码）配置，否则会造成主从节点同步失效。
    3. auth是通过明文进行传输的，所以也不是100%可靠，如果被攻击者劫持也相当危险。

15、安全的Redis之伪装危险命令

- Redis中包含了很多“危险”的命令，一旦错误使用或者误操作，后果不堪设想，例如如下命令：(Redis提供了rename-command配置解决这个问题。管理员可以对认为比较危险的命令做rename-command处理。)
    1. keys：如果键值较多，存在阻塞Redis的可能性。
    2. flushall/flushdb：数据全部被清除。
    3. save：如果键值较多，存在阻塞Redis的可能性。
    4. debug：例如debug reload会重启Redis。
    5. config：config应该交给管理员使用。
    6. shutdown：停止Redis。
- 使用了rename-command时可能会带来如下麻烦：
    1. 管理员要对自己的客户端进行修改，例如jedis.flushall()操作内部使用的是flushall命令，如果用rename-command后需要修改为新的命令，有一定的开发和维护成本。
    2. rename-command配置不支持config set，所以在启动前一定要确定哪些命令需要使用rename-command。
    3. 如果AOF和RDB文件包含了rename-command之前的命令，Redis将无法启动，因为此时它识别不了rename-command之前的命令。
    4. Redis源码中有一些命令是写死的，rename-command可能造成Redis无法正常工作。例如Sentinel节点在修改配置时直接使用了config命令，如果对config使用rename-command，会造成Redis Sentinel无法正常工作。
- 在使用rename-command的相关配置时，需要注意以下几点：
    1. 对于一些危险的命令（例如flushall），不管是内网还是外网，一律使用rename-command配置。
    2. 建议第一次配置Redis时，就应该配置rename-command，因为rename-command不支持config set。
    3. 如果涉及主从关系，一定要保持主从节点配置的一致性，否则存在主从数据不一致的可能性。

16、安全的Redis之防火墙
- 可以使用防火墙限制输入和输出的IP或者IP范围、端口或者端口范围，在比较成熟的公司都会对有外网IP的服务器做一些端口的限制，例如只允许80端口对外开放。因为一般来说，开放外网IP的服务器中Web服务器比较多，但通常存储服务器的端口无需对外开放，防火墙是一个限制外网访问Redis的必杀技。

17、安全的Redis之bind
- bind指定的是Redis和哪个网卡进行绑定，和客户端是什么网段没有关系。
- Redis3.0中bind默认值""，也就是不限制网卡的访问，但是在Redis3.2中必须显示的配置bind0.0.0.0才可以达到这种效果。
- 建议：
    1. 如果机器有外网IP，但部署的Redis是给内部使用的，建议去掉外网网卡或者使用bind配置限制流量从外网进入。
    2. 如果客户端和Redis部署在一台服务器上，可以使用回环地址（127.0.0.1）。
    3. bind配置不支持config set，所以尽可能在第一次启动前配置好。
- Redis3.2提供了protected-mode配置（默认开启），如果当前Redis没有配置密码，没有配置bind，那么只允许来自本机的访问，也就是相当于配置了bind 127.0.0.1。

18、安全的Redis之定期备份数据
- 定期备份持久化数据是一个比较好的习惯。

19、安全的Redis之不使用默认端口
- Redis 的默认端口是6379，不使用默认端口从一定程度上可降低被入侵者发现的可能性，因为入侵者通常本身也是一些攻击程序，对目标服务器进行端口扫描，例如MySQL的默认端口3306、Memcache的默认端口11211、Jetty的默认端口8080等都会被设置成攻击目标，Redis作为一款较为知名的NoSQL服务，6379必然也在端口扫描的列表中，虽然不设置默认端口还是有可能被攻击者入侵，但是能够在一定程度上降低被攻击的概率。

20、安全的Redis之使用非root用户启动
- 建议在启动Redis服务的时候使用非root用户启动。事实上许多服务，例如Resin、Jetty、HBase、Hadoop都建议使用非root启动。

21、处理bigkey
- bigkey是指key对应的value所占的内存空间比较大，例如一个字符串类型的value可以最大存到512MB，一个列表类型的value最多可以存储2^32-1个元素。如果按照数据结构来细分的话，一般分为字符串类型bigkey和非字符串类型bigkey。
    1. 字符串类型：体现在单个value值很大，一般认为超过10KB就是bigkey，但这个值和具体的OPS相关。
    2. 非字符串类型：哈希、列表、集合、有序集合，体现在元素个数过多。

22、bigkey的危害
- bigkey的危害体现在三个方面：
    1. 内存空间不均匀（平衡）：例如在Redis Cluster中，bigkey会造成节点的内存空间使用不均匀。
    2. 超时阻塞：由于Redis单线程的特性，操作bigkey比较耗时，也就意味着阻塞Redis可能性增大。
    3. 网络拥塞：每次获取bigkey产生的网络流量较大。

23、如何发现bigkey
- redis-cli--bigkeys可以命令统计bigkey的分布。
- 判断一个key是否为bigkey，只需要执行debug object key查看serializedlength属性即可，它表示key对应的value序列化之后的字节数。
- serializedlength不代表真实的字节大小，它返回对象使用RDB编码序列化后的长度，值会比strlen的结果偏小，但是对于排查bigkey有一定辅助作用，因为不是每种数据结构都有类似strlen这样的方法。
- 在实际生产环境中发现bigkey的两种方式如下：
    1. 被动收集：建议修改Redis客户端，当抛出异常时打印出所操作的key，方便排查bigkey问题。
    2. 主动检测：scan+debug object：如果怀疑存在bigkey，可以使用scan命令渐进的扫描出所有的key，分别计算每个key的serializedlength，找到对应bigkey进行相应的处理和报警，这种方式是比较推荐的方式。
- 如果键值个数比较多，scan+debug object会比较慢，可以利用Pipeline机制完成。
- 对于元素个数较多的数据结构，debug object执行速度比较慢，存在阻塞Redis的可能。
- 如果有从节点，可以考虑在从节点上执行。

24、如何删除bigkey
- 无论是什么数据结构，del命令都将其删除。但不建议这样做，因为删除bigkey通常来说会阻塞Redis服务。
- 除了string类型，其他四种数据结构删除的速度有可能很慢，这样增大了阻塞Redis的可能性。
- 以hash为例子，可以使用hscan命令，每次获取部分（例如100个）field-value，再利用hdel删除每个field（为了快速可以使用Pipeline）。

25、bigkey最佳实践思路
- 例如出现了bigkey，要思考一下可不可以做一些优化（例如拆分数据结构）尽量让这些bigkey消失在业务中，如果bigkey不可避免，也要思考一下要不要每次把所有元素都取出来（例如有时候仅仅需要hmget，而不是hgetall）。
- Redis将在4.0版本支持lazy delete free的模式，那时删除bigkey不会阻塞Redis。

26、寻找热点key

1）客户端
- 客户端其实是距离key“最近”的地方，因为Redis命令就是从客户端发出的，例如在客户端设置全局字典（key和调用次数），每次调用Redis命令时，使用这个字典进行记录。
- 使用客户端进行热点key的统计非常容易实现，但是同时问题也非常多：
    1. 无法预知key的个数，存在内存泄露的危险。
    2. 对于客户端代码有侵入，各个语言的客户端都需要维护此逻辑，维护成本较高。
    3. 只能了解当前客户端的热点key，无法实现规模化运维统计。

2）代理端
- 此架构是最适合做热点key统计的，因为代理是所有Redis客户端和服务端的桥梁。但并不是所有Redis都是采用此种架构。

3）Redis服务端
- 使用monitor命令统计热点key是很多开发和运维人员首先想到，monitor命令可以监控到Redis执行的所有命令。
- 为了减少网络开销以及加快输出缓冲区的消费速度，monitor尽可能在本机执行。
- monitor命令在高并发条件下，会存在内存暴增和影响Redis性能的隐患，所以此种方法适合在短时间内使用。
- 只能统计一个Redis节点的热点key，对于Redis集群需要进行汇总统计。

4）机器
- Redis客户端使用TCP协议与服务端进行交互，通信协议采用的是RESP。如果站在机器的角度，可以通过对机器上所有Redis端口的TCP数据包进行抓取完成热点key的统计。
- 需要一定的开发成本，但是一些开源方案实现了该功能，例如ELK（ElasticSearch Logstash Kibana）体系下的packetbeat插件，可以实现对Redis、MySQL等众多主流服务的数据包抓取、分析、报表展示。
- 由于是以机器为单位进行统计，要想了解一个集群的热点key，需要进行后期汇总。

方案 | 优点 | 缺点
---|---|---
客户端 | 实现简单 | - 内存泄漏隐患<br>- 维护成本高<br>- 只能统计单个客户端
代理 | 代理是客户端和服务端的桥梁，实现最方案最系统 | 增加代理端的开发部署成本
服务端 | 实现简单 | - Monitor本身的使用成本和危害，只能短时间使用<br>- 只能统计单个Redis节点
机器 | 对于客户端和服务端无侵入和影响 | 需要专业的运维团队开发，并且增加了机器的部署成本

下面是三种方案的思路：
- 拆分复杂数据结构：如果当前key的类型是一个二级数据结构，例如哈希类型。如果该哈希元素个数较多，可以考虑将当前hash进行拆分，这样该热点key可以拆分为若干个新的key分布到不同Redis节点上，从而减轻压力。
- 迁移热点key：以Redis- Cluster为例，可以将热点key所在的slot单独迁移到一个新的Redis节点上，但此操作会增加运维成本。
- 本地缓存加通知机制：可以将热点key放在业务端的本地缓存中，因为是在业务端的本地内存中，处理能力要高出Redis数十倍，但当数据更新时，此种模式会造成各个业务端和Redis数据不一致，通常会使用发布订阅机制来解决类似问题。

27、汇总
- bigkey的危害不容忽视：数据倾斜、超时阻塞、网络拥塞，可能是Redis生产环境中的一颗定时炸弹，删除bigkey时通常使用渐进式遍历的方式，防止出现Redis阻塞的情况。
- 通过客户端、代理、monitor、机器抓包四种方式找到热点key，这几种方式各具优势，具体使用哪种要根据当前场景来决定。
