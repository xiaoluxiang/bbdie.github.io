# Java

## Basic Lang

### 内部类

> 参考地址：https://www.cnblogs.com/GrimMjx/p/10105626.html#_label2_1

​	非静态内部类：其实例不可单独存在，不可含有静态方法。通过外部类的所持有其构造方法进行创建。<br>该方法是否有该名字的成员变量 - 直接用该变量名，<br>内部类中是否有该名字的成员变量 - 使用this.变量名。<br>外部类中是否有该名字的成员变量 - 使用外部类的类名.this.变量名<br>	静态内部类不会持有外部类的对象，只是借壳，变量访问规则参考静态变量。

### Switch

Switch基本用法。略。可接受对象会有很多，

switch 对于不同case，会有break用法。没有break会一直匹配下去。

不要忘记default:

### 代理

1. 静态代理实现较简单，只要代理对象对目标对象进行包装，即可实现增强功能，但静态代理只能为一个目标对象服务，如果目标对象过多，则会产生很多代理类。
2. JDK动态代理需要目标对象实现业务接口，代理类只需实现InvocationHandler接口。
3. 动态代理生成的类为 lass com.sun.proxy.\$Proxy4，cglib代理生成的类为class com.cglib.UserDao\$\$EnhancerByCGLIB\$\$552188b6。
4. 静态代理在编译时产生class字节码文件，可以直接使用，效率高。
5. 动态代理必须实现InvocationHandler接口，通过反射代理方法，比较消耗系统性能，但可以减少代理类的数量，使用更灵活。
6. cglib代理无需实现接口，通过生成类字节码实现代理，比反射稍快，不存在性能问题，但cglib会继承目标对象，需要重写方法，所以目标对象不能为final类。

### HashMap

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

   二叉[排序树在极端情况下会出现线性结构。例如：二叉排序树左子树所有节点的值均小于根节点，如果我们添加的元素都比根节点小，会导致左子树线性增长，这样就失去了用树型结构替换链表的初衷，导致查询时间增长。所以这是不用二叉查找树的原因。 

   2.不使用平衡二叉树 

   平衡二叉树是严格的平衡树，红黑树是不严格平衡的树，平衡二叉树在插入或删除后维持平衡的开销要大于红黑树。 

   红黑树的虽然查询性能略低于平衡二叉树但在插入和删除上性能要优于平衡二叉树。 

   选择红黑树是从功能、性能和开销上综合选择的结果。 

   3.不使用B树/B+树

   HashMap本来是数组+链表的形式，链表由于其查找慢的特点，所以需要被查找效率更高的树结构来替换。 

   如果用B/B+树的话，在数据量不是很多的情况下，数据都会“挤在”一个结点里面，这个时候遍历效率就退化成了链表。

   ```java
   /**
        * Implements Map.put and related methods.
        *
        * @param hash hash for key
        * @param key the key
        * @param value the value to put
        * @param onlyIfAbsent if true, don't change existing value
        * @param evict if false, the table is in creation mode.
        * @return previous value, or null if none
        */
       final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
           Node<K,V>[] tab; Node<K,V> p; int n, i;
         // 如果数组为空，创建数组
           if ((tab = table) == null || (n = tab.length) == 0)
               n = (tab = resize()).length;
         // 如果数组映射位置为空，直接等于替换
           if ((p = tab[i = (n - 1) & hash]) == null)
               tab[i] = newNode(hash, key, value, null);
           else {
               Node<K,V> e; K k;
             // 如果hash相同并且（key相同或者equal），直接替换
               if (p.hash == hash &&
                   ((k = p.key) == key || (key != null && key.equals(k))))
                   e = p;
               else if (p instanceof TreeNode)
                   e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
               else {
                   for (int binCount = 0; ; ++binCount) {
                       if ((e = p.next) == null) {
                           p.next = newNode(hash, key, value, null);
                           if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                               treeifyBin(tab, hash);
                           break;
                       }
                       if (e.hash == hash &&
                           ((k = e.key) == key || (key != null && key.equals(k))))
                           break;
                       p = e;
                   }
               }
               if (e != null) { // existing mapping for key
                   V oldValue = e.value;
                   if (!onlyIfAbsent || oldValue == null)
                       e.value = value;
                   afterNodeAccess(e);
                   return oldValue;
               }
           }
           ++modCount;
           if (++size > threshold)
               resize();
           afterNodeInsertion(evict);
           return null;
       }
   ```


### 日期

### SimpleDateFormate 线程不安全

多个线程之间共享变量calendar，并修改calendar。因此在多线程环境下，当多个线程同时使用相同的SimpleDateFormat对象（如static修饰）的话，如调用format方法时，多个线程会同时调用calender.setTime方法，导致time被别的线程修改，因此线程是不安全的。此外，parse方法也是线程不安全的，parse方法实际调用的是CalenderBuilder的establish来进行解析，其方法中主要步骤不是原子操作。

解决方案：1、将SimpleDateFormat定义成局部变量。2、 加一把线程同步锁：synchronized(lock)。3、使用ThreadLocal，每个线程都拥有自己的SimpleDateFormat对象副本。4、使用DateTimeFormatter代替SimpleDateFormat。 5、使用JDK8全新的日期和时间API

#### JDK 8 中的日期LocalDateTime, LocalDate, LocalTime.

> [LocalDateTime, LocalDate, LocalTime 关系](https://cloud.tencent.com/developer/article/1598040)

Export:<br>	LocalDateTime -> LocalDate/LocalTime. toLocalTime/toLocalDate.<br>	LocalDate/LocalTime -> LocalDateTime. of/at.

Reset:<br>	plus/plusXXXX:增加<br>	minus/minusXXXX:减少<br>	with/withXXXX:直接设置 

格式化:<br>	DateTimeFormatter:<br>		固定枚举:DateTimeFormatter.ISO_LOCAL_DATE<br>		自定义格式类型:DateTimeFormatter.ofPattern();<br>	Format: LocalDateTime.format(DateTimeFormatter);<br>	Parse:LocalDateTime.parse("",DateTimeFormatter);



## Feature

> what's the feature, what I think that is pretty fun, so that's 

### SPI

> spi即service provider interface

通过自定义规范（接口），主程序只需要按照规范进行API调用，Java支持按照依赖文件中的META-INF/services进行加载实现类。然后实现类完成API功能。

# IO 

> 背景就是应用程序向操作系统发起不同调用请求。操作系统作出不同响应，进而实现将内核中数据拷贝到应用程序中
>
> IO的理解方式，可以在数据的传输方式和操作方式俩个维度。Java具体实现中是通过装饰者模式

关于IO的基础定义

> 阻塞与非阻塞、同步与非同步、Linux的IO模型

## 网络IO的五种模型

>  BIO、NIO、AIO、IO多路复用、信号驱动IO

> 作者：云飞扬°
> 链接：https://www.nowcoder.com/discuss/820703
> 来源：牛客网

***BIO（Blocking IO）***<br>	阻塞型IO，用户线程发起IO请求后，必须等待内核线程准备好数据，并且将数据拷贝到用户线程存储空间，才会返回。期间用户线程处于阻塞状态，释放CPU资源。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/1639358986184-e10c4719-aa95-4093-92cc-b868160df3d7.png)

***NIO（nonblocking IO）***<br>	非阻塞型IO，发出IO操作后，不进入阻塞状态，不放弃CPU资源，开始轮询内核数据准备情况。内核准备好且收到系统调用，就将输出拷贝到用户线程。并返回

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/1639359034221-3ddfaad7-1378-4de1-8aa4-6f75a16f6364-20220904150804563.png)

***AIO (异步IO Asynchronous IO)***

​	异步IO，发出IO请求之后，就会继续做其他的事情，内核收到后准备数据并将数据拷贝到用户线程存储空间中，然后通知用户线程。用户线程直接使用

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/1639359071332-7ebcbc03-c25a-4ab6-876a-35aa0b1343be.png)

***IO多路复用 和 信号驱动IO***

1. IO多路复用<br>	在多路复用IO模型中，会有一个线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。<br>	select：轮询所有的socket <br>	poll：轮询所有的socket，但是和select相比，没有最大连接数的限制，原因是它是基于链表]()存储的 <br>	epoll：只会轮询发生了IO请求的socket。虽然连接数有上限，但是很大，1G内存的机器上可以打开10W左右的连接；
2. I/O 多路复用模型是利用select、poll、epoll可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有I/O事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll是只轮询那些真正发出了事件的流），依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。

***信号驱动IO***（signal driven IO） 

1. 在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个**信号函数**，然后用户线程会继续执行，当内核数据]()就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在**信号函数**中调用IO读写操作来进行实际的IO请求操作。 

2.  应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。 

3. 内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。信号驱动 I/O 的 CPU 利用率很高。 

***异步 IO 与信号驱动 IO***

 异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

## Java Reator 模型

No

## 零拷贝

> 系统调用次数*2=上下文切换次数
>
> 文件读取分为DMA读取和CPU读取

传统提供文件传输IO过程，是俩次（read、write）调用，四次上下文切换，四次拷贝（DMA2，CPU2）

mmap+write传输文件过程，是俩次（mmap、write）调用，四次上下文切换，三次拷贝（DMA2，CPU1）

sendfile传输文件过程，是一次（sendfile）调用，俩次上下文切换，三次拷贝（DMA2，CPU1）

sendfile+网卡支持 SG-DMA，是一次（sendfile）调用，俩次上下文切换，俩次拷贝（DMA）

## NIO

sockets是面向流的而非包导向的。它们可以保证发送的字节会按照顺序到达但无法承诺维持字节分组



# Design Principle

里氏替换原则

依赖倒置原则

接口隔离原则

单一职责原则

迪米特法则

组合/聚合原则

开闭原则

# Design Model

单例模式



## Facade模式，适配器模式，桥接模式

> 参考文章[桥接模式和适配器模式的区别](https://blog.csdn.net/xiefangjin/article/details/51056411)

### 一. 目的：

1. Facade 模式使用在给一个复杂的系统提供统一的门面\(接口),目的是简化客户端的操作,但并没有改变接口
2. Adapter 模式使用在两个部分有不同的接口的情况,目的是改变接口,使两个部分协同工作
3. 桥接模式 是为了分离抽象和实现

### 二. 使用场合

1. Facade
出现较多是这样的情况,你有一个复杂的系统,对应了各种情况,
客户看了说功能不错,但是使用太麻烦.你说没问题,我给你提供一个统一的门面.
所以Facade使用的场合多是对系统的"优化".
2. Adapter
出现是这样的情况,你有一个类提供接口A,但是你的客户需要一个实现接口B的类,
这个时候你可以写一个Adapter让把A接口变成B接口,所以Adapter使用的场合是
指鹿为马.就是你受夹板气的时候,一边告诉你我只能提供给你A(鹿),一边告诉你说
我只要B\(马),他长了四条腿,你没办法了,把鹿牵过去说,这是马,你看他有四条腿.
(当然实现指鹿为马也有两种方法,一个方法是你只露出鹿的四条腿,说你看这是马,这种方式就是
封装方式的Adapter实现,另一种方式是你把鹿牵过去,但是首先介绍给他说这是马,因为他长了四条腿
这种是继承的方式.)
3. Bridge
在一般的开发中出现的情况并不多,AWT是一个,SWT也算部分是,
如果你的客户要求你开发一个系统,这个系统在Windows下运行界面的样子是Windows的样子.
在Linux下运行是Linux下的样子.在Macintosh下运行要是Mac Os的样子.
怎么办? 定义一系列的控件Button,Text,radio,checkBox等等.供上层开发者
使用,他们使用这些控件的方法,利用这些控件构造一个系统的GUI,然后你为这些控件
写好Linux的实现,让它在Linux上调用Linux本地的对应控件,
在Windows上调用Windows本地的对应控件,在Macintosh上调用Macintosh本地的对应控件
ok,你的任务完成了.

### 三. 需求程度

1. Facade的需求程度是  "中等"  ,因为你不提供Facade程序照样能工作,只是不够好.
2. Adapter的需求程度是  "必须"  ,因为你不这么做就不能工作,除非你自己从头实现一个.
3. Bridge的需求程度是  "一般"  ,适合精益求精的人,因为你可以写三个程序给客户.

### 四. 出现时期

1. Facade出现在项目中期,再优化
2. Adapter出现在项目后期,大部分都有了,差的仅仅是接口不同
3. Bridge出现在项目前期,你想让你的系统更灵活,更cool

### 五. 在写文章的时候想到的

1. Facade很多时候是1:m的关系
2. Adapter很多是候是1:1的关系
3. Bridge很多时候是m:n的关系





# UML

> 参考文章: [30分钟学会UML类图](https://zhuanlan.zhihu.com/p/109655171)

UML中的关系有继承，实现，关联，依赖

**依赖**一般是指方法参数，临时变量，或者对静态方法的调用。强调的仅仅是使用上的需要。依赖关系用一个带虚线的箭头表示，由使用方指向被使用方，表示使用方对象持有被使用方对象的引用

**关联**分为has-a聚合，contain-a组合。关联关系有单向关联和双向关联。如果两个对象都知道（即可以调用）对方的公共属性和操作，那么二者就是双向关联。如果只有一个对象知道（即可以调用）另一个对象的公共属性和操作，那么就是单向关联。大多数关联都是单向关联，单向关联关系更容易建立和维护，有助于寻找可重用的类。在UML图中，双向关联关系用带双箭头的实线或者无箭头的实线双线表示。单向关联用一个带箭头的实线表示，箭头指向被关联的对象

**聚合**人以类聚，物以群分，各自独立，完整的生命周期。聚合关系用空心菱形加实线箭头表示，空心菱形在整体一方，箭头指向部分一方

**组合**不可分割的组成部分，部分不能脱离整体，组合关系用实心菱形加实线箭头表示，实心菱形在整体一方，箭头指向部分一方

仅从类代码本身是区分不了聚合和组合的。如果一定要区分，那么如果在删除整体对象的时候，必须删掉部分对象，那么就是组合关系，否则可能就是聚合关系。从业务角度上来看，如果作为整体的对象必须要部分对象的参与，才能完成自己的职责，那么二者之间就是组合关系，否则就是聚合关系。汽车与轮胎，汽车作为整体，轮胎作为部分。如果用在二手车销售业务环境下，二者之间就是聚合关系。因为轮胎作为汽车的一个组成部分，它和汽车可以分别生产以后装配起来使用，但汽车可以换新轮胎，轮胎也可以卸下来给其它汽车使用。如果用在驾驶系统业务环境上，汽车如果没有轮胎，就无法完成行驶任务，二者之间就是一个组合关系。再比如网上书店业务中的订单和订单项之间的关系，如果订单没有订单项，也就无法完成订单的业务，所以二者之间是组合关系。而购物车和商品之间的关系，因为商品的生命周期并不被购物车控制，商品可以被多个购物车共享，因此，二者之间是聚合关系。
