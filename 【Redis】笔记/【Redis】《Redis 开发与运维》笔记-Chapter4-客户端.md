###### 四、客户端

1、客户端通信协议

- 客户端与服务端之间的通信协议是在TCP协议之上构建的。
- Redis制定了RESP（REdis Serialization Protocol，Redis序列化协议）实现客户端与服务端的正常交互，这种协议简单高效，既能够被机器解析，又容易被人类识别。

```
// 客户端发送一条set hello world命令给服务端，按照RESP的标准，客户端需要将其封装为如下格式（每行用\r\n分隔） ：
*3
$3
SET
$5
hello
$5
world

实际传输格式为如下代码：
*3\r\n$3\r\nSET\r\n$5\r\nhello\r\n$5\r\nworld\r\n

// 这样Redis服务端能够按照RESP将其解析为set hello world命令，执行后回复的格式如下：
+OK
```

下面对命令的一些格式进行说明：
```
1、发送命令格式
RESP的规定一条命令的格式如下，CRLF代表"\r\n"。
*<参数数量> CRLF
$<参数1的字节数量> CRLF
<参数1> CRLF
. . .
$<参数N的字节数量> CRLF
<参数N> CRLF

2、返回结果格式
Redis的返回结果类型分为以下五种：
· 状态回复： 在RESP中第一个字节为"+"。例如set。
· 错误回复： 在RESP中第一个字节为"-"。例如错误命令。
· 整数回复： 在RESP中第一个字节为"："。例如incr。
· 字符串回复： 在RESP中第一个字节为"$"。例如get。
· 多条字符串回复： 在RESP中第一个字节为"*"。例如mget。

redis-cli.c源码对命令结果的解析结构如下：
static sds cliFormatReplyTTY (redisReply *r, char *prefix) {
    sds out = sdsempty ();
    switch (r->type) {
    case REDIS_REPLY_ERROR :  
        // 处理错误回复
    case REDIS REPLY STATUS :
        // 处理状态回复
    case REDIS_REPLY_INTEGER :
        // 处理整数回复
    case REDIS_REPLY_STRING :
        // 处理字符串回复
    case REDIS_REPLY_NIL :
        // 处理空
    case REDIS_REPLY_ARRAY :
        // 处理多条字符串回复
    return out;
}
```

为了看到Redis服务端返回的“真正” 结果，可以使用nc命令、telnet命令、甚至写一个socket程序进行模拟。
```
以nc命令进行演示，首先使用 nc 127.0.0.1 6379连接到Redis：
nc 127.0.0.1 6379

状态回复：set hello world的返回结果为+OK：
set hello world
+OK

错误回复：由于sethx这条命令不存在，那么返回结果就是"-"号加上错误消息：
sethx
-ERR unknown command 'sethx '

整数回复：当命令的执行结果是整数时，返回结果就是整数回复:
incr counter
:1

字符串回复：当命令的执行结果是字符串时，返回结果就是字符串回复：
get hello
$5
world

多条字符串回复： 当命令的执行结果是多条字符串时，返回结果就是多条字符串回复：
mset java jedis python redis-py
+OK
mget java python
*2
$5
jedis
$8
redis-py

注意，无论是字符串回复还是多条字符串回复，如果有nil值，那么会返回$- 1。
get not_exist_key
$-1

如果批量操作中包含一条为nil值的结果，那么返回结果如下：
mget hello not_exist_key java
*3
$5
world
$-1
$5
jedis
```
有了RESP提供的发送命令和返回结果的协议格式，各种编程语言就可以利用其来实现相应的Redis客户端。

2、Java客户端Jedis

略...

3、Python客户端redis-py

1）获取redis-py。

redis-py需要Python2.7以上版本。

如何获取安装redis-py，方法有三种：
```
第一，使用pip进行安装：
pip install redis

第二，使用easy_install进行安装：
easy_install redis

第三，使用源码安装：
wget https://github.com/andymccurdy/redis-py/archive/2.10.5.zip 
unzip redis-2.10.5.zip
cd redis-2.10.5
#安装redis-py
python setup.py install
```

2）redis-py的基本使用方法。
```
①导入依赖库：
import redis

②生成客户端连接：需要Redis的实例IP和端口两个参数：
client = redis.StrictRedis (host='127.0.0.1', port=6379)

③执行命令：redis-py的API保留了Redis API的原始风格：
# True
client.set (key, "python-redis")
# world
client.get (key)

import redis
client = redis.StrictRedis (host='127.0.0.1', port=6379)
key = "hello"
setResult = client.set (key, "python-redis") 
print setResult
value = client.get (key)
print "key :" + key + ", value :" + value
```

3）redis-py的Pipeline的使用。
```
①引入依赖，生成客户端连接：
import redis
client = redis.StrictRedis (host='127.0.0.1 ', port=6379)

②生成Pipeline：注意client.pipeline包含了一个参数，如果transaction=False代表不使用事务：
pipeline = client.pipeline (transaction=False)

③将命令封装到Pipeline中，此时命令并没有真正执行：
pipeline.set ("hello","world")
pipeline.incr ("counter")

④执行Pipeline：
# [True, 3]
result = pipeline.execute ()
```

4）redis-py的Lua脚本使用。

redis-py提供了三个重要的函数实现Lua脚本的执行：
```
eval (String script, int keyCount, String . . . params)
script_load (String script)
evalsha (String sha1, int keyCount, String . . . params)

import redis
client = redis.StrictRedis (host='127.0.0.1', port=6379)
script = "return redis.call ('get', KEYS [1])"
#输出结果为world
print client.eval (script,1,"hello")

script_load和evalsha 函数要一起使用，首先使用script_load将脚本加载到Redis中：
import redis
client = redis.StrictRedis (host='127.0.0.1', port=6379)
script = "return redis.call ('get ',KEYS [1])"
scriptSha = client.script_load (script)
print client.evalsha (scriptSha, 1, "hello");
```
- eval函数有三个参数，分别是：
    1. script：Lua脚本内容。
    2. keyCount：键的个数。
    3. params：相关参数KEYS和ARGV。
- evalsha 函数用来执行脚本的哈希值，它需要三个参数：
    1. scriptSha：脚本的SHA1。
    2. keyCount：键的个数。
    3. params：相关参数KEYS和ARGV。

4、客户端API

1）client list命令能列出与Redis服务端相连的所有客户端连接信息。
```
// 在一个Redis实例上执行client list的结果：
127.0.0.1:6379> client list
id=254487 addr=10.2.xx.234:60240 fd=1311 name= age=8888581 idle=8888581 flags=N
    db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cm
id=300210 addr=10.2.xx.215:61972 fd=3342 name= age=8054103 idle=8054103 flags=N
    sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cm
...

127.0.0.1:6390> info memory
...
maxmemory_human:4.00G.
...
127.0.0.1:6390> info clients
···
connected clients:1414                      // 当前Redis的连接数
client_longest_output_list:4869             // client_longest_output_list代表输出缓冲区列表最大对象数
client_biggest_input_buf:2097152
···

输出缓冲区对应的配置规则是：
client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
<class>：客户端类型，分为三种。a）normal：普通客户端；b）slave：slave客户端，用于复制；c）pubsub：发布订阅客户端。
<hard limit>：如果客户端使用的输出缓冲区大于<hard limit>，客户端会被立即关闭。
<soft limit>和<soft seconds>：如果客户端使用的输出缓冲区超过了<soft limit>并且持续了<soft limit>秒，客户端会被立即关闭。

Redis的默认配置是：
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

通过过Redis源码中redis.h的redisClient结构体（Redis3.2版本变为Client）可以看到固定缓冲区和动态缓冲区的实现细节：
typedef struct redisClient {
    // 动态缓冲区列表
    list *reply;
    // 动态缓冲区列表的长度 (对象个数)
    unsigned long reply_bytes;
    // 固定缓冲区已经使用的字节数
    int bufpos;
    // 字节数组作为固定缓冲区
    char buf [REDIS_REPLY_CHUNK_BYTES];
} redisClient;

可以通过config set maxclients对最大客户端连接数进行动态设置：
127.0.0.1:6379> config get maxclients
1) "maxclients"
2) "10000"
127.0.0.1:6379> config set maxclients 50
OK
127.0.0.1:6379> config get maxclients
1) "maxclients"
2) "50"

#Redis默认的timeout是0，也就是不会检测客户端的空闲
127.0.0.1:6379> config set timeout 30
OK

```
- client list输出结果的每一行代表一个客户端的信息，可以看到每行包含了十几个属性，它们是每个客户端的一些执行状态：
    1. 标识：id、addr、fd、name，这四个属性属于客户端的标识：
        1. id：客户端连接的唯一标识，这个id是随着Redis的连接自增的，重启Redis后会重置为0。
        2. addr：客户端连接的ip和端口。
        3. fd：socket的文件描述符，与lsof命令结果中的fd是同一个，如果fd=-1代表当前客户端不是外部客户端，而是Redis内部的伪装客户端。
        4. name：客户端的名字。
    2. 输入缓冲区：qbuf、qbuf-free
        1. Redis为每个客户端分配了输入缓冲区，它的作用是将客户端发送的命令临时保存，同时Redis从会输入缓冲区拉取命令并执行，输入缓冲区为客户端发送命令到Redis执行命令提供了缓冲功能。client list中qbuf和qbuf-free分别代表这个缓冲区的总容量和剩余容量。
        2. Redis没有提供相应的配置来规定每个缓冲区的大小，输入缓冲区会根据输入内容大小的不同动态调整，只是要求每个客户端缓冲区的大小不能超过1G，超过后客户端将被关闭。
    3. 输出缓冲区：obl、oll、omem
        1. Redis为每个客户端分配了输出缓冲区，它的作用是保存命令执行的结果返回给客户端，为Redis和客户端交互返回结果提供缓冲。与输入缓冲区不同的是，输出缓冲区的容量可以通过参数client-output-buffer-limit来进行设置，并且输出缓冲区做得更加细致，按照客户端的不同分为三种：普通客户端、发布订阅客户端、slave客户端。
        2. obl代表固定缓冲区的长度，oll代表动态缓冲区列表的长度，omem代表使用的字节数。
    4. 客户端的存活状态：age和idle分别代表当前客户端已经连接的时间和最近一次的空闲时间。当age等于idle时，说明连接一直处于空闲状态。
    5. 客户端的限制maxclients和timeout
        1. Redis提供了maxclients参数来限制最大客户端连接数，一旦连接数超过maxclients，新的连接将被拒绝。maxclients默认值是10000，可以通过info clients来查询当前Redis的连接数。
        2. Redis提供了timeout（单位为秒）参数来限制连接的最大空闲时间，一旦客户端连接的idle时间超过了timeout，连接将会被关闭。在实际开发和运维中，需要将timeout设置成大于0，例如可以设置为300秒，同时在客户端使用上添加空闲检测和验证等等措施。
    6. flag是用于标识当前客户端的类型，例如flag=S代表当前客户端是slave客户端、flag=N代表当前是普通客户端，flag=O代表当前客户端正在执行monitor命令。
    7. 其他参数
        1. db：当前客户端正在使用的数据库索引下标
        2. sub/psub：当前客户端订阅的频道或者模式数
        3. multi：当前事务中已执行命令个数
        4. events：文件描述符事作件(r/w)：r和w分别代表客户端套接字可读和可写
- 输入缓冲使用不当会产生两个问题：
    1. 一旦某个客户端的输入缓冲区超过1G，客户端将会被关闭。
    2. 输入缓冲区不受maxmemory控制，假设一个Redis实例设置了maxmemory为4G，已经存储了2G数据，但是如果此时输入缓冲区使用了3G，已经超过maxmemory限制，可能会产生数据丢失、键值淘汰、OOM等情况。
- 输入缓冲区过大主要是因为Redis的处理速度跟不上输入缓冲区的输入速度，并且每次进入输入缓冲区的命令包含了大量bigkey，从而造成了输入缓冲区过大的情况。还有一种情况就是Redis发生了阻塞，短期内不能处理命令，造成客户端输入的命令积压在了输入缓冲区，造成了输入缓冲区过大。
- 监控输入缓冲区异常的方法有两种：
    1. 通过定期执行client list命令，收集qbuf和qbuf-free找到异常的连接记录并分析，最终找到可能出问题的客户端。
    2. 通过info命令的info clients模块，找到最大的输入缓冲区。例如可以设置超过10M就进行报警。
- 和输入缓冲区相同的是，输出缓冲区也不会受到maxmemory的限制，如果使用不当同样会造成maxmemory用满产生的数据丢失、键值淘汰、OOM等情况。
- 实际上输出缓冲区由两部分组成：固定缓冲区(16KB)和动态缓冲区，其中固定缓冲区返回比较小的执行结果，而动态缓冲区返回比较大的结果。
- 固定缓冲区使用的是字节数组，动态缓冲区使用的是列表。当固定缓冲区存满后会将Redis新的返回结果存放在动态缓冲区的队列中，队列中的每个对象就是每个返回结果。
- 监控输出缓冲区的方法依然有两种：
    1. 通过定期执行client list命令，收集obl、oll、omem找到异常的连接记录并分析，最终找到可能出问题的客户端。
    2.  通过info命令的info clients模块，找到输出缓冲区列表最大对象数。
- 输出缓冲区出现异常的方法：
    1. 进行上述监控，设置阀值，超过阀值及时处理。
    2. 限制普通客户端输出缓冲区的，把错误扼杀在摇篮中。例如设置 client-output-buffer-limit normal 20mb 10mb 120。
    3. 适当增大slave的输出缓冲区的，如果master节点写入较大，slave客户端的输出缓冲区可能会比较大，一旦slave客户端连接因为输出缓冲区溢出被kill ，会造成复制重连。
    4. 限制容易让输出缓冲区增大的命令，例如，高并发下的monitor命令就是一个危险的命令。
    5. 及时监控内存，一旦发现内存抖动频繁，可能就是输出缓冲区过大。

下表对比client list和info clients监控输入缓冲区的优劣势

命令 | 优点 | 缺点
---|---|---
client list | 能精准分析每个客户端来定位问题 | 执行速度较慢(尤其是连接数较多的情况下)，频繁执行存在阻塞Redis的可能
info clients | 执行速度比client list快，分析过程较为简单 |  不能精准定位到客户端；不能显示所有输入缓冲区的总量，只能显示最大量

客户端类型表
序号 | 客户端类 | 说明
---|---|---
1 | N | 普通客户端
2 | M | 当前客户端是master节点
3 | S | 当前客户端是slave节点
4 | O | 当前客户端正在执行monitor命令
5 | x | 当前客户端正在执行事务
6 | b | 当前客户端正在等待阻塞时间
7 | i | 当前客户端正在等待VM I/O，但是此状态目前已经废弃不用
8 | d | 一个受监视的键已被修改，EXEC命令将失败
9 | u | 客户端未被阻塞
10 | c | 回复完成输出后，关闭连接
11 | A | 尽可能快地关闭连接


2）client setName和client getName
```
client setName xx
client getName
```
- client setName用于给客户端设置名字，这样比较容易标识出客户端的来源.

3）client kill
```
client kill ip:port
```
- client kill命令用于杀掉指定IP地址和端口的客户端。

4）client pause
```
client pause timeout (毫秒)
```
- client pause命令用于阻塞客户端timeout毫秒数，在此期间客户端连接将被阻塞。
- 该命令可以在如下场景起到作用：
    1. client pause只对普通和发布订阅客户端有效，对于主从复制(从节点内部伪装了一个客户端)是无效的，也就是此期间主从复制是正常进行的，所以此命令可以用来让主从复制保持一致。
    2. client pause可以用一种可控的方式将客户端连接从一个Redis节点切换到另一个Redis节点。

5）monitor
```
127.0.0.1:6379> monitor
OK
···
```
- monitor命令用于监控Redis正在执行的命令。
- monitor能监听到所有的命令，一旦Redis的并发量过大，monitor客户端的输出缓冲会暴涨，可能瞬间会占用大量内存。

5、客户端相关配置

除了上面介绍的部分配置外，还有下面这些配置：
- imeout：检测客户端空闲连接的超时时间，一旦idle时间达到了timeout，客户端将会被关闭，如果设置为0就不进行检测。
- maxclients：客户端最大连接数。但是这个参数会受到操作系统设置的限制。
- tcp-keepalive：检测TCP连接活性的周期，默认值为0，也就是不进行检测，如果需要设置，建议为60，那么Redis会每隔60秒对它创建的TCP连接进行活性检测，防止大量死连接占用系统资源。
- tcp-backlog：TCP三次握手后，会将接受的连接放入队列中，tcp-backlog就是队列的大小，它在Redis中的默认值是511。通常来讲这个参数不需要调整，但是这个参数会受到操作系统的影响。例如在Linux操作系统中，如果/proc/sys/net/core/somaxconn小于tcp-backlog，那么在Redis启动时会看到日志打印建议将/proc/sys/net/core/somaxconn设置更大。

6、客户端统计片段
```
127.0.0.1:6379> info clients
# Clients
connected_clients :1414
client_longest_output_list :0
client_biggest_input_buf :2097152
blocked clients :0
```
- connected_clients：代表当前Redis节点的客户端连接数，需要重点监控，一旦超过maxclients，新的客户端连接将被拒绝。
- client_longest_output_list：当前所有输出缓冲区中队列对象个数的最大值。
- client_biggest_input_buf：当前所有输入缓冲区中占用的最大容量。
- blocked_clients：正在执行阻塞命令（例如blpop、brpop、 brpoplpush）的客户端个数。

```
127.0.0.1:6379> info stats
# Stats
total_connections_received: 80
. . .
rejected_connections: 0
```
- total_connections_received：Redis自启动以来处理的客户端连接数总数。
- rejected_connections：Redis自启动以来拒绝的客户端连接数，需要重点监控。

7、Jedis客户端常见异常

1）无法从连接池获取到连接
2）客户端读写超时
3）客户端连接超时
4）客户端缓冲区异常
5）Lua脚本正在执行
6）Redis正在加载持久化文件
7）Redis使用的内存超过maxmemory配置
8）客户端连接数过大


7、客户端案例分析-Redis内存陡增

1）现象
- 服务端现象：Redis主节点内存陡增，几乎用满maxmemory，而从节点内存并没有变化.
- 客户端现象：客户端产生了OOM异常，也就是Redis主节点使用的内存 已经超过了maxmemory的设置，无法写入新的数据.

2）分析原因
```
①确实有大量写入，但是主从复制出现问题：查询了Redis复制的相关信息，复制是正常的，主从数据基本一致。
127.0.0.1:6379> dbsize      // 主节点的键个数
(integer) 2126870
127.0.0.1:6380> dbsize      // 从节点的键个数
(integer) 2126870

②其他原因造成主节点内存使用过大：排查是否由客户端缓冲区造成主节点内存陡增，使用info clients命令查询相关信息如下：
127.0.0.1:6379> info clients
# Clients
connected_clients :1891
client_longest_output_list :225698          // 输出缓冲区不太正常，最大的客户端输出缓冲区队列已经超过了20万个对象
client_biggest_input_buf :0
blocked clients :0

通过client list命令找到omem不正常的连接，一般来说大部分客户端的omem为0（因为处理速度会足够快）
redis-cli client list | grep -v "omem=0"
```

3）处理方法和后期处理

- 处理方法：只要使用client kill命令杀掉这个连接，让其他客户端恢复正常写数据即可。
- 后期处理：
    1. 从运维层面禁止monitor命令，例如使用rename-command命令重置monitor命令为一个随机字符串，除此之外，如果monitor没有做rename-command ，也可以对monitor命令进行相应的监控（例如client list）。
    2. 从开发层面进行培训，禁止在生产环境中使用monitor命令，因为有时候monitor命令在测试的时候还是比较有用的，完全禁止也不太现实。
    3. 限制输出缓冲区的大小。
    4. 使用专业的Redis运维工具(例如Cachecloud)，收到相应的报警从而快速发现和定位问题。

8、客户端案例分析-客户端周期性的超时

1）现象
```
①客户端现象：客户端出现大量超时，经过分析发现超时是周期性出现的。
②服务端现象：服务端并没有明显的异常，只是有一些慢查询操作。
```
2）分析
- 网络原因：服务端和客户端之间的网络出现周期性问题，经过观察网络是正常的。
- Redis本身：经过观察Redis日志统计，并没有发现异常。
- 客户端：由于是周期性出现问题，就和慢查询日志的历史记录对应了一下时间，发现只要慢查询出现，客户端就会产生大量连接超时，两个时间点基本一致。

3）处理方法和后期处理
- 处理方法：调整慢查询的原因。
- 后期处理： 
    1. 从运维层面，监控慢查询，一旦超过阀值，就发出报警。
    2. 从开发层面，加强对于Redis的理解，避免不正确的使用方式。
    3. 使用专业的Redis运维工具。
