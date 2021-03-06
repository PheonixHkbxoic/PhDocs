



[TOC]





## 一、什么是API网关(API Gateway)

>   分布式服务架构、微服务架构与 API 网关
>   在微服务架构里，服务的粒度被进一步细分，各个业务服务可以被独立的设计、开发、测试、部署和管理。这时，各个独立部署单元可以用不同的开发测试团队维护，可以使用不同的编程语言和技术平台进行设计，这就要求必须使用一种语言和平 台无关的服务协议作为各个单元间的通讯方式。

![img](https://imgconvert.csdnimg.cn/aHR0cDovL2Nvcy5yYWluMTAyNC5jb20vbWFya2Rvd24vaW1hZ2UtMjAxOTEwMDgxNjAyMDY4MzIucG5n?x-oss-process=image/format,png)



### API网关的定义

网关的角色是作为一个API 架构，用来保护、增强和控制对于 API 服务的访问。

API网关是一个处于应用程序或服务（提供 REST API 接口服务）之前的系统，用来管理授权、访问控制和流量限制等，这样 REST API 接口服务就被 API网关保护起来，对所有的调用者透明。因此，隐藏在 API 网关后面的业务系统就可以专注于创建和管理服务，而不用去处理这些策略性的基础设施。



### API网关的职能

![img](https://imgconvert.csdnimg.cn/aHR0cDovL2Nvcy5yYWluMTAyNC5jb20vbWFya2Rvd24vaW1hZ2UtMjAxOTEwMDgxNjAzMjUwMjEucG5n?x-oss-process=image/format,png)



### API网关的分类与功能

![img](https://imgconvert.csdnimg.cn/aHR0cDovL2Nvcy5yYWluMTAyNC5jb20vbWFya2Rvd24vaW1hZ2UtMjAxOTEwMDgxNjAzNTQ5NTcucG5n?x-oss-process=image/format,png)



## 二、Gateway是什么

Spring Cloud Gateway是Spring官方基于Spring 5.0，Spring Boot 2.0和Project Reactor等技术开发的网关，Spring Cloud Gateway旨在为微服务架构提供一种简单而有效的统一的API路由管理方式。Spring Cloud Gateway作为Spring Cloud生态系中的网关，目标是替代ZUUL，其不仅提供统一的路由方式，并且基于Filter链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。



## 三、为什么用Gateway

Spring Cloud Gateway 可以看做是一个 Zuul 1.x 的升级版和代替品，比 Zuul 2 更早的使用 Netty 实现异步 IO，从而实现了一个简单、比 Zuul 1.x 更高效的、与 Spring Cloud 紧密配合的 API 网关。
Spring Cloud Gateway 里明确的区分了 Router 和 Filter，并且一个很大的特点是内置了非常多的开箱即用功能，并且都可以通过 SpringBoot 配置或者手工编码链式调用来使用。
比如内置了 10 种 Router，使得我们可以直接配置一下就可以随心所欲的根据 Header、或者 Path、或者 Host、或者 Query 来做路由。
比如区分了一般的 Filter 和全局 Filter，内置了 20 种 Filter 和 9 种全局 Filter，也都可以直接用。当然自定义 Filter 也非常方便。

**最重要的几个概念**

![img](https://imgconvert.csdnimg.cn/aHR0cDovL2Nvcy5yYWluMTAyNC5jb20vbWFya2Rvd24vaW1hZ2UtMjAxOTEwMDgxNjA3MTM4MjIucG5n?x-oss-process=image/format,png)



![img](https://imgconvert.csdnimg.cn/aHR0cDovL2Nvcy5yYWluMTAyNC5jb20vbWFya2Rvd24vaW1hZ2UtMjAxOTEwMDgxNjA4MDkxNDYucG5n?x-oss-process=image/format,png)



![img](https://imgconvert.csdnimg.cn/aHR0cDovL2Nvcy5yYWluMTAyNC5jb20vbWFya2Rvd24vaW1hZ2UtMjAxOTEwMDgxNjA4MjU3MzEucG5n?x-oss-process=image/format,png)



## 四、Gateway怎么用

说白了 Predicate 就是为了实现一组匹配规则，方便让请求过来找到对应的 Route 进行处理，接下来我们接下 Spring Cloud GateWay 内置几种 Predicate 的使用。

* 通过时间匹配	Before,Between,After
* 通过 Cookie 匹配 Cookiee
* 通过 Host 匹配 Host
* 通过请求方式匹配 Method
* 通过请求路径匹配 Path
* 通过请求参数匹配 Query
* 通过请求 ip 地址进行匹配

```yaml
spring:
  cloud:
    gateway:
      routes:
       - id: host_foo_path_headers_to_httpbin
        uri: http://ityouknow.com
        predicates:
        - Host=**.foo.org
        - Path=/headers
        - Method=GET
        - Header=X-Request-Id, \d+
        - Query=foo, ba.
        - Query=baz
        - Cookie=chocolate, ch.p
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```


各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。

一个请求满足多个路由的谓词条件时，请求只会被首个成功匹配的路由转发