# 一.UDP

> https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453143119&idx=2&sn=d9c8716af1606e939ed589a404baebc7&scene=21#wechat_redirect

##1.首部格式：

UPD首部长度：**8字节**

* 源端口号
* 目标端口号
* UDP长度
* UDP校验和

![udp](04 UDP.assets/udp.jpeg)

##2.UDP特性：

>  无连接，不可靠，基于数据报

- 面向无连接。不需要像tcp那样三次握手。
- UDP 支持**一对一、一对多、多对一和多对多**的交互通信。
- UDP是面向报文的。
- 不可靠性。UDP 不提供可靠交付。**对方的运输层在收到 UDP 报文后，不需要给出任何确认。**
- UDP 的首部开销小，**只有 8 个字节**，比 **TCP 的 20 个字节**的首部要短

> 请注意，虽然在 UDP 之间的通信要用到其端口号，**但由于 UDP 的通信是无连接的，因此不需要使用套接字**

> UDP 对应用层交下来的报文，**既不合并，也不拆分，而是保留这些报文的边界。应用层交给 UDP 多长的报文，UDP 就照样发送，即一次发送一个报文。**

<img src="https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib0YVb206oxlvCF6yoibxGmK53OQQjcFfvWZ4ic1sRWcgXX1NU4n7UKjDnmVTQDa3yju9G4kQpsBMR5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />



##3.使用场景：

* 直播
* 实时游戏
* LoT物联网（维护TCP协议代价太大）

基于udp的协议： DHCP

TCP 是**面向连接的、可靠的、基于字节流**的传输层通信协议。

# 二.TCP

> https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453143119&idx=2&sn=d9c8716af1606e939ed589a404baebc7&scene=21#wechat_redirect

##1.头部格式

首部大小：20个字节

* 源端口号
* 目的端口号
* 序号:  给包编号，解决乱序问题。
* 确认序号：
* 首部长度：
* 控制位：
  * SYN ： 建立连接
  * ACK ：回复
  * RST ：重新连接
  * FIN :结束连接
* 窗口大小。流量控制时，需要声明一个窗口大小

![TCP](04 UDP.assets/TCP.jpeg)

##2.TCP是什么？

TCP 是**面向连接的、可靠的、基于字节流**的传输层通信协议。

- **面向连接**：一对一的连接  
- **可靠的**： 
- **字节流**： 

## 3.三次握手

> TCP 是面向连接的协议，所以使用 TCP 前必须先建立连接，而**建立连接是通过三次握手而进行的。**

连接：两个应用程序之间相互传递消息的**虚拟电路**。一旦建立连接，应用程序只能使用这个虚拟的通信线路**发送和接受消息**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843fFol7gd3035Kibg3gPMSAZQLVibf9nwEblOUaX80hoOaRLVpaYCAI44w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###3.1连接步骤：

1. 开始，客户端和服务端都处于 `CLOSED` 状态。先是服务端主动监听某个端口，处于 `LISTEN` 状态

2. 第一个报文：客户端会随机**初始化一个序列号，并且将SYN标志位置为1**。之后将第一个SYN报文发送给服务端，表示向服务端发起连接，该报文不包含引用层数据，之后客户端处于SYN_SENT状态。

3. 第二个报文：服务端收到SYN报文，**初始化一个序列号和应答号**，并且将**状态位的SYN和ACK都置为1**。之后服务端处于SYN-RCVD。(SYN-received)
4. 第三个报文：客户端返回一个**ACK为1**和**确认应答号**的应答报文。之后客户端与服务端状态处于Established。

###3.2为什么是三次握手?

- 三次握手才可以阻止历史重复连接的初始化（主要原因）
- 三次握手才可以避免资源浪费
- 三次握手才可以同步双方的初始序列号

#### 3.2.1阻止历史重复连接初始化

>  起因： 网络环境复杂，数据包不一定按顺序到达。

如何避免：

1. 一个旧的SYN报文先到达服务端
2. 此时服务端就会回复一个SYN+ACK报文给客户端
3. 客户端根据上下文比较，发现是一个历史连接，就发送RST报文给服务端，中止此次连接



<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS8436nKau10lAsztRqbyhjC1C1GRcsEz04icZmomMjwcxgeGn97BnKUoxibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

####3.2.2避免资源的浪费

如果SYN请求在**网络中阻塞**，客户端**重新发送SYN报文**，服务器每收到一个**SYN就主动建立连接**，**建立多个连接，浪费了资源**。

不使用「两次握手」和「四次握手」的原因：

- 「两次握手」：无法防止历史连接的建立，会造成双方资源的浪费，也无法可靠的同步双方序列号；
- 「四次握手」：三次握手就已经理论上**最少相对可靠连接建立**，所以不需要使用更多的通信次数。

### 3.3四次挥手

步骤：

1. 客户端将**FIN标志置为1**，发送FIN报文，之后进入FIN_wait1
2. 服务端收到报文，发**送ACK应答报文**，之后进入CLOSED_WAIT。此时不接受任何数据
3. 客户端收到**ACK报文**进入**FIN_WAIT2状态**
4. 服务端处理完数据后，**发送FIN报文**，**进入LAST_ACK**
5. 客户端收到**FIN报文**，发送**ACK报文进入TIME_WAIT状态**
6. 服务端收到ACK报文，**进入close**.
7. 客户端经过**2MSL时间后**，进入close状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843KaMMu2mHfFLZNgiaREDZ5JicRYrlaiciayQjh9HDsacxIbMT0emGUpAX5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 3.3.1为什么挥手需要四次？

* 服务端对第一个FIN报文，发送ACK表示，服务端需要**处理和发送数据**
* 结束后，发送FIN报文，表示同一关闭连接

而三次握手，将2,3步合并了。

#### 3.3.2为什么 TIME_WAIT 等待的时间是 2MSL？

`MSL` 是 Maximum Segment Lifetime，**报文最大生存时间**

TTL : IP 数据报可以经过的最大路由数

MSL =  TTL 

表示如果服务端**没有收到ACK报文，就再次发送FIN报文**。**而一来一回时间刚好2 TTL.**

####3.3.3为什么需要 TIME_WAIT 状态？

* 防止旧连接的数据包

* 保证连接正确关闭

#####3.3.3.1原因一：防止旧连接的数据包

> 假设 TIME-WAIT 没有等待时间或时间过短，被延迟的数据包抵达后会发生什么呢？

当相同端口的TCP连接复用后，**历史数据包抵达端口，造成数据错乱。**

#####3.3.3.2原因二：保证连接正确关闭

> 假设 TIME-WAIT 没有等待时间或时间过短，断开连接会造成什么问题呢？

1. 如果没有close状态，服务端没有收到客户端最后一个ACK报文。
2. 服务端一直处于LAST_ACK状态。**不能正常关闭，消耗服务器资源。**



---

如果四次挥手加入time_wait状态：

**没有收到ACK报文，就会重发FIN报文。**





![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843qohxKiaI9HTRicKXwNWLDoap0OabBhnaNeuibYrrtLvFAFWwoSgum6N2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





#UDP和TCP的区别：

**TCP 和 UDP 区别：**

*1. 连接*

- TCP 是面向连接的传输层协议，传输数据前先要建立连接。
- UDP 是不需要连接，即刻传输数据。

*2. 服务对象*

- TCP 是一对一的两点服务，即一条连接只有两个端点。
- UDP 支持一对一、一对多、多对多的交互通信

*3. 可靠性*

- TCP 是可靠交付数据的，数据可以无差错、不丢失、不重复、按需到达。
- UDP 是尽最大努力交付，不保证可靠交付数据。

*4. 拥塞控制、流量控制*

- TCP 有拥塞控制和流量控制机制，保证数据传输的安全性。
- UDP 则没有，即使网络非常拥堵了，也不会影响 UDP 的发送速率。

*5. 首部开销*

- TCP 首部长度较长，会有一定的开销，首部在没有使用「选项」字段时是 `20` 个字节，如果使用了「选项」字段则会变长的。
- UDP 首部只有 8 个字节，并且是固定不变的，开销较小。

**TCP 和 UDP 应用场景：**

由于 TCP 是面向连接，能保证数据的可靠性交付，因此经常用于：

- `FTP` 文件传输
- `HTTP` / `HTTPS`

由于 UDP 面向无连接，它可以随时发送数据，再加上UDP本身的处理既简单又高效，因此经常用于：

- `DHCP`,`DNS` 、`SNMP` 
- 直播
- 实时游戏



#三.TCP保证可靠性

>  https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453143215&idx=2&sn=e9e767ebcbd2fce4688ba71db4fbd32d&scene=21#wechat_redirect

TCP 实现可靠传输的方式之一，是通过序列号与确认应答。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgTfasBzdn2sIB39aFcqL22zhAa7v9d9vR1oZF4mibLUKouDEfKjYoZww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##3.1重传机制

* 超时重传
* 快速重传
* suck

###3.1.1超时重传

超过指定时间，没有收到对方ack确认应答报文，重发数据。

TCP 会在以下两种情况发生超时重传：

- 数据包丢失(发送)
- 确认应答丢失(接收)

#####RTO超时时间应该设置为多少呢？

超时重传时间 RTO ： **略大于包的往返时间。**

`RTT`：是**包的往返时间**。

### 3.1.2快速重传

**不以时间为驱动，而是以数据驱动重传**。

重传：收到三个相同的 ACK 报文时，重传丢失报文。

弊端：不知道该重传哪些 TCP 报文，于是就有 `SACK` 方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhggRhxrfiaRIsia0z3T4l7TxVM7J9vjFktFRKHkdva953UY6vFIpsYsOicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 3.1.3 SACK(选择性确认)

sack工作方式:  **发送方知道哪些数据没收到,只选择重传丢失数据**。

原理：TCP 头部「**选项**」字段里加一个 `SACK` 的东西，它**可以将缓存的地图发送给发送方**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhg3RbEZItSOQY1TadJpUTuibIDziaibALaEmk7JO0Ill7iaeiaGI94Wyia0NXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##3.2滑动窗口

因为TCP 是**每发送一个数据报，都要进行一次确认应答**。 发送方**收到了应答了， 再发送下一个**。

优点：减少重传，减少ack确认

缺点：数据包的**往返时间越长，通信的效率就越低**。

###窗口是什么？

实际上是操作系统开辟的**一个缓存空间**，保留已发送的数据，等到确认应答后，从缓存区中清除。 

**累计应答**：如果有某个ack丢失，可以通过下一个ack确认。

ex:  丢失ACK600 , 下一个包ACK700，ok。如果下一个ACK600，需要重传了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgz9LqxXsGESpzbick5b29No4rkqEubmVxexM2pM5DASr53mk65Qx5icZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###窗口大小由哪一方决定？

**接受方**

TCP头部的窗口大小：**接收端----->发送端 有多少缓冲区 可以接收数据**.

### 发送方滑动窗口

缓冲区分类：

* 发送收到

* 发送未收到

* 未发送可以接受

*  未发送不能接受

**发送窗口**：**已发送未确认 + 可用窗口** 

实现：接受到ACK后，移动发送窗口的第一个指针； 继续发送数据，可用窗口指针也移动。

使用两个指针：

* `SND.UNA`： 指向发送窗口第一个指针

* `SND.NXT`： 指向可用窗口第一个指针

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhge0ujfyrkBBMicd4U4qiaubDemmA9BhjDalRicd1cxrAkBQqlBQSL1oGzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### 接受方滑动窗口

接收窗口

实现：接收到数据，接受窗口指针右移。

RCV.NEXT: 一个接受窗口的指针

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhglnibEWRdZcD4QWuC61y9iaRoKlWUWyicnP0iaS01fAjsdGgKJnYJcfm0wA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##3.3流量控制

> > 如果一直无脑的发数据给对方，但对方处理不过来，那么就会导致触发重发机制，从而导致网络流量的无端的浪费。

流量控制：根据「接收方」的**实际接收能力控制发送的数据量**。

###3.3.1窗口不变：

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgrEtDiblwiaNET0Dx9BMfqyVXY6Kc00ib4BD6l1yyPEZrb2ptxV31VSmFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" />



###3.3.2窗口改变

* **操作系统缓存区调整(**发送窗口，接受窗口都是放在缓冲区中)
* **应用进程无法读取缓存区的内容**

应用程序不读取数据，可见最后窗口都收缩为 0 了，也就是发生了**窗口关闭**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgEnFtyFXUGUib92pibiacHuqQDAhbnC4AcVuMiaLs50TjjW0nbyrSnLs9Rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" />









###3.3.3窗口关闭

**如果窗口大小为 0 时，就会阻止发送方给接收方传递数据，直到窗口变为非 0 为止，这就是窗口关闭。**

引发问题：**死锁现象**，如果**接受方处理完数据后的**，**ack丢失**，那么会发生。

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgI1zQk2E30AjqibOmvbWiaNswZPhx3UcyYTyqgJTCoEJeuMEoic4t3KuOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



###3.3.4TCP 是如何解决窗口关闭时，潜在的死锁现象呢

解决： **零窗口通知，启动计时器，发送窗口探测报文，获取对方接受窗口大小。**

**只要 TCP 连接一方收到对方的零窗口通知，就启动持续计时器。**

如果持续计时器超时，就会发送**窗口探测 ( Window probe ) 报文**，而对方在确认这个探测报文时，给出自己现在的接收窗口大小。

#### 3.3.5糊涂窗口综合症

发送窗口很小，为了几个字节的数据，而TCP + IP 头有40字节，发送开销太大。



要解决糊涂窗口综合症，就解决上面两个问题就可以了

- 让接收方不通告小窗口给发送方 ：  **接受窗口 < 最大报文段** ，窗口关闭。
- 让发送方避免发送小数据：  Nagle 算法



####3.3.7怎么让发送方避免发送小数据呢？数据大小 > 最大报文段

> 最大报文段长度（MSS）,每一个报文段所能承载的最大数据长度（不包括文段头）。 

Nagle 算法:

满足一条即可发送：

- 收到之前发送数据的 `ack` 回包
- 要等到 **数据大小 >= `MSS `**   或是  **发送窗口大小 >= `MSS`**

MSS=MTU-20字节(TCP报头)-20字节(IP报头)

MSS值一般就是:1500-20-20=1460字节。

##3.4拥塞控制

> 网络拥堵，造成丢包，触发TCP重传，增加网络负担。

拥塞控制： **避免「发送方」的数据填满整个网络。**

发送窗口 = min(拥塞窗口 , 接受窗口）

拥塞窗口 `cwnd` 变化的规则：

- 只要网络中没有出现拥塞，拥塞窗口 ,就会增大；
- 但网络中出现了拥塞，拥塞窗口 就减少；

四个算法：

- 慢启动
- 拥塞避免
- 拥塞发生
- 快速恢复

### 3.4.1慢启动

慢启动:  TCP连接刚建立， 一点一点的提高发送数据包的数量。

算法： **发送方每收到一个 ACK，就拥塞窗口 cwnd 的大小就会加 1。**

**指数性的增长**

1. 初始发送1个
2. cwnd=2,发送2个
3. cwnd=2+2=4,发送4个
4. cwnd=8.

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgmialsBqYwdlLHfaykYgfgeZ3UM9bgvqoaAz07CRG8RI1Nken6JcQaqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###3.4.2拥塞避免

当慢启动达到 **慢启动门限时(65535)**，开启拥塞避免。

算法：拥塞窗口是线性增长。

### 3.4.3拥塞发生

拥塞发生是，不同重传机制，有不同的算法。我们将快速重传+快速恢复

* 超时重传

* 快速重传 +快速恢复

**快速重传的拥塞发生算法**

- `拥塞窗口 = 拥塞窗口/2` ，也就是**设置为原来的一半**;
- `启动门限 = 拥塞窗口`;

#### 快速恢复

拥塞窗口 =  启动门限 + 3   ex ：( 6  + 3)

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgR3w50EdpWF95ZM6QPpELCF3P1niazia8nBrSQUvX7e7F7LXMiaXR3iayUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)