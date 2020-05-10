

[TOC]



## 一、TCP报文

报文主要段的意思：

>   序号：表示发送的数据字节流，确保TCP传输有序，对每个字节编号
>
>   确认序号：发送方期待接收的下一序列号，接收成功后的数据字节序列号加 1。只有ACK=1时才有效。
>
>   ACK：确认序号的标志，ACK=1表示确认号有效，ACK=0表示报文不含确认序号信息
>
>   SYN：连接请求序号标志，用于建立连接，SYN=1表示请求连接
>
>   FIN：结束标志，用于释放连接，为1表示关闭本方数据流

参考链接：https://www.cnblogs.com/jainszhang/p/10641728.html



## 二、TCP三次握手

```txt
			client						server
[closed]										[closed]
		 			---->   syn	  ---->			[listen]
[syn-sent]										[syn-rcvd]
					<---- ack+syn <---			
[established]
					---->   ack   ---->
												[established]

```

-   第一次：客户端发送请求到服务器，服务器知道客户端发送，自己接收正常。SYN=1,seq=x
-   第二次：服务器发给客户端，客户端知道自己发送、接收正常，服务器接收、发送正常。ACK=1,ack=x+1,SYN=1,seq=y
-   第三次：客户端发给服务器：服务器知道客户端发送，接收正常，自己接收，发送也正常.seq=x+1,ACK=1,ack=y+1

上面分析过程可以看出，握手两次达不到让双方都得出自己、对方的接收、发送能力都正常的结论的。





## 三、TCP四次挥手

```txt
			client						server
					<==== 数据传输 ===>
[established]									[established]
	 				---->   fin	  ---->			
[fin-wait-1]
					<----   ack   <---			[close-wait]
[fin-wait-2]					
					<==== 数据传输
					
					<---- fin ack <---			[last-ack]
[time-wait] 等待2MSL
					---->   ack   ---->
												[closed]
[closed]
```





-   第一次：客户端请求断开FIN,seq=u
-   第二次：服务器确认客户端的断开请求ACK,ack=u+1,seq=v
-   第三次：服务器请求断开FIN,seq=w,ACK,ack=u+1
-   第四次：客户端确认服务器的断开ACK,ack=w+1,seq=u+1



## 四、其他问题



### 4.1为什么三次握手和四次挥手？

-   三次握手时，服务器同时把ACK和SYN放在一起发送到了客户端那里
-   四次挥手时，当收到对方的 FIN 报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方是否现在关闭发送数据通道，需要上层应用来决定，因此，己方 ACK 和 FIN 一般都会分开发送。

### 4.2为什么客户端最后还要等待2MSL？

-   客户端需要保证最后一次发送的ACK报文到服务器，如果服务器未收到，可以请求客户端重发，这样客户端还有时间再发，重启2MSL计时。



## 五、参考链接

https://baijiahao.baidu.com/s?id=1614404084382122793&wfr=spider&for=pc



## 六、TCP重传、滑动窗口、拥塞控制、流量控制

https://baijiahao.baidu.com/s?id=1664395039305097355&wfr=spider&for=pc

https://blog.csdn.net/qq_38623623/article/details/81290265