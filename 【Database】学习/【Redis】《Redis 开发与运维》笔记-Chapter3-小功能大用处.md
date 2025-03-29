###### 三、小功能大用处

1、慢查询分析

所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值，就将这条命令的相关信息(例如：发生时间，耗时，命令的详细信息)记录下来，Redis也提供了类似的功能。

注意，慢查询只统计执行命令的时间，所以没有慢查询并不代表客户端没有超时问题(网络延时、服务端待处理命令较多等等)。

2、慢查询的两个配置参数

- Redis提供了slowlog-log-slower-than和slowlog-max-len配置来解决这两个问题。从字面意思就可以看出，slowlog-log-slower-than就是那个预设阀值，它的单位是微秒(1秒=1000毫秒= 1000000微秒)，默认值是10000 ，假如执行了一条“很慢”的命令(例如keys*)，如果它的执行时间超过了10000微秒，那么它将被记录在慢查询日志中。
- 如果slowlog-log-slower-than=0会记录所有的命令，slowlog-log-slower-than<0对于任何命令都不会进行记录。
- 实际上Redis使用了一个列表来存储慢查询日志，slowlog-max-len就是列表的最大长度。一个新的命令满足慢查询条件时被插入到这个列表中，当慢查询日志列表已处于其最大长度时，最早插入的一个命令将从列表中移出。(慢查询日志是存放在Redis内存列表中的)

1）在Redis 中有两种修改配置的方法，一种是修改配置文件，另一种是使用config set命令动态修改。

```
// 将slowlog-log-slower-than设置为20000微秒，slowlog-max-len设置为1000：
config set slowlog-log-slower-than 20000
config set slowlog-max-len 1000
config rewrite                  // // 将配置持久化到本地配置文件
```

2）实现对慢查询日志的访问和管理的命令
```
1、获取慢查询日志
slowlog get [n]

127.0.0.1:6379> slowlog get
1)  1) (integer) 666
    2) (integer) 1456786500
    3) (integer) 11615
    4)  1) "BGREWRITEAOF"
2)  1) (integer) 665
    2) (integer) 1456718400
    3) (integer) 12006
    4)  1) "SETEX"
        2) "video_info_200"
        3) "300"
        4) "2"
...

2、获取慢查询日志列表当前的长度
slowlog len

127.0.0.1:6379> slowlog len
(integer) 45


3、慢查询日志重置(实际是对列表做清理操作)
slowlog reset

127.0.0.1:6379> slowlog len
(integer) 45            
127.0.0.1:6379> slowlog reset
OK
127.0.0.1:6379> slowlog len
(integer) 0
```
- slowlog get的参数n可以指定条数。
- 每个慢查询日志有4个属性组成，分别是慢查询日志的标识id、发生时间戳、命令耗时、执行命令和参数。


3、慢查询功能可以有效地帮助我们找到Redis可能存在的瓶颈，但在实际使用过程中要注意以下几点：

- slowlog-max-len配置建议： 线上建议调大慢查询列表，记录慢查询时Redis会对长命令做截断操作，并不会占用大量内存。增大慢查询列表可以减缓慢查询被剔除的可能，例如线上可设置为1000以上。
- slowlog-log-slower-than配置建议： 默认值超过10毫秒判定为慢查询，需要根据Redis并发量调整该值。由于Redis采用单线程响应命令，对于高流量的场景，如果命令执行时间在1毫秒以上，那么Redis最多可支撑OPS不到 1000。因此对于高OPS场景的Redis建议设置为1毫秒。
- 慢查询只记录命令执行时间，并不包括命令排队和网络传输时间。因此客户端执行命令的时间会大于命令实际执行时间。因为命令执行排队机制，慢查询会导致其他命令级联阻塞，因此当客户端出现请求超时，需要检查该时间点是否有对应的慢查询，从而分析出是否为慢查询导致的命令级联阻塞。
- 由于慢查询日志是一个先进先出的队列，也就是说如果慢查询比较多的情况下，可能会丢失部分慢查询命令，为了防止这种情况发生，可以定期执行slow get命令将慢查询日志持久化到其他存储中(例如MySQL)，然后可以制作可视化界面进行查询。

4、Redis提供了redis-cli、redis-server、redis-benchmark等Shell工具。

5、redis-cli详解

要了解redis-cli的全部参数，可以执行redis-cli-help命令来进行查看。

```
1）-r
$ redis-cli -r 3 ping     // 执行三次ping命令
pong
pong
pong

2）-i
$ redis-cli -r 5 -i 1 ping      // 每隔1秒执行一次ping命令，一共执行5次
PONG
PONG
PONG
PONG
PONG

$ redis-cli -r 100 -i 1 info | grep used_memory_human   // 每隔1秒输出内存的使用量，一共输出 100次
used_memory_human:2.95G
used_memory_human:2.95G
. . . . . . . . . . . . . . . . . . . . . .
used_memory_human:2.94G

3）-x
$ echo "world" | redis-cli -x set hello
OK

4）-c
5）-a
6）--scan和--pattern
7）--slave
①下面开启第一个客户端，使用--slave选项，看到同步已完成：
$ redis-cli --slave
SYNC with master, discarding 72 bytes of bulk transfer . . .
SYNC done . Logging commands from master .

②再开启另一个客户端做一些更新操作：
redis-cli
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> set a b
OK
127.0.0.1:6379> incr count
1
127.0.0.1:6379> get hello
"world"

③第一个客户端会收到Redis节点的更新操作：(PING命令是由于主从复制产生的)
redis-cli --slave
SYNC with master, discarding 72 bytes of bulk transfer . . .
SYNC done . Logging commands from master .
"PING"
"PING"
"PING"
"PING"
"PING"
"SELECT","0"
"set","hello","world"
"set","a","b"
"PING"
"incr","count"


8）--rdb
9）--pipe
下面操作同时执行了set hello world和incr counter两条命令：
echo -en '*3\r\n$3\r\nSET\r\n$5\r\nhello\r\n$5\r\nworld\r\n*2\r\n$4\r\nincr\r\ n$7\r\ncounter\r\n ' | redis-cli --pipe

10）--bigkeys
11）--eval
12）--latency
redis-cli -h {machineB} --latency
min: 0, max: 1, avg: 0.07 (4211 samples)

// 延时信息每15秒输出一次，可以通过-i参数控制间隔时间
redis-cli -h 10.10.xx.xx --latency-history
min: 0, max: 1, avg: 0.28 (1330 samples) -- 15.01 seconds range…
min: 0, max: 1, avg: 0.05 (1364 samples)    15.01 seconds range

13）--stat

14）--raw和--no-raw
①在Redis中设置一个中文的value：
$redis-cli set hello "你好"
OK

②如果正常执行get或者使用--no-raw选项，那么返回的结果是二进制格式；如果使用了--raw选项，将会返回中文。
$redis-cli get hello
"\xe4\xbd\xa0\xe5\xa5\xbd"
$redis-cli --no-raw get hello
"\xe4\xbd\xa0\xe5\xa5\xbd"
$redis-cli --raw get hello
你好
```
- -r(repeat)选项代表将命令执行多次
- -i(interval)选项代表每隔几秒执行一次命令，但是-i选项必须和-r选项一起使用。
- 注意-i的单位是秒，不支持毫秒为单位，但是如果想以每隔10毫秒执行一次，可以用-i0.0。
- -x选项代表从标准输入(stdin)读取数据作为redis-cli的最后一个参数。
- -c(cluster)选项是连接Redis Cluster节点时需要使用的，-c选项可以防止moved和ask异常。
- 如果Redis配置了密码，可以用-a(auth)选项，有了这个选项就不需要手动输入auth命令。
- --scan选项和--pattern选项用于扫描指定模式的键，相当于使用scan命令。
- --slave选项是把当前客户端模拟成当前Redis节点的从节点，可以用来获取当前Redis节点的更新操作。合理的利用这个选项可以记录当前连接Redis节点的一些更新操作，这些更新操作很可能是实际开发业务时需要的数据。
- --rdb选项会请求Redis实例生成并发送RDB持久化文件，保存在本地。可使用它做持久化文件的定期备份。
- --pipe选项用于将命令封装成Redis通信协议定义的数据格式，批量发送给Redis执行。
- --bigkeys选项使用scan命令对Redis 的键进行采样，从中找到内存占用比较大的键值，这些键可能是系统的瓶颈。
- --eval选项用于执行指定Lua脚本。
- latency有三个选项，分别是--latency、--latency-history、--latency-dist。它们都可以检测网络延迟。
    1. --latency：该选项可以测试客户端到目标Redis的网络延迟。
    2. --latency-history：--latency的执行结果只有一条，而--latency-history可以以分时段的形式了解延迟信息。
    3. --latency-dist：该选项会使用统计图表的形式从控制台输出延迟统计信息。
- --stat选项可以实时获取Redis的重要统计信息，虽然info命令中的统计信息更全，但是能实时看到一些增量的数据(例如requests)对于Redis的运维还是有一定帮助的。
- --no-raw选项是要求命令的返回结果必须是原始的格式，--raw恰恰相反，返回格式化后的结果。


6、redis-server详解

redis-server除了启动Redis外，还有一个--test-memory选项。redis-server--test-memory可以用来检测当前操作系统能否稳定地分配指定容量的内存给Redis，通过这种检测可以有效避免因为内存问题造成Redis崩溃。
```
下面操作检测当前操作系统能否提供1G的内存给Redis：
redis-server --test-memory 1024
```
- 整个内存检测的时间比较长。当输出passed this test时说明内存检测完毕，最后会提示--test-memory只是简单检测，如果有质疑可以使用更加专业的内存检测工具。
- 通常无需每次开启Redis实例时都执行--test-memory选项，该功能更偏向于调试和测试。


7、redis-benchmark详解

redis-benchmark可以为Redis做基准性能测试。
```
1）-c

2）-n<requests>
redis-benchmark -c 100 -n 20000代表100个客户端同时请求Redis，一共执行20000次。
redis-benchmark会对各类数据结构的命令进行测试，并给出性能指标.
====== GET ======
20000 requests completed in 0.27 seconds
100 parallel clients
3 bytes payload keep alive: 1
99.11% <= 1 milliseconds
100.00% <= 1 milliseconds
73529.41 requests per second

3）-q
$redis-benchmark -c 100 -n 20000 -q
PING_INLINE: 74349.45 requests per second
PING_BULK: 68728.52 requests per second
SET : 71174.38 requests per second…
LRANGE_500 (first 450 elements) : 11299.44 requests per second
LRANGE_600 (first 600 elements) : 9319.67 requests per second
MSET (10 keys) : 70671.38 requests per second

4）-r
在一个空的Redis上执行了redis-benchmark会发现只有3个键：
127.0.0.1:6379> dbsize
(integer) 3
127.0.0.1:6379> keys *
1) "counter:__rand_int__"
2) "mylist"
3) "key:__rand_int__"

$redis-benchmark -c 100 -n 20000 -r 10000

5）-P

6）-k<boolean>

7）-t
redis-benchmark -t get,set -q
SET: 98619.32 requests per
GET: 97560.98 requests per

8）--csv
redis-benchmark -t get,set --csv
"SET","81300.81"
"GET","79051.38"
```
- -c(clients)选项代表客户端的并发数量(默认是50)。
- -n(num)选项代表客户端请求总量(默认是100000)。
- -q选项仅仅显示redis-benchmark的requests per second信息
- 如果想向Redis插入更多的键，可以执行使用-r(random)选项，可以向Redis插入更多随机的键。
- -r选项会在key、counter键上加一个12位的后缀，-r 10000代表只对后四位做随机处理(-r不是随机数的个数)。
- -P选项代表每个请求pipeline的数据量(默认为1)。
- -k选项代表客户端是否使用keepalive，1为使用，0为不使用，默认值为1。
- -t选项可以对指定命令进行基准测试。
- --csv选项会将结果按照csv格式输出，便于后续处理，如导出到Excel等。


8、Pipeline概念

Redis客户端执行一条命令分为如下四个过程：
①发送命令
②命令排队
③命令执行
④返回结果
其中①+④称为Round Trip Time (RTT,往返时间)。

- Redis提供了批量操作命令(例如mget、mset等)，有效地节约RTT。大部分命令是不支持批量操作的，例如要执行n次hgetall命令，并没有mhgetall命令存在，需要消耗n次RTT。
- Pipeline(流水线)机制能改善上面这类问题，它能将一组Redis命令进行组装，通过一次RTT传输给Redis，再将这组Redis命令的执行结果按顺序返回给客户端。
- 但大部分开发人员更倾向于使用高级语言客户端中的Pipeline，目前大部分Redis客户端都支持Pipeline。


9、Pipeline性能测试

- Pipeline执行速度一般比逐条执行要快。
- 客户端和服务端的网络延时越大，Pipeline的效果越明显。

10、原生批量命令与Pipeline对比

Pipeline与原生批量命令的区别，具体包含以下几点：
- 原生批量命令是原子的，Pipeline是非原子的。
- 原生批量命令是一个命令对应多个key，Pipeline支持多个命令。
- 原生批量命令是Redis服务端支持实现的，而Pipeline需要服务端和客户端的共同实现。

11、Pipeline最佳实践

- 每次Pipeline组装的命令个数不能没有节制，否则一次组装Pipeline数据量过大，一方面会增加客户端的等待时间，另一方面会造成一定的网络阻塞，可以将一次包含大量命令的Pipeline拆分成多次较小的Pipeline来完成。
- Pipeline只能操作一个Redis实例，但是即使在分布式Redis场景中，也可以作为批量操作的重要优化手段。

12、事务

为了保证多条命令组合的原子性，Redis提供了简单的事务功能以及集成Lua脚本来解决这个问题。

事务表示一组动作，要么全部执行，要么全部不执行。

Redis提供了简单的事务功能，将一组需要一起执行的命令放到multi和exec两个命令之间。multi命令代表事务开始，exec命令代表事务结束，它们之间的命令是原子顺序执行的。

```
// 用户关注的例子
127.0.0.1:6379> multi
OK
127.0.0.1:6379> sadd user:a:follow user:b
QUEUED
127.0.0.1:6379> sadd user:b:fans user:a
QUEUED
127.0.0.1:6379> sismember user:a:follow user:b
(integer) 0
127.0.0.1:6379> exec
1) (integer) 1
2) (integer) 1
127.0.0.1:6379> sismember user:a:follow user:b
(integer) 1

127.0.0.1:6379> discard
OK
127.0.0.1:6379> sismember user:a:follow user:b
(integer) 0

127.0.0.1:6379> multi
OK
127.0.0.1:6379> sadd user:a:follow user:b
QUEUED
127.0.0.1:6379> zadd user:b:fans 1 user:a
QUEUED
127.0.0.1:6379> exec
1) (integer) 1
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value 127.0.0.1:6379> sismember user:a:follow user:b
(integer) 1

#T1：客户端1
127.0.0.1:6379> set key "java"
OK
#T2：客户端1
127.0.0.1:6379> watch key
OK
#T3：客户端1
127.0.0.1:6379> multi
OK
#T4：客户端2
127.0.0.1:6379> append key python
(integer) 11
#T5：客户端1
127.0.0.1:6379> append key jedis
QUEUED
#T6：客户端1
127.0.0.1:6379> exec
(nil)
#T7：客户端1
127.0.0.1:6379> get key
"javapython"
```
- sadd命令此时的返回结果是QUEUED，代表命令并没有真正执行，而是暂时保存在Redis中。只有当exec执行后，用户A关注用户B的行为才算完成。
- 如果要停止事务的执行，可以使用discard命令代替exec命令即可。
- 如果事务中的命令出现错误，Redis的处理机制也不尽相同：
    1. 命令错误：
    2. 运行时错误：Redis并不支持回滚功能，sadd user:a:follow user:b命令已经执行成功，开发人员需要自己修复这类问题。
- 有些应用场景需要在事务之前，确保事务中的key没有被其他客户端修改过，才执行事务，否则不执行(类似乐观锁)。Redis提供了watch命令来解决这类问题。


13、Lua用法概述

Redis将Lua作为脚本语言可帮助开发者定制自己的Redis命令，在这之前，必须修改源码。

1）数据类型及其逻辑处理

Lua语言提供了如下几种数据类型： booleans（布尔）、numbers（数值）、strings（字符串）、tables（表格），和许多高级语言相比，相对简单。

```
1、字符串
-- local代表val是一个局部变量，如果没有local代表是全局变量。 print函数可以打印出变量的值
local strings val = "world";

-- 结果是"world"
print (hello)

2、数组
local tables myArray = {"redis", "jedis", true, 88.0}
-- true
print (myArray[3])

①for
local int sum = 0
for i = 1, 100
do
    sum = sum + i
end
-- 输出结果为5050
print (sum)

for i = 1, #myArray
do
    print (myArray [i])
end

for index,value in ipairs (myArray)
do
    print (index)
    print (value)
end


②while
local int sum = 0
local int i = 0
while i <= 100
do
    sum = sum +i
    i = i + 1
end
--输出结果为5050
print (sum)


③if else
local tables myArray = {"redis", "jedis", true, 88 .0}
for i = 1, #myArray
do
    if myArray [i] == "jedis"
    then
        print ("true")
        break
    else
        --do nothing
    end
end


3、哈希
local tables user_1 = {age = 28, name = "tome"}
--user_1 age is 28
print ("user_1 age is " . . user_1 ["age"])

for key,value in pairs (user_1)
do print (key . . value)
end
```
- "--"是Lua语言的注释。
- 在Lua中，如果要使用类似数组的功能，可以用tables类型。但和大多数编程语言不同的是，Lua的数组下标从1开始计算。
- 关键字for以end作为结束符。
- 要遍历myArray，首先需要知道tables的长度，只需要在变量前加一个#号即可。
- 除此之外，Lua还提供了内置函数ipairs，使用for index，value ipairs(tables)可以遍历出所有的索引下标和值。
- while循环同样以end作为结束符。
- if以end结尾，if后紧跟then。
- 如果要使用类似哈希的功能，同样可以使用tables类型。
- strings 1..string2是将两个字符串进行连接。

2）函数定义

在Lua中，函数以function开头，以end结尾，funcName是函数名，中间部分是函数体。

```
function funcName ()
    . . .
end
// contact函数将两个字符串拼接：
function contact (str1, str2)
    return str1 . . str2
end
--"hello world"
print (contact ("hello ", "world"))
```

14、在Redis中使用Lua

在Redis中执行Lua脚本有两种方法：eval和evalsha。
```
1）eval
eval 脚本内容 key个数 key列表 参数列表

// 此时KEYS[1]="redis"，ARGV[1]="world"，所以最终的返回结果是"hello redisworld"。
127.0.0.1:6379> eval 'return "hello " . . KEYS [1] . . ARGV [1] ' 1 redis world
"hello redisworld"

2）evalsha
①加载脚本：script load命令可以将脚本内容加载到Redis内存中：

将lua_get.lua加载到Redis中，得到SHA1
# redis-cli script load "$ (cat lua_get.lua)"
"7413dc2440db1fea7c0a0bde841fa68eefaf149c"

②执行脚本：evalsha 的使用方法如下，参数使用SHA1值，执行逻辑和eval一致。
evalsha 脚本SHA1值 key个数 key列表 参数列表

127.0.0.1 :6379> evalsha 7413dc2440db1fea7c0a0bde841fa68eefaf149c 1 redis world
"hello redisworld"
```
- 如果Lua脚本较长，还可以使用redis-cli--eval直接执行文件。
- eval命令和--eval参数本质是一样的，客户端如果想执行Lua脚本，首先在客户端编写好Lua脚本代码，然后把脚本作为字符串发送给服务端，服务端会将执行结果返回给客户端。
- evalsha命令来执行Lua脚本：首先要将Lua脚本加载到Redis服务端，得到该脚本的SHA1校验和，evalsha命令使用SHA1作为参数可以直接执行对应Lua脚本，避免每次发送Lua脚本的开销。这样客户端就不需要每次执行脚本内容，而脚本也会常驻在服务端，脚本功能得到了复用。


15、Lua的Redis API

Lua可以使用redis.call函数实现对Redis的访问。
```
redis.call ("set", "hello", "world")
redis.call ("get", "hello")

放在Redis 的执行效果如下：
127.0.0.1 :6379> eval 'return redis.call ("get", KEYS [1]) ' 1 hello
"world"
```
- Lua还可以使用redis.pcall函数实现对Redis的调用，redis.call和redis.pcall的不同在于，如果redis.call执行失败，那么脚本执行结束会直接返回错误，而redis.pcall会忽略错误继续执行脚本。
- Lua可以使用redis.log函数将Lua脚本的日志输出到Redis的日志文件中，但是一定要控制日志级别。


15、Lua脚本功能为Redis开发和运维人员带来如下三个好处：
- Lua脚本在Redis中是原子执行的，执行过程中间不会插入其他命令。
- Lua脚本可以帮助开发和运维人员创造出自己定制的命令，并可以将这些命令常驻在Redis内存中，实现复用的效果。
- Lua脚本可以将多条命令一次性打包，有效地减少网络开销。


16、Redis如何管理Lua脚本

Redis提供了4个命令实现对Lua脚本的管理：
```
1、script load
script load script

2、script exists
scripts exists sha1 [sha1 … ]

127.0.0.1:6379> script exists a5260dd66ce02462c5b5231c727b3f7772c0bcc5
1) (integer) 1

3、script flush

127.0.0.1:6379> script exists a5260dd66ce02462c5b5231c727b3f7772c0bcc5
1) (integer) 1         
127.0.0.1:6379> script flush
OK
127.0.0.1:6379> script exists a5260dd66ce02462c5b5231c727b3f7772c0bcc5
1) (integer) 0

4、script kill

127.0.0.1:6379> eval 'while 1==1 do end ' 0     // 死循环，当前客户端会阻塞
127.0.0.1:6379> get hello
(error) BUSY Redis is busy running a script . You can only call SCRIPT KILL or
SHUTDOWN NOSAVE 
127.0.0.1:6379> script kill
OK
127.0.0.1:6379> get hello
"world"

```
- script load命令用于将Lua脚本加载到Redis内存中。
- script exists命令用于判断sha 1是否已经加载到Redis内存中。
- script exists返回结果代表sha 1[sha 1 …]被加载到Redis内存的个数。
- script flush命令用于清除Redis内存已经加载的所有Lua脚本。
- script kill命令用于杀掉正在执行的Lua脚本。如果Lua脚本比较耗时，甚至Lua脚本存在问题，那么此时Lua脚本的执行会阻塞Redis，直到脚本执行完毕或者外部进行干预将其结束。
- Redis提供了一个lua-time-limit参数，默认是5秒，它是Lua脚本的“超时时间”，但这个超时时间仅仅是当Lua脚本时间超过lua-time-limit后，向其他命令调用发送BUSY的信号，但是并不会停止掉服务端和客户端的脚本执行，所以当达到lua-time-limit值之后，其他客户端在执行正常的命令时，将会收到“Busy Redis is busy running a script”错误，并且提示使用script kill或者shutdown nosave命令来杀掉这个busy的脚本。
- 当script kill执行之后，客户端调用会恢复。
- 注意：如果当前Lua脚本正在执行写操作，那么script kill将不会生效。

17、Bitmaps数据结构模型

Bitmaps本身不是一种数据结构，实际上它就是字符串，但是它可以对字符串的位进行操作。

Bitmaps单独提供了一套命令，所以在Redis中使用Bitmaps和使用字符串的方法不太相同。可以把Bitmaps想象成一个以位为单位的数组，数组的每个单元只能存储0和1，数组的下标在Bitmaps中叫做偏移量。

18、Bitmaps命令

假设将每个独立用户是否访问过网站存放在Bitmaps中，将访问的用户记做1，没有访问的用户记做0，用偏移量作为用户的id。

```
1、设置值
setbit key offset value

// 将第0、5、11位用户设置为1
127.0.0.1:6379> setbit unique:users:2016-04-05 0 1
(integer) 0
127.0.0.1:6379> setbit unique:users:2016-04-05 5 1
(integer) 0
127.0.0.1:6379> setbit unique:users:2016-04-05 11 1
(integer) 0
127.0.0.1:6379> setbit unique:users:2016-04-05 15 1
(integer) 0
127.0.0.1:6379> setbit unique:users:2016-04-05 19 1
(integer) 0

2、获取值
getbit key offset

127.0.0.1:6379> getbit unique:users:2016-04-05 8
(integer) 0
127.0.0.1:6379> getbit unique:users:2016-04-05 5
(integer) 1

3、获取Bitmaps指定范围值为1的个数
bitcount [start] [end]

127.0.0.1:6379> bitcount unique:users:2016-04-05
(integer) 5

// 计算用户id在第1个字节到第3个字节之间的独立访问用户数，对应的用户id是11，15，19
127.0.0.1:6379> bitcount unique:users:2016-04-05 1 3
(integer) 3

4、Bitmaps间的运算
bitop op destkey key [key . . . .]

127.0.0.1:6379> bitop and unique:users:and:2016-04-04_03 unique:users:2016-04- unique:users:2016-04-03
(integer) 2
127.0.0.1:6379> bitcount unique:users:and:2016-04-04_03
(integer) 2

5、计算Bitmaps中第一个值为targetBit的偏移量
bitpos key targetBit [start] [end]

// 计算2016-04-04当前访问网站的最小用户id：
127.0.0.1:6379> bitpos unique:users:2016-04-04 1
(integer) 1

// 计算第0个字节到第1个字节之间，第一个值为0的偏移量
127.0.0.1:6379> bitpos unique:users:2016-04-04 0 0 1
(integer) 0             // id=0的用户
```
- 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费，通常的做法是每次做setbit 操作时将用户id减去这个指定数字。在第一次初始化Bitmaps时，假如偏移量非常大，那么整个初始化过程执行会比较慢，可能会造成Redis的阻塞(由于要申请大量内存)。
- bitcount的[start]和[end]代表起始和结束字节数。
- bitop是一个复合操作，它可以做多个Bitmaps的and(交集)、or(并集)、not(非)、xor(异或)操作并将结果保存在destkey中。
- bitops有两个选项[start]和[end]，分别代表起始字节和结束字节。

19、Bitmaps分析

当用户量很少的时候，对比用set，用Bitmaps会占用更大的内存。因为set是随着用户量一个一个增长内存的，而Bitmaps是根据id的，因此一开始就是这么大的内存，此时使用Bitmaps也不太合适，因为大部分位都是0。


20、HyperLogLog

- HyperLogLog并不是一种新的数据结构(实际类型为字符串类型)，而是一种基数算法，通过HyperLogLog可以利用极小的内存空间完成独立总数的统计，数据集可以是IP、Email、ID等。
- HyperLogLog提供了3个命令：pfadd、pfcount、pfmerge。

```
1、添加
pfadd key element [element … ]

127.0.0.1:6379> pfadd 2016_03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
(integer) 1

2、计算独立用户个数
pfcount key [key … ]

127.0.0.1:6379> pfcount 2016_03_06:unique:ids
(integer) 4
127.0.0.1:6379> pfadd 2016_03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-90"
(integer) 1        
127.0.0.1:6379> pfcount 2016_03_06:unique:ids    
(integer) 5                         // 新增uuid-90

// 向HyperLogLog插入100万个id，插入前记录一下info memory：
127.0.0.1:6379> info memory
# Memory
used_memory:835144
used_memory_human:815.57K
. . .向2016_05_01:unique:ids插入100万个用户，每次插入1000条：
elements=""
key="2016_05_01:unique:ids"
for i in `seq 1 1000000`
do
    elements="${elements} uuid-"${i}
    if [ [ $ ( (i%1000))  == 0 ]];
    then
        redis-cli pfadd ${key} ${elements}
        elements=""
    fi
done
127.0.0.1:6379> info memory
# Memory
used_memory :850616
used_memory_human :830.68K          // 内存只增加了15K左右.
127.0.0.1:6379> pfcount 2016_05_01:unique:ids   // pfcount的执行结果并不是100万
(integer) 1009838
// 如果使用集合，内存使用约84MB，但独立用户数为100万。


3、合并
pfmerge destkey sourcekey [sourcekey . . .]

```
- pfadd用于向HyperLogLog添加元素，如果添加成功返回1。
- pfcount用于计算一个或多个HyperLogLog的独立总数。
- HyperLogLog 内存占用量小得惊人，但是用如此小空间来估算如此巨大的数据，必然不是100%的正确，其中一定存在误差率。Redis官方给出的数字是0.81%的失误率。
- pfmerge可以求出多个HyperLogLog的并集并赋值给destkey。
- HyperLogLog内存占用量非常小，但是存在错误率，开发者在进行数据结构选型时只需要确认如下两条即可：
    1. 只为了计算独立总数，不需要获取单条数据。
    2. 可以容忍一定误差率，毕竟HyperLogLog在内存的占用量上有很大的优势。

21、发布订阅概述

Redis提供了基于“发布/订阅”模式的消息机制，此种模式下，消息发布者和订阅者不进行直接通信，发布者客户端向指定的频道（channel）发布消息，订阅该频道的每个客户端都可以收到该消息。

22、发布订阅的命令

Redis主要提供了发布消息、订阅频道、取消订阅以及按照模式订阅和取消订阅等命令。

```
1、发布消息
publish channel message

// 向channel:sports频道发布一条消息“Tim won the championship” 
127.0.0.1:6379> publish channel:sports "Tim won the championship" 
(integer) 0

2、订阅消息
subscribe channel [channel . . .]

(一客户端A)
127.0.0.1:6379> subscribe channel:sports
Reading messages . . . (press Ctrl-C to quit)
1) "subscribe"
2) "channel:sports"
3) (integer) 1

(另一客户端B)
127.0.0.1:6379> publish channel:sports "James lost the championship"
(integer) 1

(一客户端A)
127.0.0.1:6379> subscribe channel:sports
Reading messages . . . (press Ctrl-C to quit)
. . .
1) "message"
2) "channel:sports"
3) "James lost the championship"

3、取消订阅
unsubscribe [channel [channel . . .]]

127.0.0.1:6379> unsubscribe channel:sports
1) "unsubscribe"
2) "channel:sports"
3) (integer) 0

4、按照模式订阅和取消订阅
psubscribe pattern [pattern . . .]
punsubscribe [pattern [pattern . . .]]

127.0.0.1:6379> psubscribe it*          // 订阅以it开头的所有频道
Reading messages . . . (press Ctrl-C to quit)
1) "psubscribe"
2) "it*"
3) (integer) 1

5、查询订阅
①查看活跃的频道
pubsub channels [pattern]

127.0.0.1:6379> pubsub channels
1) "channel:sports"
2) "channel:it"
3) "channel:travel"    
127.0.0.1 :6379> pubsub channels channel:*r*
1) "channel:sports"
2) "channel:travel"

②查看频道订阅数
pubsub numsub [channel . . .]

127.0.0.1:6379> pubsub numsub channel:sports
1) "channel:sports"
2) (integer) 2

③查看模式订阅数
pubsub numpat

127.0.0.1:6379> pubsub numpat
(integer) 1             // 当前只有一个客户端通过模式来订阅
```
- publish channel返回结果为订阅者个数.
- 订阅者可以订阅一个或多个频道.
- 有关订阅命令有两点需要注意：
    1. 客户端在执行订阅命令之后进入了订阅状态，只能接收subscribe、psubscribe、unsubscribe、punsubscribe的四个命令。
    2. 新开启的订阅客户端，无法收到该频道之前的消息，因为Redis不会对发布的消息进行持久化。
- 和很多专业的消息队列系统（例如Kafka、RocketMQ）相比，Redis的发布订阅略显粗糙，例如无法实现消息堆积和回溯。但胜在足够简单，如果当前场景可以容忍的这些缺点，也不失为一个不错的选择。
- 客户端可以通过unsubscribe命令取消对指定频道的订阅，取消成功后，不会再收到该频道的发布消息。
- 除了subcribe和unsubscribe命令，Redis命令还支持glob风格的订阅命令psubscribe和取消订阅命令punsubscribe。
- 所谓活跃的频道是指当前频道至少有一个订阅者，其中[pattern]是可以指定具体的模式。


23、发布订阅的使用场景

聊天室、公告牌、服务之间利用消息解耦都可以使用发布订阅模式。

24、GEO

Redis3.2版本提供了GEO（地理信息定位）功能，支持存储地理位置信息用来实现诸如附近位置、摇一摇这类依赖于地理位置信息的功能。

```
1、增加地理位置信息
geoadd key longitude latitude member [longitude latitude member . . .]

127.0.0.1:6379> geoadd cities:locations 116.28 39.55 beijing
(integer) 1
127.0.0.1:6379> geoadd cities:locations 116.28 39.55 beijing
(integer) 0

127.0.0.1:6379> geoadd cities:locations 117.12 39.08 tianjin 114.29 38.02 shijiazhuang 118.01 39.38 tangshan 115.29 38.51 baoding
(integer) 4


2、获取地理位置信息
geopos key member [member . . .]

127.0.0.1:6379> geopos cities:locations tianjin
1)  1) "117.12000042200088501"
    2) "39.0800000535766543"

3、获取两个地理位置的距离
geodist key member1 member2 [unit]

127.0.0.1:6379> geodist cities:locations tianjin beijing km
"89.2061"

4、获取指定位置范围内的地理信息位置集合
georadius key longitude latitude radiusm |km |ft |mi [withcoord] [withdist] [withhash] [COUNT count] [asc |desc] [store key] [storedist key]
georadiusbymember key member     radiusm |km |ft |mi [withcoord] [withdist]
[withhash] [COUNT count] [asc |desc] [store key] [storedist key]

// 计算五座城市中，距离北京150公里以内的城市
127.0.0.1:6379> georadiusbymember cities:locations beijing 150 km 
1) "beijing"
2) "tianjin"
3) "tangshan"
4) "baoding"


5、获取geohash
geohash key member [member . . .]

127.0.0.1:6379> geohash cities:locations beijing
1) "wx4ww02w070"
127.0.0.1:6379> type cities:locations
zset

6、删除地理位置信息
zrem key member
```

- longitude、latitude、member分别是该地理位置的经度、纬度、成员.
- geoadd返回结果代表添加成功的个数，如果cities:locations没有包含tianjin，那么返回结果为1，如果已经存在则返回0.
- 如果需要更新地理位置信息，仍然可以使用geoadd命令，虽然返回结果为0.
- geodist的参数unit代表返回结果的单位，包含以下四种：
    1. m（meters）代表米。
    2. km（kilometers）代表公里。
    3. mi（miles）代表英里。
    4. ft（feet）代表尺。
- georadius和georadiusbymember两个命令的作用是一样的，都是以一个地理位置为中心算出指定半径内的其他地理信息位置，不同的是georadius命令的中心位置给出了具体的经纬度，georadiusbymember只需给出成员即可。
- georadius的radiusm|km|ft|mi是必需参数，指定了半径（带单位），这两个命令有很多  可选参数，如下：
    1. withcoord：返回结果中包含经纬度。
    2. withdist：返回结果中包含离中心节点位置的距离。        
    3. withhash：返回结果中包含geohash。
    4. COUNT count：指定返回结果的数量。
    5. asc|desc：返回结果按照离中心节点的距离做升序或者降序。
    6. store key：将返回结果的地理位置信息保存到指定键。
    7. storedist key：将返回结果离中心节点的距离保存到指定键。
- Redis使用geohash将二维经纬度转换为一维字符串.
- geohash有如下特点：
    1. GEO的数据类型为zset，Redis将所有地理位置信息的geohash存放在zset中。
    2. 字符串越长，表示的位置更精确。下表给出了字符串长度对应的精度，例如geohash长度为9时，精度在2米左右。
    3. 两个字符串越相似，它们之间的距离越近，Redis利用字符串前缀匹配算法实现相关的命令。
    4. geohash编码和经纬度是可以相互转换的。、
- Redis正是使用有序集合并结合geohash的特性实现了GEO的若干命令。
- GEO没有提供删除成员的命令，但是因为GEO的底层实现是zset，所以可以借用zrem命令实现对地理位置信息的删除。


geohash长度与精度对应关系表

geohash长度 | 精确度(km)
---|---
1 | 2500
2 | 630
3 | 78
4 | 20
5 | 2.4
6 | 0.61
7 | 0.076
8 | 0.019
9 | 0.002
