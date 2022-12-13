# Spring

## 概述

1. IOC，DI。开发者通过XML或者注解进行Bean定义的配置，并被spring扫描范围内。spring创建并放到IOC container并维护。通过注解完成依赖注入（DI）的方式实现了IOC。
2. AOP，面向切面编程。spring定义切面，通过拦截切入点实现不同业务模块的代理，这就是面向AOP编程
3. 组件，一站化。通过XML和注解可快速引入某些功能，spring也有着完善的开发生态

## IOC

> 参考文档
>
> [Spring中的循环依赖及解决](https://juejin.cn/post/6985337310472568839)

### 循环依赖

**问题来源**

构造对象时互相需要，导致循环依赖。构造对象时分为构造方法时需要和getter/setter时需要，构造方法时需要则无法解决（@Lazy，显性指定属性的延迟初始化，即不直接创建所依赖的对象了，而是使用动态代理创建一个代理类。只有当他首次被使用的时候才会被完全的初始化）。getter/setter时需要先通过getBean获取依赖对象。如果时原型模式Bean，则会产生OOM。如果时单例模式则可以通过三级缓存的方式解决循环依赖。

BeanPostProcessor的执行在Bean的生命周期中是处于属性注入之后的，所以对于BeanPostProcessor导致依赖对象和实际对象不一致。

**Bean创建&三级缓存**

spring扫描所有class得到所有的BeanDefinition，根据BeanDefinition反射获取并执行对象的构造方法，完成对象创建。此时对外暴露ObjectFactory，放置到三级缓存中。完成属性注入，查看对象方法是否需要完成AOP（需要判断下是否需要AOP），放入到singletonObjects中。

在每个Bean的生成过程中，都会提前暴露一个工厂，这个工厂可能用到，也可能用不到，如果没有出现循环依赖依赖本bean，那么这个工厂无用，本bean按照自己的生命周期执行，执行完后直接把本bean放入singletonObjects中即可，如果出现了循环依赖依赖了本bean，则另外那个bean执行ObjectFactory(getEarlyBeanReference)提交得到一个AOP之后的代理对象(如果有AOP的话，如果无需AOP，则直接得到一个原始对象)

ObjectFactory(getEarlyBeanReference)

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
	Object exposedObject = bean;
	if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
				SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
				exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
			}
		}
	}
	return exposedObject;
}
```



```java
// InstantiationAwareBeanPostProcessorAdapter
@Override
public Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
	return bean;
}

// AbstractAutoProxyCreator
@Override
public Object getEarlyBeanReference(Object bean, String beanName) {
	Object cacheKey = getCacheKey(bean.getClass(), beanName);
	this.earlyProxyReferences.put(cacheKey, bean);
	return wrapIfNecessary(bean, beanName, cacheKey);
}
```

而BeanPostProcessor的执行在Bean的生命周期中是处于属性注入之后的



**1、一级缓存：Map<String, Object> singletonObjects：**

（1）第一级缓存的作用：

- 用于存储单例模式下创建的Bean实例（已经创建完毕）。
- 该缓存是对外使用的，指的就是使用Spring框架的程序员。

（2）存储什么数据？

- K：bean的名称
- V：bean的实例对象（有代理对象则指的是代理对象，已经创建完毕）

**2、第二级缓存：Map<String, Object> earlySingletonObjects：**

（1）第二级缓存的作用：

- 用于存储单例模式下创建的Bean实例（该Bean被提前暴露的引用，该Bean还在创建中）。
- 该缓存是对内使用的，指的就是Spring框架内部逻辑使用该缓存。
- 为了解决第一个classA引用最终如何替换为代理对象的问题（如果有代理对象）

**3、第三级缓存：Map<String, ObjectFactory<?>> singletonFactories：**

（1）第三级缓存的作用：

- 通过ObjectFactory对象来存储单例模式下提前暴露的Bean实例的引用（正在创建中）。
- 该缓存是对内使用的，指的就是Spring框架内部逻辑使用该缓存。
- 此缓存是解决循环依赖最大的功臣

（2）存储什么数据？

- K：bean的名称
- V：ObjectFactory，该对象持有提前暴露的bean的引用

其实还要一个缓存，就是earlyProxyReferences，它用来记录某个原始对象是否进行过AOP了

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/5e0670114b787860bd190fab3ad91629.png)

而AOP可以说是Spring中除开IOC的另外一大功能，而循环依赖又是属于IOC范畴的，所以这两大功能想要并存，Spring需要特殊处理。

如何处理的，就是利用了第三级缓存**singletonFactories**。

大致过程：

从singletonFactories根据beanName得到一个ObjectFactory，然后执行ObjectFactory（getEarlyBeanReference）方法，此时会得到一个A原始对象经过AOP之后的代理对象，然后把该代理对象放入earlySingletonObjects中，然后属性填充，判断是否需要代理，在经历BeanPostProcessor处理即可放入singletonObjects中

总结一下三级缓存：

\1. singletonObjects：缓存某个beanName对应的经过了完整生命周期的bean

\2. earlySingletonObjects：缓存提前拿原始对象进行了AOP之后得到的代理对象，原始对象还没有进行属性注入和后续的BeanPostProcessor等生命周期

\3. singletonFactories：缓存的是一个ObjectFactory，主要用来去生成原始对象进行了AOP之后得到的代理对象，在每个Bean的生成过程中，都会提前暴露一个工厂，这个工厂可能用到，也可能用不到，如果没有出现循环依赖依赖本bean，那么这个工厂无用，本bean按照自己的生命周期执行，执行完后直接把本bean放入singletonObjects中即可，如果出现了循环依赖依赖了本bean，则另外那个bean执行ObjectFactory提交得到一个AOP之后的代理对象(如果有AOP的话，如果无需AOP，则直接得到一个原始对象)。

\4. 其实还要一个缓存，就是earlyProxyReferences，它用来记录某个原始对象是否进行过AOP了。



### 中间件集成

Import 可以用来简单加载类与配置类。也可以用于加载自定义的ImportSelector（动态加载字符串，搭配Enable使用），ImportBeanDefinitionRegistrar（动态直接加载类）

一些中间件如何完成Spring集成的呢？自定义注解搭配自定义注解处理器，<br>自定义处理器逻辑：通过ClassPathBeanDefinitionScanner自定义扫描器。完成类的搜集，动态加载类到ApplicationContext。<br>问题来了，自定义注解处理器如何被加载到Spring处理逻辑中，要么加注解@Component或者@Import

### Bean的生命周期

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/spring-framework-ioc-source-102.png)

图片速记

主线就是Bean的实例化和初始化的俩个过程，其中实例化就是反射调用构造方法生成对象，然后给属性赋值。初始化就是自定义Bean的init初始化方法。

spring为我们提供了若干后置处理器让开发者充分介入上述过程。

1. InstantiationAwareBeanPostProcessor在实例化前&后，让我们介入到Bean生成前后，InstantiationAwareBeanPostProcessor
2. Aware通知接口，在我们知道实例化和初始化中间的中间信息
3. BeanPostProcessor在初始化的前后。在这里Bean已经生成，故BeanPostProcessor

~~~txt
/* 
// 在Bean的实例化之前
execute InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation for user 
// 构造方法
execute User#new User()
// set 赋值
execute User#setName({})lushixiang
execute User#setAge({})24
// 在对象的实例化之后
execute InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation for user
// 对象实例化后属性的后置处理
execute InstantiationAwareBeanPostProcessor#postProcessProperties for user
// Bean 的后置通知
execute BeanNameAware#setBeanName
execute BeanFactoryAware#setBeanFactory
execute ApplicationContextAware#setApplicationContext

// 在Bean的初始化之前
MyInstantiationAwareBeanPostProcessor's postProcessBeforeInitialization user

// Bean自定义的初始化内容
execute InitializingBean#afterPropertiesSet
execute User#doInit

// 在Bean的初始化之后
MyInstantiationAwareBeanPostProcessor's postProcessAfterInitialization for user
 */
~~~

## AOP

> Aspect Oriented programming 面向切面（连接点，切入点，通知）编程。通过预编译和运行期动态代理实现程序的横向代码重复问题，解决了传统的基于继承的纵向的导致的重复问题

### AOP术语

连接点，切入点，通知，切面，引入，目标对象，织入，AOP代理

通知类型： 前置通知，后置通知，环绕通知，最终通知，异常通知

### Spring AOP和AspectJ的区别

1. AspectJ是更强意义上的AOP框架，也是事实上的AOP标准。
2. Aspect是主要通过预编译完成即编译期注入，Spring AOP完全是纯Java实现，基于运行期动态代理生成，主要通过JDK动态代理和CGLIB代理

### 配置AOP

> ```txt
> &&：要求连接点同时匹配两个切入点表达式
> ||：要求连接点匹配任意个切入点表达式
> !:：要求连接点不匹配指定的切入点表达式
> ```

#### XML schema 配置动态代理

```xml
    <aop:config>
        <!-- 配置切面 涉及到增强点 -->
        <aop:aspect ref="logAspect">
            <!-- 配置切入点 -->
            <aop:pointcut id="pointCutMethod" expression="execution(* tech.pdai.springframework.service.*.*(..))"/>
            <!-- 环绕通知 -->
            <aop:around method="doAround" pointcut-ref="pointCutMethod"/>
            <!-- 前置通知 -->
            <aop:before method="doBefore" pointcut-ref="pointCutMethod"/>
            <!-- 后置通知；returning属性：用于设置后置通知的第二个参数的名称，类型是Object -->
            <aop:after-returning method="doAfterReturning" pointcut-ref="pointCutMethod" returning="result"/>
            <!-- 异常通知：如果没有异常，将不会执行增强；throwing属性：用于设置通知第二个参数的的名称、类型-->
            <aop:after-throwing method="doAfterThrowing" pointcut-ref="pointCutMethod" throwing="e"/>
            <!-- 最终通知 -->
            <aop:after method="doAfter" pointcut-ref="pointCutMethod"/>
        </aop:aspect>
    </aop:config>
```

#### AspectJ 注解配置动态代理

```java 
    /**
     * define point cut 其实这样和具体的类名是不关联的
     */
    @Pointcut("execution(* tech.pdai.springframework.service.*.*(..))")
    private void pointCutMethod() {
    }


    /**
     * 环绕通知.
     *
     * @param pjp pjp
     * @return obj
     * @throws Throwable exception
     */
    @Around("pointCutMethod()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("-----------------------");
        System.out.println("环绕通知: 进入方法");
        Object o = pjp.proceed();
        System.out.println("环绕通知: 退出方法");
        return o;
    }

    /**
     * 前置通知.
     */
    @Before("pointCutMethod()")
    public void doBefore() {
        System.out.println("前置通知");
    }


    /**
     * 后置通知.
     *
     * @param result return val
     */
    @AfterReturning(pointcut = "pointCutMethod()", returning = "result")
    public void doAfterReturning(String result) {
        System.out.println("后置通知, 返回值: " + result);
    }

    /**
     * 异常通知.
     *
     * @param e exception
     */
    @AfterThrowing(pointcut = "pointCutMethod()", throwing = "e")
    public void doAfterThrowing(Exception e) {
        System.out.println("异常通知, 异常: " + e.getMessage());
    }

    /**
     * 最终通知.
     */
    @After("pointCutMethod()")
    public void doAfter() {
        System.out.println("最终通知");
    }
```

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/spring-framework-aop-7.png)

| Spring AOP                                       | AspectJ                                                      |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 在纯 Java 中实现                                 | 使用 Java 编程语言的扩展实现                                 |
| 不需要单独的编译过程                             | 除非设置 LTW，否则需要 AspectJ 编译器 (ajc)                  |
| 只能使用运行时织入                               | 运行时织入不可用。支持编译时、编译后和加载时织入             |
| 功能不强-仅支持方法级编织                        | 更强大 - 可以编织字段、方法、构造函数、静态初始值设定项、最终类/方法等......。 |
| 只能在由 Spring 容器管理的 bean 上实现           | 可以在所有域对象上实现                                       |
| 仅支持方法执行切入点                             | 支持所有切入点                                               |
| 代理是由目标对象创建的, 并且切面应用在这些代理上 | 在执行应用程序之前 (在运行时) 前, 各方面直接在代码中进行织入 |
| 比 AspectJ 慢多了                                | 更好的性能                                                   |
| 易于学习和应用                                   | 相对于 Spring AOP 来说更复杂                                 |

#### spring AOP 

#### Spring AOP还是完全用AspectJ？

以下Spring官方的回答：（总结来说就是 **Spring AOP更易用，AspectJ更强大**）。

- Spring AOP比完全使用AspectJ更加简单， 因为它不需要引入AspectJ的编译器／织入器到你开发和构建过程中。 如果你**仅仅需要在Spring bean上通知执行操作，那么Spring AOP是合适的选择**。
- 如果你需要通知domain对象或其它没有在Spring容器中管理的任意对象，那么你需要使用AspectJ。
- 如果你想通知除了简单的方法执行之外的连接点（如：调用连接点、字段get或set的连接点等等）， 也需要使用AspectJ。



# Spring Boot



> ApplicationContextHolder获取spring上下文？

## 注解工作原理

spring默认工作逻辑是拿到启动类的注解，这些注解被注解处理器处理。进而完成处理逻辑。

## 常用注解

| Annotation Name               | description                                                  | tips                               |
| ----------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| @SpringBootApplication        | 标记启动类                                                   |                                    |
|**AutoWire**|**AutoWire**|**AutoWire**|
| @EnableAutoConfigration       | 开启各依赖（src/main/resources的META-INF/spring.factories）的自动注入 |                                    |
| @Vaule                        | 根据配置文件完成自动注入                                     | @Value("${enable:false}") |
| @ConfigrationProperties       | 加载指定的配置文件完成自动注入                               | @configrationProperties(prefix='') |
| @EnableConfigrationProperties | 使EnableConfigurationProperties生效，并且能从Context中获取该Bean |                                    |
| **Web**                       | **Web**                                                      | **Web**                            |
| @RequesMapping                | 自定义映射访问路径                                           | @RequestMapping("path/path")       |
| @PathVariable                 |                                                              |                                    |
| @RequestParam                 |                                                              | @PathVariable("VarName") |
| @Controller                   |                                                              |                                    |
| @Service                      |                                                              |                                    |
| @Component                    |                                                              |                                    |
| @Repository                   |                                                              |                                    |
| @ResponseBody                 |                                                              |                                    |
| @RestController               |                                                              |                                    |
| **Bean**                      | **Bean**                                                     | **Bean**                           |
| @Bean                         |                                                              | @Bean(name="BeanName") |
| @PostConstruct                |                                                              |                                    |
|                               |                                                              |                                    |
|  |                                                              |                                    |
| @Import |                                                              | @Import(JavaClass.class) |
|                               |                                                              |                                    |
|                               |                                                              |                                    |
|                               |                                                              |                                    |
|                               |                                                              |                                    |
| **Predict** |                                                              |                                    |
| @ConditionOnProperty |                                                              |                                    |
|@ConditionOnExpression||@ConditionOnExpression("${enable:false}")|
| @ConditionOnClass |                                                              | @ConditionOnClass({JavaType.class}) |
| @ConditionOnMissClass |                                                              | @ConditionOnMissClass({JavaType.class}) |
| @ConditionOnMissBean | | @ConditionOnMissBean(name="") |
|  | | |
|  | | |
|  | | |
|  | | |
|  | | |
|  | | |
|  | | |
|  | | |

## 接口常见问题&解决方案

统一接口封装

参数校验

接口幂等

事务处理

外部接口调用

统一异常处理

接口文档

接口限流

接口版本控制

## 数据库访问&分库分表处理

### Mybatis

自动提交

## 定时任务schedule

## 文件下载上传

## actuator监控工具

