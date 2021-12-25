EventLoop: 一个excutor 里面有一个线程、

EventExecutor[] child

```java
children[i] = newChild(executor, args);
出啊纪念馆建
```

##NioEventLoopGroup

<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210824142044651.png" alt="image-20210824142044651"  />

<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210824142343942.png" alt="image-20210824142343942" style="zoom:67%;" />

MultithreadEventExecutorGroup

属性：

```java
     private final EventExecutor[] children;
```

构造方法：



NioEventLoopGroup

```java
@Override
protected EventLoop newChild(Executor executor, Object... args) throws Exception {
    return new NioEventLoop(this, executor, (SelectorProvider) args[0],
        ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2]);
}
```



ChannelInitializer

`protected abstract void initChannel(C ch) throws Exception;`

> 一旦Channel被注册，这个方法就会被调用。 方法返回后，此实例将从Channel的ChannelPipeline中删除。

#ServerBootstrap

##bind

