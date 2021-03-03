---

---

Java使用**线程**来执行**任务**。任务即我们要并发实现的事情，可以描述为线程执行体。线程只是任务的载体，只是任务的执行单元。明白这一点可以帮助我们更好的进行线程资源的复用和**理解线程池**。

任务可以用Runnable、Callable来描述，任务也体现在Thread中的Run方法上。

- 任务和驱动他的线程是不一样的，体现在java上是你对Thread类实际上没有任何控制权，java的线程机制来源于c的低级的p线程方法，在物理上，创建线程可能会代价高昂，因此必须保存并管理他们，这样，从实现的角度看，将任务从线程中分离出来是很有意义的，因此这样不同的runnable就可以在不同的线程实例中执行。

# 任务

Runnable、Callable的主要区别在于，Callable有**返回值**；此外Runnable任务体为run方法，Callable任务体为call方法。

Runnable没有什么好说的（如果有后面再补充），我们来看一下Callable。

1. callable可以在完成时返回一个值，也就是call方法返回**返回值**，并且可以声明抛出异常。
2. 可以接收一个泛型类型，用来指定期望的返回类型，这些与Future息息相关
   - 类型一致：Callable接口有泛型限制，Callable接口里的泛型形参类型与call()方法返回值类型相同，也要和Future类型相一致
3. 必须使用`ExecutorService.submit()`来调用callable，`submit()`方法将产生Future对象

> **Future接口**

Future接口代表Callable接口里call()方法的返回值，JAVA为Future接口提供了一个**FutureTask**实现类，该实现类实现了Future接口和Runnable接口（FutureTask实现了RunnableFuture，RunnableFuture继承自Future和Runnable，没错是多继承，但是是接口多继承不是类的多继承），可以作为Thread类的执行体，即FutureTask可以传入Thread。

（由于多线程的复杂性，不得不陈述各种各样的情形，可以先知道有这几个方法，待了解大致图景，再回来看这些方法：`isDone`、`get`、`cancel`、`isCanceled`）

下面陈述接口中的方法及其使用注意事项：

1. 用 `isDone()`方法来查询 Future是否已经完成。
2. 调用`get()`方法来获取该结果
   1. 如果 `isDone()`判断完成，调用`get()`方法获取到该结果
   2. 如果 `isDone()`判断没有完成，`get()`将阻塞，直至结果准备就绪
   3. **策略：**在试图调用`get()`来获取结果之前，先调用具有超时的`get(long timeout, TimeUnit unit)`，或者调用 `isDone()`来查看任务是否完成
3. `cancel(boolean maylnterruptltRunning)`：试图取消该Future里关联的Callable任务
   1. 调用时刻的影响
      1. 如果在任务**启动之前**调用，该任务永远不会执行
      2. 如果任务**已经开始**，`mayInterruptIfRunning`将确定是否应中断执行该任务的线程以尝试停止该任务。
      3. 如果任务**已经完成**，尝试将失败，返回false
   2. 因为其他原因也会无法取消任务，返回false
   3. 方法返回后对其他方法影响
      1. 此方法返回后，对`isDone()`的调用始终返回true
      2. 如果此方法返回true，随后调用的`isCancelled()`将始终返回true
4. `isCancelled()`：如果在Callable任务正常完成前被取消，则返回true
5. future可以捕获到异常，也就是callable内部执行错误抛出异常后，调用future.get的线程可以捕获到异常

# 线程

实线程获取方式可以分为 **常规构建** 和 **线程池创建并获取**，本节着重介绍常规创建方法和技巧，线程池创建请见之后的小节。

> **常见的创建方式**

1. new Thread()并传入Runnable

2. 继承Thread重写run方法

3. 自管理的Runnable，即在Runnable内部持有Thread变量

   这个写法和直接继承thread没有什么差异，但是**实现接口可以是我们继承另一个不同的类**，而从Thread继承将不行

4. 使用内部类将线程代码隐藏在类中

   这样的做法其实可能会造成内存泄漏，因为我们在方法结束后丢失了对这个线程的管理。

**关键的是我们传入Runnable这样的任务**。

> 一些注意事项

``` java
Thread t=new Thread(new LiftOFF());
t.start();
```

- `start()`方法会迅速的返回，并不会堵塞调用。

- 异常不能跨线程传播，必须在本地处理所有在任务内部产生的异常

- 另外也可以传入：FutureTask（实现了Runnable）

- start在构造器中调用：

  start都是在构造器中调用的，在构造器启动线程可能会变得很有问题，因为另一个任务可能会在构造器结束之前开始执行，这意味着该任务能够访问处于不稳定状态的对象，这是应该优选Executor而不是显示创建thread对象的另一个原因。应当警惕在构造器内部就执行任务体的情况。

  （<u>*todo：这句话应当放在一个别的地方感觉，专门对构造器的那个地方？*</u>）

## 生命周期

线程的状态切换（采用五状态线程模型来讨论）：

1. **新建状态：**当程序使用**new**关键字创建了一个线程之后，该线程就处于新建状态，此时仅由JVM为其**分配内存**，并**初始化**其成员变量的值
2. **就绪状态：**当线程对象调用了**start()**方法之后，该线程处于就绪状态。Java虚拟机会为其创建**方法调用栈和程序计数器**，等待调度运行
3. **运行状态：如果**处于就绪状态的线程**获得了CPU**，开始执行**run()**方法的线程执行体，则该线程处于运行状态
4. **阻塞状态：**当处于运行状态的线程失去所占用资源之后，便进入阻塞状态；或等待事件（如IO、sleep）

![](E:\_data\博文临时库\博文中的图片\drawio\Java并发类库-线程-生命周期.png)

- 有很多人采用一些别的状态来指定描绘Java的线程状态：[CS-Note](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%B9%B6%E5%8F%91.md#%E5%85%AD%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81)

从上图可以看到：

- 就绪和运行状态之间的转换通常**不受程序控制**，而是由**系统线程调度**决定。

  - **`Thread.yield()`**：表明该线程已经完成生命周期中最重要的一部分，此刻正是切换给其他任务执行一段时间的大好时机。这完全是选择性的，**可能发生也可能不发生**

    yield是在建议调度器可以让别的线程使用CPU，但并没有任何机制保证他会被采纳，yield也是在建议具有相同优先级的其他线程可以运行，对于任何重要的控制或在调整应用是，都**不能依赖于yield**

- **`join`**：

  如果某个a线程在另一个线程b上调用`t.join()`，此调用线程a将被挂起，直到目标线程b结束才恢复。也可以在`b.join()`调用时加上一个超时参数，这样如果目标线程在这段时间到期时还没有结束的话join方法总能返回。

  ```java
      Main thread-->----->--->-->--block##########continue--->---->
                   \                 |               |
  sub thread start()\                | join()        |
                     \               |               |
                      ---sub thread----->--->--->--finish    
  ```

  - <u>*todo：对join方法的调用可以被中断，做法就是在调用线程上调用interrupt方法*</u>

**run方法和start方法**：

- 启动线程**使用start()**方法，而不是run()方法。

  永远不要调用线程对象的run()方法。而是调用start()来启动线程，调用run方法相当于调用了普通对象的方法，而不是线程对象的方法

- 调用run方法，会使被调用线程运行，而**调用run的线程堵塞**

- **start只能调用一次**：不要再次调用线程对象的start()方法。只能对处于新建状态的线程调用start()方法，否则将引发IllegaIThreadStateException异常

**`isAlive()`**：

- 为了**测试**某个程是否已经**死亡**
- 当线程处于**就绪、运行、阻塞**三种状态时，该方法将返回**true**；当线程处于**新建、死亡**状态时，该方法将返回**false**

**`sleep`**：

- `Thread.Sleep(0)`的作用，就是“触发操作系统立刻重新进行一次CPU竞争，重新计算优先级”

## 常见处理

**优先级**：

- 用`getPriority()`和`setPriority()`来读取和设置线程优先级
- 优先级是**在`run()`的开头部分**设定的，在构造器内设置没有任何好处因为还没有开始执行任务
- 只使用MAX_PRIORITY、NORM_PRIORITY和MIN_PRIORITY三种级别
  - 这是为了让线程优先级具有可移植性

**后台线程**：

- main是一个非后台线程

- 必须在**线程启动之前**调用`setDaemon(boolean on)`方法才能把它设置成为后台进程。示例如下：

  ``` java
  Thread daemon = new Thread(new Runnable() {...});
  daemon.setDaemon(true);
  daemon.start();
  ```

- 后台线程创建的任何线程都自动设置为后台线程

- 后台线程在所有其他非后台线程结束后就会终止，**可能不会执行finally**语句

  这种行为是正确的，即便你基于前面对finally给出的承诺，并不希望出现这种行为，但情况仍将如此。当最后一个非后台线程终止时，后台线程会“突然”终止。因此一旦 main退出， JVM就会立即关闭所有的后台进程，而不会有任何你希望出现的确认形式。因为你不能以优雅的方式来关闭后台线程，所以它们几乎不是一种好的思想。非后台的 Executor通常是一种更好的方式，因为 Executor控制的所有任务可以同时被关闭。正如你将要在稍后看到的，在这种情况下，关闭将以有序的方式执行

## 在线程中捕获异常

由于线程的本质特性，使得我们**不能捕获从线程中逃逸的异常**。但是可以通过`Thread.UncaughtExceptionHandler`来进行异常处理，它允许在每个thread对象上附着一个**异常处理器**。

系统会检查线程专有版本，如果没有发现，则检查线程组是否有其专有的 `uncaughtException()`方法，如果也没有，再调用 `defaultUncaughtExceptionHandler`。示例如：

**策略：** 如果你知道将要在代码中处处使用相同的异常处理器，那么更简单的方式是在 Thread类中设置一个静态域，并将这个处理器设置为默认的未捕获异常处理器。这个处理器只有在不存在线程专有的未捕获异常处理器的情况下才会被调用。

``` java
public class SettingDefaultHandler {
  public static void main(String[] args) {
    Thread.setDefaultUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler()); // 为默认的异常处理器
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new ExceptionThread());
  }
}
```

# 中断线程

在某些情况下，任务必须突然的终止。即有时你希望能够终止处于阻塞状态的任务。

- 如果对于处于阻塞状态的任务，你不能等待其到达代码中可以检查其状态值的某一点，因而决定让它主动地终止，那么你就必须强制这个任务跳出阻塞状态。

Thread类包含 interrupt()方法，可以使用它终止被阻塞的任务。 interrupt()方法将设置线程的**中断状态**。

下面介绍中断方法、中断时机、中断标志位等内容

## 中断方法、时机、标志位

**中断方法：**

- 线程中断另一个线程：

  当另一个a线程在该b线程上调用  interrupt()方法时，a线程将给该b线程设定一个标志，表明该b线程已经被中断。

- Executor中断所有线程：

  在`ExecutorService`上调用` shutdownNow()`，那么它将发送一个 `interrupt()`调用给它启动的所有线程

  - 这么做是有意义的，因为当你完成工程中的某个部分或者整个程序时，通常会希望同时关闭某个特定 Executor的所有任务

- **`Future.cancel`**：

  持有Future的关键在于你可以在其上调用 cancel()，并因此可以使用它来中断某个特定任务，将true传递给 cancel()，那么它就会拥有在该线程上调用 interrupt以停止这个线程的权限

  ``` java
  ExecutorService exec = Executors.newCachedThreadPool();
  Future<?> f = exec.submit(new Runnable(){...});
  f.cancel(true); // Interrupts if running
  ```

  

**调用时刻：**如果一个线程已经被阻塞，或者试图执行一个阻塞操作，这时设置这个线程的中断状态将抛出 `InterruptedException`。

- 中断发生的唯一时刻是在任务要进入到阻塞操作中，或者已经在阻塞操作内部时

**中断状态/标志位：**

- **中断状态被复位**：
  - `InterruptedException`异常被捕获时（即抛出该异常时）将清理interrupt标志，中断状态将被复位。
    - 所以在 catch子句中，在异常被捕获的时候interrupt标志总是为假
  - 当任务调用 `Thread.isInterrupted()`时

## 不能中断的调用

存在不能中断的调用的事实有点令人烦恼，特别是在创建执行IO的任务时。因为这意味着IO具有锁住你的多线程程序的潜在可能。特别是对于基于Web的程序，这更是关乎利害。

`interrupt()`能够中断的调用：

- 对 sleep()的调用
- 任何要求抛出 `InterruptedException`的调用
  - 例如在`ReentrantLock`上堵塞的任务
- 打断被互斥所堵塞的调用

`interrupt()`不能中断的调用：

- 正在试图获取 `synchronized` 锁的任务
- 试图执行IO操作的任务

对于这类不能中断的问题，有一个略显笨拙但是有时确实行之有效的解决方案，即**关闭任务在其上发生阻塞的底层资源**：

``` java
ExecutorService exec = Executors.newCachedThreadPool();
ServerSocket server = new ServerSocket(8080);
exec.execute(new IOBlocked(socketInput));
exec.shutdownNow(); //this will not shutdown
socketInput.close(); // Releases blocked thread and then the thread will shutdown
```

## 中断检查

检查中断，保证总是可以离开任务体或无限循环

如果你只能通过在阻塞调用上抛出异常来退出，那么你就无法总是可以离开run()循环。你可以采取如下两种方法来保证退出任务。

- 可以通过调用 `Thread.isInterrupted()`来检查中断状态，这不仅可以告诉你`interrupt()`是否被调用过，而且还可以清除中断状态。

- 可以经由单一的`InterruptedException`或单一的成功的 `Thread.interrupted()`测试来得到这种通知。

虽然中断状态在上面两种方法下会被清除，但正如上文所说的：如果想要再次检查以了解是否被中断，则可以在调用  `Thread.isInterrupted()`时将**结果存储起来**。

学习下面这个代码：

（**策略：**通过学习之后的这个代码示例来保证可以退出任务体或无限循环）

在下面这个例子中，你可以在不同地点退出 Blocked3.run()：

- 在阻塞的 sleep()调用中，
- 或者在非阻塞的数学计算中。

你将看到，如果 interrupt（在注释 point2之后（即在非阻塞的操作过程中）被调用，那么首先循环将结束，然后所有的本地对象将被销毁，最后循环会经由 while语句的顶部退出。

但是，如果 interrupt()在 point1和 point2之间（在 while语句之后，但是在阻塞操作 sleep()之前或其过程中）被调用，那么这个任务就会在第一次试图调用阻塞操作之前，经由`InterruptedException`退出。在这种情况下，在异常被抛出之时唯一被创建出来的 `NeedsCleanup`对象将被清除，而你也就有了在catch子句中执行其他任何清除工作的机会。

**注意：**被设计用来响应 interrupt的类必须建立一种策略，来确保它将保持一致的状态。这通常意味着所有需要清理的对象创建操作的后面，都必须紧跟 try-finally子句，从而使得无论run()循环如何退出，清理都会发生。

``` java
class NeedsCleanup {
  private final int id;
  public NeedsCleanup(int ident) {
    id = ident;
    print("NeedsCleanup " + id);
  }
  public void cleanup() {
    print("Cleaning up " + id);
  }
}

class Blocked3 implements Runnable {
  private volatile double d = 0.0;
  public void run() {
    try {
      while(!Thread.interrupted()) {
        // point1
        NeedsCleanup n1 = new NeedsCleanup(1);
        // Start try-finally immediately after definition
        // of n1, to guarantee proper cleanup of n1:
        try {
          print("Sleeping");
          TimeUnit.SECONDS.sleep(1);
          // point2
          NeedsCleanup n2 = new NeedsCleanup(2);
          // Guarantee proper cleanup of n2:
          try {
            print("Calculating");
            // A time-consuming, non-blocking operation:
            for(int i = 1; i < 2500000; i++)
              d = d + (Math.PI + Math.E) / d;
            print("Finished time-consuming operation");
          } finally {
            n2.cleanup();
          }
        } finally {
          n1.cleanup();
        }
      }
      print("Exiting via while() test");
    } catch(InterruptedException e) {
      print("Exiting via InterruptedException");
    }
  }
}

public class InterruptingIdiom {
  public static void main(String[] args) throws Exception {
    if(args.length != 1) {
      print("usage: java InterruptingIdiom delay-in-mS");
      System.exit(1);
    }
    Thread t = new Thread(new Blocked3());
    t.start();
    TimeUnit.MILLISECONDS.sleep(new Integer(args[0]));
    t.interrupt();
  }
} /* Output: (Sample)
NeedsCleanup 1
Sleeping
NeedsCleanup 2
Calculating
Finished time-consuming operation
Cleaning up 2
Cleaning up 1
NeedsCleanup 1
Sleeping
Cleaning up 1
Exiting via InterruptedException
*///:~
```

# 参考

> 1. Java编程思想 第四版 中文版 第21章 并发
> 2. [Java多线程学习(三)---线程的生命周期](https://www.cnblogs.com/sunddenly/p/4106562.html)
> 3. [StackOverflow : What does this thread join code mean?](https://stackoverflow.com/questions/15956231/what-does-this-thread-join-code-mean)