#响应模式





![image-20210823163817784](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210823163817784.png)

##netty线程模型

![image-20210823164403160](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210823164403160.png)







#Netty组件

##1.【Bootstrap、ServerBootstrap】：
Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程序，串联各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端启动引导类。

##2.【NioEventLoopGroup】：
NioEventLoopGroup，主要管理 eventLoop 的生命周期，可以理解为一个**线程池**，内部维护了一组线程，**每个线程(NioEventLoop)**负责处理多个 Channel 上的事件，而一个 Channel 只对应于一个线程。

##3.【NioEventLoop】：
NioEventLoop 中维护了一个线程和任务队列，支持异步提交执行任务，线程启动时会调用。NioEventLoop里包含selector

NioEventLoop 的 run 方法，执行 I/O 任务和非 I/O 任务：

* I/O 任务，即 selectionKey 中 ready 的事件，如 accept、connect、read、write 等，由processSelectedKeys 方法触发。

* 非 IO 任务，添加到 taskQueue 中的任务，如 **register0、bind0 等任务，由 runAllTasks 方法触发**

##4.【Selector】：
Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel事件。
当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询(Select) 这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel 。

##4.【Future、ChannelFuture】：
正如前面介绍，在 Netty 中所有的 **IO 操作都是异步的**，不能立刻得知消息是否被正确处理。
但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件。

##5.【Channel】：

Netty 网络通信的组件，能够用于执行网络 I/O 操作。Channel 为用户提供：
1）当前网络连接的通道的状态（例如是否打开？是否已连接？）
2）网络连接的配置参数 （例如接收缓冲区大小）
3）提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成。
4）调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方。
5）支持关联 I/O 操作与对应的处理程序。
不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应。
下面是一些常用的 Channel 类型：

* 1 **NioSocketChannel**，异步的客户端 TCP Socket 连接。
* 2 **NioServerSocketChannel**，异步的服务器端 TCP Socket 连接。
* 3 NioDatagramChannel，异步的 UDP 连接。
* 4 NioSctpChannel，异步的客户端 Sctp 连接。
* 5 **NioSctpServerChannel**，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及**文件 IO**。

##6..【ChannelPipline】：
保存 **ChannelHandler 的 List**，用于处理或拦截 Channel 的入站事件和出站操作。
ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互。
在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下：

一个 Channel 包含了一个 ChannelPipeline，而 ChannelPipeline 中又维护了一个由ChannelHandlerContext 组成的双向链表，并且每个 ChannelHandlerContext 中又关联着一个ChannelHandler。

* read事件(入站事件)和write事件(出站事件)在一个双向链表中，入站事件会从链表 head 往后传递到最后一个入站的 handler

* 出站事件会从链表 tail 往前传递到最前一个出站的 handler，两种类型的handler 互不干扰

![image-20210823163436589](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210823163436589.png)



![image-20210823163440989](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210823163440989.png)

出站(out)： tail  ---- > head 中的outChannelHandler

入站(out):    head  ---->   tail 中的InChannelHandler

Client 的out事件    ==   Server 的 In事件

![Pipelien入站出战](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/Pipelien%E5%85%A5%E7%AB%99%E5%87%BA%E6%88%98.png)

####编码解码器
当你通过Netty发送或者接受一个消息的时候，就将会发生一次数据转换。入站消息会被解码：从字节转换为另一种格式（比如java对象）；如果是出站消息，会被编码成字节。

Netty提供了一系列实用的编码解码器，他们都实现了ChannelInboundHadnler或者ChannelOutcoundHandler接口。在这些类中，channelRead方法已经被重写了。以入站为例，对于每个从入站Channel读取的消息，这个方法会被调用。随后，它将调用由已知解码器所提供的decode()方法进行解码，并将已经解码的字节转发给ChannelPipeline中的下一个ChannelInboundHandler。
Netty提供了很多编解码器，比如编解码字符串的StringEncoder和StringDecoder，编解码对象的ObjectEncoder和ObjectDecoder等。

当然也可以通过集成ByteToMessageDecoder自定义编解码器。

ByteToMessageDecoder

MessageToByteEncoder

```
public abstract class MessageToByteEncoder<I> extends ChannelOutboundHandlerAdapter
public abstract class ByteToMessageDecoder extends ChannelInboundHandlerAdapter 
```

##6.【ChannelHandlerContext】：
保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象。

##7.【ChannelHandler】：
ChannelHandler 是一个接口，**处理 I/O 事件或拦截 I/O 操作**，并将其转发到其 ChannelPipeline(业务处理链)中的**下一个处理程序。**
ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，
可以继承它的子类：

* 1 **ChannelInboundHandler** 用于处理入站 I/O 事件。
* 2 **ChannelOutboundHandler** 用于处理出站 I/O 操作。
  或者使用以下适配器类：
* 1 **ChannelInboundHandlerAdapter** 用于处理入站 I/O 事件。
* 2 **ChannelOutboundHandlerAdapter** 用于处理出站 I/O 操作。



##ByteBuf详解：

ByteBuf 提供了两个索引，一个用于读取数据，一个用于写入数据。这两个索引通过在字节数
组中移动，来定位需要读或者写信息的位置。

通过readerindex和writerIndex和capacity，将buffer分成三个区域

1.已经读取的区域：[0,readerindex)
2.可读取的区域：[readerindex,writerIndex)
3.可写的区域: [writerIndex,capacity)

* byteBuf.getByte():不会更新readerIndex
* byteBuf.readByte():更新readerIndex

```
Unpooled工具类创建ByteBuf
ByteBuf byteBuf2 = Unpooled.copiedBuffer("hello,zhuge!", CharsetUtil.UTF_8);
byteBuf2.getCharSequence(0, 6, CharsetUtil.UTF_8))// 获取 0，6 的字符串
 byteBuf2.readableBytes();//可读的字节数  12
```

![image-20210823151815723](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210823151815723.png)



##ChannelInboundHandler

```JAVA
public interface ChannelInboundHandler extends ChannelHandler {
    void channelRegistered(ChannelHandlerContext var1) throws Exception;

    void channelUnregistered(ChannelHandlerContext var1) throws Exception;
	//当客户端连接服务器完成就会触发该方法
    void channelActive(ChannelHandlerContext var1) throws Exception;
	//表示 channel 处于不活动状态, 提示离线了
    void channelInactive(ChannelHandlerContext var1) throws Exception;
	// 读取客户端发送的数据 
    void channelRead(ChannelHandlerContext var1, Object var2) throws Exception;
	//  数据读取完毕处理方法
    void channelReadComplete(ChannelHandlerContext var1) throws Exception;
		
    void userEventTriggered(ChannelHandlerContext var1, Object var2) throws Exception;

    void channelWritabilityChanged(ChannelHandlerContext var1) throws Exception;
	//处理异常, 一般是需要关闭通道
    void exceptionCaught(ChannelHandlerContext var1, Throwable var2) throws Exception;
}
```















