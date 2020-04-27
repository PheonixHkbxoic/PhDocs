[TOC]



## 扩展点配置

### 来源：

Dubbo 的扩展点加载从 JDK 标准的 SPI (Service Provider Interface) 扩展点发现机制加强而来。

Dubbo 改进了 JDK 标准的 SPI 的以下问题：

-   JDK 标准的 SPI 会一次性实例化扩展点所有实现，如果有扩展实现初始化很耗时，但如果没用上也加载，会很浪费资源。
-   如果扩展点加载失败，连扩展点的名称都拿不到了。比如：JDK 标准的 ScriptEngine，通过 `getName()` 获取脚本类型的名称，但如果 RubyScriptEngine 因为所依赖的 jruby.jar 不存在，导致 RubyScriptEngine 类加载失败，这个失败原因被吃掉了，和 ruby 对应不起来，当用户执行 ruby 脚本时，会报不支持 ruby，而不是真正失败的原因。
-   增加了对扩展点 IoC 和 AOP 的支持，一个扩展点可以直接 setter 注入其它扩展点。

### 约定：

在扩展类的 jar 包内 [[1\]](http://dubbo.apache.org/zh-cn/docs/dev/SPI.html#fn1)，放置扩展点配置文件 `META-INF/dubbo/接口全限定名`，内容为：`配置名=扩展实现类全限定名`，多个实现类用换行符分隔。

### 示例：

以扩展 Dubbo 的协议为例，在协议的实现 jar 包内放置文本文件：`META-INF/dubbo/org.apache.dubbo.rpc.Protocol`，内容为：

```properties
xxx=com.alibaba.xxx.XxxProtocol
```

实现类内容 [[2\]](http://dubbo.apache.org/zh-cn/docs/dev/SPI.html#fn2)：

```java
package com.alibaba.xxx;
 
import org.apache.dubbo.rpc.Protocol;
 
public class XxxProtocol implements Protocol { 
    // ...
}
```

### 配置模块中的配置

Dubbo 配置模块中，扩展点均有对应配置属性或标签，通过配置指定使用哪个扩展实现。比如：

```xml
<dubbo:protocol name="xxx" />
```



[原文链接](http://dubbo.apache.org/zh-cn/docs/dev/SPI.html)