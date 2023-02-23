# 微服务架构基本描述



# 微服务组件

> 参考地址：[微服务架构设计](https://gudaoxuri.gitbook.io/microservices-architecture/)

## 网关

### Gateway

> 当服务被拆分之后，随之带来了调用问题，路径，权限，跨域等
>
> 引入Gateway作为统一入口	





## 注册中心

### Nacos

## 服务的熔断与降级

> 1. 设置超时时间
> 2. 设置最大尝试次数
> 3. 设置请求超时N次后在X时间不再请求实现熔断，X时间后恢复M%的请求，如果M%的请求都成功（未超时）则恢复正常关闭熔断，否则再熔断Y时间，依此循环
> 4. 引入限流和排队功能搭配灾备

## 配置中心

### 配置中心设计

> 集中管理与配置切片->动态更新->配置更新push/pull->版本管理与审计->高可用&弱依赖

#### 属性自动刷新

##### 1: 代码实现

##### 2: 代码原理

##### 3: 遇到的一些问题

1. @Value无法自动刷新，增加RefreshScope，导致初始注入即为null<br>

## 注册中心

## log

### 对Java日志组件选型的建议

slf4j已经成为了Java日志组件的明星选手，可以完美替代JCL，使用JCL桥接库也能完美兼容一切使用JCL作为日志门面的类库，现在的新系统已经没有不使用slf4j作为日志API的理由了。

日志记录服务方面，log4j在功能上输于logback和log4j2，在性能方面log4j2则全面超越log4j和logback。所以新系统应该在logback和log4j2中做出选择，对于性能有很高要求的系统，应优先考虑log4j2



## RPC

***RPC (remote process call) 远程过程调用***

1. 双方持有相同接口，调用方通过本地JDK代理实现提供实现类
2. 本地实现类通过对象序列化，将执行参数值，参数属性，接口，方法传递给远程服务线程上（传递方式，socket，http，很多自定义协议都可）
3. 远程服务线程反序列化，拿到对象，然后执行方法。对返回值进行序列化，传回给调用方
4. 本地代理对象拿到返回，反序列化，返回完成。

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-54c36e07764895d3da67c7fc624789c5_720w.jpg)

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-d690accc669d726fe122d6da6caa75a1_720w.jpg)

![img](https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-c0088ff8964a97f232081b5b2a08c068_720w.png)

# 分布式系统

## 幂等与去重

**处理逻辑**：记录请求。判断当前请求是否重复，重复则报错返回，否则正常执行。处理完成之后删除对应请求以便重试。这一流程要求请求必须有可区分是否是重试的标识，并且对业务处理的前置判断和后置更新最好在框架层面实现以对业务操作透明化，做到最小化的业务逻辑侵入。

**实现方案：**

- 借助redis携带token。rest语义上幂等直接使用URI做为请求Token再加上过期时间
- 使用MQ，

**解决方案：**数据库去重，全局锁，状态机

**注意事项：**微服务分布式下，要么使用事务保证幂等，要么要求被调用的接口也是幂等的

## 分布式锁

**正常JVM锁和分布式锁：**

- 公平与非公平锁实现顺序和优先队列，读写锁提高效率，可重入锁避免死锁，JVM提供无锁，偏向，轻量，重量级锁
- 分布式锁考虑锁的超时与释放，性能和高可用，数据一致性

**实现方式：**

- 数据库
- redis
- zookeeper

## 分布式事务