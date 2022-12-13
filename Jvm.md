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

### 字符串在内存布局中的若干种情形

> 大致的原则就是对于写明的字符串会存储在字符串常量池（方法区）中，对于new String这种对象会分配在堆上。动态产生的String对象如StringBuilder，只会在堆上。

String str1 = "lushixiang"; 这种会一个引用对象，指向常量池中字符串

String str2 = new String("lushixiang"); 这种会有一个引用对象，指向堆上String对象，其内部属性char value[]指向常量池。

String str3 = "lu"+"shixiang"; 会被编译器优化掉等效于str1

String str4 =  "lu"+new String("shixiang"); 会被编译器优化成Stringbuilder相加，然后toString。重点在于StringBuilder的toString方法是复用堆上的，即传递数组char value[]来实现的

## 第六章类文件结构

Java技术能保持良好的向后兼容性，clas 文件结构稳定性功不可没。

class文件是以八个字节为单位的二进制数据流（如何理解这句话），类似于C语言中的结构体，数据类型分为无符号数和表。无符号数u表示，其接数字表示字节长度。用来描述数字，索引，数量值或者字符串。表则为多个无符号数和其他表组成。

魔数->版本号（4）主次版本，预览版次版本需为65535->

常量池入口（资源仓库）位移计数器，从1开始，0留作后面指向常量时为不需要真指向即不需要引用任何常量池的内容。常量池主要存放了字面量和符号引用。符号引用即导入的包，类的全限定名称，字段名称描述符，方法的名称和描述符，方法句柄和方法类型。动态调用点和动态常量。在Javac编译的过程中，其实不是会描述字段方法真正的内存布局，而只是程序的格式整理。在运行时进行动态加载解析初始化。

> 程序是由字面量开始，符号引用操作，不同符号含义的确定即通过表的tag定义完成。

常量池中每一项都是一个表。并且起始字段都为表类型。根据表中子字段含义及其值回溯拼凑自己表达内容。可以熟练掌握几个类型。CONSTANT_UTF8_INFO等。

CONSTANT_Fieldref_info，CONSTANT_Methodref_info，常量类型，标明自己属于哪个类，本身的NameAndType（CONSTANT_NameAndType_info）。其CONSTANT_NameAndType_info，常量类型，Name，Type（字段或者方法）。<br>CONSTANT_MethodHandle_info，常量类型，reference_kind(值范围为1-9，代表着基本数据类型和引用类型。Java语言是静态类型的（statical typed))，参考地址[Java数据类型总结：基本类型、引用类型](https://zhishui0501.github.io/Summary-of-Java-Data-Types。reference_index，值为对常量池的有效索引。<br>Q: CONSTANT_MethodHandle_info,CONSTAND_Method_type_info,CONSTAND_NameAndType_info。到底如何描述Method。除此之外，Dynamic和invokeDynamic

**类的访问标志**：俩个字节，16个标志位，目前仅用了9个JDK9.主要是用来表示public static abstract，enum，annotation，module。

**类索引，父类索引，接口索引集合**：通过u2类型指向常量池中的CONSTANT_Class_info

**字段表集合**，字段可以用来修饰public static final volatile transient enum 编译器产生。上述这些固定表示范围+数据类型和名字非固定内容组成，即name_index和descriptior_index。插播**描述符的作用，即为字段的数据类型或方法的参数列表和返回值（先参数列表后返回值缩写符号）**。字段表不会显示父类的字段，对于内部类来说，编译器可能添加对外部类引用字段。在Class文件中，字段是允许重载的。

**方法表**：这些都是定长表示，访问标志，name，描述符，属性表。

## 第七章 虚拟机类加载机制

> 虚拟机如何加载这些Class文件，Class文件中的信息加载到虚拟机中会发生哪些变化

### 类加载的时机

#### 类加载的时机

> 类和接口基本相似，即类和接口都有\<clinit>，初始化接口中的成员变量。类在加载的时候要求其父类已初始化。但是接口的初始化不会触发父接口的初始化。接口只有在初始化中使用到父类接口是才会触发父类初始化。不可以使用静态代码块。
>
> 获取类的二进制数据流，静态存储结构存为方法区的运行时数据结构，生成Class对象，作为数据访问的入口。

JVM规范规定类加载的六个有且只有条件下，这六种条件是主动引用，需要触发类加载，其他称为被动引用。

子父类静态子段、数组-数组元素（数组其实可以认为是一种特殊数据类型）、类的静态常量会在类编译进行传播优化（理解为提前文本替换）。



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

JVM

堆中对象的生命周期及如何分配到堆中各个区

> [参考地址: 当jvm的eden区满了,进行回收时,s0区满了，此时eden区还有存活](当jvm的eden区满了,进行回收时,s0区满了，此时eden区还有存活对象没复制完，会怎样？ - Ted Mosby的回答 - 知乎 https://www.zhihu.com/question/40971445/answer/144496340)

一般来说新建对象会放到Eden区，并被分配年轻计数器用来记录对象年龄。当Eden区满又需要新建对象，则会触发minor GC，并将存活对象放入Survivor 0区。此后每次minor GC之后都会轮流移动存活对象到survivor 0 区和survivor 1区。默认经过15次GC后的对象会被放入老年代。老年代存储满并且需要对象存入，则会触发Major GC。注意点是老年代对象来源来自正常提升以及年轻代满了之后的额外提升。第二点是每次minor GC之后都会重新计算各区size和升代门槛。

Thread会有私有空余内存块。方便快速安全分配对象。空余内存块仅占总内存的1%。会优先在本线程私有空余内存块中分配，失败则通过加锁的方式确保在内存中数据操作的原子性和安全性

