---

---

# 综述

线程池首要解决的就是线程的频繁创建和销毁问题，即**线程资源的管理**。

如果不进行线程资源管理，可能会带来一系列的问题，相应的，我们进行这样的管理，就会带来额外的好处：

- **频繁申请/销毁**资源和调度资源，将带来额外的消耗
  - 提高**响应速度**，不需要的等到线程创建时间
- 对资源无限申请缺少抑制手段，易引发系统**资源耗尽**的风险
  - 降低**资源消耗**
- 系统无法合理管理内部的**资源分布**，会降低系统的稳定性
  - 提高线程的**可管理性**

由于线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，这是我们降低资源消耗的意义。另外，使用线程池可以进行统一的分配，调优和监控，线程池还具备可拓展性，允许开发人员向其中增加更多的功能，比如延时定时线程池`ScheduledThreadPoolExecutor`，就允许任务延期执行或定期执行。

---

线程池将**线程**和**任务**解耦，并不直接关联。当把两者解耦后，线程池可以应用一系列的设计：**命令设计模式**、**生产者消费者模型**、**管理功能**。

- 从之后的继承图我们会看到，线程池继承体系中有一个`ExecutorService`，这个接口提供了一种将“任务提交”与“任务执行”分离开来的机制(解耦线程和任务)，回顾设计模式，这很像命令设计模式，也是我们有了该模式的好处和便利。
- 线程池在内部构建了一个**生产者消费者模型**，线程是消费者，任务作为被消费的对象，持有任务的是阻塞队列，工作线程从阻塞队列中获取任务。这样既有一个良好的逻辑，也良好缓冲了任务。
- 线程池一方面要维护自身的生命周期，另一方面又要**同时管理线程和任务**，使两者良好的结合从而执行并行任务。

可以看到，线程池的任务繁重，也说明了我们的任务繁重（这是否是一个并不是任何时候都能满足单一职责原则的例子？）

# 继承体系

java的concurrent并发类库也处处体现着面向接口编程而不是面向实现编程，这需要我们认真体会一下从Executor到`ExecutorService`到Abstract到最后实现类的这条继承链。（当然你是一个很熟练的OO编程者就当我废话）

- 在我看来Executor不是面向线程池的（或者说面向线程Thread），而是面向Runnable，面向任务的。他有点类似于回调方法

- `ExecutorService`也仍然是面向任务的，即面向Runnable这些的，它提供了系列管理任务的待实现方法。

  提供了一种将“任务提交”与“任务执行”分离开来的机制(解耦)。类似命令设计模式。

- <u>*todo：`AbstractExecutorService`*</u>

![](E:\_data\博文临时库\博文中的图片\drawio\Java并发类库-线程池.png)

- `AbstractExecutorService`则是上层的抽象类：

  将执行任务的流程串联了起来，保证下层的实现只需关注一个执行任务的方法即可。

- 最下层的实现类`ThreadPoolExecutor`实现最复杂的运行部分

- `ForkJoin`：主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。实现了工作窃取算法来提高 CPU 的利用率

有时候我们并不是直接创建线程池，而是通过内置的或一些类库的Util类来获取工厂创建的线程池。这个工厂就是 `java.util.concurrent.Executors`

对shutdown的调用可以防止新任务被提交给这个`ExecutorService`，当前线程（本例main线程）将继续运行在shutdown被调用之前提交的所有任务，程序将在`ExecutorService`的所有任务完成之后尽快退出。

# 生命周期/状态管理

## 构造方法/核心参数

在下面讲解参数的时候，请记住以下几点：

- 线程数 = 核心线程数 + 非核心线程数
- 核心线程数即`corePoolSize`（可能小于这个值，因为还没到这个值）
- 非核心线程即`corePoolSize`到 `maximumPoolSize` 之间的线程
- 活动线程和工作线程意思相同，他们和核心线程不是一类概念

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {...;}
```

- `corePoolSize`（线程池的基本大小）（核心线程数量）：

  **当线程数少于`corePoolSize`时，线程池会一直创建新线程，或者调用`prestartAllCoreThreads`方法来提前创建所有核心线程**

- `maximumPoolSize`（线程池最大大小）:

  （这就是令人迷惑的地方了）

  就是和他的本意相同：线程池的线程的最大线程数

这两个size是如此的令人迷惑，我从[StackOverflow](https://stackoverflow.com/questions/17659510/core-pool-size-vs-maximum-pool-size-in-threadpoolexecutor)上找到一个回答：

> 🔴🔴🔴举一个例子，初始化线程池核心池大小为5，最大池大小为10，队列容量100。
>
> 开始时线程池有一个活动线程，也就是线程池最开始大小为1。随着任务的提交和执行，线程池将创建新线程，直到数量达到5。当数量到达5之后，新的任务将提交到队列，直到队列中任务数量达到队列容量100。当队列容量满后，将创建新的线程，直到达到最大线程数即10。如果这时侯又有新任务到来，并且没有县城空闲且队列满，那么就会拒绝任务。幸运的是，队列中的任务数量会不断被消耗，之后我们可以继续使用线程池。

- `keepAliveTime`（线程活动保持时间）

  非核心线程空闲后，保持存活的时间。

- `TimeUnit`（线程活动保持时间的单位）

- `workQueue`（任务队列）：

  用于保存等待执行的任务的阻塞队列

- `threadFactory`（线程工厂）：

  用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常有帮助

- `handler`（饱和策略）（拒绝策略）：

  当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。

## 状态管理

线程池运行的状态，并不是用户显式设置的，而是伴随着线程池的运行，由内部来维护。

`ThreadPoolExecutor`的运行状态有5种，分别为：RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED

运行状态的相互转化：

![](E:\_data\博文临时库\博文中的图片\drawio\Java并发类库-线程池-线程池状态转换.png)

- 对于我们来说我们只能调用 `shutdown()`和`shutdownNow()`，我们不能调用`terminated()`方法
- `shutdownNow`调用了它的所有线程的interrupt方法

**一个设计上值得学习的地方：**

**（如果为了面试，这部分可以不看）**

线程池内部使用一个变量维护两个值：**运行状态**(`runState`)和**线程数量** (`workerCount`)。在具体实现中，线程池将运行状态(`runState`)、线程数量 (`workerCount`)两个关键参数的维护放在了一起，如下代码所示：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```
高3位保存**运行状态**(`runState`)，低29位保存**线程数量** (`workerCount`)，两个变量之间互不干扰。

- 用一个变量去存储两个值，可避免在做相关决策时，出现不一致的情况，不必为了维护两者的一致，而占用锁资源。通过阅读线程池源代码也可以发现，经常出现要同时判断线程池运行状态和线程数量的情况。线程池也提供了若干方法去供用户获得线程池当前的运行状态、线程个数。这里都使用的是位运算的方式，相比于基本运算，速度也会快很多。

## 关闭线程池

调用线程池的`shutdown`或`shutdownNow`方法来关闭线程池，调用这两个方法后**均会立即从方法中返回**而不会阻塞并等待线程池关闭再返回。`shutdown`只是设置状态，而`shutdownNow`却会调用线程的**interrupt**。

- `shutdown`
  - **实现原理**：只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。
  - 只会告诉执行者服务它**不能接受新任务**，但是**已经提交的任务将继续运行**
    - 再向线程池中提交任务，将会抛`RejectedExecutionException`异常
  - 无返回值
  - 如果线程池的shutdown()方法已经调用过，重复调用没有额外效应

- `shutdownNow`
  - **实现原理**：首先将线程池的状态设置成STOP，然后遍历线程池中的工作线程，然后**逐个调用**线程的**interrupt**方法来中断线程，尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。
  - 由于调用interrupt方法，所以**无法响应interrupt中断的任务可能永远无法终止**。
  - **方法的返回值：**对于那些在**堵塞队列中等待执行的任务**，线程池并不会再去执行这些任务，而是**直接返回这些等待执行的任务**
  - `shutdownNow`调用时：
    - 如果线程处于被阻塞状态，那么线程立即退出被阻塞状态，并抛出一个`InterruptedException`异常。
    - 如果线程处于正常的工作状态，则interrupt标志位置为true，线程会继续执行不受影响
  - 关于interrupt中断，请移步终结任务或线程的讲解

**`isShutdown`和`isTerminaed`：**

- 只要调用了`shutdown`或`shutdownNow`这两个关闭方法的其中一个，`isShutdown`方法就会返回true。
- 当所有的任务都已关闭后,才表示线程池关闭成功，这时调用`isTerminaed`方法会返回true。

至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用`shutdown`来关闭线程池，如果任务不一定要执行完（可能在没有堵塞的时候被突然中断），则可以调用`shutdownNow`。

# 生产者消费者模型

（也可以说这部分讲命令模式在这里的具体实现）

`execute`方法负责`addWorker`、添加任务到队列、`reject`。

判断线程数量和具体添加线程的责任是`addWorker`方法的。任务是由`Worker`的`run`方法，具体的说是`runWorker`方法不断调用`getTask`来获取到并执行的。

这些方法即`execute`、`addWorker`、`runWorker`、`getTask`完成的工作是：**检查现在线程池的运行状态**、运行线程数、运行策略，决定接下来执行的流程：直接执行、缓冲执行或是拒绝执行。

执行的流程就是上面那个说法：

> 🔴🔴🔴举一个例子，初始化线程池核心池大小为5，最大池大小为10，队列容量100。
>
> 开始时线程池有一个活动线程，也就是线程池最开始大小为1。随着任务的提交和执行，线程池将创建新线程【point 1】，直到数量达到5。当数量到达5之后，新的任务将提交到队列，直到队列中任务数量达到队列容量100。当队列容量满后，将创建新的线程，直到达到最大线程数即10。如果这时侯又有新任务到来，并且没有县城空闲且队列满，那么就会拒绝任务。幸运的是，队列中的任务数量会不断被消耗，之后我们可以继续使用线程池。

上述注释的point：

- point 1:使用的是`addWorker`，即添加work线程

1. 



## 管理线程

线程会由线程池分配任务，当线程执行完任务后会继续获取新的任务去执行，当线程最终获取不到任务的时候，线程就会被回收。线程管理包括了：线程创建、线程执行任务、线程回收，另外还有一个AQS的机制。

**worker线程：**

线程池为了掌握线程的状态并维护线程的生命周期，设计了线程池内的工作线程Worker。我们来看一下它的部分代码：

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    final Thread thread; // Worker持有的线程
    Runnable firstTask; // 初始化的任务，可以为null
}
```

Worker这个工作线程，实现了Runnable接口，并持有一个线程thread，一个初始化的任务`firstTask`。这是一种写法：自管理的Runnable。注意这个Worker同时也继承了AQS

- thread是在调用构造方法时通过`ThreadFactory`来创建的线程，可以用来执行任务；

- `firstTask`用它来保存传入的第一个任务，这个任务可以有也可以为null。

  <img src="E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor的worker线程.jpg" style="zoom:75%;" />

**worker线程增加：**

增加线程是通过线程池中的`addWorker`方法，该方法的功能就是增加一个线程，该方法**不考虑线程池是在哪个阶段增加的该线程**，这个分配线程的策略是在上个步骤完成的，该步骤仅仅完成增加线程，并使它运行，最后返回是否成功这个结果。

<img src="E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor的worker线程增加.jpg" style="zoom:50%;" />

**worker线程执行任务：**

在Worker类中的run方法调用了`runWorker`方法来执行任务，`runWorker`方法的执行过程如下：

1. while循环不断地通过getTask()方法获取任务。
2. getTask()方法从阻塞队列中取任务。
3. 如果线程池正在停止，那么要保证当前线程是中断状态，否则要保证当前线程不是中断状态。
4. 执行任务。
5. 如果getTask结果为null则跳出循环，执行`processWorkerExit()`方法，销毁线程。

执行流程如下图所示：

<img src="E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor的worker线程执行任务.jpg" style="zoom:75%;" />

**worker线程回收：**

线程池需要管理线程的生命周期，需要在线程长时间不运行的时候进行回收。

线程池中线程的销毁依赖JVM自动的回收，线程池做的工作是根据当前线程池的状态维护一定数量的线程引用，防止这部分线程被JVM回收，当线程池决定哪些线程需要回收时，只需要将其引用消除即可。

线程池使用一张**Hash表**去持有线程的引用，这样可以通过添加引用、移除引用这样的操作来控制线程的生命周期。这个时候重要的就是如何判断线程是否在运行。

Worker被创建出来后，就会不断地进行轮询，然后获取任务去执行，

- **核心线程**可以无限等待获取任务
- **非核心线程**要限时获取任务

当Worker无法获取到任务，也就是获取的任务为空时，循环会结束，**Worker会主动消除自身在线程池内的引用**。

```java
try {
  while (task != null || (task = getTask()) != null) {
    //执行任务
  }
} finally {
  processWorkerExit(w, completedAbruptly);//获取不到任务时，主动回收自己
}
```

回收过程如下图所示：

<img src="E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor的worker线程回收.jpg" style="zoom:75%;" />

线程回收的工作是在`processWorkerExit`方法完成的。

![](E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor的worker线程回收2.jpg)

事实上，在这个方法中，将线程引用移出线程池就已经结束了线程销毁的部分。但由于引起线程销毁的可能性有很多，线程池还要判断是什么引发了这次销毁，是否要改变线程池的现阶段状态，是否要根据新状态，重新分配线程。

> 独占锁、AQS

**（如果为了面试，这部分可以不看）**

Worker是通过继承**AQS**，使用AQS来实现**独占锁**这个功能。没有使用可重入锁ReentrantLock，而是使用AQS，为的就是实现不可重入的特性去反应线程现在的执行状态。

1. lock方法一旦获取了独占锁，表示当前线程正在执行任务中。
2. 如果正在执行任务，则不应该中断线程。
3. 如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进行中断。
4. 线程池在执行shutdown方法或`tryTerminate`方法时会调用`interruptIdleWorkers`方法来中断空闲的线程，`interruptIdleWorkers`方法会使用`tryLock`方法来判断线程池中的线程是否是空闲状态；如果线程是空闲状态则可以安全回收。

在线程回收过程中就使用到了这种特性

### getTask

线程需要从任务缓存模块中不断地取任务执行，帮助线程从阻塞队列中获取任务，实现**线程管理模块和任务管理模块之间的通信**。

getTask这部分进行了多次判断，为的是控制线程的数量，使其符合线程池的状态。如果线程池现在不应该持有那么多线程，则会返回null值。工作线程Worker会不断接收新任务去执行，而当工作线程Worker接收不到任务的时候，就会开始被回收.

- 线程回收：将线程引用移出线程池

  将线程引用移出线程池就已经结束了线程销毁的部分。但由于引起线程销毁的可能性有很多，线程池还要判断是什么引发了这次销毁，是否要改变线程池的现阶段状态，是否要根据新状态，重新分配线程。

`getTask`方法执行流程如下图所示：

<img src="E:\_data\博文临时库\博文中的图片\线程池ThreadPoolExecutor任务获取示意图.jpg" style="zoom:50%;" />

## 管理任务

任务管理和阻塞队列以及拒绝策略相关。

🔴**execute方法会决定**接下来执行的流程：直接执行、缓冲执行或是拒绝执行。

- 从这个可选的流程中看到，任务如果执行则有两种可能：直接执行和缓冲执行

  - 直接执行：任务直接由新创建的线程执行
  - 缓冲执行：线程从任务队列中获取任务然后执行，执行完任务的空闲线程会再次去从队列中申请任务再去执行

  第一种情况仅出现在线程初始创建的时候，第二种是线程获取任务绝大多数的情况。

（关于阻塞队列的内容请移步并发容器，我认为这不属于线程池的讲解）

**任务拒绝**：

任务拒绝模块是线程池的**保护**部分。拒绝策略是一个接口，用户可以通过实现这个接口去定制拒绝策略，其声明如下：

``` java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

> 🔴四种`handler`（饱和策略）（拒绝策略）

1. `DiscardPolicy`：直接抛出异常
2. `DiscardPolicy`：丢掉这个任务，但是不抛出异常
3. `DiscardOldestPolicy`：丢掉最老的任务
4. `CallerRunsPolicy`：使用调用者的线程来处理
5. 当然也可以根据应用场景需要来实现`RejectedExecutionHandler`接口自定义策略。如记录日志或持久化不能处理的任务

# 参考

线程池的其他处理：例如常见线程池、线程池配置和监控，请查看另一篇文章

> 1. Java编程思想 第四版 中文版 第十二章并发
> 2. [聊聊并发（三）Java线程池的分析和使用](http://ifeve.com/java-threadpool/)
> 3. [Java线程池实现原理及其在美团业务中的实践](https://zhuanlan.zhihu.com/p/123328822?utm_source=qq&utm_medium=social&utm_oi=644097353620000768)

