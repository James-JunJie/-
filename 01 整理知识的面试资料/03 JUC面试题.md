## 1.Java线程中的七种状态

1. **初始(NEW)**：新创建了一个线程对象，但还没有调用start()方法。
2. **就绪(Ready)**:  当调用了线程对象的start()方法即启动了线程，此时线程就处于就绪状态。
3. **运行(RUNNABLE)**：Java线程中将就绪（ready）和运行中（running）两种状态笼统的成为“运行”。
   线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取cpu 的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得cpu 时间片后变为运行中状态（running）。
4. **阻塞(BLOCKED)**：表线程阻塞于锁。
5. **等待(WAITING)**：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
6. **超时等待(TIME_WAITING)**：该状态不同于WAITING，它可以在指定的时间内自行返回
7. **终止(TERMINATED)**：表示该线程已经执行完毕。

<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/930824-20180715222029724-1669695888.jpg" alt="img"  />



## Thread实例interrupt 中断线程

stop()方法的缺点：杀死线程，如果线程锁住了共享资源，杀死后无法释放锁，会导致其他线程无法获取锁。

作用：

1. **interrupt()能够打断 sleep，wait，join 的线程**。并且抛出异常，同时**清空打断标记**。
2. 可以优雅的停止运行的线程，会产生一个打断标记，在if判断有打断标记，break死循环。

* isInterrupted()  ： 不会打断标记
* interrupt() :            会清除打断标记

两阶段终止模式：

sleep()被打断，抛出异常后，**再次设置标记**，处理后事，进行break;

```java
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            while (true) {
                Thread current = Thread.currentThread();
                if (current.isInterrupted()) {
                    System.out.println("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    System.out.println("将结果保存");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    current.interrupt();//重新设置标记
                }

            }
        }
    },"t1");
    t1.start();
    Thread.sleep(100);
    t1.interrupt();
}
```

<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/twoStepCommit.png" alt="twoStepCommit" style="zoom:67%;" />

# Thread.sleep、Object.wait、LockSupport.park 的区别

* sleep是Thread的静态方法，park是LockSupport的静态方法;  wait是Object类的实例方法

* sleep和park不需要和synchronized配合使用，wait需要；

* **sleep和park不会释放对象的锁**，wait会释放锁；

**公共点**：调用三种方法之后，线程都是进入 waiting状态。

**park和unpark**：

> park和unpark方法是LockSupport的静态方法，作用是暂停线程，线程进入waiting状态，unpark唤醒线程，使线程从waiting进入就绪状态。

 先unpark了，后面park时，unpark的唤醒效果还可以起作用：

![img](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/361817-20200718175023376-618420918.png)

##2.用户级线程和内核级线程

>  为了系统的安全性分连个类型

* 用户级线程ULT**(Use**r-Level Thread)：  用户级线程，**主进程创建的伪线程，无cpu时间片使用权限。**
  * 问题：如果cpu执行中，线程1阻塞，其他线程都**阻塞了**。(**线程没有cpu时间片**)
  * 优点：避免过度创建线程，避免大量上下文切换
  * 缺点:   程序自己实现堆栈，算法。
* 内核级线程KLT(**Kernel**-Level Thread)):线程的所有**管理操作都是由操作系统内核完成的**

> 上下文切换会涉及到用户态到内核态的切换原因所在

##3.JMM是什么？

Java内存模型：它为了**屏蔽os的不同**基于**CPU缓存模型建立起来的**的一组**规范**。

* 对于硬件内存来说只有寄存器、缓存内存、主内存的概念，并没有**工作内存(线程私有数据区域)和主内存(堆内存)之分**。

* 也就是说Java内存模型对内存的划分对硬件内存并没有任何影响。

![image-20210901210751229](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210901210751229.png)

##4.JMM的三大特性可见性，原子性，有序性(volatile)

### 4.1可见性：

可见性就是指当**一个线程修改了线程共享变量的值，其它线程能够立即得知这个修改**。

实现：它借鉴cpu缓存架构，balabala,讲 mesi.

java 中 volatile 修饰的变量，支持了可见性。

![image-20210901211348324](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210901211348324.png)

### 4.2原子性

原子性指的是**一个操作是不可中断**的，即使是在多线程环境下，一个操作一旦开始就**不会被其他线程影响**.

####4.2.1为什么volatile 不保证原子性?

举一个例子：

10个线程进行 count++ ，会出现两个问题：

* 第一：**jvm中count++不是原子指令**
  * iload_1   //从局部变量表的slot_1位置加载变量到操作数栈中
  * iinc         // 对slot_1位置的变量进行+1操
  * 结果会小于预期值...

* 第二**：数据为什么会少与实际数据**？
  * 从mesi角度讲：**线程2数据的失效**，损失指令，导致本次add操作无效

<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210901212920594.png" alt="image-20210901212920594" style="zoom:80%;" />

#### 4.2.2如何保证原子性？

* 加锁
* 使用juc中Atomic包下的类。

 **synchronized和Lock**如何实现原子性？

原理：锁住临界资源，一次只有一个线程使用临界资源(**同步执行**)

**Atomic包下的类如何保证原子性？**

使用了Unsafe类，使用cas基于比较和交换的方法，保证i++这样操作的原子性。

###4.2.3有序性

> **由上个问题，线程2数据的无效，线程2不能立马读入数据，数据可能在更新，cpu不能一直等待，所以让后面的程序先行执行。**

####重排序

**为了提高性能，编译器和处理器会对指令进行重排序**

![image-20210901215449852](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210901215449852.png)

1. **编译器**重排序. class--------> 汇编 
2. **处理器**重排序(2,3). 汇编----->cpu执行    

##### java如何禁止重排序？

> 每个 **volatile写**的后面插入一个 写读屏障.

java编译器在生成指令序列时， 通过**内存屏障指令**来禁止处理器重排序

**as-if-serial语义**：不管怎么重排序，**(单线程)**程序的**执行结果**不能被改变.

>  为了遵守as-if-serial语义：编译器和处理器**不会对存在数据依赖关系 的操作做重排序**，因为这种重排序会改变执行结果，但是操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。给人幻觉，单线程程序是按照程序的顺序执行的

如果**存在数据依赖关系的操作，不会重排序**，反之不存在依赖关系，会重排序

**happens-before** 原则: 前一个操作(**执行结果**)对后一个操作可见.并不代表，前一个操作在后一个操作之前执行。

## 5.内存屏障

内存屏障插入的时间？**编译器在生成字节码时作出优化**

* storestore：写写，p26
* storeload:**写读屏障**，是一个**全能型屏障**
* loadload:读读
* loadstore:读写

#####保守策略：

* 每个**volatile写后**面，插入一个StoreLoad屏障

* 每个**volatile 读前**面，插入一个StoreLoad屏障

但内存屏障开销很昂贵。

#####JMM加屏障策略：

JMM最终选择了在每个 **volatile写**的后面插入一个**StoreLoad屏障**.

因为volatile写-读(storeload)内存语义的常见使用模式是：一个写线程写volatile变量，多个读线程读同一个volatile变量.

#####使用Unsafe类，手动加屏障

反射获得usafe类，因为他是需要系统类加载器。

```java
//手动加内存屏障
Unsafe类
    public native void loadFence();

    public native void storeFence();

    public native void fullFence();

 public static Unsafe reflectGetUnsafe()
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      return (Unsafe) field.get(null);
```



## 6.JMM同步八种原子操作

1. lock(锁定)：作用于主内存的变量，把一个变量标记为一条线程独占状态
2. unlock(解锁)：作用于主内存的变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
3. **read**(读取)：作用于主内存的变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
4. load(载入)：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中
5. use(使用)：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎
6. **assign**(赋值)：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量让他
7. **store**(存储)：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作 
8. write(写入)：作用于工作内存的变量，它把store操作从工作内存中的一个变量的值传送到主内存的变量中

主内存(lock)--->(read--->load)---->use------cpu

主内存(unlock)<---(write<--store)<---asign<----cpu

> 其中 read与load是原子操作，store和write

![image-20210901210823041](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210901210823041.png)



## 7.JVM内存区域与JMM

* JVM进程，它去申请空间时，操作**逻辑空间**，是由操作系统分配的内存。是存在的。

* JMM: 是一组**规范**，为了**屏蔽os的不同**。

## 8.什么是总线风暴？

cas与volatile 需要与内存大量的交互。

* 导致大量无效工作内存变量
* 嗅探(大量的总线交换)
* 导致**总线通信信道被占用**

##9.单例模式的双重检查锁定与延迟初始化

DCL(Double-Checked Locking) ：

dcl的优缺点:

* 缺点： 直接在getInstance()方法上加锁消耗性能

* 优点
  * 多个线程同时创建对象，**会通过加锁保证只有一个线程创建**。
  * 对象创建好后，**执行getInstance()方法不需要获得锁**。

> 但sychronized代码块内 不保证指令重排序

解决: 对myinstance加volatile修饰

第二端检锁：

```java
private volatile static Singleton myinstance;

public static Singleton getInstance() {
    if (myinstance == null) {
        synchronized (Singleton.class) {
            if (myinstance == null) {
                myinstance = new Singleton();//对象创建过程，本质可以分文三步
                //对象延迟初始化
            }
        }
    }
    return myinstance;
}
```

对象初始化的3步( myinstance = new Singleton()):

1. address = allocate //申请内存空间
2. instace(Object)实例化对象
3. myinstace = address

高并发中，如果2,3发生重排序

ex:

T1指令重排序(2,3重排序)，那么myinstance != null ,但myinstance是没有值的。

T2 判断myinstance != null,返回myinstace，而myinstance是没有实例化对象的。

![image-20210902152246278](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902152246278.png)

## 10.为什么需要加锁？

加锁目的是：**序列化访问临界资源**，即同一时刻只能有一个线程访问临界资源(**同步互斥访问**)

##11.锁分类:

* **显示锁**:  ReentrantLock，实现了JUC里的Lock，基于AQS实现，需要手动加锁跟解锁。 lock() , unLock()
* **隐式锁**:  synchronized加锁机制 ,  Jvm内置锁，不需要手动加锁与解锁。

![image-20210902162315454](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902162315454.png)

## 12.sychronized对象锁

1. 同步**实例方法**，锁是当前**实例对象**
2. 同步**类方法(static**)，锁是当前**类对象**
3. 同步**代码块**，**锁是括号里面的对象**

###sychronized底层原理:

sychronized底层通过指令：

* monitorenter/monitorexit执行（虚拟机生成）隐式锁

* 通过内部对象**Monitor**(监视器锁)实现,synchronized(Object)

![image-20210902161013027](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902161013027.png)



每一个object对象创建之后,都会在**jvm内部维护一个与之对应的monitor(管程)**

#### 管程monitor

管理**共享变量**以及对**共享变量操作**的过程。管程封装了同步操作，对进程隐蔽了同步细节,简化了同步功能的调用界面。

引入管程的原因

> 信号量机制的缺点：进程自备同步操作，P(S)和V(S)操作大量分散在各个进程中，不易管理，易发生死锁。
> 管程特点：管程封装了同步操作，对进程隐蔽了同步细节，简化了同步功能的调用界面。

我们这里就说一下ObjectMonitor中几个关键字段的含义：

- _count：记录owner线程获取锁的次数。这句话很好理解，这也决定了synchronized是可重入的。
- _owner：指向拥有该对象的线程
- _WaitSet：存放处于**wait状态的线程队列**。 
- _EntryList：存放**等待锁**而被**block的线程队列**。

![img](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/1099419-20200128201152301-1784803913.png)

##对象的内存结构：

* 对象头：Mark Word + 类型指针
  * Mark Word: 哈希码，gc分代年龄，锁状态标志，线程持有的锁，偏向线程ID
  * 类型指针: 指向它的类型元数据的指针，确定该对象是哪个类的实例
* 实例数据: 创建对象中成员变量，方法等
* 对齐填充: **8字节整数倍**

MarkWord是**动态定义的数据结构**.(JVM p51)

##实例对象存储在哪个位置?

Object实例对象一定时存在堆区的吗？

>  不一定，如果实例对象没有线程逃逸行为，则可以分配在栈上

* 堆区：有逃逸行为
* 栈区：无逃逸行为

逃逸分析（P418）: 该对象是否被其他方法、线程所引用

实例优化：

* 栈上分配：不允许对象逃逸出**线程**范围
* 标量替换：不允许对象逃逸出**方法**范围(是栈上分配的特例)



## CAS乐观锁

> 如果要阻塞和唤醒一条线程，需要陷入用户态--->内核态，需要**耗费时间**

无锁编程：基于**冲突监测的并发策略**，不管风险，先进行操作，如果**没有其他线程争用共享数据**，那**操作就直接成功了**。

CAS指令：内存位置V，旧的预期值A，设置的新值B.

cas: 当且仅当**变量V符合A时，V的值更新成B.**

###为什么需要硬件指令集的发展?

需要**操作**与**冲突监测**这两个步骤具有原子性。

使用处理器指令，x86指令有用**cmpxchg**指令完成**CAS(Compare-and-Swap)功能**

缺点：

1.资源开销比较大

* cpu压力大，但自旋会由**一个次数限制，如果超过后就会放弃时间片**.
* 大量内存交互，容易总线风暴。

2.ABA

> ABA问题:如果变量V初次读取时是A，赋值的时候仍然是A值，就能说明没有被其他线程改动？ A ---->B-------->A

解决：记录一下**变量的版本**就可以了，在变量的值发生变化时对应的版本也做出相应的变化，然后CAS操作时除了比较和预期值是否一致外，再**比较一下版本**，就知道变量有没有发生过改变了。

**atomic**包下**AtomicStampedReference**类实现了这种思路。Mysql中Innodb的多版本并发锁也是这个原理。

##锁粗化与消除

因为：StringBuffer的append方法是加sychronized的，反复进入互斥同步，消耗性能

* 锁粗化：只加锁一次，在第一个append之前
* 锁消除： 逃逸分析，数据不会被其他**线程**访问，认为**线程私有**，不用加锁

锁粗化：

```java
StringBuffer stb = new StringBuffer();
public void test1(){
    //jvm的优化，锁的粗化
    stb.append("1");
    stb.append("2");
    stb.append("3");
}
```

锁消除：StringBuffer 是方法内，栈栈帧中，虚拟机栈是线程私有的。

```java
public void test1(){
    StringBuffer stb = new StringBuffer();
    //jvm的优化，锁的粗化
    stb.append("1");
    stb.append("2");
    stb.append("3");
}
```

##自旋锁与自适应自旋锁：

如果**共享数据的锁状态只会持续一段时间**，为了这段时间去**挂起和恢复线程并不值得**。

* 自旋锁：....让后面**请求锁的线程执行一个忙循环(自旋)** , 不会放弃处理器执行时间。

* 自适应自旋：自旋时间，由**前一次**同一个**锁上的自旋时间**决定，避免了cpu自旋，浪费资源。
  * 刚刚获得锁------自旋时间增大
  * 很少获得锁------自旋时间短或无



![image-20210902163934722](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902163934722.png)



##偏向锁与轻量级锁

###1偏向锁：由同一线程多次获得锁

核心思想是，**如果一个线程获得了锁，那么锁就进入偏向模式,再次请求锁时，无需做任何操作**。

偏向锁的目标是: 减少**无竞争**且**只有一个线程使用锁**的情况下，使用**轻量级锁**产生的性能消耗（CAS）

偏向锁不会自动释放，除非有其他线程竞争，膨胀为轻量级锁

###2 轻量级锁: 所适应的场景是线程**交替执行**同步块的场合

轻量级锁的目标是： 减少**无实际竞争**情况下，使用**重量级锁产生的性能消耗**.

由于轻量级锁天**然瞄准不存在锁竞争的场景**，如果存在锁竞争但不激烈，仍然可以用**自旋锁优化**，自旋失败后再**膨胀为重量级锁。**

0无锁 1偏向锁.

![image-20210902165727796](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902165727796.png)

##synchronized锁的膨胀升级

JVM内置锁在1.5之后版本做了重大的优化，

![image-20210902165956773](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902165956773.png)

无锁：未被线程占用

无锁--->偏向锁---->轻量级锁--->重量级锁的大概步骤:

1. 如果**标志位**：01(无锁)
2. 获取偏向模式
   0. 无偏向,将mark Word 的偏向线程ID指向自己
   1. 偏向，检查ID是否是自己，不是自己CAS修改，失败-->轻量级锁
3. 当前线程栈帧建立锁记录(Lock Record)(存储锁对象Mark Word)
4. CAS尝试把对象Mark Word指向Lock Record
   1. 成功,处于**轻量级锁**
   2. 失败，自旋尝试，如果还失败达到阈值之后膨胀为重量级锁

##AQS(抽象队列同步器)是什么？

AQS:  **构建锁或者其他同步组件**的**基础框架**。

* 锁：是面向使用者的

* 同步器：面向的是锁的实现者。

##AQS支持几种同步方式？

* Exclusive-**独占**，只有**一个线程能执行**，如ReentrantLock
* Share-**共享**，多个线程可以同时执行，如Semaphore/CountDownLatch

##AQS框架的实现

* 一般通过**定义内部类承AQS**,重写指定的方法。比如tryAcquire()实现公平和非公平锁等。

2.1 AQS内部维护属性**volatile int state (32位)**

* state表示资源的**可用状态**

####2.2 State三种访问方式

* getState()、setState()、compareAndSetState() 

####2.3 AQS定义两种资源共享方式

* Exclusive-独占，只有**一个线程能执行**，如ReentrantLock
* Share-共享，多个线程可以同时执行，如Semaphore/CountDownLatch

####2.4 AQS定义两种队列

* **同步**队列(CLH):

* **条件**队列:

####2.5 AQS同步器:

自定义同步器通过内部类Sync继承AbstractQueuedSynchronizer实现

同步器的设计是基于**模板方法**模式的，重写aqs方法。

父类提供模板方法：

void acquire(int arg) : 独占是获取同步状态，如果获取同步状态成功 ，





##ReentrantLock与synchronized的区别



##ReentrantLock实现原理



##Java原子类AtomicInteger实现原理





## isInterrupted()和interrupted()区别

* isInterrupted()：**不会**清除断标记

* interrupted() :    **会清除**打断标记

因为interrupt不会强制停止线程。**所以打断标记，用于(判断)停止线程。**

##acquire(int arg)源码

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

![acquire源码](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/acquire%E6%BA%90%E7%A0%81.png)

##CyclicBarrier和CountDownLatch的区别

| CountDownLatch              | CycliBarrier(珊栏)                  |
| --------------------------- | ----------------------------------- |
| 当计数器值到达0时，开始     | 等计数器到达指定值，可以开始        |
| 内部类实现了aqs             | 使用ReentrantLock和Condition        |
| 计算为0时释放所有等待的线程 | 计数达到指定值时释放所有等待线程    |
| 不可以重复使用              | 可以重复使用                        |
| 计数为0时，无法重置         | 计数达到指定值时，计数置为0重新开始 |

```
 cyclicBarrier.await();//唤醒
 countDownLatch.await();//唤醒
```

## 为什么使用线程池？

**频繁的创建和销毁线程，消耗计算机资源**，因为创建线程需要上下文切换。

## 线程池的优点？

线程池优势

* **减少创建线程**的开销，提高性能
* 提高**响应速度**。当任务到达时，任务可以不需要的**等到线程创建就能立即执行**
* 提高线程的**可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控

> 线程创建需要用户态-->内核态   消耗性能

##什么时候使用线程池？

* **单个任务处理时间比较短**
* 需要处理的**任务数量很大**

##线程创建的几种方式

###1.Thread的run方法

```java
// 创建线程对象
Thread t = new Thread() {
public void run() {
	// 要执行的任务
	}
};
// 启动线程
t.start();
```

###2.实现Runnable

```java
Runnable runnable = new Runnable() {
public void run(){
	// 要执行的任务
 }
};
// 创建线程对象
Thread t = new Thread( runnable );
// 启动线程
t.start();
```

###3.实现Callable

```java
FutureTask<Integer> task = new FutureTask<Integer>(() -> {
    return 100;
});
Thread t = new Thread(task,"t1");
t.start();
```

###Callable接口：

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

###Runnable接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

##线程池存在5种运行状态

| RUNNING = 1                          | 线程池创建，就处于RUNNING状态                                |
| ------------------------------------ | ------------------------------------------------------------ |
| SHUTDOWN = 0   **处理**              | **不接收新**任务，**但能处理已添加的任务**。                 |
| STOP = 1    **不处理** showdownNow() | 不接收新任务，**不处理已添加的任务**，**中断正在处理的任务**。 |
| TIDYING = 2                          | **ctl记录的”有效线程的数量”为0**，线程池会变为**TIDYING**状态。 |
| TERMINATED = 3                       | 执行terminated()                                             |

​	![image-20210903150425654](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210903150425654.png)

##线程池属性

线程的大小(maximumPoolSize) = 核心线程(corePoolSize) + 非核心线程

* **CTL**：CTL( 32位 )：线程池的**运行状态(runState)**( 3位 )+ 有效**线程数量**(低29位)

* **1.corePoolSize(核心线程数)**
* **2.maximumPoolSize(最大线程数)**
* **3.keepAliveTime（运行空闲时间）**
* **4.workQueue(阻塞队列)**
  * **4.1 ArrayBlockingQueue**：基于数组结构的**有界阻塞队**列，按FIFO排序任务；
  * **4.2 LinkedBlockingQuene**：基于**链表结构的阻塞队列**，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；
  * **4.3 SynchronousQuene**：一个**不存储元素的阻塞队列**，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene
  * **4.4 priorityBlockingQuene**：具有**优先级的无界阻塞队列**；
* **5.threadFactory(线程工厂)**:
* **6.handler(饱和策略)**
  * **直接抛出异常(**AbortPolicy)，默认
  * **调用者所在线程执行**任务(CallerRunsPolicy) 
  * **丢弃阻塞队列中靠前任务**，并执行任务(DiscardOldestPolicy)
  * **直接丢弃任务**(DiscardPolicy)

## execute()执行

1.**线程数量 <  核心线程**，添加到**核心线程并执行**

2.否则，**阻塞队列没满**，**添加任务到阻塞队列**。

3.如果阻塞队列满，如果**线程数量 < 最大线程数**，那么**添加到工作线程并执行**

4.否则，执行拒接策略。

![image-20210903152219367](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210903152219367.png)

##线程池的生命周期：

![	](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

##线程池参数设置

多线程：

* **计算密集型**:  **线程数 = cpu+1**任务比较占cpu，
* **I/O型**:  **2*CPU核心数**,  任务主要时间消耗在 IO等待上，cpu压力并不大，所以线程数一般设置较大。
  * （**线程等待时间+线程CPU时间）/线程CPU时间** ）* CPU数目

###为什么这样设置？

计算密集型：**让计算型线程都占用cpu，减少上下文切换。**

I\O型：**等待时间长，cpu空闲，尽量设置大的线程数，不让cpu空闲。**

##高并发、任务执行时间短的业务怎样使用线程池？并发不高、任务执行时间长的业务怎样使用线程池？并发高、业务执行时间长的业务怎样使用线程池？

* 高并发，任务执行时间短的业务： CPU核数+1，**减少线程上下文的切换**。
* 并发不高、任务执行时间长的业务要区分开看：
  *  业务是I\O型：**(线程等待时间+线程CPU时间）/线程CPU时间** ）* CPU数目
  * 计算型：线程数 = cpu+1
* 并发高、业务执行时间长:    
  * 能否做缓存，增加服务器。
  * 业务时间长，使用中间件对任务进行拆分和解耦。

##大数组排序

* 使用内部排序，内存不够。栈oom错误。

* 使用外部排序
  * 分：将大文件拆分成小文件，在内存中排序。
  * 合：将小文件合并成大文件。（选取各个文件中的第一个数，得到最小值）

合并详细：

```java
文件内有序：
文件1：3,6,9
文件2：2,4,8
文件3：1,5,7
//选取各个文件中的第一个数，得到最小值
第一步：
这3个文件中的最小值是：min(1,2,3) = 1
上面拿出了最小值1，写入大文件.
第二步：
那么，这3个文件中的最小值是：min(5,2,3) = 2
将2写入大文件.
```

##ThreadLocal知道吗？

ThreadLocal类用来提供**线程内部的局部变量**。**能保证各个线程的变量相对独立**,不同的线程之间不会相互干扰。



##强弱引用和内存泄漏

* Memory overflow:内存溢出，**没有足够的内存提供申请者使用**。

* 内存泄漏：  堆内存**未释放或无法释放，造成系统内存的浪费**，

###强引用

>  **threadLocal泄漏**

* 如果业务使用完ThreadLocal，**栈引用(threadLocal Ref)回收**

* 但如果map中**key一直指向ThreadLocal**，会导致**threadLocal不能被回收**。

![在这里插入图片描述](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/20210124144330949.png)

###弱引用

> **entry的泄漏**

**弱引用（WeakReference）**，垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

* 如果业务使用完ThreadLocal，栈引用(threadLocal Ref)回收
* 但如果map中key一直弱引用指向ThreadLocal，会导致**threadLocal被回收**后，**key == null**。

* 1.没有删除这个Entry ； 2.CurrentThread依然运行
  * 当前线程指向map，value无法回收，但value不会被用到。

![在这里插入图片描述](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/20210124144945153.png)

###内存泄漏的真实原因

1. 没有删除Entry
2. 当前线程依然运行依然运行

**随着当前线程的结束，没有指向map的引用，map也会随着被回收。**





迭代器的快速失败

9.  了解过什么是“伪共享”吗？
10. “伪共享”出现的原因是什么？
11. 如何避免“伪共享”？
12. Java里的线程有哪些状态？
13. 什么是悲观锁？什么是乐观锁？
14. 怎么停止一个运行中的线程？
15. 说一下你对volatile的理解？
16. 并发编程三要素？
9. 线程池的优点？
19. 什么是CAS？
20. CAS的问题
23. 什么是自旋锁？
24. 什么是多线程的上下文切换？
25. 什么是线程和进程?
26. 程序计数器为什么是私有的?
27. 虚拟机栈和本地方法栈为什么是私有的?
28. 并发与并行的区别？
29. 什么是线程死锁?如何避免死锁?
30. sleep() 方法和 wait() 方法的区别和共同点?
31. 为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？
32. 什么是线程安全问题？如何解决？
33. 什么是活锁？
34. 什么是线程的饥饿问题？如何解决？
35. 什么是线程的阻塞问题？如何解决？
36. synchronized 关键字和 volatile 关键字的区别
37. 说一说几种常见的线程池及适用场景？
38. 线程池都有哪几种工作队列？
39. 什么是线程安全？
40. Java中如何获取到线程dump文件
41. Java中用到的线程调度算法是什么？
42. Thread.sleep(0)的作用是什么？
43. 单例模式的线程安全性
44. Semaphore有什么作用？
45. Hashtable的size()方法中明明只有一条语句"return count"，为什么还要做同步？
35. 同步方法和同步块，哪个是更好的选择？
