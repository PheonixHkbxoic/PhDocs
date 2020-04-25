

# JMM与volatile



[TOC]





## JMM



### 为什么会有JVM内存模型?

为了实现跨平台性.

Java语言的跨平台性的基础,不仅包括Java的解释执行和JIT(即时编译),而且还包括了内存模型的统一.

为什么要实现跨平台?跨平台是面向谁的?

Java是在什么背景下出现的?

CPU厂商众多,主机上产厂商众多,一般情况下,生产 主机的厂商还会自己开发操作系统.

造成的问题?

CPU架构不同,导致指令集不同,操作系统众多,封装的汇编不同.

对编程造成的影响?

如果要更换服务器,或者服务器有多种型号的操作系统主机,则需要根据不同的操作系统制造不同的编译器.是的,不能编译一次,到处运行.并且在这种背景下,众多的CPU和主机厂商,可以说是很费心费力.

主机的物理内存模型.为什么会出现物理主机的内存模型?

CPU的进化史

单核CPU.在这种情况下,内存的IO读写速度和CPU的执行速度相差太大,限制CPU性能的最大问题就是IO读写速度.为了解决这个问题,出现了高速缓存,CPU不再直接和内存交换数据,而是通过高速缓存.

此时IO读写问题已经解决,此时限制CPU的性能的问题是什么呢?

CPU中的运算单元没有得到充分的利用,比如说,一个CPU内的运算单元可以同时进行一万个1+1=2的运算,但是由于CPU是单线程的,如果同时有10000个1+1=2的运算,那么需要进行串行操作.导致了其他的运算单元都没用到.处理问题的方案两个,超线程技术,将一个物理CPU模拟为两个,可以同时执行两个线程.CPU乱序执行,比如一段代码需要从头到尾进行串行操作,那么也就是一个一个指令的去执行,还是串行,资源还是没有得到充分的利用.把这一段命令打乱,在不影响返回结果的情况下,一起执行.其实,乱序执行只是对于外界的一种表现.从CPU内部的角度来说,更适合叫并行执行.

当然,为了提高速度,还有多核CPU的出现.同样的,出现了多线程.那么此时又出现了新的问题.如果两个及以上的线程同时操作内存中的一个变量,且两个线程把这个值都改变了.那么在写入高速缓存时,以谁的值为准呢?在写入内存时,以谁的值为准呢?缓存一致性问题.

上面说到背景,有很多的主机厂商.那么在处理这个问题时就有了不同的解决方案.在不同的主机操作系统中有自己的缓存一致性协议.给编程带来了什么影响?

程序员在进行多线程开发时,要针对不同缓存一致性协议进行开发,针对不同的内存模型进行开发,意思也就是说在进行多线程编程时,不同主机上的代码是不一样的.移值性很差.

JVM是如何解决这个问题的?

屏蔽各种主机的缓存一致性协议,转化为JVM内存模型.在程序员编程时,只要针对JVM的内存模型来编程就可以了.

所以,跨平台是针对于程序员的.

所以在这个背景下,出现了JVM内存模型,表现就是,运行时数据区中的共享内(主内存)存部分和线程私享部分(工作内存).JVM的八种原子操作以及八种操作规则.

以上大部分内容都来自 周志明大大的 <<深入理解Java虚拟机>>,也有一些自己的想法,希望能够帮助到你.



作者：进击的码农
链接：https://www.zhihu.com/question/325469611/answer/694336275
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



### 八种原子操作

八种原子操作分别为:

1.lock 锁定 : 把主内存中的一个变量标志为一个线程独享的状态

2.unlock 解锁 : 把主内存中的一个变量释放出来

3.read 读：将主内存中的变量读到工作内存中

4.load 加载：将工作内存中的变量加载到副本中

5.use 使用：当执行引擎需要使用到一个变量时,将工作内存中的变量的值传递给执行引擎

6.assign 赋值：将执行引擎收的的值赋值给工作内存中的变量

7.store 存储：将工作内存中的变量的值传到主内存中

8.write 写入：将store得到值放到主内存的变量中



### 八种原子操作下的八种操作规则

1.read和load,store和write必须同时出现,并且按照顺序执行

2.不允许线程丢弃最后一个assign操作,在进行assign操作后,必须进行store和write

3.如果一个线程的工作内存中的一个变量没有发生assign操作,则不能够发生store和write

4.在工作内存中,如果对一个变量使用use或者store操作,则必须先执行assign和load操作

5.一个主内存中的变量在同一时刻只能被一个线程进行lock操作,但是这个线程可以进行多次lock操作,并且执行相应次数的unlock操作后,变量才会被解锁.

6.如果对一个量进行lock操作,将会清空此变量在其他线程工作内存中的值,这些线程在使用之前必须进行load或assign操作

7.一个线程如果没有对一个变量进行lock操作,则这个线程也不能对这个变量进行unlock操作

8.一个线程在进行unlock操作之前,必须先执行store和write操作



**先后顺序就是由这八种操作规则,或者说JVM内存模型的协议规定的.是JVM虚拟中内存模型的开发规范.**



![jmm_volatile](https://gitee.com/HKbxOIC/imgs/raw/master/PhDocs/jmm_volatile.png)



## volatile



当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到CPU缓存中。如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，这意味着每个线程可以拷贝到不同的 CPU cache 中。

而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache 这一步。



### 当一个变量定义为 volatile 之后，将具备两种特性：

　　1.保证此变量对所有的线程的可见性，这里的“可见性”，如本文开头所述，当一个线程修改了这个变量的值，volatile 保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存（详见：[Java内存模型](http://www.cnblogs.com/zhengbin/p/6407137.html)）来完成。

　　2.禁止指令重排序优化。有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个**内存屏障**（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。



与使用synchronized相比，声明一个volatile字段的区别在于没有涉及到锁操作。但特别的是对volatile字段进行“++”这样的读写操作不会被当做原子操作执行。

另外，有序性和可见性仅对volatile字段进行一次读取或更新操作起作用。声明一个引用变量为volatile，不能保证通过该引用变量访问到的非volatile变量的可见性。同理，声明一个数组变量为volatile不能确保数组内元素的可见性。volatile的特性不能在数组内传递，因为数组里的元素不能被声明为volatile。



由于没有涉及到锁操作，声明volatile字段很可能比使用同步的开销更低，至少不会更高。但如果在方法内频繁访问volatile字段，很可能导致更低的性能，这时还不如锁住整个方法。

如果你不需要锁，把字段声明为volatile是不错的选择，但仍需要确保多线程对该字段的正确访问。可以使用volatile的情况包括：

-   该字段不遵循其他字段的不变式。
-   对字段的写操作不依赖于当前值。
-   没有线程违反预期的语义写入非法值。
-   读取操作不依赖于其它非volatile字段的值。



### lock指令

lock 前缀的指令在多核处理器下会引发两件事情。

-   1）将当前处理器缓存行的数据写回到系统内存。
-   2）写回内存的操作会使在其他 CPU 里缓存了该内存地址的额数据无效。（MESI缓存一致性协议）



### volatile 有序性实现



#### volatile 的 happens-before 关系

-   happens-before 规则中有一条是 **volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。

#### volatile 禁止重排序

-   为了性能优化，JMM 在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了内存屏障阻止这种重排序。
-   Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。
-   JMM 会针对编译器制定 volatile 重排序规则表。

![img](https:////upload-images.jianshu.io/upload_images/5714666-390c861ed043ef94.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

volatile 重排序规则表

-   " NO " 表示禁止重排序。
    -   为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
    -   对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM 采取了保守的策略。
        -   在每个 volatile 写操作的前面插入一个 StoreStore 屏障。
        -   在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。
        -   在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。
        -   在每个 volatile 读操作的后面插入一个 LoadStore 屏障。
-   volatile 写是在前面和后面分别插入内存屏障，而 volatile 读操作是在后面插入两个内存屏障。

| 内存屏障        | 说明                                                        |
| --------------- | ----------------------------------------------------------- |
| StoreStore 屏障 | 禁止上面的普通写和下面的 volatile 写重排序。                |
| StoreLoad 屏障  | 防止上面的 volatile 写与下面可能有的 volatile 读/写重排序。 |
| LoadLoad 屏障   | 禁止下面所有的普通读操作和上面的 volatile 读重排序。        |
| LoadStore 屏障  | 禁止下面所有的普通写操作和上面的 volatile 读重排序。        |



![img](https:////upload-images.jianshu.io/upload_images/5714666-42b7250449160dc2.png?imageMogr2/auto-orient/strip|imageView2/2/w/747/format/webp)

volatile 写插入内存屏障

![img](https:////upload-images.jianshu.io/upload_images/5714666-cce9ccf139acf1a4.png?imageMogr2/auto-orient/strip|imageView2/2/w/801/format/webp)

volatile 读插入内存屏障

##  volatile 的应用场景

-   使用 volatile 必须具备的条件
    -   对变量的写操作不依赖于当前值。
    -   该变量没有包含在具有其他变量的不变式中。
-   只有在状态真正独立于程序内其他内容时才能使用 volatile。
-   *模式 #1 状态标志*
    -   也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。



```java
volatile boolean shutdownRequested;
......
public void shutdown() { shutdownRequested = true; }
public void doWork() { 
    while (!shutdownRequested) { 
        // do stuff
    }
}
```

-   模式 #2 一次性安全发布（one-time safe publication）
    -   缺乏同步会导致无法实现可见性，这使得确定何时写入对象引用而不是原始值变得更加困难。在缺乏同步的情况下，可能会遇到某个对象引用的更新值（由另一个线程写入）和该对象状态的旧值同时存在。（这就是造成著名的双重检查锁定（double-checked-locking）问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象）。



```java
public class BackgroundFloobleLoader {
    public volatile Flooble theFlooble;
 
    public void initInBackground() {
        // do lots of stuff
        theFlooble = new Flooble();  // this is the only write to theFlooble
    }
}
 
public class SomeOtherClass {
    public void doWork() {
        while (true) { 
            // do some stuff...
            // use the Flooble, but only if it is ready
            if (floobleLoader.theFlooble != null) 
                doSomething(floobleLoader.theFlooble);
        }
    }
}
```

-   模式 #3：独立观察（independent observation）
    -   安全使用 volatile 的另一种简单模式是定期 发布 观察结果供程序内部使用。例如，假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值。



```java
public class UserManager {
    public volatile String lastUser;
 
    public boolean authenticate(String user, String password) {
        boolean valid = passwordIsValid(user, password);
        if (valid) {
            User u = new User();
            activeUsers.add(u);
            lastUser = user;
        }
        return valid;
    }
}
```

-   模式 #4 volatile bean 模式
    -   在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。（这将禁止具有数组值的属性，因为当数组引用被声明为 volatile 时，只有引用而不是数组本身具有 volatile 语义）。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。



```java
@ThreadSafe
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
 
    public void setFirstName(String firstName) { 
        this.firstName = firstName;
    }
 
    public void setLastName(String lastName) { 
        this.lastName = lastName;
    }
 
    public void setAge(int age) { 
        this.age = age;
    }
}
```

-   模式 #5 开销较低的读－写锁策略
    -   volatile 的功能还不足以实现计数器。因为 ++x 实际上是三种操作（读、添加、存储）的简单组合，如果多个线程凑巧试图同时对 volatile 计数器执行增量操作，那么它的更新值有可能会丢失。
    -   如果读操作远远超过写操作，可以结合使用内部锁和 volatile 变量来减少公共代码路径的开销。
    -   安全的计数器使用 synchronized 确保增量操作是原子的，并使用 volatile 保证当前结果的可见性。如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及 volatile 读操作，这通常要优于一个无竞争的锁获取的开销。



```java
@ThreadSafe
public class CheesyCounter {
    // Employs the cheap read-write lock trick
    // All mutative operations MUST be done with the 'this' lock held
    @GuardedBy("this") private volatile int value;
 
    public int getValue() { return value; }
 
    public synchronized int increment() {
        return value++;
    }
}
```

-   *模式 #6 双重检查（double-checked）*
    -   单例模式的一种实现方式，但很多人会忽略 volatile 关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是 100%，说不定在未来的某个时刻，隐藏的 bug 就出来了。



```java
class Singleton {
    private volatile static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    } 
}
```

-   推荐懒加载优雅写法 Initialization on Demand Holder（IODH）。



```java
public class Singleton {  
    static class SingletonHolder {  
        static Singleton instance = new Singleton();  
    }  
      
    public static Singleton getInstance(){  
        return SingletonHolder.instance;  
    }  
}
```



## 参考

https://www.jianshu.com/p/157279e6efdb
[https://blog.csdn.net/devotion987/article/details/68486942](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fdevotion987%2Farticle%2Fdetails%2F68486942)



主要参考：[【Java 并发笔记】volatile 相关整理 https://www.jianshu.com/p/ccfe24b63d87](https://www.jianshu.com/p/ccfe24b63d87)

视频: [https://www.bilibili.com/video/BV1Et411n7Ro?p=1](https://www.bilibili.com/video/BV1Et411n7Ro?p=1)