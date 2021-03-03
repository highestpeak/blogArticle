<u>*todo：没有弄完*</u>

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。

除了下面说的三个组件外：

- 其他类库构件

  DelayQueue、PriorityBlockingQueue、ScheduledExecutor、Exchanger

- 任务间使用管道

  使用PipedWriter类和PipedReader类，PipedReader的建立必须在构造器和一个PipedWriter先关联

# 三个组件

- CountDownLatch：等待多个线程
- CyclicBarrier：多个线程都到达时，这些线程才会继续执行
- Semaphore：控制对互斥资源的访问线程数

**CountDownLatch：**

用来控制一个或者多个线程**等待多个线程**。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

![](E:\_data\博文临时库\博文中的图片\Java并发类库-CountDownLatch.png)

**CyclicBarrier：**

用来控制多个线程互相等待，只有**当多个线程都到达时，这些线程才会继续执行**。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

**Semaphore：**

Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

# AQS

<u>*todo:几个类的使用*</u>

lock包下面的AQS

- AbstractOwnableSynchronizer（通常地：AbstractQueuedSynchronizer简称为AQS）
- AbstractQueuedLongSynchronizer
- AbstractQueuedSynchronizer

lock的子类：ReentrantLock、ReentrantReadWriteLock

AQS（AbstractQueuedSynchronizer）

LongAddr