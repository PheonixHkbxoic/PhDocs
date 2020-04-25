[TOC]



## 一、集合简介

List 元素是有序的、可重复

ArrayList、Vector默认初始容量为10

Vector：线程安全，但速度慢

　　　　底层数据结构是数组结构

　　　　加载因子为1：即当 元素个数 超过 容量长度 时，进行扩容

　　　　扩容增量：原容量的 1倍

　　　　　　如 Vector的容量为10，一次扩容后是容量为20

ArrayList：线程不安全，查询速度快

　　　　底层数据结构是数组结构

　　　　扩容增量：原容量的 1.5倍

　　　　　　如 ArrayList的容量为10，一次扩容后是容量为15

Set(集) 元素无序的、不可重复。

HashSet：线程不安全，存取速度快

　　　　　底层实现是一个HashMap（保存数据），实现Set接口

　　　　　默认初始容量为16（为何是16，见下方对HashMap的描述）

　　　　　加载因子为0.75：即当 元素个数 超过 容量长度的0.75倍 时，进行扩容

　　　　　扩容增量：原容量的 1 倍

　　　　　　如 HashSet的容量为16，一次扩容后是容量为32

 

Map是一个双列集合

HashMap：默认初始容量为16

　　　　　（为何是16：16是2^4，可以提高查询效率，另外，32=16<<1       -->至于详细的原因可另行分析，或分析源代码）

　　　　　加载因子为0.75：即当 元素个数 超过 容量长度的0.75倍 时，进行扩容

　　　　　扩容增量：原容量的 1 倍

　　　　　　如 HashSet的容量为16，一次扩容后是容量为32

HashTable： 默认容量11，扩容方式2N+1

链接：https://blog.csdn.net/qq_32575047/java/article/details/78942965



## 二、HashMap

1.容量为2^n 便于扩容、高效计算、链表数据迁移

2.加载因子默认0.75是**提高空间利用率和 减少查询成本的折中，主要是泊松分布，0.75的话碰撞最小**

源码中大概是这样说的：

**在理想情况下,使用随机哈希码,节点出现的频率在hash桶中遵循泊松分布，同时给出了桶中元素个数和概率的对照表。**

**从上面的表中可以看到当桶中元素到达8个的时候，概率已经变得非常小，也就是说用0.75作为加载因子，每个碰撞位置的链表长度超过８个是几乎不可能的。**

3.链表转为红黑树的阈值为8，而源码实现时是>8时才会转为红黑树， <=6时会转为链表

4.key,value允许为null，key为null时放在下标为0的位置；

ConcurrentHashMap都不允许为null  至于为什么 有人如下这样说：

ConcurrentHashMap不能put null 是因为 无法分辨是key没找到的null还是有key值为null，这在多线程里面是模糊不清的，所以压根就不让put null。 

5.扩容时jdk7采用头插法 这在并发时会产生链表死循环，jdk8采用尾插法并会保留原有顺序 迁移的位置通过hash&oldCap来确定 因为这个位运算的结果只有两种可能0与hash本身， 为0时迁移至原来的下标位置，否则迁移至 原来的下标+oldCap的位置

6.具体put流程(jdk8)

![HashMap的put流程（JDK8）](https://img-blog.csdn.net/20180508173741224?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NodW1veWlu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1、hash(key)，取key的hashcode进行高位运算，返回hash值
2、如果hash数组为空，直接resize()
3、对hash进行取模运算计算，得到key-value在数组中的存储位置i
（1）如果table[i] == null，直接插入Node<key,value>
（2）如果table[i] != null，判断是否为红黑树p instanceof TreeNode。
（3）如果是红黑树，则判断TreeNode是否已存在，如果存在则直接返回oldnode并更新；不存在则直接插入红黑树，++size，超出threshold容量就扩容
（4）如果是链表，则判断Node是否已存在，如果存在则直接返回oldnode并更新；不存在则直接插入链表尾部，判断链表长度，如果大于8则转为红黑树存储，++size，超出threshold容量就扩容



## 三、链接

https://blog.csdn.net/shumoyin/java/article/details/80243419

