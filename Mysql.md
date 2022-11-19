# MySQL

## 数据类型

### 整数

tinyint,smallint,middleint,int,bigint；占用的字节数分别为4，8，16，32，64。对于int(10)，对于实际存储占用没有效果，只是为了交互显示所用。

### 浮点数

float(),double(),decimal()；可以指定列宽。即double(10,5)。不含小数点。

### 字符串

varchar()，char()。

varchar允许可变长存储，去除末尾空格。

char不允许超长，会保留末尾空格。

### 日期

datetime，timestamp。

datetime使用8字节存储，单位为秒。存储范围1001-9999。与时区无关。

timestamp使用4字节存储，单位为秒。存储范围1970-2038。与时区有关，能自动转换

### 大文本

blob，text。

## 存储引擎

### InnoDB

> 聚簇索引，在索引中包含数据。聚簇索引只有一份，因为数据不可能被存储到不同地方。

支持事务<br>支持在线热备份<br>支持行级锁<br>支持外键

### 索引

> MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。

Mysql索引是一种数据结构，采用的是B+树，是一个多路平衡查找树。特点是查找时间较短且平均，能够支持范围查找和排序。并且其每个页

在索引中包含数据。聚簇索引只有一份，因为数据不可能被存储到不同地方

## MySQL日志

### undo log

对于读取出来并将要修改的数据写入到undo log 中，这个写入不是在内存中的

### redo log

> 属于数据库引擎特有

对事务的操作进行记录，不管事务是否提交。循环写入。这里的写redo日志，实际上也包括写bin log 日志。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/db-mysql-sql-12.png)

适用于主从复制过程&数据恢复中。在下次重启的时候 MySQL 也会将 redo 日志文件内容恢复到 Buffer Pool 中

### bin log 

> 简述，定性，用处

MySQL 在提交事务的时候，不仅仅会将 redo log buffer 中的数据写入到redo log 文件中，同时也会将本次修改的数据以追加的形式记录到 bin log文件中，同时会将本次修改的bin log文件名和修改的内容在bin log中的位置记录到redo log中，最后还会在redo log最后写入 commit 标记，这样就表示本次事务被成功的提交了。**这里需要考虑日志存储形式，刷入磁盘方式**

是逻辑上的操作过程记录。属于MySQL层面，任何存储引擎都可使用该日志。

适用于主从复制过程&数据恢复中。在下次重启的时候 MySQL 也会将 redo 日志文件内容恢复到 Buffer Pool 中



## MySQL MVCC

> MVCC：Mutil-version concurrency control
>
> 明确下冲突有哪些：快照读-写冲突，当前读-写冲突，写-写冲突

首先明确下数据库存在哪些冲突

- 读-读：不存在冲突
- 读-写：存在脏读，不可重复读，幻读的问题
- 写-写：存在更新丢失的问题，即第一类更新丢失，第二类回滚更新丢失

MVCC对于以上解决了读-写冲突。可以提供可并发非阻塞读

### MVCC实现原理

> 为事务分配单向递增的时间戳，并为每次修改保存版本。读操作只读该事务最近的开始前的快照。这样对于读写即可分离。

#### MVCC 生成过程

总的来说，MVCC是通过隐藏字段指向undo log中数据。其过程是将行数据读取到buffer中，同步到undo log中，修改完成后，将当前记录隐藏字段指针指向undo log日志中。这样就形成链式快照。后台可能存在purge线程，删除undo log。

读时生成Read View时， Read View主要是用来做可见性判断的, 即当我们某个事务执行快照读的时候，对该记录创建一个Read View读视图，把它比作条件用来判断当前事务能够看到哪个版本的数据，即可能是当前最新的数据，也有可能是该行记录的undo log里面的某个版本的数据

#### MVCC 读取过程

> MVCC支持快照读时一方面是通过保存当前事务ID，另一方面是通过隐藏字段和undo log
>
> 通过隐藏字段DB_TRX_ID进行判断当前事务对此条记录是否可见，不可见，递归按指针前溯。

Read View遵循一个可见性算法，主要是将要被修改的数据的最新记录中的DB_TRX_ID（即当前事务ID）取出来，与系统当前其他活跃事务的ID去对比（由Read View维护），如果DB_TRX_ID跟Read View的属性做了某些比较，不符合可见性，那就通过DB_ROLL_PTR回滚指针去取出Undo Log中的DB_TRX_ID再比较，即遍历链表的DB_TRX_ID（从链首到链尾，即从最近的一次修改查起），直到找到满足特定条件的DB_TRX_ID, 那么这个DB_TRX_ID所在的旧记录就是当前事务能看见的最新老版本

可见与不可见判断逻辑：

> 记录的事务ID比我现在活跃的都大，不可见。比我活跃的都小，可见。如果在活跃线程中，不可见。

> **trx_list** 未提交事务ID列表，用来维护Read View生成时刻系统正活跃的事务ID
>
> **up_limit_id** 记录trx_list列表中事务ID最小的ID
>
> **low_limit_id** ReadView生成时刻系统尚未分配的下一个事务ID，也就是目前已出现过的事务ID的最大值+1

- 首先比较DB_TRX_ID < up_limit_id, 如果小于，则当前事务能看到DB_TRX_ID 所在的记录，如果大于等于进入下一个判断
- 接下来判断 DB_TRX_ID 大于等于 low_limit_id , 如果大于等于则代表DB_TRX_ID 所在的记录在Read View生成后才出现的，那对当前事务肯定不可见，如果小于则进入下一个判断
- 判断DB_TRX_ID 是否在活跃事务之中，trx_list.contains(DB_TRX_ID)，如果在，则代表我Read View生成时刻，你这个事务还在活跃，还没有Commit，你修改的数据，我当前事务也是看不见的；如果不在，则说明，你这个事务在Read View生成之前就已经Commit了，你修改的结果，我当前事务是能看见的

#### MVCC 注意事项

MVCC 在RC级别实现了RR，但快照读的结果十分依赖于快照读开始的时机。RC级别下，每次发生快照读时均会独立生成快照读

MVCC 在RR级别下，快照读的依据会是第一次发生快照读的位置

## SQL执行过程

> 数据库驱动建立应用程序与数据库的链接->查询解析器->查询优化器->存储引擎的执行器

#### 连接：

本地应用程序通过Durid维护通过数据库驱动创建的与数据库的连接，相应的数据库也有一个连接池以供客户端连接。这些连接都是线程维护处理。线程的建立连接之后，就会交给SQL接口，进行解析与优化

#### 解析：略

优化：按照他的方式进行优化，并生成执行计划。主要考虑是选择索引，即通过IO成本和CPU成本最小的索引给定执行计划。调用存储引擎接口。

#### 存储引擎-执行器

> 准备更新一条 SQL 语句
>
> MySQL（innodb）会先去缓冲池（BufferPool）中去查找这条数据，没找到就会去磁盘中查找，如果查找到就会将这条数据加载到缓冲池（BufferPool）中
>
> 在加载到 Buffer Pool 的同时，会将这条数据的原始记录保存到 undo 日志文件中
>
> innodb 会在 Buffer Pool 中执行更新操作
>
> 更新后的数据会记录在 redo log buffer 中

Buffer Pool

undo log

redo log 





## MySQL优化

> 主从复制：从机订阅主机的bin log 日志，增量读取存入从机的中继日志中，然后重放日志内。完成主从复制
>
> 读写分离：主机分则写操作及一些高实时性要求操作，从机负责读操作

### 数据库层面

通过explain，分析查询过程，查看索引覆盖情况。观察select_type, key,rows等关键指标

通过分库分表的方式

### 单次查询层面

避免使用select*，尽量使用limit，尽量通过索引。

### 复杂查询层面

查分多次小查询（有效利用缓存，避免阻塞）方便日后扩展。

# MySQL语法

## alter

- 数据库表字段的增删查改，[**MySQL修改字段名、修改字段类型**](https://blog.csdn.net/u010002184/article/details/79354136)

## MySQL on duplicate key update

> 参考[***博文***](https://www.cnblogs.com/better-farther-world2099/articles/11737376.html)

1. 语句含义，无则插入，有则更新，update后直接跟条件即可
2. 自增主键不会连续自增
3. 会产生DeathLock的问题
4. 替代方案就是先进行update，根据结果进行判断是否进行insert

## last_insert_id

> 使用[***last_insert_id***](https://blog.csdn.net/slvher/article/details/42298355)注意事项

small squirrel good night 



# MySQL常见异常

## 1093

​	mysql出现You can’t specify target table for update in FROM clause 这个错误的意思是不能在同一个sql语句中，先select同一个表的某些值，然后再update这个表。**select的结果再通过一个中间表select多一次，就可以避免这个错误**



# MySQL 锁

> mysql 中对表中的数据操作四种：select, update, insert, delete。分为俩类：一是查询，二是更新。命令作用在数据库上就存在俩种读法：快照读和当前读。多个操作当被视作事务时，就会导致同一事务中快照读升级为当前读，进而导致幻读的发生

## 更新丢失

1. 在非序列化过程中

## MySQL MVCC&幻读

> [原文地址-Java码农](https://www.jianshu.com/p/b7c53ee0ed0e)

MVCC是分为当前读和快照读<br>**针对快照读**（普通 select 语句），是**通过 MVCC 方式解决了幻读**，因为可重复读隔离级别下，事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，即使中途有其他事务插入了一条数据，是查询不出来这条数据的，所以就很好了避免幻读问题。<br>**针对当前读**（select ... for update 等语句），是**通过 next-key lock（记录锁+间隙锁）方式解决了幻读**，因为当执行 select ... for update 语句的时候，会加上 next-key lock，如果有其他事务在 next-key lock 锁范围内插入了一条记录，那么这个插入语句就会被阻塞，无法成功插入，所以就很好了避免幻读问题。

第一个例子：对于快照读， MVCC 并不能完全避免幻读现象。因为当事务 A 更新了一条事务 B 插入的记录，那么事务 A 前后两次查询的记录条目就不一样了，所以就发生幻读。

第二个例子：对于当前读，如果事务开启后，并没有执行当前读，而是先快照读，然后这期间如果其他事务插入了一条记录，那么事务后续使用当前读进行查询的时候，就会发现两次查询的记录条目就不一样了，所以就发生幻读。

所以，**MySQL 可重复读隔离级别并没有彻底解决幻读，只是很大程度上避免了幻读现象的发生。**

要避免这类特殊场景下发生幻读的现象的话，就是尽量在开启事务之后，马上执行 select ... for update 这类当前读的语句，因为它会对记录加 next-key lock，从而避免其他事务插入一条新记录。



