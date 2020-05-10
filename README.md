# PhDocs
知识文档

推荐：

[互联网-java-工程师进阶知识完全扫盲](https://doocs.github.io/advanced-java/#/?id=互联网-java-工程师进阶知识完全扫盲©)



分布式事务框架：seata, fescar



java诊断工具：arthas



## 技术栈

1.  mysql 基本语法 索引结构B+及与B树区别 索引失效情况 最左前缀 Innodb与MyIsam区别  
    再深入的话 ACID 隔离级别 事务实现 mvcc  X锁S锁 行锁  gap锁 Next-Key锁，主从复制及undo redo log   

2.  mybatis 基本语法 如何实现=>AOP 动态代理， 事务如何隔离ThreadLocal，   
    深入的话 整个解析处理流程 插件原理  

3.  spring IOC AOP 循环依赖 ABA问题 生命周期  
    深入的话 整个启动流程 常用注解解析过程 各种注册器 后置处理器 事件监听器 Bean的解析及实例化过程 SPI  

4.  springboot 常用注解 自动配置原理 启动Tomcat的过程  
    深入的话 springboot的细节很多 可以结合给的资料 本质还在spring及其注解  
    tomcat的结构 类加载原理 tomcat是否违反了类加载原理 为什么  热启动原理等   

5.  redis 常用数据结构及使用场景 如微博用户信息map，点赞incre，关注set，互相关注set交集，是否关注大V用bitmap， 是否存在 用BloomFilter 等。   
    深入的话 缓存淘汰策略 分布式锁 rdb与aof 为什么说是单线程   
    常见问题 缓存失效 缓存穿透 缓存雪崩 缓存与数据库双写一致性   
    redis主从 集群 哨兵 一致性哈希 hash slot  

6.  mq 各mq对比 技术选型 高可用 重复消费 有序性 延时及过期失效 消息积压  

7.  jdk相关 hashmap concurrenthashmap reentrantlock aqs 线程池 threadlocal synchronized valotile cas JMM内存模型  happend-before as-if-serial 指令重排序与屏障 这些实现原理 源码的大概步骤都要弄清楚  

8.  分布式理论及问题 CAP BASE 拜占庭问题 paxos zab(zookeeper原理 注册中心 分布式锁) raft 分布式事务  
    分布式配置 路由 网关 服务注册 降级 容错 治理等    

    SpringCloud => gateway hystrix loaadbalancer nacos openfeign ribbon resilience4j sentinel  

9.  其他框架 Disruptor Tomcat ES Kafka Zookeeper Netty Dubbo Shardingjdbc k8s Docker gRpc Thrift  Arthas Mycat