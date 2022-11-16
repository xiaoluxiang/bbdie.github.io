# MySQL数据类型

## 整数

tinyint,smallint,middleint,int,bigint；占用的字节数分别为4，8，16，32，64。对于int(10)，对于实际存储占用没有效果，只是为了交互显示所用。

## 浮点数

float(),double(),decimal()；可以指定列宽。即double(10,5)。不含小数点。

## 字符串

varchar()，char()。

varchar允许可变长存储，去除末尾空格。

char不允许超长，会保留末尾空格。

## 日期

datetime，timestamp。

datetime使用8字节存储，单位为秒。存储范围1001-9999。与时区无关。

timestamp使用4字节存储，单位为秒。存储范围1970-2038。与时区有关，能自动转换

## 大文本

blob，text。

# 存储引擎

## InnoDB

> 聚簇索引，在索引中包含数据

支持事务<br>支持在线热备份<br>支持行级锁<br>支持外键

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



