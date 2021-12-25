 # HTTP

HTTP是什么呢？它是**超文本传输协议**，HTTP是缩写，它的全英文名是HyperText Transfer Protocol。

超文本指**：HTML，css，JavaScript和图片，音频，视频，文件**等，HTTP的出现是为了接收和发布HTML页面。

## HTTP的消息结构







## 请求和响应

请求报文：请求行，请求头字段，空行和消息体

首部字段：

* accpet
* Accept-Charset 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/3aPj0GhFQDA0E6RyltX4BwamgouX7YPdxknnx0ARphPxtxdHHibRCPyjYs3od2MePGDmbNfBMxUqYhfFX2dDXsQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

响应结构： 响应头 + 响应空行 + 响应体

响应行：版本  状态码

响应头：

* Location: http://www.it315.org/index.jsp 【服务器告诉浏览器**要跳转到哪个页面**】

* Server:apache tomcat【服务器告诉浏览器，服务器的型号是什么】

* Content-Encoding: gzip 【服务器告诉浏览器**数据压缩的格式**】



![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2FsMVzfNxopOlFhKsfticyLAqOXeCHgG7SNTDicJIAMbakyOicJTd7DCFqofb70kxx91DR6EJsjibFHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





## HTTP Get 和 Post 区别

| Get                                 | Post                                   |
| ----------------------------------- | -------------------------------------- |
| 不安全的，请求参数拼接在url后       | 安全，请求会把**参数和值放在消息体中** |
| 有长度限制                          | 无长度限制                             |
| 览器反复的 `回退/前进` 操作是无害的 | post 操作会再次提交表单请求。          |
| 请求会产生一个 TCP 数据包           | 会产生两个 TCP 数据包， header和data   |



- get 方法一般用于请求，比如你在浏览器地址栏输入 `www.cxuanblog.com` 其实就是发送了一个 get 请求，它的主要特征是请求服务器返回资源，而 post 方法一般用于``

  `表单`的提交，相当于是把信息提交给服务器，等待服务器作出响应，get 相当于一个是 pull/拉的操作，而 post 相当于是一个 push/推的操作。

- get 方法是不安全的，因为你在发送请求的过程中，你的请求参数会拼在 URL 后面，从而导致容易被攻击者窃取，对你的信息造成破坏和伪造；

## 什么是无状态协议？怎么解决？

`无状态协议(Stateless Protocol)` 就是指**浏览器对于事务的处理没有记忆能力**。

使用cookie和session解决

cookie和session的区别：

| cookie                     | session                  |
| -------------------------- | ------------------------ |
| 放在浏览器上               | 服务器上                 |
| 不安全                     | 安全                     |
| 单个cookie数据不能超过4k   | 没有对存储数据量限制     |
| 只能存储 String 类型的对象 | 能够存储任意的 java 对象 |

 cookie将信息放在**请求头与响应头**中。

请求头中有sessionId。

## 常见状态码

* 2XX
  * 200 ： 正常处理
  * 204 成功处理，但服务器没有新数据返回，显示页面不更新
  * 206 对服务器进行范围请求，值返回一部分数据
* 3XX 
  * 301 重定向
  * 302 转发
* 4XX 客户端出错
  * 404 没有资源
* 5XX 服务器出错
  * 500 ： 内部资源出错了
  * 503 ： 服务器正忙

## 1.Http与Https的区别：

| HTTP               | HTTPS                                                        |
| ------------------ | ------------------------------------------------------------ |
| 不安全的           | 安全的，                                                     |
| 标准端口**是80**   | 标准端口是**443**                                            |
| URL 以http:// 开头 | URL 以https:// 开头                                          |
| 无需证书           | **HTTPS 就是身披了一层 SSL 的 HTTP**。 需要CA机构颁发的SSL证书 |
| 无法加密           | 对传输的数据进行加密                                         |

## 2. 什么是Http协议无状态协议?怎么解决Http协议无状态协议?

* 无状态协议：对于事务处理没有记忆能力。 
  * 当客户端一次HTTP请求完成以后，再发一次请求，HTTP并不知道当前客户端是以前访问过。

解决：

* 使用Cookie来解决无状态的问题
  * 第一次访问的时候给客户端发送一个Cookie
  * 当客户端再次来的时候，拿着Cookie(通行证)，那么服务器就知道这个是”老用户“。

##3.URI和URL的区别
URI，是uniform resource identifier，**统一资源标识符，用来唯一的标识一个资源**。
Web上可用的每种资源如HTML文档、图像、视频片段、程序等都是一个来URI来定位的	

URI一般由三部组成：

* ①访问资源的命名机制
* ②存放资源的主机名
* ③资源自身的名称，由路径表示，着重强调于资源。

URL是uniform resource locator，**统一资源定位器**，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。
URL是Internet上用来描述信息资源的字符串，主要用在各种WWW客户程序和服务器程序上，特别是著名的Mosaic。
采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服务器的地址和目录等。URL一般由三部组成：

* ①协议(或称为服务方式)
* ②存有该资源的主机IP地址(有时也包括端口号)
* ③主机资源的具体地址。如目录和文件名等



## 简述 HTTP1.0/1.1/2.0 的区别 ？

* HTTP1.0 默认是短连接，**每次与服务器的交互，都需要新开一个连接**
* HTTP1.1 是持久化连接，**建立一次连接，多次请求均由这个连接完成**！

| HTTP1.0                                               | HTTP1.1                                                      | HTTP2                      |
| ----------------------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| 短连接 （**每次与服务器的交互，都需要新开一个连接**） | 持久化连接(**建立一次连接，多次请求均由这个连接完成**)       |                            |
| 发起一次请求，需服务端响应了，才继续发送请求          | **管线化理论，同时发起多个http请求，非串行(理论阶段)**，**客户端, 但按序接收** | **多路复用，二进制分帧层** |
|                                                       | 增加host字段，实现了断点续传                                 |                            |
|                                                       |                                                              |                            |

### HTTP2

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemrmZuVZMTAl5zicJukuywvFYZMiavyw0LBkziaq5FbbbicWLeLRPOXUvu4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 在HTTP1.1中，发送一次请求时，不需要等待服务端响应了就可以发送请求了，但是回送数据给客户端的时候，**客户端**还是需要按照**响应的顺序**来一一**接收**

* 多路复用，**解决了http1.1 线头阻塞**

* **增加新的二进制分帧层**
* Hpack 对HTTP/2头部压缩
* 用frame data 封装了http1.1。

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemgYXqL5rTvMicTxd8wUSiaTAoH6Soj6Za0cXZoQXcuZvvibgdLTCRHVv6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



##HTTP的keepalive和TCP的keepalive，有什么区别？

**HTTP Keep-Alive**

http早期，每一个http请求，需要连接和断开tcp。

优点：

* 通过Keep-alive, 一次tcp连接中可以持续发送多份数据
* 减少了tcp连接建立的次数，以及TIME_WAIT状态的连接状态。

缺点：

* 长时间tcp连接容你导致系统资源无效占用。

keep-alvie timeout： http1.1默认开启，如果在这段时间，没有新的请求断开，销毁连接。

**TCP KEEPALIVE**

在客户端和服务端长时间未通讯，如何知道对方还活着？

tcp使用keepalive是一种保活机制，在保活时间内没收到，发送一个保活探测报文。

如果一定探测时间间隔后，无对端响应，继续发送，达到发送的阈值后，TCP失效。

##TCP与UDP的区别 ？



HTTPS的原理了解过吗 ？

TCP里Nagle算法了解过吗？可以禁用吗？ 在Java里怎么禁用？

HTTP协议为什么无法实现服务端推送？

websocket协议升级过程了解过吗？

websocket帧结构了解过吗

