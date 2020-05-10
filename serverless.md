# Serverless 架构

1.什么是Serverless?

```txt
Serverless不代表再也不需要服务器了，而是说：开发者再也不用过多考虑服务器的问题，计算资源作为服务而不是服务器的概念出现。Serverless是一种构建和管理基于微服务架构的完整流程，允许你在服务部署级别而不是服务器部署级别来管理你的应用部署，你甚至可以管理某个具体功能或端口的部署，这就能让开发者快速迭代，更快速地开发软件。
```

2.serverless特点?

```txt
1.Serverless意味无维护，Serverless不代表完全去除服务器，而是代表去除有关对服务器运行状态的关心和担心，它们是否在工作，应用是否跑起来正常运行等等。Serverless代表的是你不要关心运营维护问题。有了Serverless，可以几乎无需Devops了。
2.Serverless不代表某个具体技术，有些人会给他们的语言框架取名为Serverless，Serverless其实去除维护的担心，如果你了解某个具体服务器技术当然有帮助，但不是必须的。
3.Serverless中的服务或功能代表的只是微功能或微服务，Serverless是思维方式的转变，从过去：“构建一个框架运行在一台服务器上，对多个事件进行响应。”变为：“构建或使用一个微服务或微功能来响应一个事件。”，你可以使用 django or node.js 和express等实现，但是serverless本身超越这些框架概念。框架变得也不那么重要了。
4.Serverless规模扩展性方面由于充分利用云计算的特点，因此其扩展是平滑的，同时由于Serverless是基于微服务的，而一些微功能微服务的云计算是零收费，这样有助于降低整体运营费用。
```

3.应用场景?

```txt
事件驱动以及响应式架构
IoT 物联网场景中低频请求
请求对及时响应需求不够
固定时间触发计算资源利用低的业务
流量突发场景
比如短时间大流量视频转码
短周期内的流量峰值
跨云与混合云场
边缘计算
```

4.云计算经历了从 IDC -> IaaS -> PaaS -> Serverless/FaaS 的发展历程?

```txt
IaaS(Infrastructure as a Service) 
基础设施即服务，服务商提供底层/物理层基础设施资源（服务器，数据中心，环境控制，电源，服务器机房），用户需要通过 IaaS 提供的服务平台购买虚拟资源，选择操作系统、安装软件、部署程序、监控应用。
PaaS (Platform as a Service)
平台即服务，服务商提供基础设施底层服务，提供操作系统（Windows，Linux）、数据库服务器、Web 服务器、负载均衡器和其他中间件，相对于 IaaS 客户仅仅需要自己控制上层的应用程序部署与应用托管的环境。
SaaS (Software as a Service) 
软件即服务， 服务商提供基于软件的解决方案，如 OA、CRM、MIS、ERP、HRM、CM、Office 365、iCloud 等，客户不需考虑任何形式的专业技术知识，只需要通过服务商平台获取软件使用即可。
BaaS (Backend as a Service)
后端即服务，服务商为客户(开发者)提供整合云后端的服务，如提供文件存储、数据存储、推送服务、身份验证服务等功能，以帮助开发者快速开发应用。
FaaS (Function as a Service)
函数即服务，服务商提供一个平台，允许客户开发、运行和管理应用程序功能，而无需构建和维护基础架构。 按照此模型构建应用程序是实现“无服务器”体系结构的一种方式，通常在构建微服务应用程序时使用

从 IDC → IaaS，用户不用关注真实的物理资源。
从 IaaS → PaaS，用户不再关注操作系统，数据库，中间件等基础软件。
从 PaaS → BaaS/FaaS, 用户可以很少甚至不用关注 backend，app 可以简化为一个单页面程序。
```

5.Serverless/FaaS 模型?

```txt
Serverless 是基于事件驱动的编程范型，其底层的计算平台一般为轻量计算，比如容器计算 Docker。
```

6.Serverless 价值与影响?

```txt
运营成本，Serverless 将用户的服务器、数据库、中间件委托于 BaaS/FaaS，用户将不再参与基础设施及软件的维护，尤其在大规模的集群运营上成本大幅度降低；
开发成本，对比 IaaS 或者 PaaS 平台的服务器或者操作系统，Serverless 的架构中，用户操作的是服务化的组件比如存储服务，授权服务等，可以缩短开发周期，降低开发难度。
```

7.真正按需计费?

```txt
Serverless/FaaS 区别于 IaaS/PaaS 预先分配计算资源的计费方式，其计费方式通常是按请求次数及运行时间，一方面可以最大程度利用资源，另一方面真正的按需计费降低用户的资源成本。
```

8.**NoOps**?

```txt
运维的发展经历了人肉运维、自动化运维、DevOps、AiOps 等，而 Serverless 带来一种新的运维模式，这种模式下用户需要管理的只有 Code 可以认为 NoOps。
```

9.Serverless Container?

```txt
前主流的 Serverless/FaaS 框架，如 AWS Lambda、IBM OpenWhisk、Iron.io、阿里云函数计算分析来看，其底层的计算资源通常是 Docker 容器。可以认为 Serverless 构建于容器 (Docker) 之上！
```



[原文链接](https://www.cnblogs.com/limengchun/p/11936065.html)

