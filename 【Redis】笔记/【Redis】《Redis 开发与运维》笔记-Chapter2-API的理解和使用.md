###### 二、API的理解和使用

1、全局命令

1）查看所有键
```
key *

127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> set java jredis
OK
127.0.0.1:6379> set python redis-py
OK

127.0.0.1:6379> keys *
1) "python"
2) "java"
3) "hello"
```

2）键总数
```
dbsize

127.0.0.1:6379> rpush mylist a b c d e f g      // 插入一个列表类型的键值对(值是多个元素组成)
(integer) 7

127.0.0.1:6379> dbsize
(integer) 4                 // hello、java、python、mylist
```
- dbsize命令会返回当前数据库中键的总数。
- dbsize命令在计算键总数时不会遍历所有键，而是直接获取Redis内置的键总数变量，所以dbsize命令的时间复杂度是O(1)。而keys命令会遍历所有键，所以它的时间复杂度是O(n)，当Redis保存了大量键时，线上环境禁止使用。

3）检查键是否存在
```
exists key

127.0.0.1:6379> exists java
(integer) 1

127.0.0.1:6379> exists not_exist_key
(integer) 0
```
- 如果键存在则返回1，不存在则返回0.

4）删除键
```
del key [key ...]

127.0.0.1:6379> del java
(integer) 1
127.0.0.1:6379> exists java
(integer) 0
127.0.0.1:6379> del mylist
(integer) 1
127.0.0.1:6379> exists mylist
(integer) 0

127.0.0.1:6379> set a 1
OK
127.0.0.1:6379> set b 2
OK
127.0.0.1:6379> set c 3
OK
127.0.0.1:6379> del a b c
(integer) 3
```
- del是一个通用命令，无论值是什么数据结果类型，del命令都可以将其删除。
- 返回结果为成功删除键的个数，假设删除一个不存在的键，就会返回0.
- 同时del命令可以支持删除多个键。

5）键过期
```
expire key seconds

127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> expire hello 10     // 为键hello设置了10秒过期时间
(integer) 1

127.0.0.1:6379> ttl hello
(integer) 7                         // 剩余7秒
127.0.0.1:6379> ttl hello
(integer) -2                        // 说明键hello已经被删除
127.0.0.1:6379> get hello
(nil)
```
- Redis支持对键添加过期时间，当超过过期时间后，会自动删除键。
- ttl命令会返回键的剩余过期时间，它有3种返回值：
    1. 大于等于0的整数：键剩余的过期时间。
    2. -1：键没设置过期时间。
    3. -2：键不存在。

2、键的数据结构类型

```
type key

127.0.0.1:6379> set a b
OK
127.0.0.1:6379> type a
string
127.0.0.1:6379> rpush mylist a b c d e f g 
(integer) 7  
127.0.0.1:6379> type mylist
list

127.0.0.1:6379> type not_exsit_key
none
```
- 如果键不存在，则返回none。

3、数据结构和内部编码

```
graph LR

A{key}-->B1{string} 
B1{string} -->C11{raw}
B1{string} -->C12{int}
B1{string} -->C13{embstr}

A{key}-->B2{hash}
B2{hash}-->C21{hashtable}
B2{hash}-->C22{ziplist}

A{key}-->B3{list}
B3{list}-->C31{linkedlist}
B3{list}-->C32{ziplist}

A{key}-->B4{set}
B4{set}-->C41{hashtable}
B4{set}-->C42{intlist}

A{key}-->B5{zset}
B5{zset}-->B51{skiplist}
B5{zset}-->B52{ziplist}
```

- type命令实际返回的就是当前键的数据结构类型，它们(5种)分别是string(字符串)、hash(哈希)、list(列表)、set(集合)、zset(有序集合)，但这些都是Redis对外的数据结构。
- 实际上每种数据结构都有自己底层的内部编码实现，而且是多种实现，这样Redis会在合适的场景选择合适的内部编码。
- 可以看到每种数据结构都有两种以上的内部编码实现，例如list数据结构包含了linkedlist和ziplist两种内部编码。
- 同时有些内部编码，例如ziplist，可以作为多种外部数据结构的内部实现，可以通过object encoding命令查询内部编码。

```
127.0.0.1:6379> object encoding hello
"embstr"
127.0.0.1:6379> object encoding mylist
"ziplist"
```

4、单线程架构

- Redis将所有数据放在内存中，内存的响应时长大约为100ns。
- Redis使用epoll作为I/O多路复用技术的实现，再加上Redis自身的事件处理模型将epoll中的连接、读写、关闭都转换为事件，不在网络I/O上浪费过多的时间。
- 单线程避免了线程切换和竞态产生的消耗。

5、字符串

- 键都是字符串类型。
- 字符串类型的值实际可以是字符串(简单的字符串、复杂的字符串(例如JSON、XML))、数字(整数、浮点数)，甚至是二进制(图片、音频、视频)，但是值最大不能超过512MB。

1）字符串常用命令
```
①设置值：
set key value [ex seconds] [px milliseconds] [nx|xx]
setex key seconds value
setnx key value

127.0.0.1:6379> exists hello
(integer) 0
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> setnx hello redis
(integer) 0                         // setnx失败，返回结果为0。
127.0.0.1:6379> set hello jredis xx
OK                                  // set xx成功，返回结果为OK。


②获取值：
get key

127.0.0.1:6379> get hello 
"world"
127.0.0.1:6379> get not_exist_key
(nil)                               // 如果要获取的键不存在，则返回nil(空)。


③批量设置值：
mset key value [key value ...]

127.0.0.1:6379> mset a 1 b 2 c 3 d 4    // 通过mset命令一次性设置4个键值对
OK


④批量获取值：
mget key [key ...]

127.0.0.1:6379> mget a b c d        // 批量获取键a、b、c、d的值
1) "1"
2) "2"
3) "3"
4) "4"
127.0.0.1:6379> mget a b c f
1) "1"
2) "2"
3) "3"
4) (nil)


⑤计数：
incr key
decr key
incrby key increment
decrby key decrement
incrbyfloat key increment

127.0.0.1:6379> exists key
(integer) 0
127.0.0.1:6379> incr key
(integer) 1
127.0.0.1:6379> incr key
(integer) 2
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> incr hello
(error) ERR value is not an integer or out of range
```
- set命令有几个选项：
    1. ex seconds：为键设置秒级过期时间
    2. px milliseconds：为键设置毫秒级过期时间。
    3. nx：键必须不存在，才可以设置成功，用于添加。
    4. xx：与nx相反，键必须存在，才可以设置成功，用于更新。
- setex和setnx命令的作用和ex和nx选项是一样的。
- setnx可以作为分布式锁的一种实现方案(多个客户端同时指向setnx key value时，只有一个客户端能设置成功)。
- 批量获取值时，如果有些键不存在，那么它的值为nil(空)，结果是按照传入键的顺序返回。
- 批量操作命令可以有效提高开发效率。但每次批量操作所发送的命令数不是无节制的，如果数量过多可能造成Redis阻塞或者网络拥塞。
- incr命令用于对值做自增操作，返回结果分为三种情况：
    1. 值不是整数，返回错误。
    2. 值是整数，返回自增后的结果。
    3. 键不存在，按照值为0自增，返回结果为1。
- 除了incr指令，Redis提供了decr(自减)、incrby(自增指定数字)、decrby(自减指定数字)、incrbyfloat(自增浮点数)。

2）字符串不常用命令
```
①追加值：
append key value

127.0.0.1:6379> get key
"redis"
127.0.0.1:6379> append key world
(integer) 10
127.0.0.1:6379> get key
"redisworld"


②字符串长度：
strlen key

127.0.0.1:6379> get key
"redisworld"
127.0.0.1:6379> strlen key
(integer) 10
127.0.0.1:6379> set hello "世界"
OK
127.0.0.1:6379> strlen hello
(integer) 6                     // 每个中文占用3个字节。


③设置并返回原值：
getset key value

127.0.0.1:6379> getset hello world
(nil)
127.0.0.1:6379> getset hello redis
"world"


④设置指定位置的字符
setrange key offeset value

127.0.0.1:6379> set redis pest
OK
127.0.0.1:6379> setrange redis 0 b
(integer) 4
127.0.0.1:6379> get redis
"best"


⑤获取部分字符串
getrange key start end      // start和end分别是开始和结束的偏移量，偏移量从0开始计算。

127.0.0.1:6379> getrange redis 0 1
"be"
```

6、字符串内部编码

1）字符串类型的内部编码有3种：
- int：8个字节的长整型。
- embstr：小于等于39个字节的字符串。
- raw：大于39个字节的字符串。

Redis会根据当前值的类型和长度决定使用哪种内部编码实现。

```
127.0.0.1:6379> set key 8653
OK
127.0.0.1:6379> object encoding key
"int"

127.0.0.1:6379> set key "hello,world"
OK
127.0.0.1:6379> object encoding key
"embstr"

127.0.0.1:6379> set key "one string greater than 39 byte........."
OK
127.0.0.1:6379> object encoding key
"raw"
127.0.0.1:6379> strlen key
(integer) 40
```

7、字符串典型使用场景

1）缓存功能
- Redis作为缓存层，MySQL作为存储层，绝大部分请求的数据都是从Redis中获取。由于Redis具有支撑高并发的特性，所以缓存通常能起到加速读写和降低后端压力的作用。
- 
```
UserInfo getUserInfo(long id)
{
    userRedisKey = "user:info" + id
    value = redis.get(userRedisKey);
    UserInfo userInfo;
    if(value != null)
    {
        userInfo = deserialize(value);
    }
    else
    {
        userInfo = mysql.get(id);   // 从MySQL获取用户信息
        if(userInfo != null)
            redis.setex(userRedisKey, 3600, serialize(userinfo));   // 将userInfo序列化，并存入Redis
    }
    return userInfo;
}
```
**与MySQL等关系型数据库不同的是，Redis没有命令空间，而且也没有对键名有强制要求(除了不能使用一些特殊字符)。但设计合理的键名，有利于防止键冲突和项目的可维护性，比较推荐的方式是使用“业务名:对象名:id:[属性]”作为键名(也可以不是分号)。**

2）计数
- 许多应用都会使用Redis作为计数的基础工具，它可以实现快速计数、查询缓存的功能，同时数据可以异步落地到其他数据源。
```
long incrVideoCounter(long id)
{
    key = "video:playCount:" + id;
    return redis.incr(key);
}
```

3）共享Session
- 例如，一个分布式Web服务使用Redis将用户的Session进行集中管理。

4）限速
- 例如，限制用户获取短信验证码频率、限制一个IP地址不能在一秒钟之内访问超过n次。
```
phoneNum = "138xxxxxxx";
key = "shortMsg:limit:" + phoneNum;
// SET key value EX 60 NX
isExists = redis.set(key, 1, "EX 60", "NX");
if(isExists != null || redis.incr(key) <= 5)
    // 通过
else
    // 限速
```

8、哈希命令

在Redis中，哈希类型是指键值本身又是一个键值对结构，形如value={{field1, value1},...{fieldN, valueN}}。

```
①设置值：
hset key field value

127.0.0.1:6379> hset user:1 name tom        // 为user:1添加一对field-value
(integer) 1


②获取值：
hget key field

127.0.0.1:6379> hget user:1 name    // 获取user:1的name域(属性)对应的值
"tom"

127.0.0.1:6379> hget user:2 name     // 如果键或filed不存在，会返回nil。
(nil)
127.0.0.1:6379> hget user:1 age
(nil)


③删除field：
hdel key field [field ...]

127.0.0.1:6379> hdel user:1 name
(integer) 1
127.0.0.1:6379> hdel user:1 age
(integer) 0


④计算field个数：
hlen key

127.0.0.1:6379> hset user:1 name tom
(integer) 1
127.0.0.1:6379> hset user:1 age 23
(integer) 1
127.0.0.1:6379> hset user:1 city tianjin
(integer) 1
127.0.0.1:6379> hlen user:1
(integer) 3


⑤批量设置或获取field-value：
hmget key field [field ...]
hmset key field value [field value ...]

127.0.0.1:6379> hmset user:1 name mike age 12 city tianjin
OK
127.0.0.1:6379> hmget user:1 name city
1) "mike"
2) "tianjin"


⑥判断field是否存在：
hexists key field

127.0.0.1:6379> hexists user:1 name
(integer) 1


⑦获取所有field：
hkeys key

127.0.0.1:6379> hkeys user:1
1) "name"
2) "age"
3) "city"


⑧获取所有value：
hvals key

127.0.0.1:6379> hvals user:1
1) "mike"
2) "12"
3) "tianjin"


⑨获取所有的field-value：
hgetall key

127.0.0.1:6379> hgetall user:1
1) "name"
2) "mike"
3) "age"
4) "12"
5) "city"
6) "tianjin"


⑩hincrby hincrbyfloat
hincrby key field
hincrbyfloat key field


⑪计算value的字符串长度(需要Redis3.2以上)
hstrlen key field

127.0.0.1:6379> hstrlen user:1 name
(integer) 3         // hget user:1 name的value是tom，所有hstrlen的返回结果是3.
```
- 设置值如果设置成功会返回1，反之会返回0。此外，Redis提供了hsetnx命令，它们的关系就像set和setnx命令一样，只不过作用域由键变为field。
- hdel会删除一个或多个field，返回结果为成功删除field的个数。
- hmset和hget分别是批量设置和获取field-value，hmset需要的参数是key和多对field-value，hmget需要的是参数key和多个field。
- hexists判断存在时返回结果为1，不包含时返回0。
- hkeys命令应该叫hfields更为恰当，它返回指定哈希键所有的field。
- 在使用hgetall时，如果哈希元素个数比较多，会存在阻塞Redis的可能。如果开发人员只需要获取部分field，可以使用hmget，如果一定要获取全部field-value，可以使用hscan命令，该命令会渐进式遍历哈希类型。
- hincrby和hincrbyfloat，就像incrby和incrbyfloat命令一样，但是它们的作用域是field。

9、哈希内部编码

哈希类型的内部编码有两种：
- ziplist(压缩列表)：当哈希类型元素个数小于hash-max-ziplist-entries配置(默认512个)、同时所有值都小于hash-max-ziplist-value配置(默认64字节)时，Redis会使用ziplist作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀。
- hashtable(哈希表)：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)。

```
1、当field个数比较少且没有大的value时，内部编码为ziplist：
127.0.0.1:6379> hmset hashkey f1 v1 f2 v2
OK
127.0.0.1:6379> object encoding hashkey
"ziplist"

2、当有value大于64字节，内部编码会由ziplist变为hashtable：
127.0.0.1:6379> hset hashkey f3 "one string is bigger than 64 byte ...忽略..."
OK
127.0.0.1:6379> object encoding hashkey
"hashtable"

3、当field个数超过512，内部编码也会由ziplist变为hashtable：
127.0.0.1:6379> hset hashkey f1 v1 f2 v2 f3 v3 ...忽略... f513 v513
OK
127.0.0.1:6379> object encoding hashkey
"hashtable"
```

10、哈希使用场景

1）例如用户信息表。相对于使用字符串序列化缓存用户信息，哈希类型变得更加直观，并且在更新操作上会更加便捷。可以将每个用户的id定义为键后缀，多对field-value对应每个用户的属性。
```
UserInfo getUserInfo(long id)
{
    // 用户id作为key后缀
    userRedisKey = "user:info:" + id;
    // 使用hgetall获取所有用户信息映射关系
    userInfoMap = redis.hgetAll(userRedisKey);
    UserInfo userInfo;
    if(userInfoMap != null)
    {
        // 将映射关系转换为UserInfo
        userInfo = transferMapToUserInfo(userInfoMap);
    }
    else
    {
        // 从MySQL中获取用户信息
        userInfo = mysql.get(id);
        // 将userInfo变为映射关系使用hmset保存到Redis中
        redis.hmset(userRedisKey, transferUserInfoToMap(userInfo));
        // 添加过期时间
        redis.expire(userRedisKey, 3600);
    }
    return userInfo;
}
```
但是需要注意的是哈希类型和关系型数据库有两点不同之处：
- 哈希类型是稀疏的，而关系型数据库是完全结构化的，例如哈希类型每个键可以有不同的field，而关系型数据库一旦添加新的列，所有行都要为其设置值(即使为NULL)。
- 关系型数据库可以用复杂的关系查询，而Redis去模拟关系型复杂查询开发困难，维护成本高。

11、到目前为止，我们已经能够使用三种方法缓存用户信息，下面给出三种方案的实现方法和优缺点分析：
1）原生字符串类型：每个属性一个键。
```
set user:1:name tom
set user:1:age 23
set user:1:city beijing
```
- 优点：简单直观，每个属性都支持更新操作。
- 缺点：占用过多的键，内存占用量较大，同时用户信息内聚性比较差，所以此种方案一般不会在生产环境使用。

2）序列化字符串类型：将用户信息序列化后用一个键保存。
```
set user:1 serialize(userInfo)
```
- 优点：简化编程，如果合理的使用序列化可以提高内存的使用效率。
- 缺点：序列化和反序列化有一定的开销，同时每次更新属性都需要把全部数据去除进行反序列化，更新后再序列化到Redis中。

3）哈希类型：每个用户属性使用一对field-value，但是只用一个键保存。
```
hmset user:1 name tom age 23 city beijing
```
- 优点：简单直观，如果使用合理可以减少内存空间的使用。
- 缺点：要控制哈希在ziplist和hashtable两种内部编码的转换，hashtable会消耗更多内存。

12、列表概述

- 列表类型是用来存储多个有序的字符串。列表中的每个字符串称为元素，一个列表最多可以存储2^32-1个元素。
- 在Redis中，可以对列表两端插入(push)和弹出(pop)，还可以获取指定范围的元素列表、获取指定索引下标的元素等。
- 列表可以充当栈和队列的角色。

列表类型有两个特点：第一、列表中的元素是有序的，这就意味着可以通过索引下标获取某个元素或者某个范围内的元素列表。第二、列表中的元素可以是重复的。

13、列表命令

操作类型 | 操作
---|---
添加 | rpush lpush linsert
查 | lrange lindex llen
删除 | lpop rpop lrem ltrim
修改 | lset
阻塞操作 | blpop brpop

1）添加操作
```
①从右边插入元素：
rpush key value [value . . .]

127.0.0.1:6379> rpush listkey c b a         // 从右向左插入元素c、b、a
(integer) 3
127.0.0.1:6379> lrange listkey 0 -1         // 从左到右获取列表的所有元素
1) "c"
2) "b"
3) "a"


②从左边插入元素：
rpush key value [value . . .]


③向某个元素前或者后插入元素：
linsert key before|after pivot value

127.0.0.1:6379> linsert listkey before b java       // 在列表的元素b前插入java。
(integer) 4         // 返回结果4，代表当前列表的长度。
127.0.0.1:6379> lrange listkey 0 -1
1) "c"
2) "java"
3) "b"
4) "a"
```
- linsert命令会从列表中找到等于pivot的元素，在其前(before)或者后(after)插入一个新的元素value。

2）查找
```
①获取指定范围内的元素列表：
lrange key start end

127.0.0.1:6379> lrange listkey 1 3      // 获取列表的第2到第4个元素。
1) "java"
2) "b"
3) "a"


②获取列表指定索引下标的元素：
lindex key index

127.0.0.1:6379> lindex listkey -1       // 当前列表的最后一个元素为a
"a"


③获取列表长度：
llen key

127.0.0.1:6379> llen listkey
(integer) 4
```
- lrange操作会获取列表指定索引范围所有的元素。索引下标有两个特点：第一，索引下标从左到右分别是0到N-1 ，但是从右到左分别是-1到-N；第二，lrange中的end选项包含了自身，这个和很多编程语言不包含end不太相同。

3）删除
```
①从列表左侧弹出元素：
lpop key

127.0.0.1:6379> lpop listkey
"c"
127.0.0.1:6379> lrange listkey 0 -1
1) "java"
2) "b"
3) "a"


②从列表右侧弹出：
rpop key


③删除指定元素：
lrem key count value


④按照索引范围修剪列表：
ltrim key start end

127.0.0.1 :6379> lrange listkey 0 -1
1) "a"
2) "java"
3) "b"
4) "a"
127.0.0.1 :6379> ltrim listkey 1 3      // 只保留列表listkey的第2个到第4个元素。
OK
127.0.0.1 :6379> lrange listkey 0 -1
1) "java"
2) "b"
3) "a"
```
- lrem命令会从列表中找到等于value的元素进行删除，根据count的不同 分为三种情况：
    1. count>0 ，从左到右，删除最多count个元素。
    2. count<0 ，从右到左，删除最多count绝对值个元素。
    3. count=0 ，删除所有。

4）修改
```
①修改指定索引下标的元素：
lset key index newValue

127.0.0.1:6379> lset listkey 2 python   // 将列表listkey中的第3个元素设置为python
OK
127.0.0.1:6379> lrange listkey 0 -1
1) "java"
2) "b"
3) "python"
```

5）阻塞操作
```
blpop key [key . . .] timeout
brpop key [key . . .] timeout
```
- blpop和brpop是lpop和rpop的阻塞版本，它们除了弹出方向不同，使用方法基本相同。
- blpop/brpop命令包含两个参数：
    1. key[key...]：多个列表的键。
    2. timeout：阻塞时间（单位：秒）。
- 列表为空：如果timeout=3，那么客户端要等到3秒后返回；如果timeout=0，那么客户端一直阻塞等下去，如果此期间添加了数据element1 ，客户端立即返回。
- 列表不为空：客户端会立即返回。
- 在使用blpop/brpop时，有两点需要注意:
    1. 第一点，如果是多个键，那么brpop会从左至右遍历键，一旦有一个键能弹出元素，客户端立即返回。
    2. 第二点，如果多个客户端对同一个键执行brpop，那么最先执行brpop命令的客户端可以获取到弹出的值。

14、列表的内部编码

列表类型的内部编码有两种。
- ziplist(压缩列表)：当列表的元素个数小于list-max-ziplist-entries配置(默认512个)，同时列表中每个元素的值都小于list-max-ziplist-value配置时(默认64字节)，Redis会选用ziplist来作为列表的内部实现来减少内存的使用。
- linkedlist(链表) ：当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现。

```
1、当元素个数较少且没有大元素时，内部编码为ziplist：
127.0.0.1:6379> rpush listkey e1 e2 e3
(integer) 3
127.0.0.1:6379> object encoding listkey
"ziplist"

2、当元素个数超过512个，内部编码变为linkedlist：
127.0.0.1:6379> rpush listkey e4 e5 . . .忽略 . . . e512 e513
(integer) 513
127.0.0.1:6379> object encoding listkey
"linkedlist"

3、当某个元素超过64字节，内部编码也会变为linkedlist：
127.0.0.1:6379> rpush listkey "one string is bigger than 64 byte . . . . . . . . . . . . . . .. . . . . . . . . . . . . . . . . "
(integer) 4
127.0.0.1:6379> object encoding listkey
"linkedlist"
```
- Redis3.2版本提供了quicklist内部编码，简单地说它是以一个ziplist为节点的linkedlist，它结合了ziplist和linkedlist两者的优势，为列表类型提供了一种更为优秀的内部编码实现。

15、使用场景

1）消息队列

Redis的lpush+brpop命令组合即可实现阻塞队列，生产者客户端使用lrpush从列表左侧插入元素，多个消费者客户端使用brpop命令阻塞式的“抢”列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性。

2）文章列表

每个用户有属于自己的文章列表，现需要分页展示文章列表。此时可以考虑使用列表，因为列表不但是有序的，同时支持按照索引范围获取元素。

使用列表类型保存和获取文章列表会存在两个问题。
- 第一，如果每次分页获取的文章个数较多，需要执行多次hgetall操作，此时可以考虑使用Pipeline批量获取，或者考虑将文章数据序列化为字符串类型，使用mget批量获取。
- 第二，分页获取文章列表时，lrange命令在列表两端性能较好，但是如果列表较大，获取列表中间范围的元素性能会变差，此时可以考虑将列表做二级拆分，或者使用Redis3.2的quicklist内部编码实现，它结合ziplist和linkedlist的特点，获取列表中间范围的元素时也可以高效完成。

3）实际上列表的使用场景很多，在选择时可以参考以下口诀：
- lpush+lpop=Stack（栈）
- lpush+rpop=Queue（队列）
- lpush+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）

16、集合概述

- 集合(set)类型也是用来保存多个的字符串元素，但和列表类型不一样的是，集合中不允许有重复元素，并且集合中的元素是无序的，不能通过索引下标获取元素。
- 一个集合最多可以存储2^32-1个元素。
- Redis除了支持集合内的增删改查，同时还支持多个集合取交集、并集、差集。

17、集合命令

1）集合内操作
```
①添加元素：
sadd key element [element . . .]

127.0.0.1:6379> exists myset
(integer) 0
127.0.0.1:6379> sadd myset a b c
(integer) 3
127.0.0.1:6379> sadd myset a b
(integer) 0


②删除元素：
srem key element [element . . .]

127.0.0.1:6379> srem myset a b
(integer) 2
127.0.0.1:6379> srem myset hello
(integer) 0


③计算元素个数：
scard key

127.0.0.1:6379> scard myset
(integer) 1


④判断元素是否在集合中：
sismember key element

127.0.0.1:6379> sismember myset c
(integer) 1


⑤随机从集合返回指定个数元素：
srandmember key [count]

127.0.0.1:6379> srandmember myset 2
1) "a"
2) "c"
127.0.0.1:6379> srandmember myset
"d"

⑥从集合随机弹出元素：
spop key

127.0.0.1:6379> spop myset
"c"
127.0.0.1:6379> smembers myset
1) "d"
2) "b"
3) "a"


⑦获取所有元素
smembers key

127.0.0.1:6379> smembers myset
1) "d"
2) "b"
3) "a"

```
- sadd返回结果为添加成功的元素个数。
- srem返回结果为成功删除元素个数。
- scard的时间复杂度为O(1)，它不会遍历集合所有元素，而是直接用Redis内部的变量。
- sismember如果给定元素element在集合内返回1，反之返回0。
- srandmember中的[count]是可选参数，如果不写默认为1。
- spop操作可以从集合中随机弹出一个元素。
- 注意Redis从3.2版本开始，spop也支持[count]参数。
- srandmember和spop都是随机从集合选出元素，两者不同的是spop命令执行后，元素会从集合中删除，而srandmember不会。
- smembers获取集合myset所有元素，并且返回结果是无序的。
- smembers和lrange、hgetall都属于比较重的命令，如果元素过多存在阻塞Redis的可能性，这时候可以使用sscan来完成。

2）集合间操作
```
①求多个集合的交集
sinter key [key . . .]

127.0.0.1:6379> sinter user:1:follow user:2:follow
1) "sports"
2) "it"


②求多个集合的并集
suinon key [key . . .]

127.0.0.1:6379> sunion user:1:follow user:2:follow
1) "sports"
2) "it"
3) "his"
4) "news"
5) "music"
6) "ent"


③求多个集合的差集
sdiff key [key . . .]

127.0.0.1:6379> sdiff user:1:follow user:2:follow
1) "music"
2) "his"


④将交集、并集、差集的结果保存
sinterstore destination key [key . . .]
suionstore destination key [key . . .]
sdiffstore destination key [key . . .]

127.0.0.1:6379> sinterstore user:1_2:inter user:1:follow user:2:follow
(integer) 2
127.0.0.1:6379> type user:1_2:inter
set
127.0.0.1:6379> smembers user:1_2:inter
1) "it"
2) "sports"
```
- 集合间的运算在元素较多的情况下会比较耗时，所以Redis提供了上面三个命令(原命令+store)将集合间交集、并集、差集的结果保存在destination key中。

18、集合的内部编码

集合类型的内部编码有两种：
- intset(整数集合)：当集合中的元素都是整数且元素个数小于set-max-intset-entries配置(默认512个) 时，Redis会选用intset来作为集合的内部实现，从而减少内存的使用。
- hashtable(哈希表)：当集合类型无法满足intset的条件时，Redis会使用hashtable作为集合的内部实现。
```
1、当元素个数较少且都为整数时，内部编码为intset：
127.0.0.1:6379> sadd setkey 1 2 3 4
(integer) 4
127.0.0.1:6379> object encoding setkey
"intset"

2、当元素个数超过512个，内部编码变为hashtable：
127.0.0.1:6379> sadd setkey 1 2 3 4 5 6 ... 512 513
(integer) 509
127.0.0.1:6379> scard setkey
(integer) 513
127.0.0.1:6379> object encoding
"hashtable"

3、当某个元素不为整数时，内部编码也会变为hashtable：
127.0.0.1 :6379> sadd setkey a
(integer) 1
127.0.0.1 :6379> object encoding setkey
"hashtable"
```

19、集合的使用场景

1）集合类型比较典型的使用场景是标签(tag)。例如给用户打上喜好的标签(音乐、体育等等)。


```
下面使用集合类型实现标签功能的若干功能。
1、给用户添加标签
sadd user:1:tags tag1 tag2 tag5
sadd user:2:tags tag2 tag3 tag5
. . .
sadd user:k:tags tag1 tag2 tag4
. . .

2、给标签添加用户
sadd tag1:users user:1 user:3
sadd tag2:users user:1 user:2 user:3
...
sadd tagk:users user:1 user:2

3、删除用户下的标签
srem user:1:tags tag1 tag5
. . .

4、删除标签下的用户
srem tag1:users user:1
srem tag5:users user:1
. . .

5、计算用户共同感兴趣的标签(可以使用sinter命令，来计算用户共同感兴趣的标签)
sinter user:1:tags user:2:tags
```
- 用户和标签的关系维护应该在一个事务内执行，防止部分命令失败造成的数据不一致。例如上面的1和2。

2）集合类型的应用场景通常为以下几种：
- sadd=Tagging（标签）
- spop/srandmember=Random item（生成随机数，比如抽奖） 
- sadd+sinter=Social Graph（社交需求）

20、有序集合概述

有序集合保留了集合不能有重复成员的特性，但不同的是，有序集合中的元素可以排序。但是它和列表使用索引下标作为排序依据不同的是，它给每个元素设置一个分数(score)作为排序的依据。

注意：有序集合中的元素不能重复，但是score可以重复，就和一个班里的同学学号不能重复，但是考试成绩可以相同。

下表为列表、集合、有序集合三者的异同点：
数据结构 | 是否允许重复元素 | 是否有序 | 有序实现方式 | 应用场景
---|---|---|---|---
列表 | 是 | 是 | 索引下标 | 时间轴、消息队列等
集合 | 否 | 否 | 无 | 标签、社交等
有序集合 | 否 | 是 | 分值 | 排行榜系统、社交等

21、有序结合命令

1）集合内
```
①添加成员
zadd key score member [score member . . .]

127.0.0.1:6379> zadd user:ranking 251 tom
(integer) 1
127.0.0.1:6379> zadd user:ranking 1 kris 91 mike 200 frank 220 tim 250 martin (integer) 5


②计算成员个数
zcard key

127.0.0.1:6379> zcard user:ranking
(integer) 5


③计算某个成员的分数
zscore key member
127.0.0.1:6379> zscore user:ranking tom
"251"
127.0.0.1:6379> zscore user:ranking test
(nil)


④计算成员的排名
zrank key member
zrevrank key member

127.0.0.1:6379> zrank user:ranking tom
(integer) 5
127.0.0.1:6379> zrevrank user:ranking tom
(integer) 0


⑤删除成员
zrem key member [member . . .]

127.0.0.1:6379> zrem user:ranking mike
(integer) 1


⑥增加成员的分数
zincrby key increment member

127.0.0.1:6379> zincrby user:ranking 9 tom
"260"


⑦返回指定排名范围的成员
zrange    key start end [withscores]
zrevrange key start end [withscores]

127.0.0.1:6379> zrange user:ranking 0 2 withscores
1) "kris"
2) "1"
3) "frank"
4) "200"
5) "tim"
6) "220"
127.0.0.1:6379> zrevrange user:ranking 0 2 withscores
1) "tom"
2) "260"
3) "martin"
4) "250"
5) "tim"
6) "220"

⑧返回指定分数范围的成员
zrangebyscore    key min max [withscores] [limit offset count]
zrevrangebyscore key max min [withscores] [limit offset count]

127.0.0.1:6379> zrangebyscore user:ranking 200 221 withscores
1) "frank"
2) "200"
3) "tim"
4) "220"
127.0.0.1:6379> zrevrangebyscore user:ranking 221 200 withscores
1) "tim"
2) "220"
3) "frank"
4) "200"
127.0.0.1:6379> zrangebyscore user:ranking (200 +inf withscores
1) "tim"
2) "220"
3) "martin"
4) "250"
5) "tom"
6) "260"

⑨返回指定分数范围成员个数
zcount key min max

127.0.0.1:6379> zcount user:ranking 200 221     // 返回200到221分的成员的个数
(integer) 2

⑩删除指定排名内的升序元素
zremrangebyrank key start end

127.0.0.1:6379> zremrangebyrank user:ranking 0 2
(integer) 3

⑪删除指定分数范围的成员
zremrangebyscore key min max

127.0.0.1:6379> zremrangebyscore user:ranking (250 +inf     // 将250分以上的成员全部删除
(integer) 2
```
- zadd返回结果代表成功添加成员的个数。
- 有关zadd命令有两点需要注意：
    1. Redis3.2为zadd命令添加了nx、xx、ch、incr 四个选项：
        1. nx：member必须不存在，才可以设置成功，用于添加。
        2. xx：member必须存在，才可以设置成功，用于更新。
        3. ch：返回此次操作后，有序集合元素和分数发生变化的个数。
        4. incr：对score做增加，相当于后面介绍的zincrby。
    2. 有序集合相比集合提供了排序字段，但是也产生了代价，zadd的时间复杂度为O(log (n))，sadd的时间复杂度为O(1)。
- 和集合类型的scard命令一样，zcard的时间复杂度为O(1)。
- zscore如果成员不存在则返回nil。
- zrank是从分数从低到高返回排名，zrevrank反之。(排名从0开始计算)
- zrem返回结果为成功删除的个数。
- zincrby返回增加后的分数。
- 有序集合是按照分值排名的，zrange是从低到高返回，zrevrange反之。
- zrangebyscore按照分数从低到高返回，zrevrangebyscore反之。
    1. [limit offset count]选项可以限制输出的起始位置和个数。
    2. min和max还支持开区间(小括号)和闭区间(中括号)，-inf和+inf分别代表无限小和无限大。
- withscores选项会同时返回每个成员的分数。
- zremrangebyrank删除第start到第end名的成员，返回结果为成功删除的个数。

2）集合外
```
①交集
zinterstore destination numkeys key [key . . .] [weights weight [weight . . .]] [aggregate sum|min|max]

127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1 user:ranking:2
(integer) 3
127.0.0.1 :6379> zrange user:ranking:1_inter_2 0 -1 withscores
1) "mike"
2) "168"
3) "martin"
4) "875"
5) "tom"
6) "1139"

127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1 user:ranking:2 weights 1 0.5 aggregate max          // user:ranking:2的权重变为0.5，并且聚合效果使用max
(integer) 3
127.0.0.1 :6379> zrange user:ranking:1_inter_2 0 -1 withscores
1) "mike"
2) "91"
3) "martin"
4) "312.5"
5) "tom"
6) "444"

②并集
zunionstore destination numkeys key [key ...] [weights weight [weight ...] [aggregate sum|min|max]

127.0.0.1:6379> zunionstore user:ranking:1_union_2 2 user:ranking:1 user:ranking :2
(integer) 7
127.0.0.1:6379> zrange user:ranking:1_union_2 0 -1 withscores
1) "kris"
2) "1"
3) "james"
4) "8"
5) "mike"
6) "168"
7) "frank"
8) "200"
9) "tim"
10) "220"
11) "martin"
12) "875"
13) "tom"
14) "1139"
```
- zinterstore的命令参数：
    1. destination：交集计算结果保存到这个键。
    2. numkeys ：需要做交集计算键的个数。
    3. key[key...] ：需要做交集计算的键。
    4. weights weight[weight...]：每个键的权重，在做交集计算时，每个键中的每个member会将自己分数乘以这个权重，每个键的权重默认是1。
    5. aggregate sum|min|max：计算成员交集后，分值可以按照sum(和) 、min(最小值) 、max(最大值)做汇总，默认值是sum。

22、有序集合内部编码

有序集合类型的内部编码有两种：
- ziplist(压缩列表) ：当有序集合的元素个数小于zset-max-ziplist-entries配置(默认128个)，同时每个元素的值都小于zset-max-ziplist-value配置(默认64字节)时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存的使用。
- skiplist(跳跃表)：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降。

```
1、当元素个数较少且每个元素较小时，内部编码为skiplist：
127.0.0.1:6379> zadd zsetkey 50 e1 60 e2 30 e3
(integer) 3
127.0.0.1:6379> object encoding zsetkey
"ziplist"

2、当元素个数超过128个，内部编码变为ziplist：
127.0.0.1:6379> zadd zsetkey 50 e1 60 e2 30 e3 12 e4 ...忽略... 84 e129
(integer) 129
127.0.0.1:6379> object encoding
"skiplist"

3、当某个元素大于64字节时，内部编码也会变为hashtable：
127.0.0.1:6379> zadd zsetkey 20 "one string is bigger than 64 byte . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . "
(integer) 1
127.0.0.1:6379> object encoding
"skiplist"
```

23、有序集合使用场景

有序集合比较典型的使用场景就是排行榜系统。
```
1、添加用户赞数
zadd user:ranking:2016_03_15 mike 3

2、取消用户赞数
zrem user:ranking:2016_03_15 mike

3、展示获取赞数最多的十个用户
zrevrangebyrank user:ranking:2016_03_15	0 9

4、展示用户信息以及用户分数
hgetall user:info:tom
zscore user:ranking:2016_03_15 mike     // 用户分数
zrank user:ranking:2016_03_15 mike      // 用户排名
```


24、单个键管理
```
1、键重命名
rename key newkey

127.0.0.1:6379> get python
"jedis"
127.0.0.1:6379> set python jedis
OK
127.0.0.1:6379> rename python java      // 如果在rename之前，键java 已经存在，那么它的值也将被覆盖
OK
127.0.0.1:6379> get python
(nil)
127.0.0.1:6379> get java
"jedis"

127.0.0.1:6379> set java jedis
OK
127.0.0.1:6379> set python redis-py
OK
127.0.0.1:6379> renamenx java python
(integer) 0                             // 返回结果是0代表没有完成重命名
127.0.0.1:6379> get java
"jedis"
127.0.0.1:6379> get python
"redis-py"

2、随机返回一个键
randomkey

127.0.0.1:6379> dbsize
1000
127.0.0.1:6379> randomkey
"hello"
127.0.0.1:6379> randomkey
"jedis"

3、键过期
127.0.0.1:6379> expireat hello 1469980800   // 将键hello在2016-08-01 00:00:00(秒级时间戳为1469980800)过期
(integer) 1

127.0.0.1:6379> expire not_exist_key 30
(integer) 0

127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> expire hello -2
(integer) 1
127.0.0.1:6379> get hello
(nil)

127.0.0.1:6379> hset key f1 v1
(integer) 1
127.0.0.1:6379> expire key 50
(integer) 1
127.0.0.1:6379> ttl key
(integer) 46
127.0.0.1:6379> persist key
(integer) 1
127.0.0.1:6379> ttl key
(integer) -1

127.0.0.1:6379> expire hello 50
(integer) 1
127.0.0.1:6379> ttl hello
(integer) 46
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> ttl hello
(integer) -1


4、迁移键
①move key db

②dump key
restore key ttl value

第一步：在源Redis上执行dump：
redis-source> set hello world
OK
redis-source> dump hello "\x00\x05world\x06\x00\x8f<T\x04%\xfcNQ"

第二步：在目标Redis上执行restore：
redis-target> get hello
(nil)
redis-target> restore hello 0 "\x00\x05world\x06\x00\x8f<T\x04%\xfcNQ"
OK
redis-target> get hello
"world"

上面2步对应的伪代码如下：
Redis sourceRedis = new Redis ("sourceMachine", 6379);
Redis targetRedis = new Redis ("targetMachine", 6379);
targetRedis.restore("hello", 0, sourceRedis.dump(key));

③migrate
migrate host port key|"" destination-db timeout [copy] [replace] [keys key [key ...]]

127.0.0.1:6379> migrate 127.0.0.1 6379 hello 0 1000 replace
OK

127.0.0.1:6379> migrate 127.0.0.1 6380 "" 0 5000 keys key1 key2 key3
OK
```
- 为了防止被强行rename，Redis提供了renamenx命令，确保只有newKey不存在时候才被覆盖。
- 在使用重命名命令时，有两点需要注意：
    1. 由于重命名键期间会执行del命令删除旧的键，如果键对应的值比较大，会存在阻塞Redis的可能性，这点不要忽视。
    2. 如果rename和renamenx中的key和newkey如果是相同的，在Redis3.2和之前版本返回结果略有不同。Redis3.2中会返回OK，Redis3.2之前的版本会提示错误。
- 除了expire、ttl命令以外，Redis还提供了expireat、pexpire、pexpireat、pttl、persist等一系列命令，下面分别进行说明：
    1. expire key seconds ：键在seconds秒后过期。
    2. expireat key timestamp ：键在秒级时间戳timestamp后过期。
- ttl命令和pttl都可以查询键的剩余过期时间，但是pttl精度更高可以达到毫秒级别，有3种返回值：
    1. 大于等于0的整数：键剩余的过期时间（ttl是秒，pttl是毫秒）。
    2. -1：键没有设置过期时间。
    3. -2：键不存在。
- expireat命令可以设置键的秒级过期时间戳。
- 除此之外，Redis2.6版本后提供了毫秒级的过期方案：
    1. pexpire key milliseconds：键在milliseconds毫秒后过期。
    2. pexpireat key milliseconds-timestamp键在毫秒级时间戳timestamp后过期。
- 注意：但无论是使用过期时间还是时间戳，秒级还是毫秒级，在Redis内部最终使用的都是pexpireat。
- 在使用Redis相关过期命令时，需要注意以下几点：
    1. 如果expire key的键不存在，返回结果为0。
    2. 如果过期时间为负值，键会立即被删除，犹如使用del命令一样。
    3. persist命令可以将键的过期时间清除。
    4. 对于字符串类型键，执行set命令会去掉过期时间，这个问题很容易在开发中被忽视。
    5. Redis不支持二级数据结构(例如哈希、列表)内部元素的过期功能，例如不能对列表类型的一个元素做过期时间设置。
    6. setex命令作为set+expire的组合，不但是原子执行，同时减少了一次网络通讯的时间。
- 迁移键功能非常重要，因为有时候我们只想把部分数据由一个Redis迁移到另一个Redis(例如从生产环境迁移到测试环境)，Redis发展历程中提供了move、dump+restore、migrate三组迁移键的方法。
- move命令用于在Redis内部进行数据迁移，Redis内部可以有多个数据库，彼此在数据上是相互隔离的，move key db就是把指定的键从源数据库移动到目标数据库中，但笔者认为多数据库功能不建议在生产环境使用。
- dump+restore可以实现在不同的Redis实例之间进行数据迁移的功能，整个迁移的过程分为两步：
    1. 在源Redis上，dump命令会将键值序列化，格式采用的是RDB格式。
    2. 在目标Redis上，restore命令将上面序列化的值进行复原，其中ttl参数代表过期时间，如果ttl=0代表没有过期时间。
- 有关dump+restore有两点需要注意：第一，整个迁移过程并非原子性的，而是通过客户端分步完成的。第二，迁移过程是开启了两个客户端连接，所以dump的结果不是在源Redis和目标Redis之间进行传输。
- migrate命令也是用于在Redis实例间进行数据迁移的，实际上migrate命令就是将dump、restore、del三个命令进行组合，从而简化了操作流程。migrate命令具有原子性，而且从Redis3.0.6版本以后已经支持迁移多个键的功能，有效地提高了迁移效率，migrate在水平扩容中起到重要作用。
- migrate实现过程和dump+restore基本类似，但是有3点不太相同：第一，整个过程是原子执行的，不需要在多个Redis实例上开启客户端的，只需要在源Redis上执行migrate命令即可。第二，migrate命令的数据传输直接在源Redis和目标Redis上完成的。第三，目标Redis完成restore后会发送OK给源Redis，源Redis接收后会根据migrate对应的选项来决定是否在源Redis上删除对应的键。
- migrate的参数进行的说明：
    1. host：目标Redis的IP地址。
    2. port：目标Redis的端口。
    3. key|""：在Redis3.0.6版本之前，migrate只支持迁移一个键，所以此处是要迁移的键，但Redis3.0.6版本之后支持迁移多个键，如果当前需要迁移多个键，此处为空字符串""。
    4. destination-db：目标Redis的数据库索引，例如要迁移到0号数据库，这里就写0。
    5. timeout：迁移的超时时间(单位为毫秒)。
    6. [copy]：如果添加此选项，迁移后并不删除源键。
    7. [replace]：如果添加此选项，migrate不管目标Redis是否存在该键都会正常迁移进行数据覆盖。
    8. [keys key[key...]]：迁移多个键，例如要迁移key1、key2、key3，此处填写“keys key1 key2 key3”。

```
// 如下是Redis源码中，set命令的函数setKey，可以看到最后执行了removeExpire（db ，key）函数去掉了过期时间
void setKey (redisDb *db, robj *key, robj *val) {
    if (lookupKeyWrite (db,key) == NULL) {
        dbAdd (db,key,val);
    } else {
        dbOverwrite (db,key,val);
    }
    incrRefCount (val);
    // 去掉过期时间
    removeExpire (db,key);
    signalModifiedKey (db,key);
}
```

**笔者建议使用migrate命令进行键值迁移。**

下表为move、dump+restore、migrate三种迁移方式的异同点：
命令 | 作用域 | 原子性 | 支持多个键
---|---|---|---
move | Redis实例内部 | 是 | 否
dump+restore | Redis实例之间 | 否 | 否
migrate | Redis实例之间 | 是 | 是


25、遍历键

Redis提供了两个命令遍历所有的键，分别是keys和scan。
```
1、全量遍历键
keys pattern

127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> mset hello world redis best jedis best hill high
OK
127.0.0.1:6379> keys *
1) "hill"
2) "jedis"
3) "redis"
4) "hello"
127.0.0.1:6379> keys [j,r]edis      // 匹配以j,r开头，紧跟edis字符串的所有键
1) "jedis"
2) "redis"
127.0.0.1:6379> keys hll*
1) "hill"
2) "hello"

redis-cli keys video* | xargs redis-cli del     // 删除所有以video字符串开头的键

2、渐进式遍历
scan cursor [match pattern] [count number]

// 第一次执行scan0，返回结果分为两个部分：第一个部分6就是下次scan需要的cursor，第二个部分是10个键：
127.0.0.1:6379> scan 0
1) "6"
2)  1) "w"
    2) "i"
    3) "e"
    4) "x"
    5) "j"
    6) "q"
    7) "y"
    8) "u"
    9) "b"
    10) "o"
127.0.0.1:6379> scan 6
1) "11"
2)  1) "h"
    2) "n"
    3) "m"
    4) "t"
    5) "c"
    6) "d"
    7) "g"
    8) "p"
    9) "z"
    10) "a"
127.0.0.1:6379> scan 11
1) "0"
2)  1) "s"
    2) "f"
    3) "r"
    4) "v"
    5) "k"
    6) "l"
```
- 实际上keys命令是支持pattern匹配的。
- 如果要获取所有的键，可以使用keys pattern命令。
- pattern使用的是glob风格的通配符：
    1. *代表匹配任意字符。
    2. ?代表匹配一个字符。
    3. []代表匹配部分字符，例如[1,3]代表匹配1,3 ，[1-10]代表匹配1到10的任意数字。
    4. \x用来做转义，例如要匹配星号、问号需要进行转义。
- 如果Redis包含了大量的键，执行keys命令很可能会造成Redis阻塞，所以一般建议不要生产环境下使用keys命令。但有时候确实有遍历键的需求该怎么办，可以在以下三种情况使用：
    1. 在一个不对外提供服务的Redis从节点上执行，这样不会阻塞到客户端的请求，但是会影响到主从复制。
    2. 如果确认键值总数确实比较少，可以执行该命令。
    3. 使用scan命令渐进式的遍历所有键，可以有效防止阻塞。
- Redis从2.8版本后，提供了一个新的命令scan，它能有效的解决keys命令存在的问题。和keys命令执行时会遍历所有键不同，scan采用渐进式遍历的方式来解决keys命令可能带来的阻塞问题，每次scan命令的时间复杂度是O(1)，但是要真正实现keys的功能，需要执行多次scan。Redis存储键值对实际使用的是hashtable的数据结构。
- scan的参数进行的说明：
    1. cursor是必需参数，实际上cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回当前游标的值，直到游标值为0，表示遍历结束。
    2. match pattern是可选参数，它的作用的是做模式的匹配，这点和keys的模式匹配很像。
    3. count number是可选参数，它的作用是表明每次要遍历的键个数，默认值是10，此参数可以适当增大。
- 除了scan以外，Redis提供了面向哈希类型、集合类型、有序集合的扫描遍历命令，解决诸如hgetall、smembers、zrange可能产生的阻塞问题，对应的命令分别是hscan、sscan、zscan，它们的用法和scan基本类似。
- 渐进式遍历可以有效的解决keys命令可能产生的阻塞问题，但是scan并非完美无瑕，如果在scan的过程中如果有键的变化(增加、删除、修改)，那么遍历效果可能会碰到如下问题： 新增的键可能没有遍历到，遍历出了重复的键等情况，也就是说scan并不能保证完整的遍历出来所有的键，这些是我们在开发时需要考虑的。

```
// 以sscan为例子进行说明，当前集合有两种类型的元素，例如分别以old:user和new:user开头，先需要将old:user开头的元素全部删除:
String key = "myset";
// 定义pattern
String pattern = "old:user*";
// 游标每次从0开始
String cursor = "0";
while (true) {
    // 获取扫描结果
    ScanResult scanResult = redis.sscan (key, cursor, pattern);
    List elements = scanResult.getResult ();
    if (elements != null && elements.size () > 0) {
        // 批量删除
        redis.srem (key, elements);
    }
    // 获取新的游标
    cursor = scanResult.getStringCursor ();
    // 如果游标为0表示遍历结束
    if ("0".equals (cursor)) {
        break;
    }
}
```

26、数据库管理

Redis提供了几个面向Redis数据库的操作，它们分别是dbsize、select、flushdb/flushall命令

```
1、切换数据库
select dbIndex

127.0.0.1:6379> set hello world     // 默认进到0号数据库
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> select 15           // 切换到15号数据库
OK
127.0.0.1:6379 [15]> get hello      // 因为15号数据库和0号数据库是隔离的，所以get hello为空
(nil)

2、flushdb/flushall

127.0.0.1:6379> dbsize
(integer) 4                 // 当前0号数据库有四个键值对
127.0.0.1:6379> select 1
OK
127.0.0.1:6379 [1]> dbsize
(integer) 3
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> select 1
OK
127.0.0.1:6379 [1]> dbsize
(integer) 3
127.0.0.1:6379> flushall        // 在任意数据库执行flushall会将所有数据库清除.
OK
127.0.0.1:6379> select 1
OK
127.0.0.1:6379 [1]> dbsize
(integer) 0
```
- 与关系型数据库用字符来区分不同数据库名不同，Redis只是用数字作为多个数据库的实现。Redis默认配置中是有16个数据库。
- 当使用redis-cli -h{ip} -p{port}连接Redis时，默认使用的就是0号数据库，当选择其他数据库时，会有[index]的前缀标识，其中index就是数据库的索引下标。
- Redis3.0中已经逐渐弱化这个功能，例如Redis的分布式实现Redis Cluster只允许使用0号数据库，只不过为了向下兼容老版本的数据库功能，该功能没有完全废弃掉。废弃的原因有三点：
    1. Redis是单线程的。如果使用多个数据库，那么这些数据库仍然是使用一个CPU，彼此之间还是会受到影响的。
    2. 多数据库的使用方式，会让调试和运维不同业务的数据库变的困难，假如有一个慢查询存在，依然会影响其他数据库，这样会使得别的业务方定位问题非常的困难。
    3. 部分Redis的客户端根本就不支持这种方式。即使支持，在开发的时候来回切换数字形式的数据库，很容易弄乱。
- 笔者建议如果要使用多个数据库功能，完全可以在一台机器上部署多个Redis实例，彼此用端口来做区分，因为现代计算机或者服务器通常是有多个CPU的。这样既保证了业务之间不会受到影响，又合理地使用了CPU资源。
- flushdb/flushall命令用于清除数据库，两者的区别的是flushdb只清除当前数据库，flushall会清除所有数据库。
- flushdb/flushall命令可以非常方便的清理数据。如果当前数据库键值数量比较多，flushdb/flushall存在阻塞Redis 的可能性。
