# Spring

## IOC

### 循环依赖

BeanFactory 创建后，为了解决循环依赖的问题，存在三级缓存

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