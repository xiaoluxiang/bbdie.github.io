# 微服务架构基本描述

# 分布式组件

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

<img src="https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-54c36e07764895d3da67c7fc624789c5_720w.jpg" alt="img" style="zoom:67%;" />

<img src="https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-d690accc669d726fe122d6da6caa75a1_720w.jpg" alt="img" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/xiaoluxiang/picCollect/main/workDesign/img/v2-c0088ff8964a97f232081b5b2a08c068_720w.png" alt="img" style="zoom:50%;" />

# 分布式系统

**分布式系统**定义：若干独立计算机共同对外提供服务，但对于系统用户来说，就像是一台计算机在提供服务

**分布式系统面临的问题：**全局时钟，网络异常（网络不可靠，且不确保顺序），节点宕机，结果三态，存储数据丢失。

**分布式系评价统指标：**性能，扩展性，高可用，一致性

**分布式系统理论基础：**CAP原理和BASE原理（AP的补充，关注最终一致性）

## 分布式一致性算法

> 参考地址：https://pdai.tech/md/algorithm/alg-domain-distribute-x-consistency-hash.html

**一致性Hash算法**

评价指标：平衡性（分配结果多均衡），单调性（节点增加，原有尽量不动），分散性（避免相同的内容被分配到不同的缓存区），负载（避免不同的内容被分配到相同的缓冲区

为了将映射空间设置成环形，为了防止动态平衡性，增加为实际节点增加诸多虚拟节点。

**paxos算法**

***prepare阶段：***proposer生成全局唯一递增的的proposal ID，携带该ID发送prepare请求。accept收到请求后做出不接受小于等于该ID的prepare请求，不接受小于当前ID的proposal请求。并返回自己当前最大proposal ID和value；

***accept阶段：***proposal在收到多数响应之后，选择proposal最大的ID的里的value（均为空则可以自定义），外加自己的proposal ID作为本次发起的提案。acceptor在不违背之前的承诺持久化最大的proposal ID和value；proposer在收到多数acceptor的accept之后，标志决议形成，通知learners。

Multi-Paxos算法：通过basic-paxos选出leader，由leader提proposal。leader节点宕机之后，重新选举。

**Raft算法：**

ZAB算法

## 幂等与去重

**处理逻辑**：记录请求。判断当前请求是否重复，重复则报错返回，否则正常执行。处理完成之后删除对应请求以便重试。这一流程要求请求必须有可区分是否是重试的标识，并且对业务处理的前置判断和后置更新最好在框架层面实现以对业务操作透明化，做到最小化的业务逻辑侵入。

**实现方案：**

- 借助redis携带token。rest语义上幂等直接使用URI做为请求Token再加上过期时间
- 使用MQ，

**解决方案：**数据库去重，全局锁，状态机

**注意事项：**微服务分布式下，要么使用事务保证幂等，要么要求被调用的接口也是幂等的

## 分布式锁

setnx set if not exists

**正常JVM锁和分布式锁：**

- 公平与非公平锁实现顺序和优先队列，读写锁提高效率，可重入锁避免死锁，JVM提供无锁，偏向，轻量，重量级锁
- 分布式锁考虑锁的超时与释放，性能和高可用，数据一致性

**实现方式：**

- 数据库 
- redis
- zookeeper

## 分布式事务

俩阶段提交