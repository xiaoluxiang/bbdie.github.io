# 编程思想

1. 循环的是实现方式可以通过递归也可以通过for/while实现。例如数据结构中树的定义。<br>在递归过程中，一般需要先进行递归条件判断，然后再对当前条件计算判断。<br>在循环过程中，退出条件的设定是很关键的。一定要覆盖，避免死递归

   ```java
   /*
    * (递归实现)查找"二叉树x"中键值为key的节点
    */
   private BSTNode<T> search(BSTNode<T> x, T key) {
       if (x==null)
           return x;
   
       int cmp = key.compareTo(x.key);
       if (cmp < 0)
           return search(x.left, key);
       else if (cmp > 0)
           return search(x.right, key);
       else
           return x;
   }
   
   public BSTNode<T> search(T key) {
       return search(mRoot, key);
   }
   
   ```

2. 移位操作完成运算

   对于一些实时性要求非常高的功能，使用移位，与，或，掩码。

3. 

# 事件思想

文件读写导出（Redis文件 文件导出）

- 文件名称，路径
- 文件导出内容格式（是否压缩）
- 多次重复频繁写，策略
- 文件导出过程并发问题
- 文件异常处理



# 概念思想

1. 服务冗余设计

   - 服务提供冗余-高可用。

     正常的负载均衡，读写分离

     不正常的主从切换。灾备

   - 服务保障冗余-高可靠

2. 数据冗余设计

   - 数据一致性问题
   - 数据同步问题
   - 数据源切换问题

# 设计思想

队列节点的设计

1. 内部类，通过指针/引用方式使用