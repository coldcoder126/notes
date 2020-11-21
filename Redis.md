# 一、概述

Redis全称REmote Dictionary Server ,是一种基于键值对的NoSQL数据库，Redis中的值可以是由string(字符串)、hash(哈希)、list(列表)、set(集合)、zset(有序集合)、Bitmaps(位图)、HyperLogLog、GEO(地理信息定位)等多种数据结构和算法组成。

## 特点

- 速度快
  - 单线程，避免了线程切换和竞态产生的消耗
  - 多路IO复用
  - C语言编写
  - 数据存放在内存中
- 基于键值对

## 功能

- 提供了键过期功能，用来是实现缓存
- 提供了发布订阅功能，用来实现消息系统。
- 支持Lua脚本功能，可以利用Lua创造出新的Redis命令
- 提供了简单的事务功能，能在一定程度上保证事务特性
- 提供了流水线(Pipeline)功能，客户端能将一批命令一次性传到Redis，减少了网络开销

# 二、基本操作

## Linux下的安装

>1. 获取资源及解压
>
>2. 安装
>
>3. 移动配置文件到安装目录
>
>   `mv redis.conf  /usr/local/redis/etc`
>
>4. 打开守护进程
>
>   `vi redis.conf  #将daemonize 置为yes`
>
>5. 将redis 加入到开机启动
>
>   ```shell
>   cd redis-x.x.x
>   make
>   cd src 
>   make install PREFIX=/usr/local/redis
>   ```

## 配置文件启动

`redis-server /usr/local/redis/etc/redis.conf`

## Redis 命令行客户端

`redis-cli -p {port}` 	默认本地连接

`redis-cli -h {host} -p {port}` 	远程连接

`redis-cli shutdown` 	停止redis服务

# 三、Redis命令

##数据库管理命令

`select dbIndex`  	切换数据库

`flushdb` 	清除当前数据库

`flushall` 	清除所有数据库

##全局键命令

### 键查询

`keys *` 	查看当前数据库所有键

`dbsize`	键总数

`exists key` 	检查键是否存在，存在返回1，不存在返回0

`type key` 	键的数据结构类型

`object encoding key` 	查询内部编码

`randomkey` 	随机返回一个键

`keys pattern` 	全量遍历键，pattern为通配符，其中：

- *代表任意字符
- . 代表一个字符
- []代表部分字符，[1,3]代表1或3，[1-3]代表1到3
- \x用来转义

`scan cursor [match pattern] [count number]`    渐进扫描以解决keys命令可能遇到的阻塞问题

- cursor 是一个游标，第一次遍历从0开始，每次scan遍历完都返回当前游标的值，知道游标为0表示遍历结束
- [match pattern] 做模式匹配
- [count number] 每次要遍历的键个数，默认为10

`hscan` `sscan` `zscan` 用法同上



### 键过期

`expire <key> <seconds>` 	对键添加过期时间，过期自动删除键，seconds为秒数

- expire key 键不存在返回0
- 过期时间为值，键直接删除
- persist命令可以将键的过期时间清除
- 对于字符串类型的键，执行set命令会去掉过期时间
- 不支持二级数据结构内部元素的过期功能，例如不能对列表类型的一个元素做过期时间设置
- setex命令作为set+expore命令的组合，不但是原子执行，而且同时减少了一次网络通讯的时间

`expireat key timestamp`	键在秒级时间戳timestamp后过期

`pexpire key milliseconds` 	键在milliseconds毫秒后过期

`pexpireat key milliseconds-timestamp` 	键在毫秒级时间戳后过期（内部实现始终使用这种）

`ttl|pttl key ` 	查看键过期剩余时间，ttl精度为秒，pttl精度为毫秒，其返回值

- 大于等于0的整数：键剩余的过期时间
- -1    键没有设置过期时间
- -2    键不存在

### 键修改/移动

`rename key newkey` 	键重命名，如果newkey 已存在，则其值会被覆盖

`renamenx key newkey` 	键重命名，只有newkey不存在时才会执行成功



`del key [key ...]` 	删除键，删除成功返回删除的个数，键不存在返回0



`move key db` 	把指定的键从源数据库移动到目标数据库中

`dump key` `restore key ttl value` 	dump+restore可以实现在不同的redis实例中间进行数据迁移的功能，整个迁移非原子性，而是分为两步：

- 在源redis上，dump命令会将键值序列化，格式采用的是RDB格式
- 在目标redis上，restore命令将上面序列化的值进行复原，其中ttl参数代表过期时间，如果ttl为0代表没有过期时间

`migrate host port key|"" destination-db timeout [copy] [replace] [keys key [key ...]]`

migrate命令是将dump、restore、del三个命令进行组合，从而简化了操作流程，该命令有原子性

- host：目标redis的IP地址
- port：目标redis的端口
- key|"" ：迁移一个键或多个键
- destination-db：目标redis的数据库索引
- timeout：迁移的超时时间
- [copy] ：如果添加此选项，迁移后并不删除源键
- [replace]：如果添加此选项，不管目标key是否存在都会覆盖
- [keys key [key ...]]：迁移多个键，例如迁移k1和k2，此处为keys k1 k2





## string 

**内部编码**

字符串类型的内部编码有3种，Redis会根据当前值的类型和长度决定使用哪种内部编码实现。

- int ：8个字节的长整型
- embstr：小于等于39个自检的字符串
- raw：大于等于39个字节的字符串 

**设置值**

`set key vlaue [ex seconds] [px milliseconds] [nx|xx]` 	设置值

​	`ex seconds` 	为键设置秒级过期时间

​	`px milliseconds` 为键设置毫秒级过期时间

​	`nx` 	键必须不存在才可以设置成功，用于添加

​	`xx` 	键必须存在才可以设置成功，用于更新

`setnx` 	相当于`set`加上`nx`参数，使用方法同set	

`setxx` 	相当于`set`加上`xx`参数，使用方法同上

`mset key value [key value ...]` 	批量设置值



`append key value` 	追加值

`getset key value` 	设置并返回原值

`setrange <key> <起始位置> <value>` 	从起始位置开始用value覆写key所存储的字段

**获取值**

`get key` 	获取值，不存在则返回nil

`mget key [key ...]` 	批量获取值

`getrange <key> <起始位置> <结束位置>` 	获得值的范围，从0开始，类似substring功能





---

**计数**

`incr key` 	对值做自增操作（原子性）

- 值不是整数，返回错误
- 值是整数，返回自增后的结果
- 键不存在，新创建value并设值为0，自增并返回结果1

`decr key` 	自减

`incrby key increment ` 	自增指定数字

`decrby key decrement ` 	自减指定数字

`incrbyfloat key increment ` 	自增指定浮点数

## 哈希

在Redis中哈希类型是指键值本身又是一个键值对的结构，形如value={{field1,value1},...,{fieldN,valueN}}

哈希类型的内部编码有两种：

- ziplist 	当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个）、同时所有值都小于hash-max-ziplist-value配置（默认64字节），redis会使用ziplist作为内部实现，更紧凑
- hashtable    当哈希类型不满足ziplist的条件时，redis会使用hashtable作为hash内部实现

**设置值**

`hset key field value` 	设置值，成功返回1，失败返回0

`hsetnx` 	类似setnx命令

`hmset key field value [field value ...]` 	批量设置field-value

`hmegt key field [field ...]` 	批量获取field-value

`hkeys key` 	获取所有field ··

`hvals key` 	获取所有value

`hgetall key` 	获取所有field-value

`hget key field` 	获取值，如果键或field不存在会返回nil

`hexists key field` 	判断field 是否存在

`hdel key field [field ...]` 删除一个或多个field，返回结果为成功删除field的个数

`hlen key` 	计算field个数

`hscan` 	

`hincrby/hincrbyfloat` 	使field自增

`hstrlen key field` 	计算value字符串长度  

## 列表

list类型是简单的字符串列表，按照插入顺序排序，一个列表可以存储2^32-1 个元素，List底层使用的是双向链表。在redis中可以对列表两端插入(push)和弹出(pop)，还可以获得指定范围的元素列表，指定索引下标的元素。

**添加**

`rpush key value [value ...]` 	从右边插入元素

`lpush key value [value ...]` 	从左边插入元素

 `linsert key before|after pivot value` 	向某个元素前或后插入元素

`rpoplpush <key1> <key2>`		从key1左边弹出一个值，插入到key2右侧 

**查找**

`lrange <key> start end` 	获取指定索引范围内的元素列表，索引下表从左到右是0~N-1，从右向左是-1~-N（`lrange key 0 -1`从左到右获取列表的所有元素） 

`lindex key index` 	获取列表指定索引下标的元素

`llen key` 	获取列表长度 

**删除**

`lpop key`	从列表左侧弹出元素 

`rpop key` 	从列表右侧弹出元素

`lrem key count value` 	删除指定元素，该命令会从列表中找到等于value的元素进行删除

- count>0从左到右，删除最多count个元素
- count<0从右向左，删除最多|count|个元素
- count=0删除所有

`ltrim key start end `      按照索引范围修剪列表，只保留start到end中间的元素

**修改**

`lset key index newValue` 	修改指定索引下标的元素

**阻塞操作**

`blpop key [key ...] timeout`  `brpopkey [key ...] timeout` 

这是`lpop`和`rpop`的阻塞版本，timeout 为阻塞秒数，timeout为0时，客户端一直阻塞等下去。

## 集合

集合(set)类型也是用来保存多个字符串，但和list不同的是，集合中不允许有重复的元素，且集合中的元素是无序的，不能通过索引下标获取元素。一个集合最多可以存放2^32-1个元素

**内部编码**

集合的内部编码有两种：

- intset：整数集合，当集合中元素都是整数且元素个数小于set-max-intset-entries配置(默认512)是，用此实现
- hashtable ：哈希表，集合类型不满足intset条件时，使用hashtable



**添加**

`sadd key element [element ...]` 	返回添加成功的元素个数

**删除元素**

`srem key element [element ...]` 	返回删除成功元素的个数

**计算元素的个数**

`scard key` 	返回集合内元素的数量

**判断元素是否在集合中**

`sismember key element` 	存在返回1，否则返回0

**随机从集合返回指定个数元素**

`srandmember key [count]` 	[count]可选，不写默认为1

**从集合随机弹出元素**

`spop key [count]` 	该指令执行成功后元素会从集合删除，[count]可选，不写默认为1

**获取所有元素**

`smembers key` 	

###集合间的操作

**求多个集合的交集**

`sinter key [key ...]`

**求多个集合的并集**

`suinon key [key ...]`

**求多个集合的差集**

`sdiff key [key ...]`

**将交集、并集、差集的结果保存**

`sinterstore destination key [key ...]`

`suionstore destination key [key ...]`

`sdiffstore destination key [key ...]`



## 有序集合

有序集合(sorted set或zset)保留了集合不能有重复成员的特性，但其中的元素可以排序。它给每元素设置一个分数(score)作为排序的依据。

**内部编码**

- ziplist(压缩列表)：当有序结合的元素个数小于zset-max-ziplist-entries配置(默认128)，同时每个元素的值都小于zset-max-ziplist-value配置(默认64字节)时，redis会采用ziplist实现
- skiplist(跳跃表)：当不满足ziplist使用条件时，会使用skiplist实现

###集合内的操作

**添加成员**

`zadd key score member [score member ...]` 	添加成员及分数

- nx：member必须不存在，才可以设置成功，用于添加
- xx：member必须存在，才可以设置成功，用于更新
- ch：返回操作后集合内发生变化的元素个数
- incr：对scoer 做增加，相当于zincrby

**计算成员的个数**

`zcard key` 	返回成员的个数

**查看成员的分数**

`zscore key member` 	返回成员的分数，不存在则返回nil

**计算成员的排名**

`zrank key member ` 	按分数从到到底返回排名

`zrevrank key member ` 	按分数从低到高返回排名

**删除成员**

`zrem key member [member ...]` 返回成功删除的成员个数

**增加成员的分数**

`zincrby key increment member` 	给member 增加increment分，操作成功返回score

**返回指定排名范围的成员**

`zrange key start end [withscores]`		从低到高返回，加上[withscores]连同分数一起返回

`zrevrange key start end [withscores]`   	从高到低返回，加上[withscores]连同分数一起返回

**返回指定分数范围的成员**

`zrangebyscore key min max [withscores] [limit offset count]` 	按分数从低到高返回，，

`zrevrangebyscore key min max [withscores] [limit offset count]` 	按分数从高到低返回，按分数从低到高返回，[withscores]返回每个成员的分数， [limit offset count]限制输出的起始范围和个数。

- [withscores]返回每个成员的分数
- [limit offset count]限制输出的起始范围和个数
- [min] [max] 支持开区间(小括号)和闭区间(中括号)，-inf和+inf代表无限大和无限小，例：(200 +inf

**返回指定分数范围成员个数**

`zcount key min max` 	

**删除指定排名内的升序元素**

`zremrangebyrank key start end`

**删除指定分数范围的成员**

`zremrangebyscore key min max`

###集合间的操作

**交集**

`zinterstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregae sum|min|max]`

- destination：交集计算结果保存到这个键
- numkeys：需要做交集计算键的个数
- key[key ...]：需要做交集计算的键
- weights weight[weight ...]：每个键的权重，在做交集运算时，每个键中的每个member会将知己陈述乘以这个权重，每个键的权重默认是1
- aggregate sum|min|max：计算成员交集后，分值可以按照sum(和)、min(最小值)、max(最大值)做汇总，默认值是sum。

**并集**

`zunionstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregae sum|min|max]`  	做并集运算，参数同上

# 四、常用功能

## 慢查询

**慢查询日志**：系统在命令执行前后，计算每条命令执行的时间，当超过预设阈值，就将这条命令的相关信息(时间，耗时等详细信息)记录下来。

## redis-cli详解

## redis-server详解

`redis-server [--option]`

- `--test-memory memoryCount` ：简单检测系统是否能稳定地分配指定容量(单位:M)的内存给redis

## Pipeline

pipeline（流水线）将一组redis命令进行组装，一次传输给redis，减少每次网络传输的时间，再将这组redis命令执行的结果按顺序返回给客户端。此功能常在编程语言中实现

## 事务与Lua

为了保证多条命令组合的原子性，redis提供了简单的事务功能以及集成Lua脚本来解决这个问题。

###redis中的事务

redis提供了简单的事务：将一组命令放在`multi`和`exec`两个命令之间，此种方法不支持事务回滚，没有隔离级别的概念也不保证原子性。

redis事务的本质就是一组命令的集合，一次性、顺序性地执行一个队列中的一系列命令，在这个过程中，其他客户端提交的命令不会插入到当前事务执行命令的队列中。并且当前执行队列中如果有执行失败的命令，之前的命令不会回滚，之后的命令不会停止。

###Lua

Lua是一个轻量小巧的脚本语言，提供了booleans(布尔)、numbers(数值)、strings(字符串)、tables(表格)数据类型。Lua中注释为“--”

#### string

定义一个字符串类型的数据

```Lua
local strings val = "world"		//local代表局部变量，没有local则代表全局变量
```

#### 数组

使用类似数组的功能，用tables类型实现，Lua的数组下标从1开始计算

```Lua
local tables myArray = {"redis","java",true,88.0}
```

##### for

```Lua
local int sum = 0
for i = 1, 100
do
    sum = sum + i
end
print(sum)		--输出结果为5050
--------------------------------------
for i = 1,#myArray		-- 在变量前加#可以求出数组程度
--------------------------------------
for index,value in ipairs(myArray) 	-- Lua提供内置函数ipairs,此句可以遍历所有的索引下标和值
```

#####while

while同样将循环流程放在do和end之间

##### if else

```Lua
for i = 1,#myArray
do
    if myArray[i] == "java"
    then					--if后紧跟then
        print("true")
        break				--break作为条件语句结束
    else
        --do nothing
    end						--
end
```

##### 哈希

要使用类似哈希的功能，也可以使用tables类型

```Lua
local tables user1 = {age = 28,name = "tom"}
print("user1 age is:"..user1["age"])	--str1..str2连接两个字符串
for key,value in pairs(user1)
do 
    print(key..value)					--遍历打印pairs
end
```

#####函数定义

函数以function 开头，以end结尾，funcName是函数名，中间部分是函数体

```Lua
function funcName()
    --函数体
end
```

###redis与Lua

**在redis中执行Lua脚本**

好处

- 原子性
- 可定制
- 一次性打包减少网络开销

有两种方法：

`eval 脚本内容 key个数 key列表 参数列表` 

```Lua
> eval 'return "hello"..KEY[1]..ARGV[1]' 1 redis world		--"helloredisworld"
```

KEY[1]="redis",ARGV[1]="world" 

`evalsha 脚本SHA1值 key个数 key列表 参数列表`

此方法需要先将Lua脚本加载到redis服务端，得到该脚本的SHA1校验和，使用SHA1作为参数可以直接执行对应脚本

```Lua
# redis-cli script load "$(cat lua_get.lua)"
"7413dc..."			--返回一串SHA1字符串
> evalsha 7413dc... 1 redis world
"helloredisworld"
```

**在Lua中使用redis API**

Lua可以使用redis.call函数实现对redis的访问

```Lua
redis.call("set","hello","word")	
redis.call("get","hello")			
```

Lua还可以使用redis.pcall，不同的是redis.call执行失败会返回错误；redsi.pcall会忽略错误继续执行脚本

**redis管理Lua脚本**

`script load script` 	用于将Lua脚本加载到redis内存中

`script exists sha1 [sha1 ...]` 	判断sha1是否加载到内存中

`script flush` 	清除redis内存已经加载的所有Lua脚本

`script kill `		杀死正在执行的Lua脚本，如果脚本正在执行写操作，该命令不会生效

##Bitmaps

可以实现对 位 的操作

大量数据（百万以上）的时候使用Bitmaps比使用set能节省存储空间

## HyperLogLog

一种基数算法，通过HyperLogLog可以利用极小的内存完成独立的总数的统计,存在误差率，约0.81%

它提供了3个命令：

`pfadd key element [element ...]`  	向HyperLogLog中添加元素，添加成功返回1

`pfcount key [key ...]`  	计算一个或多个HyperLogLog的独立（不重复）总数

`pfmerge destkey sourcekey [sourcekey ...]` 	求出多个HyperLogLog的并集并赋值给destkey  

##发布订阅

redis提供了基于“发布/订阅”模式的消息机制，消息发布者和订阅者不需要直接通信，发布者客户端向指定的频道发布消息，订阅该频道的每个客户端都可以收到该消息。和专业的消息队列系统相比，redis的发布订阅无法实现消息堆积和回溯，但足够简单。

**发布消息**

`publish channel message` 	返回订阅者的个数

**订阅消息**

`subscribe channel [channel ...]`

**取消订阅**

`unsubscribe [channel...]`

**按照匹配模式订阅和取消订阅**

`psubscribe pattern  `

`punsubscribe [pattern ...]`

**查询订阅**

`pubsub channels [pattern]`		查看活跃(至少有一个订阅者)频道

`pubsub numsub [channel ...]` 	查看频道订阅数

`pubsub numpat` 	查看模式订阅数

## GEO

GEO(地理信息定位)

# 客户端

了解redis服务端和客户端的通信协议以及主流编程语言的redis客户端使用方法。

- 客户端和服务端之间的通信协议是在TCP协议之上构建的
- redis指定了RESP(REdis Serialization Protocol) 实现客户端与服务端的正常交互

###使用Jedis

**获取redis实例**

```java

----直接使用jedis对象-------
Jedis jedis = new Jedis("127.0.0.1",6379);	//生成一个Jedis对象，这个对象负责和指定Jedis实例通信
--------使用连接池----------
GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
JedisPool jedisPool = new JedisPool(poolConfig,"127.0.0.1",6379);
jedis = jedisPool.getResource();

jedis.set("hello","world");					//jedis执行set操作
String value = jedis.get("hello")			//jedis执行get操作
jedis.close();
```

直接使用Jedis并初始化Jedis

`Jedis (final String host, final int port, final int connTimeout,final int soTimeout)`

- host ：Redis实例的所在机器IP
- port ：端口
- connTimeout ：客户端连接超时
- soTimeout：客户端读写超时

**Redis中Pipeline的使用方法**

```java
Jedis jedis = new Jedis(host,port);
Pipeline pipeline = jedis.pipelined();
pipeline.del(key) ...		//封装多个命令
pipeline.sync();			//完成命令的调用
//pipeline.syncAndReturnAll()  将pipeline的命令进行返回
```

**Jedis的Lua脚本**

Jedis中执行Lua脚本和redis-cli十分类似，jedis提供了三个函数实现Lua脚本的执行

`Object eval(String script, int keyCount, String... params)
` 

`Object evalsha(String sha1, int keyCount, String... params)`
`String scriptLoad(String script)`

```java
String key = "hello";
String script = "return redis.call('get',KEY[1])";
Object result = jedis.eval(script,1,key);
```

# 持久化

Redis支持RDB和AOF两种持久化机制，持久化功能可以有效避免因进程退出而造成的数据丢失问题，下次重启时利用之前持久化的文件即可实现数据恢复。

如果只使用Redis作为缓存，持久化可以不打开

RDB保存的是数据，AOF保存的是指令

**两种持久化方式如何选择**

- 官方推荐两个都启用。
- 如果对数据完整性要求不高可以单独使用RDB
- 不建议单独使用AOF
- 如果只做缓存使用可以不用持久化

## RDB

RDB持久化是在指定的时间间隔内把当前进程数据生成快照保存到硬盘的过程，恢复是是直接将快照文件读到内存中。RDB默认开启。

**备份如何执行**

Redis会单独fork出一个子进程来进行持久化，先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。

RDB的优点：

- RDB是一个紧凑压缩的二进制文件，节省磁盘空间，非常适用于备份
- Redis加载RDB恢复数据远快于AOF的方式

RDB的缺点：

- 无法做到实时持久化/秒级持久化（AOF适合）
- 使用特定二进制格式保存，存在老版本无法兼容新版RDB格式的问题
- Redis意外宕机，最后一次持久化之后的数据可能丢失

**如何配置：**

在redis.conf中配置文件名称，默认为dump.rdb

```
#The filename where to dump the DB
dbfilename dump.rdb

#默认保存在当前目录下
dir ./

#保存策略 save <seconds> <changes> 默认为以下，满足一个就会自动触发持久化
save 900 1
save 300 10
save 60 10000

#当redis无法写入磁盘时，直接关掉Redis的写操作以防止新数据无法持久化
stop-writes-on-bgsave-error yes
#存储快照后，再进行一次数据校验，会多消耗10%的性能，可酌情关闭
rebchecksum yes
```

触发RDB持久化过程分为手动触发和自动触发

**自动触发**

配置文件可配置自动触发规则

redis正常关闭会触发持久化

**手动触发**

对应命令为save(已废弃)和bgsave(主流)

- save命令会阻塞当前Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会造成长时间阻塞
- bgsave命令：Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短

**RDB的恢复**

将备份文件拷贝到工作目录下，重启Redis后，备份数据会直接加载

## AOF

AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的，解决了实时性数据持久化的问题。AOF默认不开启。

**使用AOF**

开启AOF功能需要设置配置：

```
appendonly yes  #开启AOF，开启后系统优先取AOF的持久化文件加载
appendfilename "appendonly.aof" 	#配置文件名称
#文件路径和RDB共用配置
```

其持久化流程为

1. 所有的写入命令会追加到aof_buf（缓冲区）中
2. AOF缓冲区根据对应的策略向硬盘做同步操作
3. 随着AOF文件越来越大，当文件超过阈值大小时，Redis会对AOF文件进行重写，达到压缩的目的
4. 当Redis服务器重启时，可以加载AOF文件进行数据恢复。

**优点**

- 备份机制更稳健，丢失数据的概率更低
- 有可读的日志文本

**缺点**

- 比RDB占用更多的磁盘空间
- 恢复备份的速度更慢
- 每次读写都同步，有一定的性能压力

AOF持久化开启且存在AOF文件时，优先加载AOF文件；AOF关闭或AOF文件不存在时，加载RDB文件

加载AOF /RDB文件成功后，Redis启动成功。

AOF/RDB 文件存在错误时，Redis启动失败并发音错误信息

# 主从复制

分布式系统中为了解决单点问题，通常会把数据复制多个副本部署到其他机器，满足故障恢复和负载均衡等需求。Redis也提供了复制功能，实现了相同数据的多个Redis副本。参与复制的Redis实例划分为主节点和从节点。从节点只能有一个主节点主节点可以有多个从节点。**复制的数据流是单向的，只能由主节点到从节点。Master以写为主，Slave以读为主。**

用处：

- 读写分离，性能扩展
- 容灾快速恢复

## 配置

使用`info replication`可以查看当前redis的信息

**建立复制**

配置复制的方式有三种：

- 在配置文件中加入`slaveof {masterHost} {masterPort}`随redis启动生效
- 在redis-server启动命令后加入 `--slaveof {masterHost} {masterPort}`生效
- 直接使用命令：`slaveof {masterHost} {masterPort}`生效

主从节点复制成功建立后，可以使用`info replication`命令查看复制相关状态

**断开复制**

在子节点上执行`slaveof no one`可以断开与主节点的复制关系，断开复制后不会抛弃原有数据，无法继续获取变化数据，自身角色变为master

`slaveof {newMasterHost} {newMasterPort}` 切换到另一个主节点，此操作会删除从节点当前所有数据

**一些问题**

1. 从服务器断开后，再连接到主服务器仍然会同步到主服务上的所有内容
2. 主服务器shutdown之后，从服务器会原地待命

**传输延迟问题**

`repl-disable-tcp-nodelay`参数用于控制是否关闭TCP_NODELAY，默认关闭

- 关闭时，主从之间延迟变小，但增加网络带宽消耗。适合同机房部署
- 开启时，节省带宽，增大延迟。适合远程

##拓扑

**一主一从**

最简单的结构。若只在从节点开启AOF，需要注意主节点重启之前，先断开从节点，否则重启之后从节点数据也会被清空。

**一主多从**

应用端可以利用多个从节点实现读写分离。读占比较大的场景，可以将比较耗时的读命令在其中一台从节点上执行，防止慢查询造成阻塞。而对于写并发量高的场景，多个从节点会导致主节点命令的多次发送从而过度消耗网络带宽，并加重了主节点的负载，影响服务的稳定性。

**树状主从结构**

使得从节点不但可以复制主节点的数据，同时可以作为其他节点的主节点继续向下复制。通过引入复制中间层可以有效降低主节点的负载和需要传输给从节点的数据量。

**哨兵模式sentinel**

一个哨兵服务器监控主机是否正常，故障了可以根据投票自动将从服务器变为主服务器

## 原理

###复制过程

1. 保存主节点信息
2. 从节点内部通过每秒运行的定时任务维护复制相关逻辑
3. 发送ping命令
4. 权限验证
5. 同步数据集
6. 命令持续复制

###数据同步

**全量复制**

一般用于初次复制的场景，主节点一次性将数据发送给从节点

**部分复制**

用于处理在主从复制中因为网络闪退等原因造成的数据丢失场景，从节点再次连上主节点时，会补发对视数据给从节点

#Redis集群

容量不够，如何扩容

并发写操作，redis如何分摊

redis集群实现了对redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总量数据的1/N。redis集群通过分区来提供一定程度的可用性，即时集群中有一部分节点失效或无法进行通讯，集群也可以继续处理命令请求。





# 阿里巴巴Redis开发规范

##键值设计

### 1、key名设计

**可读性和可管理性**

以业务名(或数据库名)为前缀(防止key冲突)，用冒号分隔，比如业务名:表名:id

```java
ugc:video:1
```

**简洁性**

保证语义的前提下，控制key的长度，当key较多时，内存占用也不容忽视，例如：

```java
user:{uid}:friends:messages:{mid}简化为u:{uid}:fr:m:{mid}。
```

**不要包含特殊字符**

反例：包含空格、换行、单双引号以及其他转义字符

### 2、value设计

**拒绝bigkey**

防止网卡流量、慢查询，string类型控制在10KB以内，hash、list、set、zset元素个数不要超过5000。

反例：一个包含200万个元素的list。

非字符串的bigkey，不要使用del删除，使用hscan、sscan、zscan方式渐进式删除，同时要注意防止bigkey过期时间自动删除问题(例如一个200万的zset设置1小时过期，会触发del操作，造成阻塞，而且该操作不会不出现在慢查询中(latency可查))，查找方法和删除方法
选择适合的数据类型

例如：实体类型(要合理控制和使用数据结构内存编码优化配置,例如ziplist，但也要注意节省内存和性能之间的平衡)
反例：

```java
set user:1:name tom
set user:1:age =19
set user:1:favor football
```

正例：

```java
hmset user:1 name tom age =19 favor football
```

**控制key的生命周期**

redis不是垃圾桶，建议使用expire设置过期时间(条件允许可以打散过期时间，防止集中过期)，不过期的数据重点关注idletime。

##命令使用

**1、O(N)命令关注N的数量**

例如hgetall、lrange、smembers、zrange、sinter等并非不能使用，但是需要明确N的值。有遍历的需求可以使用hscan、sscan、zscan代替。

**2、禁用命令**

禁止线上使用keys、flushall、flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理。

**3、合理使用select**

redis的多数据库较弱，使用数字进行区分，很多客户端支持较差，同时多业务用多数据库实际还是单线程处理，会有干扰。

**4、使用批量操作提高效率**

原生命令：例如mget、mset。

非原生命令：可以使用pipeline提高效率。

但要注意控制一次批量操作的元素个数(例如500以内，实际也和元素字节数有关)。

注意两者不同：

- 原生是原子操作，pipeline是非原子操作。
- pipeline可以打包不同的命令，原生做不到
- pipeline需要客户端和服务端同时支持。

**5、不建议过多使用Redis事务功能**

Redis的事务功能较弱(不支持回滚)，而且集群版本(自研和官方)要求一次事务操作的key必须在一个slot上(可以使用hashtag功能解决)

**6、Redis集群版本在使用Lua上有特殊要求**

1、所有key都应该由 KEYS 数组来传递，redis.call/pcall 里面调用的redis命令，key的位置，必须是KEYS array, 否则直接返回error，"-ERR bad lua script for redis cluster, all the keys that the script uses should be passed using the KEYS arrayrn"
2、所有key，必须在1个slot上，否则直接返回error, "-ERR eval/evalsha command keys must in same slotrn"

**7、monitor命令**

必要情况下使用monitor命令时，要注意不要长时间使用。