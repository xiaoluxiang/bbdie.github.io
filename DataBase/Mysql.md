# SQL数据类型

## 整数

tinyint,smallint,middleint,int,bigint；占用的字节数分别为4，8，16，32，64。对于int(10)，对于实际存储占用没有效果，只是为了交互显示所用。

## 浮点数

float(),double(),decimal()；可以指定列宽。即double(10,5)。不含小数点。

## 字符串

varchar()，char()。blob，text

varchar允许可变长存储，去除末尾空格。
char不允许超长，会保留末尾空格。
blog：二进制存储，没有规则顺序；
text：字符串存储，有排序规则；

## 日期

datetime，timestamp。

datetime使用8字节存储，单位为秒。存储范围1001-9999。与时区无关。
timestamp使用4字节存储，单位为秒。存储范围1970-2038。与时区有关，能自动转换
YYYY: week year；yyyy:normal year；MM：月份； d：月份中的天数；D：年份中的天数；H：24小时；h：12小时；m：分钟；s：秒；S：毫秒；

## 真值表

and or / true, false, null;

**and**: false> null > true;
**or**: true > null > false;

其他对于null的相等不等判断均为null；

# 存储引擎

## 技术类型

### MyISAM

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/249993-20170605164152387-2112745537.png)

### INNODB

InnoDB的数据文件本身就是索引文件,叶节点包含了完整的数据记录。这种索引叫做聚集索引

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/249993-20170606162710418-2028069737.png)

支持事务<br>支持在线热备份<br>支持行级锁<br>支持外键

## 区别和联系

一是主索引的区别，InnoDB的数据文件本身就是索引文件。而MyISAM的索引和数据是分开的。

二是辅助索引的区别：InnoDB的辅助索引data域存储相应记录主键的值而不是地址。而MyISAM的辅助索引和主索引没有多大区别。

InnoDB的主索引文件上，直接存放该行数据，称为聚簇索引。非聚簇索引指向对主键的引用。

Myisam中，主索引和次索引都指向物理行。

补充：索引覆盖

索引覆盖是指如果查询的列恰好是索引的一部分，那么查询只需要在索引文件上进行，不需要回行到磁盘再找数据。这种查询速度非常快，称为“索引覆盖”。

# SQL执行过程

> 数据库驱动建立应用程序与数据库的链接->查询解析器->查询优化器->存储引擎的执行器

## 连接：

本地应用程序通过Durid维护通过数据库驱动创建的与数据库的连接，相应的数据库也有一个连接池以供客户端连接。这些连接都是线程维护处理。线程的建立连接之后，就会交给SQL接口，进行解析与优化

## 解析：略

优化：按照他的方式进行优化，并生成执行计划。主要考虑是选择索引，即通过IO成本和CPU成本最小的索引给定执行计划。调用存储引擎接口。

## 存储引擎-执行器

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

# MySQL

> MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。
>
> 一般某种算法都会对应一种或多种数据结构。MySQL的索引就是实现MySQL高速获取数据的数据结构

MySQL索引是一种数据结构，采用的是B+树，是一个多路平衡查找树。其每一个节点可对应着物理存储的页。特点是查找时间较短且平均，能够支持范围查找和排序。

Null对索引使用上的影响

## MySQL语法

> alter参考地址：[MySQL修改字段名、修改字段类型](https://blog.csdn.net/u010002184/article/details/79354136)

### 基本语法

**显示数据库信息**

```sql
use XXdatabases;
show databases; show tables; show variable like '';
-- 显示表信息
show create table tableName;show index from XX;
```

**数据表及表结构的增删改查**

```mysql
--  表的增删改查，需要加table
create table [if not exists] XX(,,,)comment '';
drop table XX;

-- 表名，表索引，表字段
-- alter表->索引->字段增改删(add, change, modify, drop)
alter table XX rename to newTableName;

alter table XX add Index indexName(lineName);
alter table XX add column columnName...;
alter table XX add column columnName... [[before][after]] columnName;
alter table XX modify column columnName ...;
alter table XX change column oldColumnName newColunmName...;

alter table XX drop index indexName;
alter table XX drop column columnName;
```

**表记录的增删改查**

```mysql
-- 表内容的增删改查
select, update, insert, delete,truncate;

-- 算数运算符：
-- 逻辑运算符：and or in exists like [not in  not exists]
-- group by和having是组合使用的
select [distinct] XX from XX [join on] where [算数运算符] [逻辑运算符] group having order by limit [union];

update XX set XX where XX;
update a,b set a.XX = b.XX where a.xx = b.xx;

-- join详解，参照数学上的交集，数据的横向补长，union会纵向补充
-- join详解，参照数学上的交集
select a.XX, b.XX from a [left join, inner join, right join] b on a.XX = b.XX;
```

**特殊SQL**

```mysql
/*
插入时判断是否增长，参考[***博文***](https://www.cnblogs.com/better-farther-world2099/articles/11737376.html)
1. 自增主键不会连续自增
2. 会产生DeathLock的问题
3. 替代方案就是先进行update，根据结果进行判断是否进行insert
*/
insert into XX () values() on duplicate key update XX;


[***last_insert_id***](https://blog.csdn.net/slvher/article/details/42298355)
```



## 索引

> MySQL的聚集索引：主索引文件和数据文件是同一份文件，所以数据一般不会被存储到其他地方。一般索引来说他的data域会指向对应的主索引

### 背景

> 参考内容：[扇区、磁盘块、页。磁盘是如何存储数据的：磁盘的物理结构](https://blog.nowcoder.net/n/eabf311f51794625b1fd87d392c4f45d)

扇区：磁盘上的磁道被等份若干弧道，弧道的区域被称为扇区，是物理读写的基本单位，并且这是磁盘物理层面的概念

磁盘块：磁盘上若干相邻扇区组成的磁盘块，IO的基本单位（一个磁盘块只有一个文件），是操作系统操作磁盘的逻辑概念

页：由磁盘的的2^n个磁盘块组成，内存进行操作的基本单位

### 技术选型

红黑树：数据结构的深度比较深，查找效率随数据量的上升而显著下降。逻辑上相邻的数据物理上可能不在一处，无法使用局部性原理。渐进复杂度为O(h).

AVL树：需要保持严格平衡，数据插入删除修改操作量大。并且查找效率随数据量的上升而显著下降

B-tree（平衡多路查找树）：B-Tree相对于AVLTree缩减了节点个数，使每次磁盘I/O取到内存的数据都发挥了作用，从而提高了查询效率

B+tree：因为非叶子节点上也存data，这会导致非叶节点上存储key减少，实际存储数据能显著影响查询次数。所以，非叶子节点只存储key值信息。所有的叶节点上都有前后指针。所有数据均存在对应的叶子节点上。

局部性原理指的是当一个数据被用到，其附近的数据也通常也被用道。因此磁盘读取数据时会顺序的往后读取一段数据。

### 技术实现

B+tree结构模型

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/249993-20170531161139243-491884410.png)

在索引中包含数据。聚簇索引只有一份，因为数据不可能被存储到不同地方

### 性能分析

B+ tree通过将节点大小设置为页大小，读取和存储的时候只需要一次IO，还可以利用局部性原理。由于非叶子节点上不存数据，树的出度一般会比较大，对于渐进复杂度logdN。一般不会超过三次，非常可观。

### 课题评价&未来展望

评价一个数据结构作为索引的优劣最重要的指标就是在查找过程中磁盘I/O操作次数的渐进复杂度。换句话说，索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数。在此B+tree属实是一个不错的选择

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
> 为事务分配单向递增的时间戳，并为每次修改保存版本。读操作只读该事务最近的开始前的快照。这样对于读写即可分离
>
> 明确下冲突有哪些：快照读-写冲突，当前读-写冲突，写-写冲突

首先明确下数据库存在哪些冲突

- 读-读：不存在冲突
- 读-写：存在脏读，不可重复读，幻读的问题
- 写-写：存在更新丢失的问题，即第一类更新丢失，第二类回滚更新丢失

MVCC对于以上解决了读-写冲突。可以提供可并发非阻塞读

### MVCC 生成过程

总的来说，MVCC是通过隐藏字段指向undo log中数据。其过程是将行数据读取到buffer中，同步到undo log中，修改完成后，将当前记录隐藏字段指针指向undo log日志中。这样就形成链式快照。后台可能存在purge线程，删除undo log。

读时生成Read View时， Read View主要是用来做可见性判断的, 即当我们某个事务执行快照读的时候，对该记录创建一个Read View读视图，把它比作条件用来判断当前事务能够看到哪个版本的数据，即可能是当前最新的数据，也有可能是该行记录的undo log里面的某个版本的数据

### MVCC 读取过程

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

### MVCC 注意事项

MVCC 在RC级别实现了RR，但快照读的结果十分依赖于快照读开始的时机。RC级别下，每次发生快照读时均会独立生成快照读

MVCC 在RR级别下，快照读的依据会是第一次发生快照读的位置

### MySQL MVCC&幻读

> mysql 中对表中的数据操作四种：select, update, insert, delete。分为俩类：一是查询，二是更新。命令作用在数据库上就存在俩种读法：快照读和当前读。多个操作当被视作事务时，就会导致同一事务中快照读升级为当前读，进而导致幻读的发生
>
> [原文地址-Java码农](https://www.jianshu.com/p/b7c53ee0ed0e)

**MVCC原理**

可重复读隔离级是由 MVCC（多版本并发控制）实现的，实现的方式是启动事务后，在执行第一个查询语句后，会创建一个 Read View，**后续的查询语句利用这个 Read View，通过这个  Read View 就可以在 undo log 版本链找到事务开始时的数据，所以事务过程中每次查询的数据都是一样的**，即使中途有其他事务插入了新纪录，是查询不出来这条数据的，所以就很好了避免幻读问题。

事务 A 执行了这条当前读语句后，就在对表中的记录加上 id 范围为 (2, +∞] 的 next-key lock（next-key lock 是间隙锁+记录锁的组合）。然后，事务 B 在执行插入语句的时候，判断到插入的位置被事务 A 加了  next-key lock，于是事务 B 会生成一个插入意向锁，同时进入等待状态，直到事务 A 提交了事务。这就避免了由于事务 B 插入新记录而导致事务 A 发生幻读的现象

MVCC是分为当前读和快照读<br>**针对快照读**（普通 select 语句），是**通过 MVCC 方式解决了幻读**，因为可重复读隔离级别下，事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，即使中途有其他事务插入了一条数据，是查询不出来这条数据的，所以就很好了避免幻读问题。<br>**针对当前读**（select ... for update 等语句），是**通过 next-key lock（记录锁+间隙锁）方式解决了幻读**，因为当执行 select ... for update 语句的时候，会加上 next-key lock，如果有其他事务在 next-key lock 锁范围内插入了一条记录，那么这个插入语句就会被阻塞，无法成功插入，所以就很好了避免幻读问题。

第一个例子：对于快照读， MVCC 并不能完全避免幻读现象。因为当事务 A 更新了一条事务 B 插入的记录，那么事务 A 前后两次查询的记录条目就不一样了，所以就发生幻读。

第二个例子：对于当前读，如果事务开启后，并没有执行当前读，而是先快照读，然后这期间如果其他事务插入了一条记录，那么事务后续使用当前读进行查询的时候，就会发现两次查询的记录条目就不一样了，所以就发生幻读。

所以，**MySQL 可重复读隔离级别并没有彻底解决幻读，只是很大程度上避免了幻读现象的发生。**

要避免这类特殊场景下发生幻读的现象的话，就是尽量在开启事务之后，马上执行 select ... for update 这类当前读的语句，因为它会对记录加 next-key lock，从而避免其他事务插入一条新记录。

## MySQL优化

> - 数据库schema设计优化
> - SQL查询优化
> - 锁策略优化
> - 存储引擎优化
> - 服务器配置优化
> - 主从，读写分离
> - 集群，负载均衡

### 数据库层面

表结构字段设计的是否合理；

分库分表；

通过explain，分析查询过程，查看索引覆盖情况。观察select_type, key,rows等关键指标

- 查询优化https://www.cnblogs.com/myseries/p/11262054.html

### 控制层面

**单次查询层面**

避免使用select*，尽量通过索引。避免索引失效。

- 避免索引失效：
  - 不在条件里的索引做运算
  - 违反最左前缀原则
  - 索引范围条件右边的列
  - 使用不等!=或 <>
  - or：如果存在部分，则不查询。还得看索引优化器，其再看统计信息；
  - in：如果数据库数据量少在伴随着回表，或者in的情况太多；
  - 索引类型隐形转换，例字符串不适用单引号，参考地址：https://zhuanlan.zhihu.com/p/30955365

**复杂查询层面**

查分多次小查询（有效利用缓存，避免阻塞）方便日后扩展。

提高效率

- 避免使用子查询
- 避免使用order by，limit，区分in和exists
- 批量插入

### 架构设计上

ES

## MySQL集群

> 主从复制：从机订阅主机的bin log 日志，增量读取存入从机的中继日志中，然后重放日志内。完成主从复制
>
> 读写分离：主机负责写操作及一些高实时性要求操作，从机负责读操作





**大量的关注**/粉丝设计

设计表结构，关注了就增加关注与被关注记录。此时考虑的单条记录的其他属性

水平拆分->按主键取模，确定表

垂直拆分->分为关注表与被关注表

考虑业务场景下的优化，分数数量直接成为字段之一，业务上禁止查询。

**对于很多的待做的事情**

专门的数据库表来记录，然后定时处理

MQ的定时消息

**秒杀**

动静分离，将静态资源部署到CDN上。极少数允许动态请求

页面按钮先置为灰色

读多写少，使用Redis作预先缓存。存在缓存击穿的问题（建议先预热）。存在缓存穿透，大量的直接穿透访问。缓存雪崩，缓存大量同时过期。在Redis之前再来一份分布式锁大量请求数据库 。->自旋锁

库存不足和库存超卖和回退库存

下单等利用MQ解耦，解决消息丢失（落表，等生产者来写），消息重复（落表，回写生产者），垃圾消息（重试次数）。

限流，nginx（IP，用户），验证码，接口限流。调整业务规则

**回调函数**：把函数的指针（地址）作为参数传递给另一个函数，当这个指针调用其所指向的函数时，就称这是回调函数。

**多态：**同一段代码执行时却表现出不同的行为状态。简单的说，可以理解为允许将不同的子类类型的对象动态赋值给父类类型的变量，通过父类的变量调用方法在执行时实际执行的可能是不同的子类对象方法，因而表现出不同的执行效果。

**闭包**：函数和对其周围状态的引用捆绑在一起构成闭包。也就是说，闭包可以让你从内部函数访问外部函数作用域。

**异步调用**：promise将异步调用以同步的流程表达出来，避免嵌套回调函数，简化了回调函数传入的接口实现。

**匿名函数**：lamda函数在常见的命令式编程语言中以匿名函数的形式出现，比如无参数的代码块或者箭头函数
