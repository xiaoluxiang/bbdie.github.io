1. [***内存屏障***](https://www.zhihu.com/question/325469611/answer/1650954047)：LoadLoad Barriers <br>示例：Load1; LoadLoad; Load2 <br>该屏障确保Load1数据的装载先于Load2及其后所有装载指令的的操作 <br>StoreStore Barriers <br>示例：Store1; StoreStore; Store2 <br>该屏障确保Store1立刻刷新数据到内存(使其对其他处理器可见)的操作先于Store2及其后所有存储指令的操作 <br>LoadStore Barriers <br>示例：Load1; LoadStore; Store2 <br>确保Load1的数据装载先于Store2及其后所有的存储指令刷新数据到内存的操作 <br>StoreLoad Barriers <br>示例：Store1; StoreLoad; Load2 <br>该屏障确保Store1立刻刷新数据到内存的操作先于Load2及其后所有装载装载指令的操作。它会使该屏障之前的所有内存访问指令(存储指令和访问指令)完成之后,才执行该屏障之后的内存访问指令。
2. ***[数据依赖性 as-if-serial语义](https://blog.csdn.net/cold___play/article/details/104031253)***: 编译器和处理器可能会对操作做重排序。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。<br>这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。<br>as-if-serial语义
   as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）
3. **[as-if-serial规则和happens-before规则](https://xmmarlowe.github.io/2021/04/28/%E5%B9%B6%E5%8F%91/as-if-serial%E8%A7%84%E5%88%99%E5%92%8Chappens-before%E8%A7%84%E5%88%99/) **<br>*as-if-serial*语义的意思指：**不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。** 编译器、runtime和处理器都必须遵守as-if-serial语义。
   为了遵守as-if-serial语义，**编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果**<br><br>*happens-before（先行发生）*规则:JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。具体的定义为：
   1. **如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。**
   2. 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。**如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么JMM允许这种重排序。**
   3. as-if-serial规则和happens-before规则的区别
      **as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变**。
      as-if-serial语义给编写单线程程序的程序员创造了一个幻觉：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻觉：正确同步的多线程程序是按happens-before指定的顺序来执行的。
      as-if-serial语义和happens-before这么做的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。