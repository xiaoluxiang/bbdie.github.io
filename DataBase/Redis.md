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

> Redis（Remote Dictionary Server）是一种支持key-value等多种数据结构的存储系统。提供字符串，哈希，列表，队列，集合结构直接存取。基于内存，可持久化。支持网络。可用于缓存，事件发布或订阅，高速队列等场景。
>
> 特点：读写性能优异，数据类型丰富，原子性，丰富的特性，持久化，发布订阅，分布式

## redisObject对象内存布局

> 对于不同的数据结构会有不同的操作方式，不同的存储方式，所以需要一个对象类型系统。一是为了承接命令（都为受检查的类型，以供判断），二是为了多态，能够调用不同的操作函数

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-object-1.png)

type决定数据类型，encoding决定编码方式，lru记录空转时间，refcount记录引用次数，ptr指向具体存储地址

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-object-2-2.png)

## 命令的类型检查和多态

1. 检查Key值是否存在，即通过获取该key的redisObject是否存在
2. 检查该Object的所属类型与命令是否匹配
3. 多态的调用函数

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-object-3.png)

### 对象共享&对象的销毁

对象的共享：redis会把一些常用对象预先分配，共享使用，以减少CPU频繁创建销毁。其中包括各种命令的返回值（ok，error，QUEUE等），0到`REDIS_SHARED_INTEGERS`的整数

对象的销毁：redis的对象结构中存在refcount属性，指示引用次数。为0时，内存结构将会被释放

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

> 数据结构设计当中，length开头，直接标明当前结构的整体长度

### 简单动态字符串 - sds

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-ds-x-3.png)

sds设计了头部&存储内容&\0的存储结构，支持动态扩容。其中头部包含length，alloc，flag（注意这里的length和alloc的数学关系）。

len&alloc：常数级时间复杂度获取字符串实际长度，拥有了实际长度，搭配alloc就可以极大避免内存溢出和内存泄漏，能实现内存预分配和惰性回收。另外采用二进制存储比C语言中字符串存储方式安全。在实际应用中支持文本和二进制数据流

C语言中的字符串是以/0（空字符串结尾）不记录字符串长度。是通过遍历获取到字符串的长度。扩大合并或缩小时均可能会引起内存问题（内存溢出或泄漏），在实际用途中也只能存储文本（因为二进制数据流可能存在\0导致异常）



>  空间预分配是指sds在初始内存扩展时，会分配给他们大小为所需大小的一倍，但不超过1M。存在内存浪费现象，并且不会被释放。如果存在内存问题，是需要手动修改redis服务

### 压缩列表 - ZipList

> 考虑到每个末尾识别，List存在pop操作

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-redis-ds-x-6.png)

一般结构 `<prevlen> <encoding> <entry-data>`

`prevlen`：前一个entry的大小，编码方式见下文；

`encoding`：不同的情况下值不同，用于表示当前entry的类型和长度；

`entry-data`：真是用于存储entry表示的数据；

**ZipList选择不定长存储Entry，故需要描述整个List的属性和当前Entry属性（还包括前一个Entry长度属性），所以带来了大量存储的优化，和动态扩容下最坏情况的劣化**

### 快表 - QuickList

> quickList采用双链表方式，quickList节点属性描述的整个quickList级属性，

正常数组每个元素占用的大小是一样的，并且取决于最大的元素。ZipList解决了存储效率问题，带来了使用时扩缩容带来的重分配问题，所以设计QuickList，并提供QuickList.fill，可以根据经验调参

### 字典/哈希表 - Dict

### 整数集 - IntSet

### 跳表 - ZSkipList

# RDB和AOF机制

> Redis 数据存储在内存，停机会导致内存数据丢失，如果从后端服务恢复有难度且影响后端服务性能，所以要自行备份和恢复
>
> Redis严格意义上存在四种数据持久化的方式，RDB，AOF，VM，DISKSTORE。VM被舍弃，DISKSTORE目前尚未推出

## RDB

> Redis DataBase 内存快照，即某一时刻的内存全量快照，经压缩存储的全量快照

### 触发方式

手动：sava或者bgsave。save是阻塞式，直到完成全部操作。bgsave先fork子进程（过程是阻塞的）然后子进程完成RDB的操作![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/redis-x-rdb-1.png)

自动触发：

- 执行debug reload命令重新加载redis时也会触发bgsave操作；
- redis.conf中配置`save m n`，即在m秒内有n次修改时，自动触发bgsave生成rdb文件；
- 默认情况下执行shutdown命令时，如果没有开启aof持久化，那么也会触发bgsave操作；
- 主从复制时，从节点要从主节点进行全量复制时也会触发bgsave操作，生成当时的快照发送到从节点；

### 执行过程

RDB执行bgsave时，主进程fork子进程（过程阻塞），子进程正常读取并生成**全份文件备份**（被压缩的二进制快照文件），对于并发问题通过copy-write解决。将备份文件写入磁盘（注意IO瓶颈）。最后通过信号通知主进程。

在save过程中出现意外也不用担心，redis是通过修改文件名的方式原子替换源文件。

## AOF

> Append Only File 只是追加的文件，记录的是增量变化

### 触发方式

默认关闭，配置开启后，每当写操作时

### 执行过程

当执行写操作时，主线程会先执行完毕后，再写入AOF文件。相当于先后顺序写俩份文件，至于怎么刷入磁盘可配置

**AOF重写：**为了解决AOF文件可能过大的隐患，还会有AOF重写即合并中间冗余操作。触发方式是文件起步大小或者增量差距。AOF重写也采用内存copy-write方式解决。方式是fork出的子线程有着相同的地址映射表，父线程更新时会复制然后更新新的内存页。在重写过程中新的数据写入会存在AOF重写缓存区，采用Linux管道技术尽早重放命令，解决AOF重写缓存区过大的问题

**阻塞：**父线程会在重写线程的fork，bigKey写入，AOF文件替换过程中。不用原AOF文件是徒增竞争&污染原文件

## RDB&AOF组合

> 避免RDB所需频繁fork子进程，同时也避免了AOF文件过大的问题，避免AOF重写开销

Redis在加载AOF文件后，不会加载RDB文件，没有AOF，才会加载RDB文件

**使用建议：**

- 降低fork频率
- 避免redis内存过大，防止fork过长，同时内存也不能过小，防止fork失败
- 同时使用RDB&AOF。或者索性关闭持久化
- 单台机器多个实例时，防止同时持久化，可以从架构上一主一备。主提供服务，备提供备份服务
