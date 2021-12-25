ThreadLocal全面解析

> https://blog.csdn.net/weixin_44050144/article/details/113061884

学习目标

* 了解ThreadLocal的介绍
* 掌握ThreadLocal的运用场景
* 了解ThreadLocal的内部结构
* 了解ThreadLocal的核心方法源码
* 了解ThreadLocalMap的源码

#1.hreadLocal介绍

##1.1 官方介绍

ThreadLocal类用来提供**线程内部的局部变量**。**能保证各个线程的变量相对独立**,不同的线程之间不会相互干扰。


> 总结:
>
> 1. 线程并发: 在多线程并发的场景下
> 2. 传递数据: 我们可以通过ThreadLocal在同一线程，不同组件中传递公共变量
> 3. 线程隔离: 每个线程的变量都是独立的，不会互相影响

##ThreadLocal使用：

| 方法声明                  | 描述                       |
| ------------------------- | -------------------------- |
| ThreadLocal()             | 创建ThreadLocal对象        |
| public void set( T value) | 设置当前线程绑定的局部变量 |
| public T get()            | 获取当前线程绑定的局部变量 |
| public void remove()      | 移除当前线程绑定的局部变量 |

##ThreadLocal 与synchronized

强调的是**线程数据隔离的问题**，并不是多线程共享数据的问题。

synchronized：同步访问，性能慢。

## ThreadLocal内部结构

构造了静态内部类(ThreadLocalMap)

Map里面存储ThreadLocal对象（key）和线程的变量副本（value）

![在这里插入图片描述](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/20210124120125266.png)

set（）源码

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

##Thread属性

每个Thread线程内部都有一个Map (ThreadLocalMap)

ThreadLocal.ThreadLocalMap **threadLocals** ；

##ThreadLocalMap

ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部的Entry也是独立实现。

## 弱引用和内存泄漏

- Memory overflow:内存溢出，**没有足够的内存提供申请者使用**。
- Memory leak: 内存泄漏是指程序中已动态分配的堆内存由于某种原因程序**未释放或无法释放，造成系统内存的浪费**，导致程序运行速度减慢甚至系统崩溃等严重后果。内存泄漏的堆积终将导致内存溢出。

###强引用

**threadLocal泄漏**

* 如果业务使用完ThreadLocal，栈引用(threadLocal Ref)回收

* 但如果map中key一直指向ThreadLocal，会导致**threadLocal不能被回收**。

![在这里插入图片描述](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/20210124144330949.png)

###弱引用

**弱引用（WeakReference）**，垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

**entry的泄漏**

* 如果业务使用完ThreadLocal，栈引用(threadLocal Ref)回收
* 但如果map中key一直弱引用指向ThreadLocal，会导致**threadLocal被回收**后，**key == null**。

* 1.没有删除这个Entry ； 2.CurrentThread依然运行
  * 当前线程指向map，value无法回收，但value不会被用到。



![在这里插入图片描述](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/20210124144945153.png)

##内存泄漏的真实原因

1. 没有删除Entry
2. 当前线程依然运行依然运行

**随着当前线程的结束，没有指向map的引用，map也会随着被回收。**

​	