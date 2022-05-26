###### 八、理解内存

1、内存使用统计

可通过执行info memory命令获取内存相关指标。

下表为info memory详细解释

属性名| 属性说明
---|---
used_memory | Redis分配器分配的内存总量，也就是内部存储的所有数据内存占用量
used_memory_human | 以可读的格式返回used_memory
used_memory_rss | 从操作系统的角度显示Redis进程占用的物理内存总量
used_memory_peak | 内存使用是最大值，表示used_memory的峰值
used_memory_peak_human | 以可读的格式返回used_memory_peak
used_memory_lua | Lua引擎所消耗的内存大小
mem_fragmentation_ratio | used_memory_rss/used_memory比值，表示内存碎片率
mem_allocator | Redis所使用的内存分配器。默认为jemalloc

- 重点关注的指标有：used_memory_rss和used_memory 以及它们的比值mem_fragmentation_ratio：
    1. 当mem_fragmentation_ratio>1时，说明used_memory_rss-used_memory多出的部分内存并没有用于数据存储，而是被内存碎片所消耗，如果两者相差很大，说明碎片率严重。
    2. 当mem_fragmentation_ratio<1时，这种情况一般出现在操作系统把Redis内存交换（Swap）到硬盘导致，出现这种情况时要格外关注，由于硬盘速度远远慢于内存，Redis性能会变得很差，甚至僵死。

2、内存消耗划分

Redis进程内消耗主要包括：自身内存+对象内存+缓冲内存+内存碎片，其中Redis空进程自身内存消耗非常少，通常used_memory_rss在3MB左右，used_memory在800KB左右，一个空的Redis进程消耗内存可以忽略不计。

另外三种内存消耗：
1）对象内存
- 对象内存是Redis内存占用最大的一块，存储着用户所有的数据。
- Redis所有的数据都采用key-value数据类型，每次创建键值对时，至少创建两个类型对象：key对象和value对象。
- 对象内存消耗可以简单理解为sizeof（keys）+sizeof（values）。
- 键对象都是字符串，在使用Redis时很容易忽略键对内存消耗的影响，应当避免使用过长的键。
- 每种value对象类型根据使用规模不同，占用内存不同。在使用时一定要合理预估并监控value对象占用情况，避免内存溢出。

2）缓存内存
- 缓冲内存主要包括：客户端缓冲、复制积压缓冲区、AOF缓冲区。
    1. 客户端缓冲指的是所有接入到Redis服务器TCP连接的输入输出缓冲。输入缓冲无法控制，最大空间为1G，如果超过将断开连接。输出缓冲通过参数client-output-buffer-limit控制。
        1. 普通客户端：除了复制和订阅的客户端之外的所有连接，Redis的默认配置是：client-output-buffer-limit normal 0 0 0，Redis并没有对普通客户端的输出缓冲区做限制，一般普通客户端的内存消耗可以忽略不计，但是当有大量慢连接客户端接入时这部分内存消耗就不能忽略了，可以设置maxclients做限制。特别是当使用大量数据输出的命令且数据无法及时推送给客户端时， 如monitor命令，容易造成Redis服务器内存突然飙升。
        2. 从客户端：主节点会为每个从节点单独建立一条连接用于命令复制，默认配置是：client-output-buffer-limit slave 256mb 64mb 60 。当主从节点之间网络延迟较高或主节点挂载大量从节点时这部分内存消耗将占用很大一部分，建议主节点挂载的从节点不要多于2个，主从节点不要部署在较差的网络环境下，防止复制客户端连接缓慢造成溢出。
        3. 当使用发布订阅功能时，连接客户端使用单独的输出缓冲区，默认配置为：client-output-buffer-limit pubsub 32mb 8mb 60，当订阅服务的消息生产快于消费速度时，输出缓冲区会产生积压造成输出缓冲区空间溢出。
    2. 复制积压缓冲区： Redis在2.8版本之后提供了一个可重用的固定大小缓冲区用于实现部分复制功能，根据repl-backlog-size参数控制，默认1MB。对于复制积压缓冲区整个主节点只有一个，所有的从节点共享此缓冲区，因此可以设置较大的缓冲区空间，如100MB，这部分内存投入是有价值的，可以有效避免全量复制。
    3. AOF缓冲区：这部分空间用于在Redis重写期间保存最近的写入命令。AOF缓冲区空间消耗用户无法控制，消耗的内存取决于AOF重写时间和写入命令量，这部分空间占用通常很小。

3）内存碎片

- Redis默认的内存分配器采用jemalloc，可选的分配器还有：glibc(就是GNU C Library的malloc吧?)、tcmalloc。内存分配器为了更好地管理和重复利用内存，分配内存策略一般 采用固定范围的内存块进行分配。
- jemalloc针对碎片化问题专门做了优化，一般不会存在过度碎片化的问题，正常的碎片率（mem_fragmentation_ratio）在1.03左右。

出现高内存碎片问题时常见的解决方式如下：
- 数据对齐：在条件允许的情况下尽量做数据对齐。
- 安全重启：重启节点可以做到内存碎片重新整理，因此可以利用高可用架构，如Sentinel或Cluster，将碎片率过高的主节点转换为从节点，进行安全重启。

3、子进程内存消耗

- 子进程内存消耗主要指执行AOF/RDB重写时Redis创建的子进程内存消耗。
- Redis执行fork操作产生的子进程内存占用量对外表现为与父进程相同，理论上需要一倍的物理内存来完成重写操作。但Linux具有写时复制技术（copy-on-write），父子进程会共享相同的物理内存页，当父进程处理写请求时会对需要修改的页复制出一份副本完成写操作，而子进程依然读取fork时整个父进程的内存快照。
- 如果在高并发写的场景下开启THP ，子进程内存消耗可能是父进程的数倍，极易造成机器物理内存溢出，从而触发SWAP或OOM killer。

子进程内存消耗总结如下：
- Redis产生的子进程并不需要消耗1倍的父进程内存，实际消耗根据期间写入命令量决定，但是依然要预留出一些内存防止溢出。
- 需要设置sysctl vm.overcommit_memory=1允许内核可以分配所有的物理内存，防止Redis进程执行fork时因系统剩余内存不足而失败。
- 排查当前系统是否支持并开启THP，如果开启建议关闭，防止copy-on-write期间内存过度消耗。

4、设置内存上限

Redis主要通过控制内存上限和回收策略实现内存管理。

Redis使用maxmemory参数限制最大可用内存。限制内存的目的主要有：
- 用于缓存场景，当超出内存上限maxmemory时使用LRU等删除策略释放空间。
- 防止所用内存超过服务器物理内存。


需要注意，maxmemory限制的是Redis实际使用的内存量，也就是used_memory统计项对应的内存。由于内存碎片率的存在，实际消耗的内存可能会比maxmemory设置的更大，实际使用时要小心这部分内存溢出。
通过设置内存上限可以非常方便地实现一台服务器部署多个Redis进程的内存控制。
得益于Redis单线程架构和内存限制机制，即使没有采用虚拟化，不同的Redis进程之间也可以很好地实现CPU和内存的隔离性。

5、动态调整内存上限

Redis的内存上限可以通过config set maxmemory进行动态修改，即修改最大可用内存。
```
Redis-1> config set maxmemory 6GB
Redis-2> config set maxmemory 2GB
```
- 通过动态修改maxmemory，可以实现在当前服务器下动态伸缩Redis内存的目的。
- 当超出系统物理内存限制而不能简单的通过调整maxmemory来达到扩容的目的时，需要采用在线迁移数据或者通过复制切换服务器来达到扩容的目的。
- Redis默认无限使用服务器内存，为防止极端情况下导致系统内存耗尽，建议所有的Redis进程都要配置maxmemory。
- 在保证物理内存可用的情况下，系统中所有Redis实例可以调整maxmemory参数来达到自由伸缩内存的目的。

6、内存回收策略

Redis的内存回收机制主要体现在以下两个方面：
1）删除到达过期时间的键对象。
2）内存使用达到maxmemory上限时触发内存溢出控制策略。

1. 删除过期键对象
    1. 惰性删除：惰性删除用于当客户端读取带有超时属性的键时，如果已经超过键设置的过期时间，会执行删除操作并返回空，这种策略是出于节省CPU成本考虑，不需要单独维护TTL链表来处理过期键的删除。但是单独用这种方式存在内存泄露的问题，当过期键一直没有访问将无法得到及时删除，从而导致内存不能及时释放。
    2. 定时任务删除：Redis内部维护一个定时任务，默认每秒运行10次（通过配置hz控制）。定时任务中删除过期键逻辑采用了自适应算法，根据键的过期比例、使用快慢两种速率模式回收键。流程说明如下：
        1. 定时任务在每个数据库空间随机检查20个键，当发现过期时删除对应的键。
        2. 如果超过检查数25%的键过期，循环执行回收逻辑直到不足25%或运行超时为止，慢模式下超时时间为25毫秒。
        3. 如果之前回收键逻辑超时，则在Redis触发内部事件之前再次以快模式运行回收过期键任务，快模式下超时时间为1毫秒且2秒内只能运行1次。
        4. 快慢两种模式内部删除逻辑相同，只是执行的超时时间不同。
2. 内存溢出控制策略
    1. 当Redis所用内存达到maxmemory上限时会触发相应的溢出控制策略。具体策略受maxmemory-policy参数控制，Redis支持6种策略：
        1. noeviction：默认策略，不会删除任何数据，拒绝所有写入操作并返回客户端错误信息（error）OOM command not allowed when used memory，此时Redis只响应读操作。
        2. volatile-lru：根据LRU算法删除设置了超时属性（expire）的键，直到腾出足够空间为止。如果没有可删除的键对象，回退到noeviction策略。
        3. allkeys-lru：根据LRU算法删除键，不管数据有没有设置超时属性，直到腾出足够空间为止。
        4. allkeys-random：随机删除所有键，直到腾出足够空间为止。
        5. volatile-random：随机删除过期键，直到腾出足够空间为止。
        6. volatile-ttl：根据键值对象的ttl属性，删除最近将要过期数据。如果没有，回退到noeviction策略。

- Redis所有的键都可以设置过期属性，内部保存在过期字典中。由于进程内保存大量的键，维护每个键精准的过期删除机制会导致消耗大量的CPU，对于单线程的Redis来说成本过高，因此Redis采用惰性删除和定时任务删除机制实现过期键的内存回收。
- 内存溢出控制策略可以采用config set maxmemory-policy{policy}动态配置。当Redis因为内存溢出删除键时，可以通过执行info stats命令查看evicted_keys指标找出当前Redis服务器已剔除的键数量。
- 每次Redis执行命令时如果设置了maxmemory参数，都会尝试执行回收内存操作。当Redis一直工作在内存溢出（used_memory>maxmemory）的状态下且设置非noeviction策略时，会频繁地触发回收内存的操作，影响Redis服务器的性能。
- 如果当前Redis有从节点，回收内存操作对应的删除命  令会同步到从节点，导致写放大的问题。
- 建议线上Redis内存工作在maxmemory>used_memory状态下，避免频繁内存回收开销。
- 对于需要收缩Redis 内存的场景，可以通过调小maxmemory来实现快速回收。比如对一个实际占用6GB内存的进程设置maxmemory=4GB，之后第一次执行命令时，如果使用非noeviction策略，它会一次性回收到maxmemory指定的内存量，从而达到快速回收内存的目的。注意，此操作会导致数据丢失和短暂的阻塞问题，一般在缓存场景下使用。

7、内存优化-redisObject对象

- Redis存储的所有值对象在内部定义为redisObject结构体。
- Redis存储的数据都使用redisObject来封装，包括string、hash、list、set、zset在内的所有数据类型。
- redisObject结构体每个字段的详细说明如下：
    1. type字段：表示当前对象使用的数据类型，Redis主要支持5种数据类型：string、hash、list、set、zset。可以使用type {key}命令查看对象所属类型，type命令返回的是值对象类型，键都是string类型。
    2. encoding字段：表示Redis内部编码类型，encoding在Redis内部使用，代表当前对象内部采用哪种数据结构实现。
    3. lru字段：记录对象最后一次被访问的时间，当配置了maxmemory和maxmemory-policy=volatile-lru或者allkeys-lru时，用于辅助LRU算法删除键数据。可以使用object idletime{key}命令在不更新lru字段情况下查看当前键的空闲时间。
    4. refcount字段：记录当前对象被引用的次数，用于通过引用次数回收内存，当refcount=0时，可以安全回收当前对象空间。使用object refcount{key}获取当前对象引用。当对象为整数且范围在[0-9999]时，Redis可以使用共享对象的方式来节省内存。
    5. *ptr字段： 与对象的数据内容相关，如果是整数，直接存储数据；否则表示指向数据的指针。Redis在3.0之后对值对象是字符串且长度<=39字节的数据，内部编码为embstr类型，字符串sds(simple dynamic string，简单动态字符串)和redisObject一起分配，从而只要一次内存操作即可。
- 可以使用scan+object idletime命令批量查询哪些键长时间未被访问，找出长时间不访问的键进行清理，可降低内存占用。
- 高并发写入场景中，在条件允许的情况下，建议字符串长度控制在39字节以内，减少创建redisObject内存分配次数，从而提高性能。

8、内存优化-缩减键值对象

降低Redis内存使用最直接的方式就是缩减键（key）和值（value）的长度。
- key长度： 如在设计键时，在完整描述业务情况下，键值越短越好。如 user:{uid}:friends:notify:{fid}可以简化为u:{uid}:fs:nt:{fid}。
- value长度：值对象缩减比较复杂，常见需求是把业务对象序列化成二进制数组放入Redis。

- 值对象除了存储二进制数据之外，通常还会使用通用格式存储数据，比如：json、xml等作为字符串存储在Redis中。这种方式优点是方便调试和跨语言，但是同样的数据相比字节数组所需的空间更大，在内存紧张的情况下，可以使用通用压缩算法压缩json、xml后再存入Redis，从而降低内存占用，例如使用GZIP压缩后的json可降低约60%的空间。
- 当频繁压缩解压json等文本数据时，开发人员需要考虑压缩速度和计算开销成本，这里推荐使用Google的Snappy压缩工具，在特定的压缩率情况下效率远远高于GZIP等传统压缩工具，且支持所有主流语言环境。

9、内存优化-共享对象池

- 共享对象池是指Redis内部维护[0-9999]的整数对象池。
- 创建大量的整数类型redisObject存在内存开销，每个redisObject内部结构至少占16字节，甚至超过了整数自身空间消耗。- 所以Redis内存维护一个[0-9999]的整数对象池，用于节约内存。
- 除了整数值对象，其他类型如list、hash、set、zset内部元素也可以使用整数对象池。
- 因此开发中在满足需求的前提下，尽量使用整数对象以节省内存。


整数对象池在Redis 中通过变量REDIS_SHARED_INTEGERS定义，不能通过配置修改。可以通过object refcount命令查看对象引用数验证是否启用整数对象池技术。
```
// 设置键foo等于100时，直接使用共享池内整数对象，因此引用数是2，再设置键bar等于100时，引用数又变为3 
redis> set foo 100
OK
redis> object refcount foo
(integer) 2
redis> set bar 100
OK
redis> object refcount bar
(integer) 3
```

**需要注意的是对象池并不是只要存储[0-9999]的整数就可以工作。当设置maxmemory并启用LRU相关淘汰策略如：volatile-lru，allkeys-lru时，Redis禁止使用共享对象池。**

- 为什么开启maxmemory和LRU淘汰策略后对象池无效？
    - LRU算法需要获取对象最后被访问时间，以便淘汰最长未访问数据，每个对象最后访问时间存储在redisObject对象的lru字段。对象共享意味着多个引用共享同一个redisObject，这时lru字段也会被共享，导致无法获取每个对象的最后访问时间。如果没有设置maxmemory，直到内存被用尽Redis也不会触发内存回收，所以共享对象池可以正常工作。
    - 综上所述，共享对象池与maxmemory+LRU策略冲突，使用时需要注意。对于ziplist编码的值对象，即使内部数据为整数也无法使用共享对象池，因为ziplist使用压缩且内存连续的结构，对象共享判断成本过高，

- 为什么只有整数对象池？
    - 首先整数对象池复用的几率最大，其次对象共享的一个关键操作就是判断相等性，Redis之所以只有整数对象池，是因为整数比较算法时间复杂度为O（1），只保留一万个整数为了防止对象池浪费。如果是字符串判断相等性，时间复杂度变为O（n），特别是长字符串更消耗性能（浮点数在Redis内部使用字符串存储）。对于更复杂的数据结构如hash、list等，相等性判断需要O（n^2）。对于单线程的Redis来说，这样的开销显然不合理，因此Redis只保留整数共享对象池。

10、内存优化-字符串优化

所有的键都是字符串类型，值对象数据除了整数之外都使用字符串存储。

1）字符串结构
- Redis没有采用原生C语言的字符串类型而是自己实现了字符串结构，内部简单动态字符串（simple dynamic string，SDS）。
- Redis 自身实现的字符串结构有如下特点：
    1. O（1）时间复杂度获取：字符串长度、已用长度、未用长度。 
    2. 可用于保存字节数组，支持安全的二进制数据存储。
    3. 内部实现空间预分配机制，降低内存再分配次数。
    4. 惰性删除机制，字符串缩减后的空间不释放，作为预分配空间保留。、

2）预分配机制
- 因为字符串（SDS）存在预分配机制，日常开发中要小心预分配带来的内存浪费。
- 字符串之所以采用预分配的方式是防止修改操作需要不断重分配内存和字节数据拷贝。但同样也会造成内存的浪费。字符串预分配每次并不都是翻倍扩容，空间预分配规则如下：
    1. 第一次创建len属性等于数据实际大小，free等于0 ，不做预分配。
    2. 修改后如果已有free空间不够且数据小于1M，每次预分配一倍容量。如原有len=60byte，free=0，再追加60byte ，预分配120byte，总占用空间：60byte+60byte+120byte+1byte。
    3. 修改后如果已有free空间不够且数据大于1MB，每次预分配1MB数据。如原有len=30MB，free=0，当再追加100byte ，预分配1MB，总占用空间：1MB+100byte+1MB+1byte。
- 尽量减少字符串频繁修改操作如append、setrange，改为直接使用set修改字符串，降低预分配带来的内存浪费和内存碎片化。

3）字符串重构
- 字符串重构： 指不一定把每份数据作为字符串整体存储，像json这样的数据可以使用hash结构，使用二级结构存储也能帮我们节省内存。同时可以使用hmget、hmset命令支持字段的部分读取修改，而不用每次整体存取。


11、内存优化-编码优化

1）了解编码
- 编码不同将直接影响数据的内存占用和读写效率。使用object encoding{key}命令获取编码类型。

- type和encoding对应关系如下表：

类型 | 编码方式 | 数据结构
---|---|---
string | raw | 动态字符串编码
string | embstr | 优化内存分配的字符串编码
string | int | 整数编码
hash | hashtable | 散列表编码
hash | ziplist | 压缩列表编码
list | linkedlist | 双向链表编码
list | ziplist | 压缩列表编码
list | quicklist | 3.2版本新的列表编码
set | hashtable | 散列表编码
set | intset | 整数集合编码
zset | skiplist | 跳跃表编码
zset | ziplist | 压缩列表编码


2）控制编码类型

- 编码类型转换在Redis写入数据时自动完成，这个转换过程是不可逆的，转换规则只能从小内存编码向大内存编码转换。

- Redis之所以不支持编码回退，主要是数据增删频繁时，数据向压缩编码转换非常消耗CPU，得不偿失。

- hash、list、set、zset内部编码配置如下表：

类型 | 编码 | 决定条件
---|---|---
hash | ziplist | 满足所有条件：value最大空间(字节)<=hash-max-ziplist-value <br> field个数<=hash-max-ziplist-entries
hash | hashtable | 满足任意条件：value最大空间(字节)>hash-max-ziplist-value <br> field个数>hash-max-ziplist-entries
list | ziplist | 满足所有条件：value最大空间(字节)<=list-max-ziplist-value <br> 链表长度<=list-max-ziplist-entries
list | linkedlist | 满足任意条件：value最大空间(字节)>list-max-ziplist-value <br> 链表长度>list-max-ziplist-entries
list | quicklist | 3.2版本新编码：<br> 废弃 list-max-ziplist-entries和list-max-ziplist-entries 配置 <br> 使用新配置：<br> list-max-ziplist-size：表示最大压缩空间或长度 <br> 最大空间使用[-5-1]范围配置，默认-2表示8KB <br> 正整数表示最大压缩长度 <br> list-compress-depth：表示最大压缩深度，默认=0不压缩
set | intset | 满足所有条件：<br> 元素必须为整数 <br> 集合长度<=set-max-intlist-entries
set | hashtable | 满足任意条件：<br> 元素非整数类型 <br> 集合长度>hash-max-ziplist-entries
zset | ziplist | 满足所有条件：<br> value最大空间(字节)<=zset-max-ziplist-value <br> 有序集合长度<=zset-max-ziplist-entries
zset | skiplist | 满足任意条件：<br> value最大空间(字节)>zset-max-ziplist-value <br> 有序集合长度>zset-max-ziplist-entries

- 可以使用config set命令设置编码相关参数来满足使用压缩编码的条件。
- 对于已经采用非压缩编码类型的数据如hashtable、linkedlist等，设置参数后即使数据满足压缩编码条件，Redis也不会做转换，需要重启Redis重新加载数据才能完成转换。

3）ziplist编码

- ziplist编码主要目的是为了节约内存，因此所有数据都是采用线性连续的内存结构。
- ziplist编码是应用范围最广的一种，可以分别作为hash、list、zset类型的底层数据结构实现。
- 首先从ziplist编码结构开始分析，它的内部结构类似这样： <zlbytes><zltail><zllen><entry-1><entry-2><....><entry-n> <zlend> 。
- 一个ziplist可以包含多个entry（元素），每个entry保存具体的数据（整数或者字节数组）。

- ziplist结构字段含义：
    1. zlbytes：记录整个压缩列表所占字节长度，方便重新调整ziplist空间。类型是int-32，长度为4字节。
    2. zltail：记录距离尾节点的偏移量，方便尾节点弹出操作。类型是int-32，长度为4字节。
    3. zllen：记录压缩链表节点数量，当长度超过216-2时需要遍历整个列表获取长度，一般很少见。类型是int-16，长度为2字节。
    4. entry：记录具体的节点，长度根据实际存储的数据而定。
        1. prev_entry_bytes_length：记录前一个节点所占空间，用于快速定位上一个节点，可实现列表反向迭代。
        2. encoding：标示当前节点编码和长度，前两位表示编码类型：字符串/整数，其余位表示数据长度。
        3. contents：保存节点的值，针对实际数据长度做内存占用优化。 
    5. zlend：记录列表结尾，占用一个字节。

- ziplist数据结构的特点：
    1. 内部表现为数据紧凑排列的一块连续内存数组。
    2. 可以模拟双向链表结构，以O（1）时间复杂度入队和出队。
    3. 新增删除操作涉及内存重新分配或释放，加大了操作的复杂性。
    4. 读写操作涉及复杂的指针移动，最坏时间复杂度为O（n^2）。
    5. 适合存储小对象和长度有限的数据。

- 使用ziplist可以分别作为hash、list、zset数据类型实现。使用ziplist编码类型可以大幅降低内存占用。ziplist实现的数据类型相比原生结构，命令操作更加耗时，不同类型耗时排序：list<hash<zset。
- ziplist压缩编码的性能表现跟值长度和元素个数密切相关，正因为如此Redis提供了{type}-max-ziplist-value和{type}-max-ziplist-entries相关参数来做控制ziplist编码转换。最后再次强调使用ziplist压缩编码的原则：追求空间和时间的平衡。
- 针对性能要求较高的场景使用ziplist，建议长度不要超过1000 ，每个元素大小控制在512字节以内。
- 命令平均耗时使用info Commandstats命令获取，包含每个命令调用次数、总耗时、平均耗时，单位为微秒。

3）intset编码

- intset编码是集合（set）类型编码的一种，内部表现为存储有序、不重复的整数集。当集合只包含整数且长度不超过set-max-intset-entries配置时被启用。

- intset的字段结构含义：
    1. encoding：整数表示类型，根据集合内最长整数值确定类型，整数类型划分为三种： int-16 、int-32 、int-64。
    2. length：表示集合元素个数。
    3. contents：整数数组，按从小到大顺序保存。

- intset保存的整数类型根据长度划分，当保存的整数超出当前类型时，将会触发自动升级操作且升级后不再做回退。升级操作将会导致重新申请内存空间，把原有数据按转换类型后拷贝到新数组。
- 使用intset编码的集合时，尽量保持整数范围一致，如都在int-16范围内。防止个别大整数触发集合升级操作，产生内存浪费。
- 当使用整数集合时尽量使用intset编码。
- 使用ziplist编码的hash类型依然 比使用hashtable编码的集合节省大量内存。


12、内存优化-控制键的数量

当使用Redis存储大量数据时，通常会存在大量键，过多的键同样会消耗大量内存。Redis本质是一个数据结构服务器，它为我们提供多种数据结构，如hash、list、set、zset等。使用Redis时不要进入一个误区，大量使用get/set这样的API，把Redis当成Memcached使用。对于存储相同的数据内容利用Redis的数据结构降低外层键的数量，也可以节省大量内存。

hash结构降低键数量分析：
- 根据键规模在客户端通过分组映射到一组hash对象中，如存在100万个键，可以映射到1000个hash中，每个hash保存1000个元素。
- hash的field可用于记录原始key字符串，方便哈希查找。
- hash的value保存原始值对象，确保不要超过hash-max-ziplist-value限制。


同样的数据使用ziplist编码的hash类型存储比string类型节约内存。节省内存量随着value空间的减少越来越明显。hash-ziplist类型比string类型写入耗时，但随着value空间的减少，耗时逐渐降低。

使用hash重构后节省内存量这种内存优化技巧的关键点：
- hash类型节省内存的原理是使用ziplist编码，如果使用hashtable编码方式反而会增加内存消耗。
- ziplist长度需要控制在1000以内，否则由于存取操作时间复杂度在O（n）到O（n^2）之间，长列表会导致CPU消耗严重，得不偿失。
- ziplist适合存储小对象，对于大对象不但内存优化效果不明显还会增加命令操作耗时。
- 需要预估键的规模，从而确定每个hash结构需要存储的元素数量。
- 根据hash长度和元素大小，调整hash-max-ziplist-entries和hash-max-ziplist-value参数，确保hash类型使用ziplist编码。


关于hash键和field键的设计：
- 当键离散度较高时，可以按字符串位截取，把后三位作为哈希的field，之前部分作为哈希的键。如：key=1948480哈希key=group:hash:1948，哈希field=480。
- 当键离散度较低时，可以使用哈希算法打散键，如：使用crc32（key）&10000函数把所有的键映射到“0-9999”整数范围内，哈希field存储键的原始值。
- 尽量减少hash键和field的长度，如使用部分键内容。

- 客户端需要预估键的规模并设计hash分组规则，加重客户端开发成本。
- hash重构后所有的键无法再使用超时（expire）和LRU淘汰机制自动删除，需要手动维护删除。
- 对于大对象，如1KB以上的对象，使用hash-ziplist结构控制键数量反而得不偿失。
- 对于大量小对象的存储场景，非常适合使用ziplist编码的hash类型控制键的规模来降低内存。
- 使用ziplist+hash优化keys后，如果想使用超时删除功能，开发人员可以存储每个对象写入的时间，再通过定时任务使用hscan命令扫描数据，找出hash内超时的数据项删除即可。


13、【汇总】内存优化的思路包括：
- 精简键值对大小，键值字面量精简，使用高效二进制序列化工具。
- 使用对象共享池优化小整数对象。
- 数据优先使用整数，比字符串类型更节省空间。
- 优化字符串使用，避免预分配造成的内存浪费。
- 使用ziplist压缩编码优化hash、list等结构，注重效率和空间的平衡。 
- 使用intset编码优化整数集合。
- 使用ziplist编码的hash结构降低小对象链规模。
