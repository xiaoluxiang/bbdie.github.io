# 虚拟机类加载机制

## 类加载的时机

1. JVM规范规定类加载的六个有且只有条件下，这六种条件是主动引用，需要触发类加载，其他称为被动引用。
2. 子父类静态子段、数组-数组元素（数组其实可以认为是一种特殊数据类型）、类的静态常量会在类编译进行传播优化（理解为提前文本替换）。
3. 接口与类很相似，同样有\<clinit>()类构造器，但是类在加载的时候要求其父类已初始化。接口只有在初始化中使用到父类接口是才会触发父类初始化。

### 加载

加载工作：<br>获取类的二进制数据流，静态存储结构存为方法区的运行时数据结构，生成Class对象，作为数据访问的入口。





# JVM生产问题

## JVM问题排查方法

> 参考资料
>
> ​	1. [CPU占用过高，死锁，内存泄漏，内存溢出](https://www.jianshu.com/p/28a2fba71105)]

> 总的思路都是通过查看JVM的堆栈信息，其中堆代表着内存使用（新生老年），栈代表着执行的方法情况。

### cpu占用过高

> cpu占用过高，那么一定是线程在疯狂执行中，所以定位到具体栈信息即可，反查代码

1. 通过`top`查看实时进程信息，得到PID
2. 通过`top -Hp PID` 查看线程实时信息 processId
3. `jstack PID | grep processId -A 10`分析具体栈空间信息，执行代码位置，复查代码缺陷

### 死锁

> 线程死锁，没有明显特征，不占用CPU，消耗内存也有限，一般体现为多线程任务长久执行无结果，得考虑死锁的问题

1. `jps`查看Java线程情况
2. jstack 查看特定进程执行堆栈
3. 如果发生死锁，JVM是可以自己检出的，所以查看jstack所打印出的最后信息

### 内存泄漏-内存占用过高-内存溢出

> 内存泄漏，导致内存不足，先体现于程序耗时增加，因为有频繁的GC。严重遇到OOM的问题，

1. 在OOM时自动打印相关日志`-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=heap.bin`
2. 未发生OOM时
   1. 通过`jps`得到进程号
   2. 通过`jstat`分析GC情况，查看GC频率，GC前后内存各区域变化
   3. 主动通过`jmap`dump出内存快照



### JVM 内存占用过高

```shell
# pidValue 指定特定线程值
# 查看JVM指定线程的堆内存
jhsdb jmap --heap --pid pidValue

# 打印出指定线程
jmap -histo:live pidValue
```

![image-20221009142832256](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/image-20221009142832256.png)

# GC

> 性能优化解决方案总体思路：1. 单体应用业务逻辑优化，操作读写方式优化。2. 单体应用JVM优化
>
> [CMS 垃圾回收机制](https://www.cnblogs.com/Leo_wl/p/5393300.html)

### 案例一 Major GC和Minor GC频繁

> [美团GC优化参考](https://tech.meituan.com/2017/12/29/jvm-optimize.html)

**动态年龄计算**：Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值。在本案例中，调优前：Survivor区 = 64M，desired survivor = 32M，此时Survivor区中age<=2的对象累计大小为41M，41M大于32M，所以晋升年龄阈值被设置为2，下次Minor GC时将年龄超过2的对象被晋升到老年代。

下图展示了CMS各个阶段可以标记的对象，用不同颜色区分。 <br>1. Init-mark初始标记(STW) ，该阶段进行可达性分析，标记GC ROOT能直接关联到的对象，所以很快。<br> 2. Concurrent-mark并发标记，由前阶段标记过的绿色对象出发，所有可到达的对象都在本阶段中标记。 <br>3. Remark重标记(STW) ，暂停所有用户线程，重新扫描堆中的对象，进行可达性分析，标记活着的对象。因为并发标记阶段是和用户线程并发执行的过程，所以该过程中可能有用户线程修改某些活跃对象的字段，指向了一个未标记过的对象，如下图中红色对象在并发标记开始时不可达，但是并行期间引用发生变化，变为对象可达，这个阶段需要重新标记出此类对象，防止在下一阶段被清理掉，这个过程也是需要STW的。特别需要注意一点，这个阶段是以新生代中对象为根来判断对象是否存活的。 <br>4. 并发清理，进行并发的垃圾清理。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/09cf176b.png)

新生代对象持有老年代中对象的引用，这种情况称为**“跨代引用”**。因它的存在，Remark阶段必须扫描整个堆来判断对象是否存活，

# 深入理解Java虚拟机

## 第X章 Java内存布局

内存布局发展概览

> 参考[地址](https://www.cnblogs.com/better-farther-world2099/articles/13889075.html)

<img src = 'https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-36899d1d42f4c9ff395181b3dd1e2233_r.jpg' height = 450px>

<img src = 'https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-7fd50080cbaf8e581104cadabe1b2023_r.jpg' height = 400px>

<img src = 'https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-7104247fc099753760cd68e4df6ee862_r.jpg' height = 370px>

JDK 6-8经历了取消永久代的过程，也就是永久代被取消，其中静态变量和字符串常量池被移到堆上。其余脱离JVM内存，放到本地内存上。

## 第七章 虚拟机类加载机制

> 虚拟机如何加载这些Class文件，Class文件中的信息加载到虚拟机中会发生哪些变化

### 类加载的时机

#### 类加载的时机

> 类和接口基本相似，即类和接口都有\<clinit>，初始化接口中的成员变量。但是接口的初始化不会触发父接口的初始化。不可以使用静态代码块。

加载->验证->准备->解析->初始化->使用->卸载

1. 验证->准备->解析统称为连接
2. 加载，验证，准备，初始化，卸载。按顺序开始，解析具体开始时间可自由发挥，是为了支持动态绑定。
3. 初始化的时机。主动引用必须初始化。被动引用，自由发挥。
   - new，getstatic，putstatic，invokestatic，这四种字节码必须初始化。new -> 1. new。2. 父类。3. reflect。

4. 初始化的时机。被动引用
   - 子类引用父类静态字段不会触发子类初始化
   - 通过数组引用类，不会触发类的初始化
   - 静态常量的使用，不会触发该类的初始化

#### 类加载的过程

过程简述

1. 通过类的全限定名获取类的二进制数据流
2. 将数据流所代表的静态存储接口转为方法区的运行时数据结构
3. 在内存中生成其类型的Class对象。作为方法去该类的数据访问入口

其他

1. 类的二进制数据流获取可供程序员自定义。
2. 数组类的创建是虚拟机在内存中动态构建出来的。





