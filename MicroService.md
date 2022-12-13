# 微服务术语

## 网关

## 注册中心

## Nginx



# 网关

## Gateway

> 当服务被拆分之后，随之带来了调用问题，路径，权限，跨域等
>
> 引入Gateway作为统一入口	





# 注册中心

## Nacos

### 配置中心

#### 属性自动刷新

##### 1: 代码实现

##### 2: 代码原理

##### 3: 遇到的一些问题

1. @Value无法自动刷新，增加RefreshScope，导致初始注入即为null<br>
2. 

### 注册中心

# log

### 对Java日志组件选型的建议

slf4j已经成为了Java日志组件的明星选手，可以完美替代JCL，使用JCL桥接库也能完美兼容一切使用JCL作为日志门面的类库，现在的新系统已经没有不使用slf4j作为日志API的理由了。

日志记录服务方面，log4j在功能上输于logback和log4j2，在性能方面log4j2则全面超越log4j和logback。所以新系统应该在logback和log4j2中做出选择，对于性能有很高要求的系统，应优先考虑log4j2



# RPC

***RPC (remote process call) 远程过程调用***

1. 双方持有相同接口，调用方通过本地JDK代理实现提供实现类
2. 本地实现类通过对象序列化，将执行参数值，参数属性，接口，方法传递给远程服务线程上（传递方式，socket，http，很多自定义协议都可）
3. 远程服务线程反序列化，拿到对象，然后执行方法。对返回值进行序列化，传回给调用方
4. 本地代理对象拿到返回，反序列化，返回完成。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-54c36e07764895d3da67c7fc624789c5_720w.jpg)

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-d690accc669d726fe122d6da6caa75a1_720w.jpg)

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-c0088ff8964a97f232081b5b2a08c068_720w.png)