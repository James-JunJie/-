##DNS服务器

- DNS域名解析 ：域名 ---->  ip地址，因为域名服务器是按照树状结构组织的，因而域名查找是使用递归的方法，并通过缓存的方式增强性能；
- 全局的负载均衡。

###DNS解析流程

1. 首先从本地DNS缓存中获取。
2. 如果没有，本地DNS服务器，会转向根域名服务器
3. 转向顶级域名服务器，一直向上查找

![dns](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/dns.png)

### DNS负载均衡

负载均衡： 一个域名 --->多个ip时，使用负载均衡选出ip。





## CDN

选择最近的服务器









