* 查看进程：ps -ef | grep  XXX 

* 显示正在执行的进程，与ps不用，它可以执行一段时间更新线程

* 杀死进程：kill 
  * -9表示强行杀死

* **解压：**tar -zxvf 目录

* systemctl stop firewalld # 临时关闭防火墙

* systemctl disable firewalld # 禁止开机启动

* 重启网络服务：service network restart

* 查看端口22:netstat -an | grep 22

---

## 文件：

* 改名/移动 mv oldNameFile   newNameFIle
* 删除 ： rm
* vim情况下 搜索  /
* 创建文件：touch  文件名
* 创建目录：mkdir  目录   | (mkdir -p  多级目录)
* 删除目录：rmdir MyDocuments
* 复制文件：cp   -r 递归复制
  * cp /home/aaa.txt  /home/bbb.txt
* 定位文本中词
* 



# RPM

查看rpm软件安装位置：rpm -ql  软件名

查看是否安装软件：rpm -qa | grep 软件名

![image-20211021150714380](C:/Users/Jun/AppData/Roaming/Typora/typora-user-images/image-20211021150714380.png)

查看文件属于哪个rpm包：rpm -qf  [文件路径] /etc/passwd

删除rpm软件包：rpm -e 软件名字

安装rpm包：

# yum

Yum是前端软件包管理器。是基于RPM包管理，能够从指定服务器下载RPM包并且安装，可以自动处理依赖关系。

查看yum服务器是否有需要安装的软件：

* yum list | grep xx

安装指定的yum包

* yum install xxx

> 
