Collections转化成并发容器：

这些方法大都以synchronized为前缀，使用这些方法转化成并发容器，是在操作上加上了synchronized锁，并且含有一个可以传入锁对象的重载版本（即在什么对象上加锁）。

LinkedBlockingQueue的底层结构， LinkedBlockingQueue是怎么实现阻塞的；

`CopyOnWrite`：

写操作在一个复制的容器上进行，读操作还是在原始容器中进行，读写分离，互不影响。

写操作：需要加锁，防止并发写入时导致写入数据丢失。写操作结束之后需要把原始容器指向新的复制容器。

**适用场景：**

在写操作的同时允许读操作，大大提高了读操作的性能，因此很**适合读多写少**的应用场景

但是 `CopyOnWrite`有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 `CopyOnWrite` **不适合内存敏感**以及对**实时性**要求很高的场景。

常见

- `CopyOnWriteArrayList`



`Vector`：

实现与 ArrayList 类似，使用了 synchronized 进行同步

Vector 每次扩容请求其大小的 2 倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍



`ConcurrentHashMap`：

这个类在1.7使用分段锁，段为segment，并发度与segment数量相等

在1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized，1.8 的实现也在链表过长时会转换为红黑树。

虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。

<u>*todo：1.8中的具体加锁流程？*</u>

`HashTable`：

是整个数据结构一把锁，使用 synchronized 来保证线程安全。



`BlockingQueue`：

并发List

Vector

它的实现与 ArrayList 类似，都实现了List接口，它们都是有序的集合(存储有序)，底层是数组。但是使用了 synchronized 进行同步，并且因此降低了性能。

**扩容：**Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在**扩容时使容量 capacity 增长 capacityIncrement**。如果这个**参数的值小于等于 0或者没有设置**，扩容时每次都令 capacity 为原来的**两倍**。

CopyOnWriteArrayList 

<u>*todo: 暂不进行在这里讲解*</u>

Collections.synchronizedList()

``` java
List<String> synList = Collections.synchronizedList(new ArrayList(...));
List<String> synList = Collections.synchronizedList(new LinkedList(...));
```

<u>*todo: 暂不进行在这里讲解,更多请移步util类库讲解的文章*</u>



HashTable

全表锁

ConcurrentHashMap之前是分段锁，现在是每个数组位置是一个锁

Hashtable和ConcurrentHashMap，哪些方法是有锁的，哪些是没有的

线程安全吗？线程不安全的出现场景？怎么使之安全？

有哪些线程安全的 Map

todo: 补充内容

> HashMap和Hashtable的区别(不保证正确)

**共同点：**

- 从存储结构和实现来讲基本上都是相同的，都是实现Map接口~

**区别：**

- **同步性：**

- - HashMap是非同步的
  - Hashtable是同步的
  - 需要同步的时候，我们往往不使用，而使用ConcurrentHashMap[ConcurrentHashMap基于JDK1.8源码剖析](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484161&idx=1&sn=6f52fb1f714f3ffd2f96a5ee4ebab146&chksm=ebd74200dca0cb16288db11f566cb53cafc580e08fe1c570e0200058e78676f527c014ffef41&scene=21#wechat_redirect)

- **是否允许为null：**

- - HashMap允许为null
  - Hashtable不允许为null

- **contains方法**

- - 这知识点是在牛客网刷到的，没想到这种题还会有(我不太喜欢)….
  - Hashtable有contains方法
  - HashMap把Hashtable的contains方法去掉了，改成了containsValue和containsKey

- **继承不同：**

- - HashMap

    extends AbstractMap

  - public class Hashtable

    extends Dictionary





任务缓冲通过一个阻塞队列来实现的。阻塞队列缓存任务，工作线程从阻塞队列中获取任务。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

<img src="E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor阻塞队列示意表.jpg" style="zoom:75%;" />

> 关于`workQueue`可以选择以下几个阻塞队列

1. `ArrayBlockingQueue`：是一个基于**数组结构**的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
2. `LinkedBlockingQueue`：一个基于**链表结构**的阻塞队列，此队列按FIFO （先进先出） 排序元素，**吞吐量**通常要高于`ArrayBlockingQueue`。静态工厂方法`Executors.newFixedThreadPool()`使用了这个队列。
3. `SynchronousQueue`：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，**吞吐量**通常要高于`LinkedBlockingQueue`，静态工厂方法`Executors.newCachedThreadPool`使用了这个队列。
4. `PriorityBlockingQueue`：一个具有优先级得无限阻塞队列。