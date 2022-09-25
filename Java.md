# 代理

1. 静态代理实现较简单，只要代理对象对目标对象进行包装，即可实现增强功能，但静态代理只能为一个目标对象服务，如果目标对象过多，则会产生很多代理类。
2. JDK动态代理需要目标对象实现业务接口，代理类只需实现InvocationHandler接口。
3. 动态代理生成的类为 lass com.sun.proxy.\$Proxy4，cglib代理生成的类为class com.cglib.UserDao\$\$EnhancerByCGLIB\$\$552188b6。
4. 静态代理在编译时产生class字节码文件，可以直接使用，效率高。
5. 动态代理必须实现InvocationHandler接口，通过反射代理方法，比较消耗系统性能，但可以减少代理类的数量，使用更灵活。
6. cglib代理无需实现接口，通过生成类字节码实现代理，比反射稍快，不存在性能问题，但cglib会继承目标对象，需要重写方法，所以目标对象不能为final类。



# BIO、NIO、AIO、IO多路复用、信号驱动IO

作者：云飞扬°
链接：https://www.nowcoder.com/discuss/820703
来源：牛客网

***BIO（Blocking IO）***<br>	阻塞型IO，用户线程发起IO请求后，必须等待内核线程准备好数据，并且将数据拷贝到用户线程存储空间，才会返回。期间用户线程处于阻塞状态，释放CPU资源。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/1639358986184-e10c4719-aa95-4093-92cc-b868160df3d7.png)

***NIO（nonblocking IO）***<br>	非阻塞型IO，发出IO操作后，不进入阻塞状态，不放弃CPU资源，开始轮询内核数据准备情况。内核准备好且收到系统调用，就将输出拷贝到用户线程。并返回

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/1639359034221-3ddfaad7-1378-4de1-8aa4-6f75a16f6364-20220904150804563.png)

***AIO (异步IO Asynchronous IO)***

​	异步IO，发出IO请求之后，就会继续做其他的事情，内核收到后准备数据并将数据拷贝到用户线程存储空间中，然后通知用户线程。用户线程直接使用

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/1639359071332-7ebcbc03-c25a-4ab6-876a-35aa0b1343be.png)

***IO多路复用 和 信号驱动IO***

1. IO多路复用<br>	在多路复用IO模型中，会有一个线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。<br>	select：轮询所有的socket <br>	poll：轮询所有的socket，但是和select相比，没有最大连接数的限制，原因是它是基于[链表]()存储的 <br>	epoll：只会轮询发生了IO请求的socket。虽然连接数有上限，但是很大，1G内存的机器上可以打开10W左右的连接；
2. I/O 多路复用模型是利用select、poll、epoll可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有I/O事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll是只轮询那些真正发出了事件的流），依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。

***信号驱动IO***（signal driven IO） 

1. 在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个**信号函数**，然后用户线程会继续执行，当内核[数据]()就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在**信号函数**中调用IO读写操作来进行实际的IO请求操作。 

2.  应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。 

3. 内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。信号驱动 I/O 的 CPU 利用率很高。 

***异步 IO 与信号驱动 IO***

 异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

# HashMap

参考文章地址：https://www.nowcoder.com/discuss/820700

***数据结构***

JDK1.7之前是数组加链表<br>JDK1.8之后是数组+链表+红黑树

***put/get的过程***

put<br>1. 判断数组是否为空，为空则创建数组。<br>2. 计算key值（key.hashCode()&(length-1)），定位到数组位置，<br>3. 判断该位置是否为null，为null，则设置该值，不为null，则通过hashcode和equals进行比较，相同更新，不同则判断该点是否为红黑树节点，是则直接插入，不是则链表插入，<br>4. 链表插入完成之后，当前链表长度大于8且数组长度大于64则转为红黑树。链表长度大于8且数组长度小于64则进行数组扩容。<br>5. 最后当插入操作完成后，进行数组整体容量和负载因子判断，是否需要进行扩容。

get：<br>1. 对于给定key的hashcode值定位数组位置，如果为null，返回null<br>2. 不为null，判断是否为红黑树节点，是则按照红黑树查找，不是则按照链表查找

***HashMap的扩容机制***

1. 初始16，容量是2的n次方，负载因子默认为0.75。解释以上三者原因<br>时间和空间上需要权衡，太小会频繁扩容，太大浪费空件，而且16是2的幂次方<br>计算位置时是通过hashcode&(n-1)，n为2的幂次方可以确保n-1值的全为1，避免不必要的hash冲突的发生<br>当为1的时候，不会扩容。为0.5的时候，虽然可能减少hash冲突，但是会频繁扩容，而且空间利用率低

2. HashMap扩容过程

   > 伯努利试验（同样的条件下重复地、相互独立地进行的一种随机试验），试验次数是否很大，p是否很小即可
   >
   > 二项分布趋于泊松分布，用泊松分布的概率值作二项分布概率值的近似

3. HashMap为什么线程不安全

   > 更新丢失会有俩种，撤销回滚更新丢失，覆盖更新丢失

   1. 1.7中采用头插法会造成死锁和数据丢失
   2. 1.8采用尾插法，避免死锁，数据更新丢失在多线程仍无法避免

4. HashMap如何解决Hash冲突

   拉链法（链地址法），采用单向链表，链表长度大于等于8可能转为红黑树，链表长度所见缩减小于6，会转为链表

5. HashMap为什么使用[红黑树](https://www.nowcoder.com/jump/super-jump/word?word=红黑树)而不是B树或[平衡二叉树](https://www.nowcoder.com/jump/super-jump/word?word=平衡二叉树)AVL或二叉查找树

   1.不使用二叉查找树

   二叉[排序](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=排序)树在极端情况下会出现线性结构。例如：二叉[排序](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=排序)树左子树所有节点的值均小于根节点，如果我们添加的元素都比根节点小，会导致左子树线性增长，这样就失去了用树型结构替换[链表](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=链表)的初衷，导致查询时间增长。所以这是不用二叉查找树的原因。 

   2.不使用平衡二叉树 

   平衡二叉树是严格的平衡树，红黑树是不严格平衡的树，[平衡二叉树](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=平衡二叉树)在插入或删除后维持平衡的开销要大于[红黑树](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=红黑树)。 

   [红黑树](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=红黑树)的虽然查询性能略低于[平衡二叉树](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=平衡二叉树)，但在插入和删除上性能要优于[平衡二叉树](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=平衡二叉树)。 

   选择[红黑树](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=红黑树)是从功能、性能和开销上综合选择的结果。 

   3.不使用B树/B+树

   HashMap本来是数组+[链表](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=链表)的形式，[链表](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=链表)由于其查找慢的特点，所以需要被查找效率更高的树结构来替换。 

   如果用B/B+树的话，在[数据](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=数据)量不是很多的情况下，[数据](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=数据)都会“挤在”一个结点里面，这个时候遍历效率就退化成了[链表](applewebdata://51410658-CE82-4A58-BD95-062F998FE491/jump/super-jump/word?word=链表)。

# RPC

***RPC (remote process call) 远程过程调用***

1. 双方持有相同接口，调用方通过本地JDK代理实现提供实现类
2. 本地实现类通过对象序列化，将执行参数值，参数属性，接口，方法传递给远程服务线程上（传递方式，socket，http，很多自定义协议都可）
3. 远程服务线程反序列化，拿到对象，然后执行方法。对返回值进行序列化，传回给调用方
4. 本地代理对象拿到返回，反序列化，返回完成。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-54c36e07764895d3da67c7fc624789c5_720w.jpg)

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-d690accc669d726fe122d6da6caa75a1_720w.jpg)

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-c0088ff8964a97f232081b5b2a08c068_720w.png)

# Lambda表达式、函数式接口

1. 函数式接口，通过接口实现匿名类，我们可以实现类的一些属性

# NIO

sockets是面向流的而非包导向的。它们可以保证发送的字节会按照顺序到达但无法承诺维持字节分组

# JC

> 性能优化解决方案总体思路：1. 单体应用业务逻辑优化，操作读写方式优化。2. 单体应用JVM优化
>
> [CMS 垃圾回收机制](https://www.cnblogs.com/Leo_wl/p/5393300.html)

### 案例一 Major GC和Minor GC频繁

> [美团GC优化参考](https://tech.meituan.com/2017/12/29/jvm-optimize.html)

**动态年龄计算**：Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值。在本案例中，调优前：Survivor区 = 64M，desired survivor = 32M，此时Survivor区中age<=2的对象累计大小为41M，41M大于32M，所以晋升年龄阈值被设置为2，下次Minor GC时将年龄超过2的对象被晋升到老年代。

下图展示了CMS各个阶段可以标记的对象，用不同颜色区分。 <br>1. Init-mark初始标记(STW) ，该阶段进行可达性分析，标记GC ROOT能直接关联到的对象，所以很快。<br> 2. Concurrent-mark并发标记，由前阶段标记过的绿色对象出发，所有可到达的对象都在本阶段中标记。 <br>3. Remark重标记(STW) ，暂停所有用户线程，重新扫描堆中的对象，进行可达性分析，标记活着的对象。因为并发标记阶段是和用户线程并发执行的过程，所以该过程中可能有用户线程修改某些活跃对象的字段，指向了一个未标记过的对象，如下图中红色对象在并发标记开始时不可达，但是并行期间引用发生变化，变为对象可达，这个阶段需要重新标记出此类对象，防止在下一阶段被清理掉，这个过程也是需要STW的。特别需要注意一点，这个阶段是以新生代中对象为根来判断对象是否存活的。 <br>4. 并发清理，进行并发的垃圾清理。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/09cf176b.png)

新生代对象持有老年代中对象的引用，这种情况称为**“跨代引用”**。因它的存在，Remark阶段必须扫描整个堆来判断对象是否存活，