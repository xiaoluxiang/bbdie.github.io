# Mysql on duplicate key update用法及优缺点 

> 参考[***博文***](https://www.cnblogs.com/better-farther-world2099/articles/11737376.html)

1. 语句含义，无则插入，有则更新，update后直接跟条件即可
2. 自增主键不会连续自增
3. 会产生DeathLock的问题
4. 替代方案就是先进行update，根据结果进行判断是否进行insert

# last_insert_id()函数使用的注意事项

> 使用[***last_insert_id***](https://blog.csdn.net/slvher/article/details/42298355)注意事项

small squirrel good night 

# MySQL常见异常错误

## 1093

​	mysql出现You can’t specify target table for update in FROM clause 这个错误的意思是不能在同一个sql语句中，先select同一个表的某些值，然后再update这个表。**select的结果再通过一个中间表select多一次，就可以避免这个错误**