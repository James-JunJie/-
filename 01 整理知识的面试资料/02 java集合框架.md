#集合

## 常用集合类？

Map接口和Collection接口是所有集合框架的父接口： 

1. Collection接口的子接口包括：Set接口和List接口,Queue接口
2. Map接口的实现类主要有：**HashMap**、**TreeMap**、**Hashtable**、 **ConcurrentHashMap**等 
3. Set接口的实现类主要有：**HashSet**、**TreeSet**、**LinkedHashSet**等
4. Queue: **ArrayDeque**，**PriorityQueue**
5. List接口的实现类主要有：**ArrayList**、**LinkedList**、**Stack**以及**Vector**等



![图片](https://mmbiz.qpic.cn/mmbiz_png/og5r7PHGHogIVfibksSZVTgKDVVTRrDeibyE5q0VfMIHfQspLicibz0j05ibrib20wWXSwE6zzMsHwZQPbdkWZGXYN2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Collection接口：

![image-20210830172219283](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830172219283.png)



Map接口：

![image-20210830172233214](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830172233214.png)

## 1.List

### 1.1ArrayList

属性：

* 初始化容量： 10

<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830204017744.png" alt="image-20210830204017744" style="zoom:150%;" />

**ArrayList**底层其实就是一个数组**，ArrayList中有**扩容这么一个概念，正因为它扩容，所以它能够实现动态增长

#### 1.1.1构造方法

未指定返回空。

![image-20210830204355753](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830204355753.png)



#### 1.1.2.add(E e)

到目前为止，我们就可以知道add(E e)的基本实现了：

*  首先去检查一下数组的容量是否足够
  * 足够：直接添加
  * 不足够：扩容
* 扩容到原来的1.5倍
*  第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。



![image-20210830204945661](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830204945661.png)

ensureCapacityInternal()

![image-20210830205109394](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830205109394.png)

grow()

![image-20210830205215929](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210830205215929.png)

Arryas.copyOf()方法底层：

System.arraycopy()方法

```
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

###1.3Iterator

terator也是一个接口，它只有三个方法：

•  hasNext()

•   next()

•   remove()

我们**遍历集合 (Collection) 的元素都可以使用Iterator**，至于它的具体实现是以内部类的方式实现的！

##1.ArrayList和LinkedList有什么区别？

* 底层数据结构：
  * ArrayList：基于动态数组
  * LinkedList:  基于双向链表
* 查询数组：
  * ArrayList: get(i) 时间复杂度O(1)
  * LinkedList:需要遍历链表 ，O(n)
* 插入数组：
  * ArrayList:  直接add(E e)不考虑扩容，O(1)； 指定位置插入，需要调用Array.copy()   
  * LinkedList: 遍历消耗时间，但插入，删除不耗时间。

## 2.ArrayList和Vector的区别?

**共同点：**

* 这两个类都实现了List接口，它们都是**有序**的集合(存储有序)，
* **底层是数组**。我们可以按位置索引号取出某个元素
* **允许元素重复和null**。
* **默认初始容量为10**

**区别：**

*  **同步性：**
  *   ArrayList是非同步的 ,即便需要同步的时候，我们可以使用**Collections工具类来构建出同步的ArrayList**而不用Vector
  *  Vector是同步的 

*  **扩容大小：**
  * Vector  :  原来的2倍
  * ArrayList :  原来的1.5倍

## 3.HashMap和Hashtable的区别

**共同点：**

•       从存储结构和实现来讲基本上都是相同的，都**实现Map接口**

区别：

* 同步性
  * **HashMap是非同步的**
  * Hashtable是同步的
* **是否允许为null**:
  * **HashMap允许为null**
  * Hashtable不允许为null
* **contains**
  * HashMap把Hashtable的contains方法去掉了，改成了**containsValue和containsKey**
  * Hashtable有contains方法
* 初始容量
  * hashMap：16
  * hashTable: 11
* 扩容
  * hashMap：2倍
  * hashTable : 扩容策略是**2倍+1**

除留余数法：
f(key) = key mod p (p<=m)  散列表长为m
**p是 <= 表长的质数(接近m)** **p 不应该是2的整数幂，选择接近2的整数幂的素数最好。**

##4. 集合框架底层的数据结构

###List集合

**Arraylist和Vector**使用的是 **Object 数组**， LinkedList使用**双向循环链表**

###Set集合

HashSet（无序，唯一）：基于 **HashMap** 实现的，HashSet的值作为key，value是Object类型的常量

LinkedHashSet继承HashSet，并且通过 **LinkedHashMap** 来实现的

TreeSet（有序，唯一）： 红黑树(自平衡的排序二叉树。)

###Map集合

HashMap由**数组+链表+红黑树**组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，当链表长度大于阈值（默认为8）并且数组长度大于64时，将链表转化为红黑树

LinkedHashMap（有序） 继承自 HashMap，底层仍然是数组+链表+红黑树组成。另外，LinkedHashMap 在此基础上，**节点之间增加了一条双向链表**，使得可以保持键值对的插入顺序

HashTable： 无序，**数组+链表**组成的，数组是 HashTable的主体，链表则是主要为了解决哈希冲突而存在的

TreeMap ： 有序，红黑树

##3. 集合框架的扩容

* ArrayList和Vector默认初始容量为10，当**元素个数超过容量长度**时都进行进行扩容，**ArrayList扩容为原来的1.5倍**，而**Vector扩容为原来的2倍**

* HashSet和HashMap默认初始容量为16，加载因子为0.75：即当元素个数超过容量长度的0.75倍时，进行扩容，扩容为原来的2倍。HashSet基于 HashMap 实现的，因此两者相同

* HashTable：默认**初始容量为11**，加载因子为0.75，扩容策略是**2倍+1**，如 初始的容量为11，一次扩容后是容量为23

##  ArrayList集合加入1万条数据，应该怎么提高效率

ArrayList的默认初始容量为10，要插入大量数据的时候需要不断扩容，而扩容是非常影响性能的。因此，现在明确了10万条数据了，我们可以**直接在初始化的时候就设置ArrayList的容量**！



##什么是Hash函数和Hash表？

* Hash表：**采用散列技术**将**记录**存储在一块**连续的存储空间**中。

* Hash函数: **存储位置**与**关键字之**间的**对应关系f.**

`存储位置 = f(关键字)`

##Hash函数构造？

* 直接定址法 f(key) =a*key + b
* 数字分析法
* 平方取中法
* 折叠法
* 除留余数法 f(key) = key mod p (p <=m)
* 随机数法

##解决冲突的方式？
* 1.开放地址法 一旦发生冲突，寻找下一个空的散列地址
  * 1.1 线性探测法 1.2二次探测法 1.3 随机探测法
* 2.再散列函数法
* 3.链地址法

##HashMap在Java7和Java8中的实现有什么不同？

对比：

* 插入方式，**避免产生链表环**：
  * 7在遇到Hash冲突之后回把数据按照**头插法**插入到链表中
  * 8是用的**尾插法**
* 数组长度必须大于64，链表的长度达到了8，树化。
* hash()函数
  *  1.7 hash()函数是 采取大量的异或操作
  * 1.8 hash()函数， hash ^ (hash >>> 16)

##HashMap1.7 有时候会死循环，你知道是什么原因吗？

jdk1.7 ,多线程环境下，resize()扩容，导致链表形成环。

*  java1.7 hashMap会产生死锁

*  java1.8不会死锁，但多线程下，数据会丢失，put()的时候，会覆盖头部节点。----------concurrenMap使用CAS去解决数据丢失

##HashMap1.7的扩容过程

首先创建一个2倍大小的数组，进行tranfer()操作。

transfer:

![image-20210902214720014](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210902214720014.png)

coding

```java
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e; 
            e = next;
        }
    }
}
```

##ConcurrentHashMap是怎么实现的？

采用**锁分段技术**提高并发访问效率。**当一个线程占用锁访问其中一个段数据的时候，其他的数据也能被其他线程访问**。

segment数组 + HashEntry数组

segment继承可重入锁(ReentrantLock) 。

####put() 源码：

f = tabAt(tab, i = (n - 1) & hash)    

* **CAS**：Node[i]= =null ,插入在头节点，避免了值的丢失。put（）

* **锁桶** : Node[i] != null, 加锁：  synchronized (f) {

