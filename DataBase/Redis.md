# 缓存更新策略

## redis 缓存更新策略

> 隐含背景是正常缓存读取是读操作未命中，则读数据库然后更新缓存信息。由此可以看出让写线程更新缓存很大概率上会造成缓存与数据库数据不一致的情况
>
> 缓存删除后，理由读线程会读取数据库再更新缓存的特点，实现缓存和数据库同步，故建议对于缓存只需要删除操作即可
>
> 这几种策略，综合考虑双写和单读的情况，即可分析出利弊

### 先更新缓存后更新数据库

更新缓存速度快，更新数据库速度慢。当存在多个线程并发更新数据库，缓存和数据库的更新不具有原子性

### 先更新数据库再更新缓存

同先更新缓存再更新数据库同样的问题

### 先删除缓存再更新数据库

更新缓存速度快，更新数据库操作慢。先删除缓存，主动造成较长时间的不一致，而且由于利用读设置新缓存，会造成永久不一致。

### 先更新数据库再删除缓存

更新完之后再删除，比较完美。

### 终极解决方案

手动重试-> 在删除缓存失败后，将消息存入消息中间件中。可以额外消费消息。不断重试

框架重试-> 应用程序订阅数据库的binlog日志，搭配消息中间件，不断重试

# Redis 数据结构及其应用场景

> Redis（Remote Dictionary Server）是一种支持key-value等多种数据结构的存储系统。支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化可用于缓存，事件发布或订阅，高速队列等场景。
>
> 特点：读写性能优异，数据类型丰富，原子性，丰富的特性，持久化，发布订阅，分布式

## redisObject对象内存布局

> 对于不同的数据结构会有不同的操作方式，不同的存储方式，所以需要一个对象类型系统。一是为了承接命令（类型判断），二是为了多态调用不同的操作函数

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-object-1.png)

type决定数据类型，encoding决定编码方式，lru记录空转时间，refcount记录引用次数，ptr指向具体存储地址

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-object-2-2.png)

## 命令的类型检查和多态

1. 检查Key值是否存在，即通过获取该key的redisObject是否存在
2. 检查该Object的所属类型与命令是否匹配
3. 多态的调用函数

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-object-3.png)

## 数据类型

### String

​	操作命令

| 命令   | 简述                   | 使用              |
| ------ | ---------------------- | ----------------- |
| GET    | 获取存储在给定键中的值 | GET name          |
| SET    | 设置存储在给定键中的值 | SET name value    |
| DEL    | 删除存储在给定键中的值 | DEL name          |
| INCR   | 将键存储的值加1        | INCR key          |
| DECR   | 将键存储的值减1        | DECR key          |
| INCRBY | 将键存储的值加上整数   | INCRBY key amount |
| DECRBY | 将键存储的值减去整数   | DECRBY key amount |

889使用场景

- **缓存**： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。
- **计数器**：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。
- **session**：常见方案spring session + redis实现session共享

### List

数据结构

lpush+lpop=Stack(栈)

lpush+rpop=Queue（队列）

lpush+ltrim=Capped Collection（有限集合）

lpush+brpop=Message Queue（消息队列）

操作命令

| 命令   | 简述                                                         | 使用             |
| ------ | ------------------------------------------------------------ | ---------------- |
| RPUSH  | 将给定值推入到列表右端                                       | RPUSH key value  |
| LPUSH  | 将给定值推入到列表左端                                       | LPUSH key value  |
| RPOP   | 从列表的右端弹出一个值，并返回被弹出的值                     | RPOP key         |
| LPOP   | 从列表的左端弹出一个值，并返回被弹出的值                     | LPOP key         |
| LRANGE | 获取列表在给定范围上的所有值                                 | LRANGE key 0 -1  |
| LINDEX | 通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。 | LINDEX key index |

使用场景

- **微博TimeLine**: 有人发布微博，用lpush加入时间轴，展示新的列表信息。
- **消息队列**

### Set

数据结构

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

操作命令

| 命令      | 简述                                  | 使用                 |
| --------- | ------------------------------------- | -------------------- |
| SADD      | 向集合添加一个或多个成员              | SADD key value       |
| SCARD     | 获取集合的成员数                      | SCARD key            |
| SMEMBERS  | 返回集合中的所有成员                  | SMEMBERS key member  |
| SISMEMBER | 判断 member 元素是否是集合 key 的成员 | SISMEMBER key member |

使用场景

- 标签（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。

- **点赞，或点踩，收藏等**，可以放到set中实现

### Zset

数据结构

操作命令

| 命令   | 简述                                                     | 使用                           |
| ------ | -------------------------------------------------------- | ------------------------------ |
| ZADD   | 将一个带有给定分值的成员添加到有序集合里面               | ZADD zset-key 178 member1      |
| ZRANGE | 根据元素在有序集合中所处的位置，从有序集合中获取多个元素 | ZRANGE zset-key 0-1 withccores |
| ZREM   | 如果给定元素成员存在于有序集合中，那么就移除这个元素     | ZREM zset-key member1          |

使用场景

- 排行榜：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行

### Hash

​	数据结构

​	操作命令

| 命令    | 简述                                     | 使用                          |
| ------- | ---------------------------------------- | ----------------------------- |
| HSET    | 添加键值对                               | HSET hash-key sub-key1 value1 |
| HGET    | 获取指定散列键的值                       | HGET hash-key key1            |
| HGETALL | 获取散列中包含的所有键值对               | HGETALL hash-key              |
| HDEL    | 如果给定键存在于散列中，那么就移除这个键 | HDEL hash-key sub-key1        |

使用场景

- **缓存**： 能直观，相比string更节省空间，的维护缓存信息，如用户信息，视频信息等

### Bitmap

数据结构

操作命令

使用场景

#### HyperLogLogs

是一个带有 0.81% 标准错误的近似值

#### geospatial

 两地之间的距离, 方圆几里的人

## 底层数据结构

### 简单动态字符串 - sds

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-ds-x-3.png)

C语言中的字符串是以/0（空字符串结尾）不记录字符串长度。是通过遍历获取到字符串的长度。扩大合并或缩小时均可能会引起内存问题，在实际用途中也只能存储文本（因为二进制数据流可能存在\0导致一场）

sds为了能支持动态扩容，设计了头部&存储内容&\0的存储结构。其中头部包含length，alloc，flag（注意这里的length和alloc的数学关系）。在内存使用这方面，sds也通过空间换时间的方式，进行空间预分配与惰性释放。在实际应用中支持文本和二进制数据流

>  空间预分配是指sds在初始内存扩展时，会分配给他们大小为所需大小的一倍，但不超过1M。存在内存浪费现象，并且不会被释放。如果存在内存问题，是需要手动修改redis服务

### 压缩列表 - ZipList

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-ds-x-6.png)

一般结构 `<prevlen> <encoding> <entry-data>`

`prevlen`：前一个entry的大小，编码方式见下文；

`encoding`：不同的情况下值不同，用于表示当前entry的类型和长度；

`entry-data`：真是用于存储entry表示的数据；

快表 - QuickList

正常数组每个元素占用的大小是一样的，并且取决于最大的元素

字典/哈希表 - Dict
整数集 - IntSet
跳表 - ZSkipList

# RDB和AOF机制

> Redis 数据存储在内存，停机会导致内存数据丢失，如果从后端服务恢复有难度且影响后端服务性能
>
> Redis严格意义上存在四种数据持久化的方式，RDB，AOF，VM，DISKSTORE

## RDB

> Redis DataBase 内存快照，即某一时刻的内存全量快照，经压缩存储的全量快照

### 触发方式

手动：sava或者bgsave。save是阻塞式，直到完成全部操作。bgsave先fork子进程（过程是阻塞的）然后子进程完成RDB的操作![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/redis-x-rdb-1.png)

自动触发：

- redis.conf中配置`save m n`，即在m秒内有n次修改时，自动触发bgsave生成rdb文件；
- 主从复制时，从节点要从主节点进行全量复制时也会触发bgsave操作，生成当时的快照发送到从节点；
- 执行debug reload命令重新加载redis时也会触发bgsave操作；
- 默认情况下执行shutdown命令时，如果没有开启aof持久化，那么也会触发bgsave操作；

### 使用细节

RDB全量备份的过程要考虑快照的备份频率，IO瓶颈。bgsave过程中，多线程并发通过copy-on-write来解决。在save过程中出现意外也不用担心，redis是原子替换源文件。
