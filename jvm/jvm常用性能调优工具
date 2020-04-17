#  GC



[TOC]



## 1.查看进程

```shell
ps -ef # 查看进程状态 -e等同于-A显示所有线程 -f 全格式/所有列 SystemV标准
ps aux BSD标准
jps -l 详细显示java进程状态
jinfo [optional] pid 打印java进程所有参数
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
```



## 2.查看进程的内存映像信息jmap

```shell
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

jmap dump到指定的文件

>   jmap -dump:format=b,file=/var/jvmdump.dat

jhat 解析dump文件

>   jhat -port 9999 /var/jvmdump.dat

可以打印浏览器查看并利用OQL时行查询筛选



## 3.查看堆内存使用统计jstat

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。

命令的格式如下：

　　jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

​		如：jstat -gc 6600 1s 10

### 命令选项

```shell
jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```



### 垃圾回收统计

```shell
jstat -gc 6600 1s 5
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
10752.0 10752.0  0.0    0.0   65536.0   3445.3   175104.0    9299.4   19072.0 18343.5 2432.0 2264.7      2    0.017   1      0.023    0.040
10752.0 10752.0  0.0    0.0   65536.0   3445.3   175104.0    9299.4   19072.0 18343.5 2432.0 2264.7      2    0.017   1      0.023    0.040
10752.0 10752.0  0.0    0.0   65536.0   3445.3   175104.0    9299.4   19072.0 18343.5 2432.0 2264.7      2    0.017   1      0.023    0.040
10752.0 10752.0  0.0    0.0   65536.0   3445.3   175104.0    9299.4   19072.0 18343.5 2432.0 2264.7      2    0.017   1      0.023    0.040
10752.0 10752.0  0.0    0.0   65536.0   3445.3   175104.0    9299.4   19072.0 18343.5 2432.0 2264.7      2    0.017   1      0.023    0.040
```



各列代表的含义：

>   ```
>   S0C：第一个幸存区的大小
>   S1C：第二个幸存区的大小
>   S0U：第一个幸存区的使用大小
>   S1U：第二个幸存区的使用大小
>   EC：伊甸园区的大小
>   EU：伊甸园区的使用大小
>   OC：老年代大小
>   OU：老年代使用大小
>   MC：方法区大小
>   MU：方法区使用大小
>   CCSC:压缩类空间大小
>   CCSU:压缩类空间使用大小
>   YGC：年轻代垃圾回收次数
>   YGCT：年轻代垃圾回收消耗时间
>   FGC：老年代垃圾回收次数
>   FGCT：老年代垃圾回收消耗时间
>   GCT：垃圾回收消耗总时间
>   ```



其它选项详细情况暂不详说

链接：[【JVM】jstat命令详解---JVM的统计监测工具](https://www.cnblogs.com/sxdcgaq8080/p/11089841.html)



## 4.其它调优工具

>   jconsole
>
>   VisualVM

第三方调优工具

>   MAT
>
>   GChisto
>
>   GCViewer



## 5.查看线程执行状态stack

jstack pid

### 5.1 死锁

是指在一组进程中的各个进程均占有不会释放的资源 但因互相申请被其他进程所站用不会释放的资源而处于的一种永久等待状态 

死锁的四个必要条件 

1 互斥条件(Mutual exclusion)：资源不能被共享，只能由一个进程使用。 

2 请求与保持条件(Hold and wait)：已经得到资源的进程可以再次申请新的资源。

3 非剥夺条件(No pre-emption)：已经分配的资源不能从相应的进程中被强制地剥夺。 

4 循环等待条件(Circular wait)：系统中若干进程组成环路，该环路中每个进程都在等待相邻进程正占用的资源。 

 java中产生死锁可能性的最根本原因是 1 是多个线程涉及到多个锁 这些锁存在着交叉 所以可能会导致了一个锁依赖的闭环 2 默认的锁申请操作是阻塞的  

### 5.2 避免死锁 

1 破坏死锁的循环等待条件 

2 破坏死锁的请求与保持条件 使用lock的特性 为获取锁操作设置超时时间 这样不会死锁（至少不会无尽的死锁） 

3 设置一个条件遍历与一个锁关联 该方法只用一把锁 没有chopstick类 将竞争从对筷子的争夺转换成了对状态的判断 仅当左右邻座都没有进餐时才可以进餐 提升了并发度