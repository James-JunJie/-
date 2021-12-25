##1.你了解过哪些IO模型？

###1.1BIO

**同步阻塞模型**，**一个客户端连接对应一个处理线程**

 ####缺点：

1. IO代码里read操作是**阻塞操作**，**不做读写操作,会导致线程阻塞**，浪费资源。
2. 如果使用多线程，那会导致**服务器线程太多，压力太大。**

服务端多线程：![image-20210827203532871](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827203532871.png)

###1.2.NIO:

是**同步非阻塞的**，它的**一个线程可以处理多个请求**，客户端发送的**请求都会注册到多路复用器selector上**，多路复用器轮询**连接有IO请求就进行处理。**

**selector的底层：**

![image-20210915200343924](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210915200343924.png)

select 和 poll ：遍历，轮询channel，如果有事件进行处理。

##### epoll

底层是**哈希表结构**，采用**事件通知方式**，每当**有IO事件就绪**，系统注册的**回调函数就会被调用**，时间复杂度O(1).

由以前的主动轮询，变为被动通知。

####应用场景：

NIO方式适用于**连接数目多，时间短** 的架构， 比如聊天服务器， 弹幕系统， 服务器间通讯

#### NIO的组件

channel(通道)， Buffer(缓冲区)，Selector(选择器)	

1. chanel类似于流，每个channel对应一个buffer缓冲区，buffer底层是一个数组
2. selector可以对应多个客户端的连接，连接之后，将channel的key注册到selector上。
3. 由selector根据channel发生的读写事件，进行处理，这样就不会阻塞

![image-20210916093056225](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210916093056225.png)

为什么不阻塞？

因为accept(),read()操作会判断是否有这样的事件。在决定是否执行。

###1.3AIO:

**异步非阻塞**：aio对于**nio的代码封装，所以保存了非阻塞的**，**异步是它通过回调通知去处理**，**由操作系统完成后，回调通知程序启动线程去处理**。

场景:  **适合连接数量多，时间长**



* BIO：同步阻塞。
* NIO：同步非阻塞。
* AIO: 异步非阻塞，是将nio进行封装。
* 同步：需要等待该方法执行完。
* 异步：不需要等待该方法执行完。直接执行主函数，不需要等该方法执行完。
* 阻塞：accept() 等待客户端连接，阻塞
* 非阻塞：非阻塞，nio，selector中有事件，进行执行。

##2.什么是Reactor模型？Reactor的3种版本都知道吗？

> https://cloud.tencent.com/developer/article/1488120

Reactor模型基于**事件驱动**。

- Reactor： **负责监听和分配事件**，将**I/O事件分派给对应的Handler**。新的事件包含连接建立就绪、读就绪、写就绪等。
- Acceptor：**处理客户端新连接**，并分派请求到处理器链中。
- Handler：将自身与事件绑定，处理业务，执行非阻塞读/写任务，完成channel的读入，完成处理业务逻辑后，负责将结果写出channel。可用资源池来管理。

### 2.1单线程模型

理论上一个nio线程可以实现多个io复用，非阻塞。

缺点：

io事件耗费时间。



![image-20210827203941114](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827203941114.png)



###2.2单Reactor多线程模型

将读写事件交给线程池运行。

![image-20210827204906536](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827204906536.png)

缺点：

如果有大量客户端，**那么有大量读写事件，需要Reactor(Selecttor) 处理分发事件**，分发事件耗费时间。

对比nio，的selector.select()；

![image-20210827205321704](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827205321704.png)



###2.3主从Reactor多线程模型

一主一从。

使用一主多从。

MainReactor：**负责所有新连接**。aceept()事件，将**socketchannel 注册到多个从响应器中**。

subReactor:  多个响应器。处理读写的分发。

![image-20210823163817784](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210823163817784.png)




##3.了解过粘包拆包吗？为什么会出现粘包拆包？怎么处理粘包拆包？

###3.1粘包拆包原因？

TCP是个“流”协议，所谓流，就是没有界限的一串数据。

根据TCP缓冲区的实际情况进行包的划分。所以在业务上认为，一**个完整包TCP拆分成多个包，多个小包封装成大的数据包发送**

数据从发送方到接收方需要经过**操作系统的缓冲区**，而造成粘包和拆包的主要原因就在这个缓冲区上。

* 粘包可以理解为**缓冲区数据堆积**，导致多个请求数据粘在一起。

* 拆包可以理解为**发送的数据大于缓冲区**，进行拆分处理。

 ![img](https://gitee.com/mo-se-de-feng/notes/raw/master/images/2.3.1%20%E6%95%B0%E6%8D%AE%E7%BC%93%E5%86%B2%E5%8C%BA.PNG)

几种情况：

粘包：两个独立包，合并在一起。

![image-20210827210327071](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827210327071.png)

###3.2Netty中的粘包和拆包解决方案？

拆包： 用户可以**自己定义自己的编码器进行处理**

由于拆包比较复杂，代码比较处理比较繁琐，Netty提供了4种解码器来解决，分别如下：

1. **固定长度的拆包器** FixedLengthFrameDecoder，每个应用层数据包的都拆分成都是固定长度的大小
2. **分隔符拆包器 **：DelimiterBasedFrameDecoder，每条数据有固定的格式（开始符、结束符），这种方法简单易行，但选择开始符和结束符的时候一定要注意每条数据的内部一定不能出现开始符或结束符。
3. **数据包长度的拆包器** LengthFieldBasedFrameDecoder，**读取传输的数据长度之后，从缓存区中读取数据。**



##4.UDP协议会有粘包拆包的问题吗？为什么？

不会。

UDP（user datagram protocol，用户数据报协议）是**无连接的，面向报文的**。不使用块的**合并优化算法**，,  由于UDP支持的是一对多的模式，所以接收端的skbuff(套接字缓冲区）采用了链式结构来记录每一个到达的UDP包，在每个**UDP包中就有了消息头（消息来源地址，端口等信息）**，这样，对于接收端来说，就容易进行**区分处理**了。 

由于**UDP有消息保护边界**(一次将一个整包接受)，不会发生粘包拆包问题，因此粘包拆包问题只发生在TCP协议中。

##5.Netty 是什么？

 Netty 是一个基于 JAVA NIO 类库的**异步通信框架**。

##6.为什么要用 Netty？

Netty 对 JDK 自带的 NIO 的 API 进行了良好的封装。

简单，**异步非阻塞**、基于**事件驱动、高性能、高可靠性**和**高可定制性。**



##7.Netty 的应用场景了解么？

* 在分布式系统中，各个节点之间需要远程服务调用，高性能的 RPC 框架必不可少。rpc框架使用netty作为通信组件
* 游戏行业
* 即时通讯系统

##8.Netty 的零拷贝了解么？

首先linux下零拷贝的实现有三种：

* 直接I/O
* 减少内核空间缓冲区-->用户空间缓冲区的拷贝次数
* 写时复制

netty主要使用了第二种，创建并使用直接内存，java程序直接操作这部分空间。

相对于传统IO：磁盘/网卡控制器缓冲区 --DMA-->内核空间缓存区---->用户空间缓存区，省下了上下文的切换，以及减少了内核空间缓冲区--->用户空间缓冲区。

在Linux中我们使用mmap + write方式，或者 sendfile方式去实现(sendfile省区了系统调用的上下文切换)

直接内存：它的**内存申请比较慢，但是访问效率高。**

在java中我们使用ByteBuffer操作直接内存，其中用到Usafe类去allocat()分配直接内存，以及freeMemeroy()释放直接内存。

ByteBuffer使用了虚引用，使用Cleaner（虚引用）监测ByteBuffer对象，一旦ByteBuffer对象被垃圾回收，那么Referencehandler线程会掉用Cleaner的clean的方法调用Unsafe的freeMemory()来释放直接内存

 

##9.长连接和短连接

* 长连接，也叫持久连接，在**TCP层握手成功后**，**不立即**断开连接，并在此连接的基础上进行**多次消息（包括心跳）交互**，直至连接的任意一方（客户端OR服务端）主动断开连接，此过程称为一次完整的长连接。HTTP 1.1相对于1.0最重要的新特性就是引入了长连接。

* 短连接，顾名思义，与长连接的区别就是，**客户端收到服务端的响应后**，**立刻发送FIN消息**，主动释放连接。也有服务端主动断连的情况，凡是在**一次消息交互（发请求-收响应）之后立刻断开连接**的情况都称为短连接。

## 10长连接，短连接场景？

* 需要**频繁交互**的场景使用长连接，如即时通信工具（微信/QQ，QQ也有UDP），相反则使用短连接。
* **维持长连接**会有一定的系统开销

##10.Netty 的心跳机制了解么？

所谓心跳, 即在 TCP 长连接中, 客户端和服务器之**间定期发送的一种特殊的数据包, 通知对方自己还在线**, 以**确保 TCP 连接的有效性**。
在 Netty 中, 实现心跳机制的关键是 IdleStateHandler。

读超时 | 写超时 | 读写超时，在超时后会触发一个Event事件，发生了超时调用**userEventTriggered**方法。

**IdleStateHandler**构造器：

```java
1 public IdleStateHandler(int readerIdleTimeSeconds, int writerIdleTimeSeconds, int allIdleTimeSeconds) {
2 this((long)readerIdleTimeSeconds, (long)writerIdleTimeSeconds, (long)allIdleTimeSeconds, TimeUnit.SECONDS);
3 }
```

这里解释下三个参数的含义：

* **readerIdleTimeSeconds**: **读超时**. 即当在指定的时间间隔内没有从 Channel 读取到数据时, 会触发一个 READER_IDLE 的IdleStateEvent 事件.
* **writerIdleTimeSeconds**: 写超时. 即当在指定的时间间隔内没有数据写入到 Channel 时, 会触发一个 WRITER_IDLE 的IdleStateEvent 事件.
* allIdleTimeSeconds: **读/写超时**. 即当在指定的时间间隔内没有读或写操作时, 会触发一个 ALL_IDLE 的 IdleStateEvent 事件.

注：这三个参数默认的时间单位是秒。

若需要指定其他时间单位，可以使用另一个构造方法：
IdleStateHandler(boolean observeOutput, long readerIdleTime, long writerIdleTime, long allIdleTime, TimeUnit unit)

要实现Netty服务端心跳检测机制需要在服务器端的ChannelInitializer中加入如下的代码：
` pipeline.addLast(new IdleStateHandler(3, 0, 0, TimeUnit.SECONDS));`

## 11.Netty 中有哪些重要组件？

* EventLoopGroup组件，这主要用于创建bossGroup和workerGroup线程组。默认数量是cpu核数的两倍

* Bootstrap：客户端启动器。

* ServerBootstrap服务端启动器，调用group()设置线程组, chnnel()设置Channel类型，NioServerSocketChannel，设置handler

* channelPipline：一个channel包含一个ChannelPipeline，它之中维护了由ChannelHandlerContext 组成的双向链表，ChannelHandlerContext 关联了一个ChannelHandler
* ChannelHandler分为出站，入站事件，在使用netty时主要就编写handler。



##请概要讲述一下序列化？

使用序列化，主要用于网络传输和持久化。

序列化：Java 对象----->字节流

反序列化：字节流----> Java 对象

java中实现 Serializable 接口，实现序列化。





1. 说说Netty的执行流程？

2. Netty高性能体现在哪些方面？

   

![image-20210827203026937](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827203026937.png)



![image-20210827203056316](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210827203056316.png)



概述下什么是DDOS攻击和SYN洪水攻击

HTTP和HTTPS的区别

