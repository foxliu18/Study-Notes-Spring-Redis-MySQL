<center> <b><h1>Redis</h1></b></center>

## 概述

Redis(Remote Dictionary Server)，即远程字典服务

redis会周期性的把更新数据写入磁盘或者把修改操作写入追加的记录文件,并且在次基础上实现的master-slave同步



------

Redis能干嘛

1、内存存储、持久化、内存中是断电即事失、所以说持久化很重要（RDB、AOF）

2、效率高、可以用于告诉缓存

3、发布订阅系统

4、地图信息分析

5、计数器、定时器（浏览量）

6、...

> 特性

1. 多样的数据类型

2. 持久化

3. 集群

4. 事务

   ...

> 学习中需要的东西

1. 公众号
2. Redis官网
3. Redis中文网站
4. GitHub

> 基础知识

默认16个数据,默认使用第一个

可以使用select进行切换数据库

``` bash
127.0.0.1:6379> keys * # 查看所用key
1) "hello"
2) "name"
127.0.0.1:6379> 
```

清除当前数据库

``` bash
127.0.0.1:6379> FLUSHDB
OK
```

清除全部数据库

``` bash
127.0.0.1:6379> FLUSHALL
OK
```

> redis是单线程的

Redis是基于内存的,CPU不是Redis的性能瓶颈,Redis的瓶颈是机器的内存和网络带宽,既然可以使用单线程来实现,就使用单线程实现

Redis是C语言写的,完全不比同样使用key-value的Memcached差

**Redis为什么单线程还这么快**

1、误区1: 高性能服务器一定是多线程的?

2、误区而:多线程(CPU上下文切换)一定比单线程效率高!

先去CPU>内存>硬盘的速度有所了解

核心: Redis是将多有数据存放在内存中的,所以说使用单线程去操作效率就是最高的,多线程(CPU上下文切换,是耗时的操作),对于内存系统来说,如果没有上下文奇幻效率就是虽高的!多次读写都是在一个CPU上的,在内存情况下,这个就是最佳方案

## 数据类型

### String

底层存储结构：简单动态字符串（SDS）

1. 能转整数的：int
2. 小于等于44字节：embstr
3. 大于44字节： raw

使用场景： 

+ 单值缓存：SET key value; GET key

+ 对象缓存: 

  1. SET user:1 value(json数据)

  2. MSET user:1:name zhunge user:1:balance 1888

      MGET user:1:name user:1:balance

+ 分布式锁

  SETNX product:10001 true    //返回1代表获取锁成功

  SETNX product:10001 true   //其他线程获取锁返回0，表示失败

  ... 执行业务...

  DEL product:10001                //执行完业务释放锁

  SET product:10001 true ex 10 nx      //防止程序一万终止导致死锁，增加锁有效期，为防止意外超时，可以增加自动锁续命，redisson有实现

+ 计数器

  INCR article：readcount:{文章id}

  GET article：readcount:{文章id}     //记录点阅次数

+ 分布式系统全局序列号

  INCRBY orderId 100       //redis批量生成序列号提升性能能，用于存在分库分表数据库自增id

  

### List

底层存储结构：

			1. 压缩列表
   			2. 双向链表

+ List常用操作

  LPUSH key value [value...]  //一个或多个值插入到list头（左）

  RPUSH key value [value...]  //一个或多个值插入到list尾（右）

  LPOP key                              //移除并返回key列表的头元素

  RPOP key                              //移除并返回key列表的尾元素

  LRANGE key start stop          //返回key表指定区间内元素，偏移量start-end

  BLPOP key [key...] timeout   //从表头弹出一个元素，如果没有阻塞等待timeout秒。如果timeout=0则一直等待

  BRPOP key [key...] timeout   //从表尾弹出一个元素，如果没有阻塞等待timeout秒。如果timeout=0则一直等待

+ 重用数据结构

  Stack = LPUSH + LPOP

  Queue = LPUSH + RPOP

  Blocking MQ = LPUSH + BRPOP

### Set

底层存储结构：

1. 哈希表
2. 整数数组

+ Set常用操作

  SADD key member [member...]     //往集合key中存入元素，元素存在则忽略，key不存在则新建

  SREM key member [member...]      //从集合key中删除元素

  SMEMBERS key                                 //获取集合key中所有元素

  SCARD key                                         //获取集合key的元素个数

  SISMEMBER key member                 //判断member元素是否存在集合key中

  SRANDMEMBER key [count]             //从集合key中选出count个元素，元素不从key中删除

  SPOP key [count]                              //从集合key中选出count个元素，元素从key中删除

+ Set运算操作

  SINTER key [key...]               //交集运算

  SINTERSTORE destination key [key...]      //将交集结果存入新集合destination中

  SUNION key [key...]     //并集运算

  SUNIONSTORE destination key [key...]      //将并集结果存入新集合destination中

  SDIFF key [key...]                                           //差集运算

  SDIFFSTORE destination key [key...]             //将差集结果存入新集合destination中 

###  Hash

底层存储结构：

1. 哈希表
2. 压缩列表

+ 对象缓存：

  HMSET user {userId}:name zhuge {userId}:balance 1888

  HMSET user 1:name zhuge 1:balance 1888

  HMGET user 1:name 1:balance

==优点==

1. 同类数据归类整合存储，方便数据管理
2. 相比string操作消耗内存与CPU更小

==缺点==

1. 过期功能不能使在field（对象字段）上，只能用在key上（对象key）
2. Redis集群架构下不适合大规模使用

### Zset

底层存储结构：

1. 压缩列表
2. 跳表

+ Zset常用操作

  ZADD key score member [[score member] ...]    //往有序集合key中加入带分值元素

  ZREM key member [member ...]             //从有序集合key中删除元素

  ZSCORE key member                              //返回有序集合key中元素member的分值

  ZINCRBY key increment member           //为有序集合key中元素member的分值加上increment

  ZCARD key                                            //返回有序集合key中元素个数

  ZRANGE key start stop [WITHSCORES]          //正序获取有序集合key从start到stop下标的元素

  ZREVRANGE key start stop [WITHSCORES]    //倒序获取有序集合key从start到stop下标的元素

+ Zset集合操作

  ZUNIONSTORE destkey numkeys key [key ...]    //并集计算

  ZINTERSTORE destkey numkeys key [key ...]     //交集计算

##  事务

MySql ACID

原子性: 同时成功同时失败

Redis单条命令保证原子性, 但redis事务不保证原子性

Redis事务本质: 一组命令的集合! 一个事物的所有命令都会被序列化,会按照顺序执行! 一次性,顺序性,排他性

-- redis事务没有隔离级别的概念 --

所有命令在事务中,并没有被直接执行, 只有发起执行命令的时候才会执行! exec

redis的事务: 

+ 开启事务(multi)
+ 命令入队(...)
+ 执行事务(exec)

> 正常执行事务

``` bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "v2"
4) OK

```

> 放弃事务

``` bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> DISCARD
OK
127.0.0.1:6379> exec
(error) ERR EXEC without MULTI
127.0.0.1:6379> get k4
(nil)

```

> 编译型异常(代码有问题,命令有错),事务中所有的命令都不会执行

``` bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> getset k3
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> getk5
(error) ERR unknown command `getk5`, with args beginning with: 

```



> 运行时异常(I/O), 如果事务队列存在语法性错误,那么执行命令的时候, 其他命令可以正常执行, 错误命令抛出异常

``` bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
4) "v2"
```

> 监控

悲观锁:

+ 很悲观, 什么时候都会出问题, 无论做什么都会加锁

乐观锁:

+ 很乐观， 认为什么时候都不会出问题， 所以不会上锁，更新数据的时候去判断一下， 在此期间是否有人修改过这个数据
+ 获取version
+ 更新的时候比较version

> 正常执行

``` bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> WATCH money # 监视money对象
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20

```

测试多线程修改后,使用watch当做redis的乐观锁操作

```bash
# 线程1
127.0.0.1:6379> WATCH money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 40
QUEUED
127.0.0.1:6379> INCRBY out 40
QUEUED
127.0.0.1:6379> exec # 事务执行失败
(nil)
# 线程2 修改money
127.0.0.1:6379> set money 1000
OK
127.0.0.1:6379> 
```

如果修改失败获取最新的的值

``` bash
127.0.0.1:6379> UNWATCH
OK
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 100
QUEUED
127.0.0.1:6379> INCRBY out 100
QUEUED
127.0.0.1:6379> exec
1) (integer) 900
2) (integer) 120

```

## Jedis

使用Java操作Redis

> Jedis 是redis官方推荐的java链接开发工具! 使用java操作Redis的中间件, 如果使用java操作redis, 那么一定要对jedis十分熟悉

## Redis.conf详解

启动的时候,就通过配置文件启动

## Redis 持久化

Redis是内存数据库, 如果部将内存中的数据库状态保存到磁盘, 那么一旦服务器进程退出, 服务器中的数据状态也会消失. 所以redis提供了持久化功能.

### RDB(Redis DataBase)

> 什么是RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘, 也就是行话讲的Snapshot快照, 它恢复时是将快照文件直接读到内存里

Redis会单独创建(fork)一个子进程来进行持久化, 会先将数据写入到一个临时文件中, 持久化过程都结束了, 在用这个临时文件替换上次持久化好的文件. 整个过程中, 主进程是不进行任何IO操作的. 这就确保了极高的性能. 如果需要进行大规模的数据恢复, 且对于数据恢复的完整性不是非常敏感, 那RDB方式要比AOF方式更加的高效. RDB的缺点是最后一次初九化后的数据可能丢失

默认持久化模式就是RDB, 一般不需要修改这个配置

==RDB保存的文件名是 dump.rdb == 

> 触发机制

1, save的规则满足,会自动触发RDB规则

2, 执行flushall命令, 也会触发我们的RDB规则

3, 退出Redis, 也会产生RDB文件

备份就会自动生成dump.rdb文件

> 如何恢复RDB文件

1, 只要将RDB文件放到redis启动目录, redis启东市就会自动恢复数据

2, 查看需要存在的位置

**优点**

1. 适合大规模的数据恢复
2. 对数据完整性要求不高

**缺点**

1. 需要一定的时间间隔进操作,如果redis意外宕机了, 这个最后一次修改数据就没有了
2. fork进程的时候会占用一定的内存空间

### AOF(Append Only File)

将我们所有的命令都记录下来,恢复的时候就把记录写命令全部执行一遍

以日志的形式来记录每个写操作, 将redis执行过得所有写命令记录下来,秩许追加文件但不可以改写文件,redis启动之初会读取文件重新构建数据, 换言之, redis重启德华就根据日志文件的内容将指令从前到后执行一次以完成数据的恢复工作

==AOF保存的是appendonly.aof文件==

> appendonly yes

默认不开启, 需要手动配置

重启, redis就可以生效了

如果这个AOF文件有错位, 这时候redis是启动不起来的, 需要修复这个AOF文件,redis提供一个工具redis-check-aof --fix

**优点**

1. 每次修改都同步,文件的完整型更加好
2. 默认每秒同步一次,可能会丢失一秒的数据
3. 从不同步,效率更高

**缺点**

1. 相对数据文件来说, aof远远大于RDB, 修复的速度也比RDB慢
2. AOF运行效率也要比RDB慢, 所以我们redis默认配置就是RDB持久化

## Redis缓存穿透和雪崩

Redis缓存的使用,极大的提升了应用程序性能和效率, 特别是数据查询方面,但同时他也带来了一些问题. 其中, 最要害的问题就是数据一致性问题, 从严格意义上讲,这个问题无解.如果对数据一致性要求很高,那么就不能使用缓存.

另外一些典型问题就是,缓存穿透,缓存雪崩和缓存击穿.目前,业界也都有比较流行的解决方案

### 缓存穿透

> 概念

缓存穿透的概念很简单,用户想要查寻一个数据,发现redis内存数据库没有, 也就是缓存没有命中, 于是向持久层数据库查寻. 发现也没有, 于是本次查寻失败. 当用户很多的时候, 缓存都没没有命中, 于是都去请求了持久层数据库. 这回给持久层数据库造成很大的压力, 这个时候就相当于出现了缓存穿透

> 解决方案

**布隆过滤器**

布隆过滤器是一种数据结构, 对所有可能查询的参数以hash形式存储, 在控制层先进性校验, 不符合则丢弃, 从而避免了对底层存储系统的查询压力

**缓存空对象**

当存储层不命中后, 即使返回的空对象也将其缓存起来, 同时会设置一个过期的时间, 之后在访问这个数据将会从缓存中获取, 保护了后端数据源.

但这种方法会存在两个问题:

1. 如果空值能够被缓存起来,  这就意味着缓存需要更多的空间存储更多的键, 因为这当中可能会有很多的空值的键
2. 即使对空值设置了过期时间, 还是会存在缓存层和存储层的数据会有一段时间窗口的不一致,这对于需要保持一致性的业务会有影响.

### 缓存击穿

> 概述

这里需要注意和缓存击穿的区别, 缓存击穿,是指一个key非常热点, 在不停的扛着大并发, 大并发集中对这一个典进行访问, 当这个key在失效的瞬间, 持续的大并发就穿破缓存, 直接请求数据库,.

当某个key在过期的瞬间, 有大量的请求并发访问, 这类数据一般是热点数据, 由于缓存过了期, 会同时访问数据来查询最新数据, 并且回写缓存, 会导致数据库瞬间压力过大

> 解决方案

**设置热点数据永不过期**

从缓存层来看, 没有设置过期时间,所以不会出现热点key过期后产生的问题

**加互斥锁**

分布式锁: 使用分布式锁, 保证对于每个key同时只有一个线程去查询后端服务, 其他线程没有获得分布式锁的权限, 因此只需要等待即可. 这种方式将高并发的压力转移到了分布式锁, 因此对分布式锁的考验很大.

### 缓存雪崩

> 缓存雪崩

缓存雪崩, 指在某一个时间段, 缓存集中过期失效

产生雪崩的原因之一, 比如在写文本的时候, 马上就要到双十一零点,很宽就会迎来一波抢购, 这波商品时间比较集中的放入了缓存,假设缓存一个小时.那么到了凌晨一点的时候,这批商品的缓存就都过期了.. 而对这批商品的访问查询, 都落到了数据库上, 对于数据库而言, 就会产生周期性的压力波峰. 于是所有的请求都会达到了存储层, 存储层的调用会暴增, 造成存储层也会挂掉的情况

其实集中过期, 倒不是非常致命, 比较致命的缓存雪崩, 是缓存服务器某个节点宕机或断网. 因为自然形成的缓存雪崩, 一定是在某个时间段集中创建缓存, 这个时候,数据库也是可以顶住压力的. 无非就是对数据库产生周期性的压力而已. 而缓存服务节点的宕机, 对数据库服务造成的压力是不可预知的, 很有可能瞬间就把数据压垮

> 解决方案

**Redis高可用**

这个思想的含义是, 既然redis可能挂掉, 那就多增设几台redis, 这样一台挂掉时候其他的还可以继续工作, 其实就是搭建集群

**限流降级**

这个解决方案得思想是,在缓存失败后,通过加锁或者队列来控制读数据库写缓存的线程数量. 比如对某个key只允许一个线程查寻数据和写缓存, 其他线程等待

**数据预热**

数据预热的含义就是在正式部署前, 我先把可能的数据预先访问一遍, 这样部分可能大量访问的数据

就会加载到缓存中. 在即将发生大并发访问前手动触发加载缓存不同的key, 设置不同的过期时间, 让缓存失效的时间尽量均匀