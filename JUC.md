# Java并发编程基础

> 遇到一个有趣的问题，如果一个线程拿到另一个线程B的锁，然后把调用B的wait()方法，B被加入到等待队列中，谁会把B拿到同步队列中。

1. [***内存屏障***](https://www.zhihu.com/question/325469611/answer/1650954047)：<br>LoadLoad Barriers <br>示例：Load1; LoadLoad; Load2 <br>该屏障确保Load1数据的装载先于Load2及其后所有装载指令的的操作 <br>StoreStore Barriers <br>示例：Store1; StoreStore; Store2 <br>该屏障确保Store1立刻刷新数据到内存(使其对其他处理器可见)的操作先于Store2及其后所有存储指令的操作 <br>LoadStore Barriers <br>示例：Load1; LoadStore; Store2 <br>确保Load1的数据装载先于Store2及其后所有的存储指令刷新数据到内存的操作 <br>StoreLoad Barriers <br>示例：Store1; StoreLoad; Load2 <br>该屏障确保Store1立刻刷新数据到内存的操作先于Load2及其后所有装载装载指令的操作。它会使该屏障之前的所有内存访问指令(存储指令和访问指令)完成之后,才执行该屏障之后的内存访问指令。

2. ***[数据依赖性 as-if-serial语义](https://blog.csdn.net/cold___play/article/details/104031253)***: <br>编译器和处理器可能会对操作做重排序。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。<br>这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。<br>as-if-serial语义
   as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）

3. ***[as-if-serial规则和happens-before规则](https://xmmarlowe.github.io/2021/04/28/%E5%B9%B6%E5%8F%91/as-if-serial%E8%A7%84%E5%88%99%E5%92%8Chappens-before%E8%A7%84%E5%88%99/)***<br>*as-if-serial*语义的意思指：**不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。** 编译器、runtime和处理器都必须遵守as-if-serial语义。
   为了遵守as-if-serial语义，**编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果**<br><br>*happens-before（先行发生）*规则:JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。具体的定义为：

   1. **如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。**
   2. 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。**如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么JMM允许这种重排序。**
   3. as-if-serial规则和happens-before规则的区别
      **as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变**。
      as-if-serial语义给编写单线程程序的程序员创造了一个幻觉：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻觉：正确同步的多线程程序是按happens-before指定的顺序来执行的。
      as-if-serial语义和happens-before这么做的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。

4. <font color = "blue">Object.join()部分源码:</font>

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

5. ***[ThreadLocal](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581251653666)***:<br>它可以在一个线程中传递同一个对象，实际上，可以把`ThreadLocal`看成一个全局`Map<Thread, Object>`：每个线程获取`ThreadLocal`变量时，总是使用`Thread`自身作为key：

   ```java
   Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
   ```

   因此，`ThreadLocal`相当于给每个线程都开辟了一个独立的存储空间，各个线程的`ThreadLocal`关联的实例互不干扰。

   最后，特别注意`ThreadLocal`一定要在`finally`中清除：

6. ***超时等待模式***：<br>超时等待模式是一种线程间通讯的方式，等待可能是其他线程处理的过程信息，这种可以通过volae变量值方式进行超时判断。等待的是其他线程的运行状态则可以通过Thread#join方法。详情可以看到

7. ***<font color = "red">线程池</font>***<br>线程池的本质就是使用了一个线程安全的工作队列连接工作者线程和客户端 线程，客户端线程将任务放入工作队列后便返回，而工作者线程则不断地从工作队列上取出 工作并执行。当工作队列为空时，所有的工作者线程均等待在工作队列上，当有客户端提交了 一个任务之后会通知任意一个工作者线程，随着大量的任务被提交，更多的工作者线程会被 唤醒。



# 线程池

## 源码部分

Executor->ExecutorService->AbstractExecutorService->ThreadPoolExecutor

执行者->功能丰富的执行者->抽象的初步的具有一定功能的执行者->线程池版的执行者

## Java中的线程池

> ctl是一个Integer值，它是对线程池运行状态和线程池中有效线程数量进行控制的字段.**Integer值一共有32位，其中高3位表示”线程池状态”，低29位表示”线程池中的任务数量”**

### 基本信息

1. 在最后即线程池满，线程满了，任务队列也满了的时候，采取拒绝策略前，还是要判断下线程池内所有的线程状态是否全部在工作中，如果全部工作中，那么再采用拒绝策略<br>![image-20220929220136720](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20220929220136720.png)
2. 什么时候创建多于corePoolSize呢，在任务队列满了之后，maximumPoolSize允许创建多余的线程，即会创建多余线程。
3. Java的线程既是工作单元，也是执行机制。从JDK 5开始，把工作单元与执行机制分离开 来。工作单元包括Runnable和Callable，而执行机制由Executor框架提供。换句话说，任务执行和任务调度的分离

### CacheThreadPoolExecutor

1. SynchronousQueue在cacheThreadPool中使用其实还是很精巧的。`SynchronousQueue`队列的插入必须等待队列为空或者前一次poll操作完成。是同步阻塞的。但是cacheThreadPool可以拥有INTERGER.MAX_VALUE。线程允许缓存60s，还真是挺符合Cache这个定义的。缓存是有有效期的。

### scheduledThreadPoolExecutor

1. 作为定时执行类线程池，处理周期性任务`ScheduledFutureTask`。开发者往队列里面加任务，线程自动取，然后执行任务，再修改任务时间信息，扔回任务队列。
2. condition中所有的线程都是在等待中，如果当前加入`ScheduledFutureTask`是被插入到头节点中，则会唤醒所有condition中的线程进行新一次的判断,<br>唤醒所有线程后是不是所有线程都需要进行一次判断，功能上是不是存在重复呢？

# Atomic类

> CAS算法是由硬件直接支持来保证原子性的，有三个操作数：**内存位置V、旧的预期值A和新值B，当且仅当V符合预期值A时，CAS用新值B原子化地更新V的值，否则，它什么都不做。**CAS也并不完美，它存在"ABA"问题，CAS只关注了比较前后的值是否改变，而无法清楚在此过程中变量的变更明细，这就是所谓的ABA漏洞。 

