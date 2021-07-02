# 为什么使用dubbo？

**微服务架构趋势的演变**

## 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

单一应用的痛点在于系统耦合度较高，往往只是一处的改动，而需要影响这个服务系统重新部署；且如果过大，本地调试都需要启动特别久的时间，在开发效率上来讲，会花费更多的时间在应用的启动上；

## 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，提升效率的方法之一是将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

## 分布式服务架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

## 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

基于以上需求和背景，dubbo的产生可以解决以上问题。

# dubbo介绍

## dubbo架构

![dubbo架构图](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20210621163610.png)

| 节点     | 角色说明                               |
| -------- | -------------------------------------- |
| rovider  | 暴露服务的服务提供方                   |
| onsumer  | 调用远程服务的服务消费方               |
| egistry  | 服务注册与发现的注册中心               |
| onitor   | 统计服务的调用次数和调用时间的监控中心 |
| ontainer | 服务运行容器                           |

## 支持的配置

启动检查、重试次数、通讯端口、分组名称、超时时间、集群容错、负载均衡等等，重点记录下集群容错和负载均衡

## 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为`failover`重试。

![dubbo invoke过程](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20210629114121.png)

- 这里的 Invoker 是 Provider 的一个可调用 Service 的抽象，Invoker 封装了 Provider 地址及 Service 接口信息
- Directory 代表多个 Invoker，可以把它看成 List<Invoker> ，但与 List 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- Cluster 将 Directory 中的多个 Invoker 伪装成一个 Invoker，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- Router 负责从多个 Invoker 中按路由规则选出子集，比如读写分离，应用隔离等
- LoadBalance 负责从多个 Invoker 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选

具体的集群容错机制如下：

1、Failover Cluster  失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。

2、Failfast Cluster 快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

3、Failsafe Cluster 失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

4、Failback Cluster 失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

5、Forking Cluster 并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。

6、Broadcast Cluster 广播调用所有提供者，逐个调用，任意一台报错则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息

## 负载均衡

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为`random`随机调用。

1、Random LoadBalance 随机，按权重设置随机概率。在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

2、RoundRobin LoadBalance
轮询，按公约后的权重设置轮询比率。存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二上

3、LeastActive LoadBalance 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

4、ConsistentHash LoadBalance 一致性 Hash，相同参数的请求总是发到同一提供者。当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。

# dubbo超时

划重点，dubbo超时是针对消费端的，毕竟dubbo通过netty实现的NIO模式。

经测试，默认dubbo的超时时间是1s，在超时之后，消费端会抛出timeout这类异常，如下：

```java
com.alibaba.dubbo.rpc.RpcException: Invoke remote method timeout. method: addUser, provider: dubbo://10.0.75.1:21881/com.example.service.DemoService?anyhost=true&application=demo-consumer&check=false&default.check=false&dubbo=2.6.2&generic=false&interface=com.example.service.DemoService&methods=findUser,addUser&pid=20384&register.ip=10.0.75.1&remote.timestamp=1569415296300&side=consumer&timestamp=1569415630331, cause: Waiting server-side response timeout by scan timer. start time: 2019-09-25 20:47:17.394, end time: 2019-09-25 20:47:18.401,
```

由于dubbo的集群容错机制，他会继续重试两次，直到在设置次数范围内全部失败才抛出该异常，在项目中使用时，设置全局重试次数=0，在消费端通过@Refrence引入的provider服务时，可以进行单独的重试次数设定，比如查询类的可以设置重试1次；但操作类，例如新增数据要求幂等性时，则默认走全局配置的重试次数即可。

# dubbo异常处理

在服务端抛出异常后，默认情况下，如果不做特别设定，会在dubbo的exceptionHandler内的invoke函数来对异常进行一次包装，这个包装会把本身抛出的异常当成一个rpcException的msg来进行处理，所以，如果项目内有自定义的异常（服务端定义，而消费端并未定义的异常），会通过dubbo后成为另一种异常。

那么，在有自定义异常，并且想让服务端直接把异常抛到消费端怎么办？  

首先，消费端和服务端最好定义异常统一，最起码两个工程内都认可，这里说的认可就是可以序列化，比如你这样定义：

```java
public class RestStatusException extends RuntimeException implements Serializable
```

在消费端可能要在application内加一列这个配置：

```java
dubbo.provider.filter=-exception
```

加上这个设置之后，provider不打印异常堆栈信息，不通过dubbo的ExceptionFilter的invoke进行异常包装而导致消费端返回异常一大坨字符给用户，这样做暴露方会把所有的原始异常暴露给使用者，使多个dubbo服务仿佛是一个主体（一个项目）那样简单来进行异常管理。

# dubbo为什么要有超时机制？

最简单的回答，比如没有超时时间的限制，dubbo采用netty框架来实现服务于服务之间的通信，默认处理线程池是200个大小，设想一下，如果没有超时时间，某一个或者多个模块的处理时间较长，调用频率较高时，容易出现拒绝请求这类的问题，同时netty的调用也注定了客户端会不断的轮询去获取服务端响应的结果；总结就是两句，一是避免调用服务端被拒绝，二是避免客户端死循环。

# 注册中心挂掉后服务之间可以通信吗

可以，因为dubbo远程调用时基于netty实现通信，注册中心只是实现了动态注册服务以及动态订阅最新服务，在服务启动时，消费者端会将注册中心内注册的服务地址和端口在本地缓存一份，所以即使注册中心挂了仍然不影响通信，但是如果这是服务的ip、端口发生变化的话，由于消费端缓存的是老的ip和端口，这种情况下才会发生异常。

