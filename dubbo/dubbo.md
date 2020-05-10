

[TOC]





### 一、Dubbo是什么？
Dubbo是阿里巴巴开源的基于 Java 的高性能 RPC（一种远程调用） 分布式服务框架（SOA），致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。



### 二、为什么要用Dubbo？

 因为是阿里开源项目，国内很多互联网公司都在用，已经经过很多线上考验。内部使用了 Netty、Zookeeper，保证了高性能高可用性。

```txt
1、使用Dubbo可以将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，可用于提高业务复用
灵活扩展，使前端应用能更快速的响应多变的市场需求。
2、分布式架构可以承受更大规模的并发流量。
```



### 三、Dubbo 和 Spring Cloud 有什么区别？

```txt
1、通信方式不同：Dubbo 使用的是 RPC 通信，而Spring Cloud 使用的是HTTP RESTFul 方式。
Spring Cloud提供了微服务的一整套解决方案：服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等

2、组成不一样：
dubbo的服务注册中心为Zookeerper，服务监控中心为dubbo-monitor,无消息总线，服务跟踪、批量任务等组件
spring-cloud的服务注册中心为spring-cloud netflix  enruka，服务监控中心为spring-boot admin,有消息总线，

```



### 四、节点角色说明



![dubbo-architecture](https://gitee.com/HKbxOIC/imgs/raw/master/PhDocs/dubbo/dubbo-architecture.jpg)

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |



### 五、注册中心

>   Zookeeper, Multicast, Nacos, Redis, Simple
>
>   推荐使用Zookeeper作为注册中心
>



### 六、通讯协议

 

>   1.dubbo
>
>   Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况
>
>   2.rm
>   RMI协议采用JDK标准的java.rmi.\* 实现，采用阻塞式短连接和JDK标准序列化方式
>
>   3.Hessian
>   Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌Jetty作为服务器实现
>
>   4.http
>   采用Spring的HttpInvoker实现
>
>   5.Webservice
>   基于CXF的frontend-simple和transports-http实现
>
>   6.thift, redis, grpc, rest, memccached



### 七、Dubbo内置了哪几种服务容器？

```txt
三种服务容器：
1、Spring Container
2、Jetty Container
3、Log4j Container
```



### 八、在 Provider 上可以配置的 Consumer 端的属性有哪些？

```txt
1、timeout：方法调用超时
2、retries：失败重试次数，默认重试 2 次
3、loadbalance：负载均衡算法，默认随机
4、actives 消费者端，最大并发调用限制
```



### 九、Dubbo启动时如果依赖的服务不可用会怎样？

>   Dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，默认
>    check="true"，可以通过 check="false" 关闭检查。



### 十、Dubbo推荐使用什么序列化框架，你知道的还有哪些？

```txt
推荐使用Hessian序列化，还有Duddo、FastJson、Java自带序列化；
```



### 十一、Dubbo默认使用的是什么通信框架，还有别的选择吗？

```txt
Dubbo 默认使用 Netty 框架，也是推荐的选择，另外内容还集成有Mina、Grizzly。
```



### 十二、服务提供者能实现失效踢出是什么原理？

```txt
服务失效踢出基于zookeeper的临时节点原理。
```



### 十三、Dubbo服务之间的调用是阻塞的吗？

```txt
默认是同步等待结果阻塞的，支持异步调用。
Dubbo 是基于 NIO 的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对
多线程开销较小，异步调用会返回一个 Future 对象。
```

Dubbo暂时不支持分布式事务。



### 十五、Dubbo的管理控制台能做什么？

```txt
管理控制台主要包含：路由规则，动态配置，服务降级，访问控制，权重调整，负载均衡，等管理功能。
注：dubbo源码中的dubbo-admin模块打成war包，发布运行即可得到dubbo控制管理界面。
```



### 十六、Dubbo 服务暴露的过程

```bash
Dubbo 会在 Spring 实例化完 bean 之后，在刷新容器最后一步发布 ContextRefreshEvent 事件的时候，通知
实现了 ApplicationListener 的 ServiceBean 类进行回调 onApplicationEvent 事件方法，Dubbo 会在这个方法
中调用 ServiceBean 父类 ServiceConfig 的 export 方法，而该方法真正实现了服务的（异步或者非异步）发
布
```





### 十七、集群提供了哪些负载均衡策略？

> 1.Random LoadBalance
>   随机选取提供者策略，有利于动态调整提供者权重。截面碰撞率高，调用次数越多，分布越均匀；
> 2.RoundRobin LoadBalance
>   轮循选取提供者策略，平均分布，但是存在请求累积的问题；
> 3.LeastActive LoadBalance
>   最少活跃调用策略，解决慢提供者接收更少的请求；
> 4.ConsistantHash LoadBalance
>   一致性Hash策略，使相同参数请求总是发到同一提供者，一台机器宕机，可以基于虚拟节点，分摊至其他提供者，避免引起提供者的剧烈变动；
> 缺省时为Random随机调用

 

### 十八、集群容错方案有哪些？

> 1.Failover Cluster
>   失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。
> 2.Failfast Cluster
>   快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。
> 3.Failsafe Cluster
>   失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。
> 4.Failback Cluster
>   失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。
> 5.Forking Cluster
>   并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks=”2″ 来设置最大并行数。
> 6.Broadcast Cluster
>   广播调用所有提供者，逐个调用，任意一台报错则报错 。通常用于通知所有提供者更新缓存或日志等本地资源信息。



### 十九、Dubbo的核心功能？

主要是以下3个核心功能：

>   Remoting：网络通信框架，提供对多种NIO框架抽象封装，包括“同步转异步”和“请求-响应”模式的信息交换方式。
>
>   Cluster：服务框架，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
>
>    Registry：服务注册，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

 