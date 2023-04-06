# Summary

1. 今日思考解耦的不同方式，解耦的主要思路就是达到一种可持久化的中间状态，这种中间状态并且可以继续转为其他状态（中间态或最终态），或者维持不变
2. 思考中间件的设计思想，之前理解为过滤器有点简陋，今天在操作蚂蚁平台，对于他的应用包和任务调度（任务调度分为不同类型），他的作用起的增强的，简化，过滤
3. 工程结构的设计，其中包括核心任务实现，从入到出的处理逻辑，逻辑的层次化划分。对于层次的划分，很经典的是应用层，持久层，
4. SQL灵活的结果可以实现多种查询需求
5. JDK提供了ThreadPool的实现
6. 接口参数设计上，可以参考WEB的request请求，一些固定的报文头，再加上一个Map类型



# 接口、lambda表达式与内部类

1. 接口的默认实现，便于接口演化&源代码兼容
2. 接口与抽象类的相同方法签名的方法，类为主，接口冲突需要程序员手动解决
3. ***对于Function接口的compsoe方法和andThen方法需要以一种动作的结合来考虑，而不是结果传递，是动作传递***--->待复习
4. 内部类有多种形式，常规内部类，局部内部类，匿名内部类，静态内部类。除静态内部类，其他不允许有静态方法。其实可以理解，其他内部类一般都是主类实例化之后才会存在，不可能在主类没有实例化，就能有其实例化后对象的静态方法。
5. 对于接口的提供抽象类的半实现，还可以通过接口的默认实现方式进行实现
6. Java程序设计语言中，所有链表实际上都是双向链接的
7. 面向接口的过程中，接口类型引用如果不存在对应方法变量，需要强转才能使用其方法或变量
8. List中的remove 方法重载，基本数据类型与Object类型
9. New 一个静态内部类（泛型）作为结果返回



# Redis 中间件

1. Redis如何使用
   - 在Maven依赖中添加redis的starter
   - 在配置文件中配置redis的配置信息
   - 注入Redis对象，即可使用
2. Redis的template对象

# 并发

1. 并发会有很多种类，
   - 通知等待机制
   - Lock
   - syn
   - volatile
   - Callable,Runable,future
   - 原子类
2. 了解并发的发生原因
   - 资源的竞争
   - 并发对数据的破坏方式
3. 降低死锁，阻塞的方法
   - 减少锁的粒度
   - 降低锁的持有时间

# 代理

```java
package com.lushixiang.aop;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class TestAopExample {


    public static void main(String[] args) {
        Person person = new Person();
        Object result = Proxy.newProxyInstance(person.getClass().getClassLoader(), person.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
                System.out.println("before Method execute ...");
                System.out.println("now method name: " + method.getName());
                Object invoke = method.invoke(person, objects);
                System.out.println("after Method execute....");
                System.out.println("Method execute result: " + invoke.toString());
                if ("StringAop".equals(method.getName())) {
                    System.out.println("change result to Hello World");
                    return "Hello world";
                }
                return null;
            }
        });
        ((AopInterface) result).stringAop();
    }
}
// JDK 动态代理
class Person implements AopInterface{

    @Override
    public String stringAop() {
        System.out.println("method start execute...");
        String s = "hello world";
        System.out.println("method execute over...");
        return s;
    }
}

```

@CallerSensitive 

Q：JDK 动态代理 源码分析(**分析目的**，生成的Class文件到底如何实现接口中方法，然后调用自定义的InvocationHandler)<br>A：通过Class对象的参数为InvocationHandler的初始化方法实现的。猜测实现原理是保留该对象，然后在接口方法中调用InvocationHandler.invoker

Q：如果是保留该对象，如何实现对象的不变。<br>A：finall 

```java
// 通过预定义的GetProxyClass0方法生成对应Proxy的子类，并且实现了intfs
Class<?> cl = getProxyClass0(loader, intfs);
// 拿到他的 constructorParams = { InvocationHandler.class } 构造方法
final Constructor<?> cons = cl.getConstructor(constructorParams);
// 通过构造方法生成代理对象
return cons.newInstance(new Object[]{h})

/* 
有一些值得参考的写法
*/
private static final WeakCache<ClassLoader, Class<?>[], Class<?>> proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

// 再提供proxyClassCache的get方法，代码看起来确实有点意思

```

# MySQL

1. 

# 线程池的使用

使用自定义配置构造池

# 注解

```java
@EnableConfigurationProperties(DataSyncServerClientAutoProperties.class);
@PreDestroy
```

# MS

1. 良好的文档管理工具



# Java Lang

1. protected修饰符用于提供包访问权限。非静态成员使用法对于同包内使用同public，不同包内只能使用子类方法（引用类型必须为子类）。当然可以通过子类重写该方法时通过调用父类的super.XX来加强。
   静态成员就只有通过子类访问

# Decimal

1. `equal`方法比较会比较精度
2. 大小比较使用他的API



# 注解

过滤器中通过反射校验特定注解的修饰的属性情况

# 算法

1: 在操作上存在重复现象-重复操作可用备忘录或者动态规划进行优化

2: 查找上存在可优化行为，hash表-遍历查找，可以使用hash表将查找的复杂度降为0(1) 

领域模型和数据模型侧重点不同，导致结果不同

代码是用来看的。写清楚明白的代码，而不是平铺式，过程式，要面向对象编程

不是你可不可以的问题，是应不应该的问题





