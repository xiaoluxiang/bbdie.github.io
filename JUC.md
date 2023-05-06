# Java并发编程基础

> 遇到一个有趣的问题，如果一个线程拿到另一个线程B的锁，然后把调用B的wait()方法，B被加入到等待队列中，谁会把B拿到同步队列中。
>
> 无锁开发->非阻塞乐观锁->读写锁分离->降低锁的粒度&减少锁的持有时间

## Java内存模型中的术语

1. [***内存屏障***](https://www.zhihu.com/question/325469611/answer/1650954047)：

   LoadLoad Barriers <br>示例：Load1; LoadLoad; Load2 <br>该屏障确保Load1数据的装载先于Load2及其后所有装载指令的的操作 <br>StoreStore Barriers <br>示例：Store1; StoreStore; Store2 <br>该屏障确保Store1立刻刷新数据到内存(使其对其他处理器可见)的操作先于Store2及其后所有存储指令的操作 <br>LoadStore Barriers <br>示例：Load1; LoadStore; Store2 <br>确保Load1的数据装载先于Store2及其后所有的存储指令刷新数据到内存的操作 <br>StoreLoad Barriers <br>示例：Store1; StoreLoad; Load2 <br>该屏障确保Store1立刻刷新数据到内存的操作先于Load2及其后所有装载装载指令的操作。它会使该屏障之前的所有内存访问指令(存储指令和访问指令)完成之后,才执行该屏障之后的内存访问指令。

2. ***[数据依赖性 as-if-serial语义](https://blog.csdn.net/cold___play/article/details/104031253)***: <br>as-if-serial语义
   as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变

   编译器和处理器可能会对操作做重排序。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。<br>这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。<br>

3. ***[as-if-serial规则和happens-before规则](https://xmmarlowe.github.io/2021/04/28/%E5%B9%B6%E5%8F%91/as-if-serial%E8%A7%84%E5%88%99%E5%92%8Chappens-before%E8%A7%84%E5%88%99/)***<br>*as-if-serial*语义的意思指：**不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。** 编译器、runtime和处理器都必须遵守as-if-serial语义。为了遵守as-if-serial语义，**编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果**<br><br>*happens-before（先行发生）*规则:JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。具体的定义为：

   1. **如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。**
   2. 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。**如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么JMM允许这种重排序。**
   3. as-if-serial规则和happens-before规则的区别
      **as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变**。
      as-if-serial语义给编写单线程程序的程序员创造了一个幻觉：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻觉：正确同步的多线程程序是按happens-before指定的顺序来执行的。
      as-if-serial语义和happens-before这么做的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。

## CPU实现中的术语

1. 基础术语![image-20221106134257124](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106134257124.png)
	1. 原子操作![image-20221106173756054](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106173756054.png)对于被处理器缓存的缓存行内容，如果在Lock期间被锁定，回写内存时，不必声明Lock#信号，直接修改内部地址。通过缓存一致性协议禁止其他处理器的修改。对于无法缓存或者缓存内容超过缓存行单位长度，则无法使用缓存锁定。

## Java语言中的术语

1. ### <font color = 'blue'> volatile</font>

   ***含义***：在多处理器开发中保证了共享变量的可见性。（可见性即确保所有线程看到的变量的值是一致的）

   ***实现***：JVM会向处理器发出lock指令（锁总线，可导致访问任何共享内存，现处理器不一定锁总线），指令导致将本缓存行的数据写回系统内存，并使其他处理器对该内存地址的数据无效（对于这个其他处理器的数据无效。是因为各处理器间使用缓存一致性协议，通过嗅探总线上的数据来检查自己缓存的数据是否失效，并且也会阻止俩个及以上的处理器同时修改同一内存地址的数据）

   ***优点***：相较于synchronized，更为轻量。且不会引发上下文切换和调度

   ***缺点**：

   - 当单个对象不足机器的缓存行并且放到同一缓存行，会导致伪共享的现象。可使用填充字节的方式（可能会被JDK优化以致方法失效）

   ![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/java-thread-x-key-volatile-2.png)

2. ### <font color = 'blue' >synchronized</font>

   ***含义***：Java中每一个对象（对象头）都可以当作锁，对于普通方法，锁的是实例对象，对于静态方法锁的是类对象，对于代码块锁的是指定的对象。执行代码过程前需获得锁，正常或异常退出需要释放锁。

   ***实现***：

   - 代码块同步是使用了monitorenter和monitorexit指令实现的。方法同步JVM未作详细说明。（任何对象都持有monitor对象，指令或尝试获取与释放monitor对象）

   - 数据结构

     ynchronized使用的锁是在对象头中实现的。数组类型使用三个字长。非数组类型使用俩个字长![image-20221106141443140](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106141443140.png)

     Mark Word组成![image-20221106141546701](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106141546701.png)Mark Word的变化情况![image-20221106141616902](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106141616902.png)64位下的Mark Word ![](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106141645223.png)

   - 数据操作

     偏向锁是一种乐观锁，锁的状态位同无锁，都是01，考虑大部分情况下都是单一线程重复获取与释放锁，所以在锁对象的markword记录偏向线程号。如果不是我，则看锁标志位，是则cas竞争锁，不是则cas使偏向锁指向自己。<br>偏向锁的撤销：发生竞争后才会主动撤销，并且等到全局安全点。先看该线程是否活着，活着先执行，等待的CAS，CAS不成功则去撤销偏向锁。进行锁的升级。

     轻量锁是一种乐观锁，线程先在自己的栈空间中创建用于存储锁记录的空间，并将markword信息复制过来。然后反向替换Mark Word为该空间的指针。反向替换失败则尝试自旋。自旋失败，则升级为重量级锁，自己陷入阻塞。<br>轻量级锁的撤销：将markword的信息替换回去，失败的膨胀成重量级锁。然后解锁需要唤起阻塞线程

   ***优缺点***：![image-20221106174703143](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221106174703143.png)

3. ### <font color ='blue'>原子操作</font>

   ***含义***：不可中断的一系列操作。

   ***实现***：通过锁和CAS实现

   - CAS通过处理器的CMPXCHG。从Java 1.5开始，JDK的并发包里提供了一些类来支持原子操作，如AtomicBoolean(用原子 方式更新的boolean值)、AtomicInteger(用原子方式更新的int值)和AtomicLong(用原子方式更 新的long值)。这些原子包装类还提供了有用的工具方法，比如以原子的方式将当前值自增1和 自减1
   - 利用锁机制实现原子操作，锁机制确保了只有获得锁的线程才能操作锁定的内存区域。除了偏向锁，JVM实现锁的方式都用了循环 CAS，即当一个线程想进入同步块的时候使用循环CAS的方式来获取锁，当它退出同步块的时 候使用循环CAS释放锁。

   ***优缺点***：

   - CAS

     ABA的问题，可以通过在变量里增加单调版本号来实现<br>自旋时间长消耗大<br>只能保证一个共享变量的原子操作。对多个共享变量操作时，循环CAS就无法保证操作的原子 性，这个时候就可以用锁。有一个取巧的办法，就是把多个共享变量合并成一个共享变量来 操作

4. ### <font color = "blue">Object.join()部分源码:</font>
```java
public class Thread implements Runnable {
    ...
    public final void join() throws InterruptedException {
        join(0);
    }
    ...
    public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        if (millis == 0) { //判断是否携带阻塞的超时时间，等于0表示没有设置超时时间
            while (isAlive()) {//isAlive获取线程状态，无线等待直到previousThread线程结束
                wait(0); //调用Object中的wait方法实现线程的阻塞
            }
        } else { //阻塞直到超时
            while (isAlive()) { 
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
    ...
```

Q: 问题在于谁拿到该对象，A线程中，使用B.join()方法，A是获取到B是否存活，存活则弃锁继续等待wait()，否则返回结束。

4. ***[ThreadLocal](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581251653666)***:<br>它可以在一个线程中传递同一个对象，实际上，可以把`ThreadLocal`看成一个全局`Map<Thread, Object>`：每个线程获取`ThreadLocal`变量时，总是使用`Thread`自身作为key：

   ```java
   Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
   ```

   因此，`ThreadLocal`相当于给每个线程都开辟了一个独立的存储空间，各个线程的`ThreadLocal`关联的实例互不干扰。

   最后，特别注意`ThreadLocal`一定要在`finally`中清除：

5. ***超时等待模式***：<br>超时等待模式是一种线程间通讯的方式，等待可能是其他线程处理的过程信息，这种可以通过volae变量值方式进行超时判断。等待的是其他线程的运行状态则可以通过Thread#join方法。详情可以看到

6. ***<font color = "red">线程池</font>***<br>线程池的本质就是使用了一个线程安全的工作队列连接工作者线程和客户端 线程，客户端线程将任务放入工作队列后便返回，而工作者线程则不断地从工作队列上取出 工作并执行。当工作队列为空时，所有的工作者线程均等待在工作队列上，当有客户端提交了 一个任务之后会通知任意一个工作者线程，随着大量的任务被提交，更多的工作者线程会被 唤醒。



# Java内存模型

> 多线程，解决的事控制不同线程操作相对顺序。即不同线程的同步问题。大体上有俩种方式，即共享内存模型和消息传递。Java采用的是共享内存模型。所以不同线程通讯是隐形的。故对于程序员来说，这个过程是透明的
>
> Java内存模型是对栈空间与堆空间的抽象。是概念模型。共享的是整个堆内存的数据。通过堆空间即主内存实现跨线程隐形控制交互与同步

### <font color = 'blue'>1: 重排序现象</font>

   ***现象***：略

   ***发生原因***：在编译器编译，cpu指令执行，硬件IO操作环节。优化程序性能发生重排序现象。

   ***发生场景***：

   - 编译器优化的重排序，编译器有编译重排序规则禁止非法重排序
   - 指令并行的重排序，编译器通过在特定操作位置插入内存屏障禁止非法重排序
   - 内存系统的重排序。无。

   ***发生影响***：

### <font color = 'blue'>2: happens-before</font>

   含义：一个操作按顺序排在另一个操作的前面，并且第一个操作的结果对第二个操作可见。不要求前一个操作必须在后一个操作前执行。即顺序和执行过程与执行结果的关系。

   特性：<br>如果前一个操作的结果需要对其他操作可见，那么他们需要有happens-before关系。<br>happens-before 是一种折中，要求程序员按逻辑顺序写。要求程序执行满足逻辑结果。win-win

### <font color = 'blue'>3: 数据依赖性</font>

   ***含义***：数据依赖性来源于操作（写与读）的关联性。对于写后读，写后写，读后写来说存在关联。

### <font color = 'blue'>4: as-if-serial</font>

   *as-if-serial*语义的意思指：**不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。** 编译器、runtime和处理器都必须遵守as-if-serial语义。为了遵守as-if-serial语义，

### <font color = 'blue'>5: 重排序规则</font>

​	**编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果**

​	**单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果**

​	**多线程程序中，对存在控制依赖的操 作重排序，可能会改变程序的执行结果**

### <font color = 'blue'>6: 数据竞争</font>

​	当程序未正确同步时，就可能会存在数据竞争，即一个读一个写，并且这俩个线程没有就此通过同步来排序。JMM保证了如果程序是正确同步的，程序的执行将具有顺序一致性(Sequentially Consistent)——即程 序的执行结果与该程序在顺序一致性内存模型中的执行结果相同。马上我们就会看到，这对 于程序员来说是一个极强的保证。这里的同步是指广义上的同步，包括对常用同步原语 (synchronized、volatile和final)的正确使用。

### <font color = 'blue'>7: 顺序一致性模型&JMM</font>

定义：所有的线程按照程序预定义的顺序进行，所有线程只能看到单一的顺序，并且自己的操作对其他线程立即可见。（即操作全局唯一内存空间，而且操作也是串行化的）

使用：在顺序一致性模型中，正确同步的程序的执行顺序是确定的，并且在总的顺序也是确定的。未正确同步的程序，执行顺序是确定的，对于每个线程看到的顺序也是确定的，就是总的顺序可能是不确定的。

JMM：不正确同步程序，不保证与顺序一致性模型一致，并且每个线程看到的顺序也可是不一致的。因为JMM要保持一致需要禁止大量的重排序，再因为总体上程序也是无序的，这样做没有意义。但是JMM提供最小安全性，即确保要么是之前的值，要么是默认值（JMM在创建对象的时候会对内存区域进行清零，然后存入值）对于正确同步的程序，JVM会在结果一致的情况下，尽量让编译器和处理器进行重排序（临界区内的代码，可能允许重排序）

### <font color = 'blue'>8: volatile内存语义</font>

·线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程发出了(其对共享变量所做修改的)消息。 ·

线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的(在写这个volatile变量之前对共享变量所做修改的)消息

内存语义的实现：插入内存屏障实现禁止重排序的目的。插入屏障策略十分保守。会在volatile读写前后均插入内存load/store屏障。volatile的内存语义被增强，禁止与普通变量进行重排序。其读写仅具有原子性与可见性。相对于synchronized。功能弱，但灵活。



# Java中的锁

> 乐观悲观锁
>
> 阻塞和自旋锁
>
> 公平锁和非公平锁
>
> 无锁->偏向锁->轻量级锁->重量级锁

公平锁和非公平锁。公平锁保证所有线程不会饿死，但吞吐率较低。非公平锁：线程尝试加锁时碰上解锁，可能会直接获得锁，这样减少线程的唤醒，整体吞吐率较高。但有的线程会等待很久

# Java多线程

new Thread 搭配call

# 线程池

## 源码部分

Executor->ExecutorService->AbstractExecutorService->ThreadPoolExecutor

执行者->功能丰富的执行者->抽象的初步的具有一定功能的执行者->线程池版的执行者

## Java中的线程池

> ctl是一个Integer值，它是对线程池运行状态和线程池中有效线程数量进行控制的字段.**Integer值一共有32位，其中高3位表示”线程池状态”，低29位表示”线程池中的任务数量”**

### 基本逻辑

> 描述线程池的生命周期变化情况

![image-20220929220136720](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20220929220136720.png)

1. 由上图，初始时线程池未达到corePoolSize时，一旦提交任务就选择新建线程
2. 当线程池满，就提交到任务队列中，如果任务队列已满，则查看能否继续新建线程即maximum是否允许新建
3. 正常超过corePoolSize的线程按照疯狂去队列获取任务，知道满足超时时间后销毁
4. Java的线程既是工作单元，也是执行机制。从JDK 5开始，把工作单元与执行机制分离开 来。工作单元包括Runnable和Callable，而执行机制由Executor框架提供。换句话说，任务执行和任务调度的分离

### 使用方法

> 线程池的客户是任务，任务的特点决定线程池的选型
>
> 考虑任务属性是CPU密集型orIO密集型，确定线程池大小->考虑线程池所需资源，确定线程超时时间->考虑优先级->考虑执行时间，确定线程队列

提交：execute（无返回值，直接抛出异常）submit（有返回值，异常自己get）

关闭：shutdown，shutdownNow

监控：ThreadPoolExecutor提供很多属性用于监控，也可以通过继承重写一些方法，获得线程池，任务执行的更多详细信息

## Executor框架



![](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20230217140059784.png)

Executor主要是由任务，任务的执行，异步计算的结果

主要成员：任务（Runable，CallAble），任务的执行（ThreadPoolExecutor及其子三个，ScheduleThreadPoolExecutor及其子二），异步计算的结果（Future）



CacheThreadPoolExecutor

1. SynchronousQueue在cacheThreadPool中使用其实还是很精巧的。`SynchronousQueue`队列的插入必须等待队列为空或者前一次poll操作完成。是同步阻塞的。但是cacheThreadPool可以拥有INTERGER.MAX_VALUE。线程允许缓存60s，还真是挺符合Cache这个定义的。缓存是有有效期的。

scheduledThreadPoolExecutor

1. 作为定时执行类线程池，处理周期性任务`ScheduledFutureTask`。开发者往队列里面加任务，线程自动取，然后执行任务，再修改任务时间信息，扔回任务队列。
2. condition中所有的线程都是在等待中，如果当前加入`ScheduledFutureTask`是被插入到头节点中，则会唤醒所有condition中的线程进行新一次的判断,<br>唤醒所有线程后是不是所有线程都需要进行一次判断，功能上是不是存在重复呢？

# 并发类

## 并发操作原子类



## 原子操作类

> JDK中的Unsafe提供了三种CAS方法，compareAndSwapObject、compare- AndSwapInt和compareAndSwapLong。所以对应的可以Object，Int，Long三者的原子更新
>
> AtomicBoolean，AtomicInteger，AtomicLong
>
> AtomicBooleanArray，AtomicIntegerArray，AtomicreferenceArray
>
> AtomicReference，AtomicReferenceFieldUpdater，AtomicMarkableReference
>
> AtomicIntegerFieldUpdater，AtomicLongFieldUpdater，AtomicStampedReference

原子更新基本类型3，数组4（通过构造方法引入的数组是不会改变的），引用类型3，字段3，感受一下API的命名getAndSet，addAndSet，compareAndSet。

都是将值传递进去，包装后按照调用Atomic那边的API，完成对原对象的操作



> CAS算法是由硬件直接支持来保证原子性的，有三个操作数：**内存位置V、旧的预期值A和新值B，当且仅当V符合预期值A时，CAS用新值B原子化地更新V的值，否则，它什么都不做。**CAS也并不完美，它存在"ABA"问题，CAS只关注了比较前后的值是否改变，而无法清楚在此过程中变量的变更明细，这就是所谓的ABA漏洞。 

## 并发工具类

### CountDownLatch

通过传递变量countDownLatch完成等待与同步的作用，可用于不同线程间（也可以是步骤节点）的同步，是通知主持有者，然后自己可以正常执行

```java
// 常用API
// 构造方法
public CountDownLatch(int count) {  };  //参数count为计数值
// 功能方法
public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown() { };  //将count值减1
```

### CyclicBarrier

通过传递变量cyclicBarrier完成等待与同步的作用，可用于不同线程间（也可以是步骤节点）的同步，是通知cyclicBarrier，我已抵达，阻塞，等待被通知

可以被reset，即可以重试

提供多种信息属性，了解CyclicBarrier状态

```java
// 常用API
// 构造方法
// 所有线程到达之后barrierAction会开始执行
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
// 功能方法
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

### Semaphore

通过传递变量semaphore完成等待与同步的作用，用于控制访问特定资源的线程数量。尝试获取令牌，获取成功则进行操作，最后释放令牌。

提供多种方法，了解Semaphore的状态

```java
// 常用API
// 构造方法
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
// 功能方法
    public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

### Exchange

通过传递变量exchange完成线程间交换作用，尝试exchange方法后，原地阻塞，知道其他线程完成交换

```java
// 常用API
// 构造方法
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
// 功能方法
    public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

# JUC思维脑图

## 理解并发

**多线程为出现并发问题**：
CPU为了平衡内存的速度会有缓存，带来了可见性问题（缓存和共享内存数据不一致）
操作系统为了充分利用CPU采用了时分复用的方案，带来了原子性问题（操作可能完成后就切换上下文，寄存器和程序计数器）
编译执行程序为了优化程序采用了重新编排执行次序，导致有序性问题（程序执行顺序和编写顺序不一致，编译重排序和cpu重排序）

**Java是如何解决多线程并发问题**：
通过Java内存模型（按需禁用缓存和编译优化）：（happens-before原则/as-if-serial原则）+synchronized/volatile/final解决

happens-before：规定先行原则即一个操作无需控制就能先行另一个操作完成，如果具有happens-before关系，则一个操作的发生再另一个操作之前，且结果对下一个操作可见。具体有单一线程原则、管程锁定规则（锁）、volatile、线程启动规则、线程加入原则、线程中断、对象终结规则、传递性。

**实现线程安全的方式**：
悲观的互斥同步（synchronized/lock），乐观的冲突检测在补偿（CAS），无同步方案（栈内存，threadLocal，pure code）

## 理解线程

**线程的生命周期**：
新建、可运行、阻塞（超时）、等待（超时）、终止。

**线程的使用（中断interrupt，互斥，协作）**
线程新建：extend Thread; new Thread(runable/callable); ThreadPoolExecutor
普通线程协作机制：
	Thread.sleep(XX); Thread.yield(XX);  join()
	下面是属于JUC下面Thread的方法：threadName.interrupt(); threadName.interrupted(); interrupt：设置中断；interrupted：检测中断并清除中断状态；isInterrupted：返回中断状态；
	wait/notify/notifyAll；线程的阻塞与唤醒，是需要搭配synchronized使用；
线程池的协作：shutdown：等待所有任务执行完毕后结束。shutdownNow：调用每个线程的interrup方法；
线程之间的同步协作：synchronized；Lock(condition的await，signal，signall。正常都有的lock和unlock）;

## 理解JUC相关关键字

### volatile详解

可见性：lock指令+插入内存屏障禁止重排序实现可见性
有序性：插入内存屏障实现有序性

### final详解

**使用**:
类（所有方法隐形final），方法（private 隐形final，子类虽同名但不同无法父类调用），参数（无法修改参数指向），变量（并非皆为编译器常量，static final定义时赋值，blank final允许后续赋值）

## 理解Java中的锁

**悲观锁/乐观锁**：悲观锁在使用资源时任务竞争一定会发生。乐观锁认为读的过程无，更新需要检测一下；
**自旋锁/适应性自旋锁**：阻塞和唤醒一个线程是需要操作系统切换CPU状态来完成的。所以自旋锁就是不放弃CPU执行时间，认为锁很快就会被释放；
**无锁/偏向锁/轻量级锁/重量级锁**：详见synchronized锁的优化；
**公平锁和非公平锁**：是指新加入竞争的锁能不能不排队直接竞争资源。正常都会有一个等待队列。如果竞争失败则乖乖排队。
**可重入锁和不可重入锁**：同一个线程嵌套调用的锁方法能否获取相同的锁。
**排他锁和共享锁**：读写分离；

### synchronized详解

**synchronized使用**：
锁代码块（显式指定对象），锁方法（修饰方法，public void synchronized），锁静态方法（class）

**原理分析**：
**加锁**：通过使用cpu指令monitorenter/monitorexit实现，monitor计数器为0，则立即获取锁然后加1，别的等待。已经拿到锁则继续加1表示重入。否则已被获取等待释放。
**解锁**：monitorexit会导致monitor计数器减1，如果不是0表示当前线程仍然持有这把锁；
**可重入**：就是通过monitor计数器，尝试获取的加一，程序退出则减一，线程异常则清零。
保证可见性：

**JVM锁的优化**：
锁粗化：减少不必要的lock和unlock操作，将多个连续的锁合并，减少加锁解锁的性能消耗；
锁消除：通过逃逸分析，判断哪些无需锁保护的操作；
无锁：
偏向锁：如果竞争对象未被锁定，则复制`Mark Word`到线程栈空间创建锁记录存储。然后CAS的修改`Mark Word`使其指向自己。偏向锁是非主动撤销过程，需要等到竞争发生之后再到全局安全点（当前线程不在执行字节码），寻找指向线程判断是否存活，再遍历其栈记录，要么恢复到无锁，要么升级为轻量级锁。
轻量级锁：就是发生竞争之后，开始自旋（也会有适应性自旋，认为竞争一会就会消失）；
重量级锁：竞争成功的正常执行，失败的加入阻塞等待队列中；

**注意点**：锁对象不能为空，作用域不能太大，要避免死锁。尽量不用；

### Lock详解

默认是非公平锁，当然可以通过显式指定为非公平锁

API：lock(); unlock(); tryLock(); tryLock(long, TimeUtil);

**Synchronized和lock**：
实现层面不同：lock是JDK实现的，
Lock可以显示指定为公平锁，可以绑定多个条件，可以被中断，需要自己手动释放
synchronized为非公平锁，不可被中断，执行完毕或者异常会自动释放锁

