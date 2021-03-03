# Copy-On-Write

Copy-On-Write简称COW（不会产奶的奶牛），是一种用于程序设计中的优化策略。Copy On Write技术在Linux和文件系统中均有应用，Linux通过Copy On Write技术极大地**减少了Fork的开销**，文件系统通过Copy On Write技术一定程度上保证**数据的完整性**。

- （<u>*todo: 专门写一篇文章去阐述Linux上的这个事情*</u>）

而在Java的concurrent包中也有他的身影，本节就是阐述concurrent包中的Copy On Write技术。

CopyOnWrite容器是写时复制的容器。它的的基本思路是，从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种**延时**懒惰**策略**。

concurrent包中含有依据COW的组件：**CopyOnWriteArrayList和CopyOnWriteArraySet**

- **添加元素的过程：**

  通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素（添加过程需要加锁），添加完元素之后，再将原容器的引用指向新的容器。

  ``` java
      public boolean add(E e) {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              Object[] elements = getArray();
              int len = elements.length;
              Object[] newElements = Arrays.copyOf(elements, len + 1);
              newElements[len] = e;
              setArray(newElements);
              return true;
          } finally {
              lock.unlock();
          }
      }
      final void setArray(Object[] a) {
          array = a;
      }
  ```

  - 在添加的时候是需要**加锁的**，但是这个锁是,否则多线程写的时候会Copy出N个副本出来。

  - **读元素**的时候**不**需要**加锁**，如果读的时候有多个线程正在向容器添加数据，**读**还是会读到**旧数据**，因为写的时候不会锁住旧的容器

- **优点：**

  可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以**CopyOnWrite容器**也是一种**读写分离的思想**，读和写不同的容器。

- **缺点：**

  - **内存占用**问题

    在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象。如果这些对象占用的内存比较大，则会占用比较多的资源，造成长时间GC或频繁的GC。

    **压缩容器中的元素或使用其他并发容器**，如ConcurrentHashMap

  - **数据一致性**问题

    只能保证数据的最终一致性，不能保证数据的实时一致性

- 应用场景：

  读多写少的并发场景。例子比如：

  - 白名单，黑名单
  - 商品类目的访问和更新场景

- 类库组件：**CopyOnWriteArrayList和CopyOnWriteArraySet**

  CopyOnWriteArrayList和CopyOnWriteArraySet分别实现了List接口和AbstractSet接口，其中CopyOnWriteArraySet含有一个CopyOnWriteArrayList实例域，进而实现COW应用。因此它们两个行为的类似

# ConcurrentHashMap

- JDK 1.7：分段锁，Segment继承自重入锁 ReentrantLock，并发度与 Segment 数量相等

- JDK 1.8：CAS+Synchronized，数组+链表+红黑树，使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。

- 在 Java 8 之后， JDK 却弃用了分段锁策略，更新后使用 synchronized+cas，因为他们认为：

  1. 加入多个分段锁浪费内存空间。
  2. 生产环境中， map 在放入时竞争同一个锁的概率非常小，分段锁反而会造成更新等操作的长时间等待。
  3. 为了提高 GC 的效率

- 不使用HashTable的原因：

  HashTable虽然是线程安全的，但是是通过整个来加锁的方式，当一个线程在写操作的时候，另外的线程则不能进行读写

ConcurrentHashMap有一个重要的属性`sizeCtl`：

（ConcurrentHashMap的初始化也对sizeCtl进行了赋值）

- 负数代表正在进行初始化或扩容操作

- -1代表正在初始化

- -N 表示有N-1个线程正在进行扩容操作

- 正数或0代表hash表还没有被初始化，

  这个数值表示初始化或下一次进行扩容的大小，这一点类似于扩容阈值的概念。它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。

Node是最核心的内部类，基本与HashMap的定义类似，但是差别是他对value和next属性设置了volatile同步锁。不允许调用setValue方法直接改变Node的value域。

TreeBin包装了很多TreeNode节点，oncurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这个类带了读写锁。

在ConcurrentHashMap中，随处可以看到U, 即Unsafe。这是用了CAS操作实现无锁化修改值。

ConcurrentHashMap最常用的就是put和get两个方法：

- 有一个最重要的不同点就是ConcurrentHashMap不允许key或value为null值

1. 如果一个或多个线程正在对ConcurrentHashMap进行扩容操作，当前线程也要进入扩容的操作中。这个扩容的操作之所以能被检测到，是因为transfer方法中在空结点上插入forward节点，如果检测到需要插入的位置被forward节点占有，就帮助进行扩容；（helpTransfer方法）
2. 如果检测到要插入的节点是非空且不是forward节点，就对这个节点加锁，这样就保证了线程安全。尽管这个有一些影响效率，但是还是会比hashTable的synchronized要好得多。

helpTransfer是一个协助扩容的方法。这个方法被调用的时候，当前ConcurrentHashMap一定已经有了nextTable对象，首先拿到这个nextTable对象，调用transfer方法。回看上面的transfer方法可以看到，当本线程进入扩容方法的时候会直接进入复制阶段。



Size相关的方法
对于ConcurrentHashMap来说，这个table里到底装了多少东西其实是个不确定的数量，因为不可能在调用size()方法的时候像GC的“stop the world”一样让其他线程都停下来让你去统计，因此只能说这个数量是个估计值。对于这个估计值，ConcurrentHashMap也是大费周章才计算出来的。

https://blog.csdn.net/u010723709/article/details/48007881

ConcurrentHashMap的get操作



SynchronizedMap 和 ConcurrentHashMap 有什么区别？

- SynchronizedMap 一次锁住整张表来保证线程安全，所以每次只能有一个线程来访为 map。

<u>*todo:ConcurrentHashMap 使用了一种不同的迭代方式。在这种迭代方式中，当iterator 被创建后集合再发生改变就不再是抛出ConcurrentModificationException，取而代之的是在改变时 new 新的数据从而不影响原有的数据，iterator 完成后再将头指针替换为新的数据 ，这样 iterator线程可以使用原来老的数据，而写线程也可以并发的完成改变。*</u>

# ConcurrentLinkedQueue?

*<u>todo: ConcurrentLinkedQueue</u>*

*<u>todo: ConcurrentLinkedDeque</u>*

实现一个线程安全的队列有两种实现方式：一种是使用阻塞算法，另一种是使用非阻塞算法。

使用阻塞算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现，而非阻塞的实现方式则可以使用循环CAS的方式来实现，ConcurrentLinkedQueue是使用非阻塞的方式来实现线程安全队列的

# BlockingQueue

*<u>todo: BlockingQueue</u>*

*<u>todo: [聊聊并发（七）——Java中的阻塞队列](http://ifeve.com/java-blocking-queue/)</u>*

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。（即**生产和消费**）

阻塞队列常用于**生产者和消费者的场景**，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

提供了7个阻塞队列。分别是

- ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
- LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
- PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
- DelayQueue：一个使用优先级队列实现的无界阻塞队列。
  - 队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口
  - 可以用于：
    - 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。
    - 定时任务调度。使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，从比如TimerQueue就是使用DelayQueue实现的。
- SynchronousQueue：一个不存储元素的阻塞队列。
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

> ## 阻塞队列的实现原理

使用通知模式实现。所谓通知模式，就是当生产者往满的队列里添加元素时会阻塞住生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。即**使用await、signal实现**

*<u>todo: [聊聊并发（七）——Java中的阻塞队列](http://ifeve.com/java-blocking-queue/)</u>*

*<u>todo: 并发容器和普通容器的迭代器的不同</u>*

<u>*todo: ConcurrentSkipListMap ==> ConcurrentSkipListSet*</u>

# 参考

> 1. [聊聊并发-Java中的Copy-On-Write容器](

