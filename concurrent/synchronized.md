

[TOC]



## 一、Synchronized基本知识

1.synchronized锁的是对象

2.位置：在方法上锁住的是当前对象，如果是静态方法上锁住的是当前类 或者

```java
synchronized(obj){// 对obj对象加锁
    
    // TODO 业务代码逻辑 如：i++;
    
}// 释放锁
```

3.对象组成：

markword, KlassPointer, [如果是数组这里还会有数据的长度信息,]实例数据, 对齐填充

32位JVM中Object占用8个字节：markword为4个字节，KlassPointer为4个字节

64位JVM中Object占用8个字节：markword为*个字节，KlassPointer为8个字节(如果开启指针压缩则是4个字节)

java对齐为8*N，静态属性不算在对象大小内

![这里写图片描述](https://gitee.com/HKbxOIC/imgs/raw/master/PhDocs/concurrent\java-object-structure.png)

查看类及对象占用大小：org.openjdk.jol:jol-core这个jar包来查看

```java
String objClzz = ClassLayout.parseClass(Object.class).toPrintable();
```

JDK8x64的JVM结果如下：

```txt
15:59:43.049 [main] INFO cn.pheker.learnjava.sync.ObjectHeader - objClzz: java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```



4.加锁的原理是在对象头的markword中添加锁标志位(64位JVM markword如下)

![img](https://gitee.com/HKbxOIC/imgs/raw/master/PhDocs/concurrent\java-object-markword.png)

4种锁状态，级别由低到高依次为：**无锁状态**、**偏向锁状态**、**轻量级锁状态**、**重量级锁状态**。这几个状态会随着竞争情况逐渐升级。

注意：锁可以升级但不能降级。

![img](https://gitee.com/HKbxOIC/imgs/raw/master/PhDocs/concurrent/synchronized-lock.png)

锁状态说明及升级图示

当然了，在谈这四种状态之前，我们还是有必要再简单了解下 synchronized 的原理。

在使用 synchronized 来同步代码块的时候，经编译后，会在代码块的起始位置插入 **monitorenter指令**，在结束或异常处插入 **monitorexit指令。当执行到 monitorenter 指令时，将会尝试获取对象所对应的 **monitor的所有权，即尝试获得对象的锁。

Synchronized修饰方法：

Synchronized方法同步不再是通过插入monitorentry和monitorexit指令实现，而是由方法调用指令来读取运行时常量池中的ACC_SYNCHRONIZED标志隐式实现的，如果方法表结构（method_info Structure）中的ACC_SYNCHRONIZED标志被设置，那么线程在执行方法前会先去获取对象的monitor对象，如果获取成功则执行方法代码，执行完毕后释放monitor对象，如果monitor对象已经被其它线程获取，那么当前线程被阻塞。


偏向锁在 JDK 6 及之后版本的 JVM 里是默认启用的。可以通过 JVM 参数关闭偏向锁：

```java
-XX:-UseBiasedLocking=false
```

关闭之后程序默认会进入轻量级锁状态。



**自旋锁** 默认是10次，可以使用 -XX:PreBlockSpin

在 JDK1.4.2 中引入，使用 -XX:+UseSpinning 来开启。JDK 6 中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。



## 二、锁的升级与优化

1、锁升级
锁的4中状态：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态（级别从低到高）

（1）偏向锁：

为什么要引入偏向锁？

因为经过HotSpot的作者大量的研究发现，大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。

偏向锁的升级

当线程1访问代码块并获取锁对象时，会在java对象头和栈帧中记录偏向的锁的threadID，因为偏向锁不会主动释放锁，因此以后线程1再次获取锁的时候，需要比较当前线程的threadID和Java对象头中的threadID是否一致，如果一致（还是线程1获取锁对象），则无需使用CAS来加锁、解锁；如果不一致（其他线程，如线程2要竞争锁对象，而偏向锁不会主动释放因此还是存储的线程1的threadID），那么需要查看Java对象头中记录的线程1是否存活，如果没有存活，那么锁对象被重置为无锁状态，其它线程（线程2）可以竞争将其设置为偏向锁；如果存活，那么立刻查找该线程（线程1）的栈帧信息，如果还是需要继续持有这个锁对象，那么暂停当前线程1，撤销偏向锁，升级为轻量级锁，如果线程1 不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程。

偏向锁的取消：

偏向锁是默认开启的，而且开始时间一般是比应用程序启动慢几秒，如果不想有这个延迟，那么可以使用-XX:BiasedLockingStartUpDelay=0；

如果不想要偏向锁，那么可以通过-XX:-UseBiasedLocking = false来设置；

（2）轻量级锁

为什么要引入轻量级锁？

轻量级锁考虑的是竞争锁对象的线程不多，而且线程持有锁的时间也不长的情景。因为阻塞线程需要CPU从用户态转到内核态，代价较大，如果刚刚阻塞不久这个锁就被释放了，那这个代价就有点得不偿失了，因此这个时候就干脆不阻塞这个线程，让它自旋这等待锁释放。

轻量级锁什么时候升级为重量级锁？

线程1获取轻量级锁时会先把锁对象的对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间（称为DisplacedMarkWord），然后使用CAS把对象头中的内容替换为线程1存储的锁记录（DisplacedMarkWord）的地址；

如果在线程1复制对象头的同时（在线程1CAS之前），线程2也准备获取锁，复制了对象头到线程2的锁记录空间中，但是在线程2CAS的时候，发现线程1已经把对象头换了，线程2的CAS失败，那么线程2就尝试使用自旋锁来等待线程1释放锁。

但是如果自旋的时间太长也不行，因为自旋是要消耗CPU的，因此自旋的次数是有限制的，比如10次或者100次，如果自旋次数到了线程1还没有释放锁，或者线程1还在执行，线程2还在自旋等待，这时又有一个线程3过来竞争这个锁对象，那么这个时候轻量级锁就会膨胀为重量级锁。重量级锁把除了拥有锁的线程都阻塞，防止CPU空转。

 

*注意：为了避免无用的自旋，轻量级锁一旦膨胀为重量级锁就不会再降级为轻量级锁了；偏向锁升级为轻量级锁也不能再降级为偏向锁。一句话就是锁可以升级不可以降级，但是偏向锁状态可以被重置为无锁状态。

（3）这几种锁的优缺点（偏向锁、轻量级锁、重量级锁）



2、锁粗化
按理来说，同步块的作用范围应该尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，缩短阻塞时间，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。 
但是加锁解锁也需要消耗资源，如果存在一系列的连续加锁解锁操作，可能会导致不必要的性能损耗。 
锁粗化就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，避免频繁的加锁解锁操作。

3、锁消除
Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，经过逃逸分析，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间



![synchronized原理](https://gitee.com/HKbxOIC/imgs/raw/master/PhDocs/synchronized_principle.png)



## 疑问

epoch用处？



## 链接

https://blog.csdn.net/tongdanping/java/article/details/79647337