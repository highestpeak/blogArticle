---

---

<u>*todo： join也算，sleep也算，锁的释放上，是否共享上*</u>

# ThreadLocal

**ThreadLocal是为了能够在当前线程中有属于自己的变量，不解决且不能解决同步问题、共享资源问题**。由于不需要共享信息，自然就不存在竞争问题了，从而保证了某些情况下线程的安全，以及避免了某些情况需要考虑线程安全必须同步带来的性能损失。

本地存储跟除了对变量的共享，ThreadLocal保证不会出现竞争条件，它相当于提供了线程的局部变量，每个线程都可以通过set()和get()来对这个局部变量进行操作，实现了线程的数据隔离。

- 一个线程使用ThreadLocal创建的变量只能被它自己访问，其他线程无法访问和修改
- 同一个 ThreadLocal 所包含的对象，在不同的 Thread 中有不同的副本（不同的实例）。例如：对`ThreadLocal<String>`而言即为不同的 Thread 持有不同的String 类型变量。（尽管他们可能名字一样？）
- ThreadLocal 变量通常被`private static`修饰（策略）。当一个线程结束时，它所使用的所有 ThreadLocal 相对的实例副本都可被回收。
  - 同一个类同一个线程不同该类实例共享相同的实例
  - 所有此类的对象（只要是这个线程内定义的）都可以操纵这个ThreadLocal变量
- ThreadLocal位于堆上，而不是栈上
- 其实ThreadLocal使用起来感觉和普通对象差不多，就是加上了上述限制。例如`ThreadLocal<String>`我们就可以视其为一个String来使用

## 实现

很自然的我们可以想到使用键值对，也就是map来实现。但是即使这样也有两种实现方式，即：

- map的键是thread、值为对应的对象。map是ThreadLocal维护的
- 🔵map的键是对象类型、值为对应的对象。map是线程维护的
  - 这是Java所采用的方案
  - 键实际上是ThreadLocal类型，例如`ThreadLocal<String>`

![](E:\_data\博文临时库\博文中的图片\drawio\Java并发类库-线程协作-ThreadLocal.png)

- 这里只给出了Java采用的方案的图，另外的图可以查看：http://www.jasongj.com/java/threadlocal/
- 从图中可以看到，一个ThreadLocal只能存储一个Object对象，如果需要存储多个Object对象那么就需要多个ThreadLocal，这也是上面我们说的：ThreadLocal使用起来感觉和普通对象差不多，例如`ThreadLocal<String>`我们就可以视其为一个String来使用
- 这个map结构的几个问题：
  - 于每个线程访问某 ThreadLocal 变量后，都会在自己的 Map 内维护该 ThreadLocal 变量与具体实例的映射，如果不删除这些引用（映射），则这些 ThreadLocal 不能被回收，可能会造成内存泄漏

具体实现：

- 每个Thread维护着一个`ThreadLocalMap`的引用，`ThreadLocalMap`是ThreadLocal的内部类，用Entry来进行存储
- set()方法：实际上就是往`ThreadLocalMap`设置值，key是ThreadLocal对象，值是传递进来的对象
- get()方法：实际上就是往`ThreadLocalMap`获取值，key是ThreadLocal对象
- ThreadLocal本身并不存储值，它只是作为一个key来让线程从`ThreadLocalMap`获取value。

``` java
public class ThreadLocal<T> {
    ...;
    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;
		   ...;
        }
    }
    ...;
}
```

`ThreadLocalMap`的Entry继承自`WeakReference`弱引用，弱引用的特点是：不管内存是否充足，如果只被`WeakReference`关联，那么就会被回收。在我们这里就是：如果只被`ThreadLocalMap`的Entry的key指向，那么就会被回收。

在使用ThreadLocal的时候，最好在线程最后的finally块调用ThreadLocal的remove，remove函数会在内部把Entry中的值Object设置为null，进而让JVM可以进行回收。

``` java
new Thread(()->{
   try{
       ...;
   }finally{
       threadLocal.remove();
   }
});
```

## 适用场景

只要不涉及线程之间合作的，就可以用ThreadLocal来保证线程安全：

- **每个线程需要有自己单独的实例**
  - 也可以在线程内部构建一个单独的实例，但ThreadLocal 以非常方便的形式满足该需求。
- 实例需要在多个方法中共享，但不希望被多线程共享
  - 也可以通过方法间引用传递的形式实现，但ThreadLocal 使得代码耦合度更低，且实现更优雅。

:star::star::star::star:实际应用中**举例**如：

-  **Java Web 应用中Session**，每个线程有自己单独的 Session 实例，很多地方都需要操作 Session
   -  实现单个线程单例以及单个线程上下文信息存储，比如交易id等
-  **Android中Looper类**就是利用了ThreadLocal的特性
-  承载一些线程相关的数据，类似线程的全局变量，避免在方法中来回传递参数
-  管理**数据库的Connection**，这时**每个线程相当于一个事务**
   - ThreadLocal能够实现**当前线程的操作都是用同一个Connection，保证了事务**
   - Hibernate对Connection的管理也是采用了相同的手法

---

**其他问题：**

- ThreadLocal造成内存泄漏的原因？

  `ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露。`ThreadLocalMap` 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法

  ThreadLocal内存泄漏解决方案？

  - 每次使用完ThreadLocal，都调用它的remove()方法，清除数据。
  - 在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理。

# wait & notify

## wait

wait会在等待外部世界产生变化的时候将任务挂起，并且只有在notify或notifyAll发生时，这个任务才会被唤醒并去检查所产生的变化。因此**wait不是忙等待**。对wait的调用的时候，线程将被挂起，对象上的**锁将被释放**，这就意味着另一个任务可以获得这个锁。

🔴**只能在同步控制方法或同步控制块里调用wait、 notify和 notifyAll，也就是我们必须拥有目标对象的锁**

- 如果在非同步控制方法里调用这些方法，程序能通过编译，但运行的时候，将得到`IllegalMonitorStateException`异常，并伴随着一些含糊的消息，比如“当前线程不是拥有者”。（消息的意思是，调用 wait、notify和notifyAll的任务在调用这些方法前必须“拥有”（获取）对象的锁）

- 如果要向对象x发送 notifyAll，那么就必须在能够取得x的锁的同步控制块中这么做：

  ``` java
  synchronized(x){
      x.notifyAll();
  }
  ```


有两种形式的wait：接受亳秒数作为参数；不接受任何参数（也是更常用形式）。

- 对于接受亳秒数作为参数的wait而言，wait与sleep的区别如下：
  1. 在wait期间对象锁是释放的。
  2. 可以通过notify、 notifyAll，或者令时间到期，从wait中恢复执行。

wait、 notify以及 notifyAll有一个比较特殊的方面，那就是这些方法是基类Object的部分，而不是属于Thread的一部分。这是有道理的，因为这些方法操作的锁也是所有对象的一部分。

## notify

在实际场景下，可能有多个任务在单个对象上处于wait状态，因此调用notifyAll比只调用 notify要更安全。

- 使用 notify而不是notifyAll是一种优化。使用 notify时，在众多等待同一个锁的任务中只有一个会被唤醒，如果你希望使用 notify，就必须保证被唤醒的是恰当的任务。
- （<u>*todo：尚未搞懂这里？？？*</u>）另外，为了使用notify，所有任务必须等待相同的条件，因为如果你有多个任务在等待不同的条件，那么你就不会知道是否唤醒了恰当的任务。如果使用 notify，当条件发生变化时，必须只有一个任务能够从中受益。
- （<u>*todo：尚未搞懂这里？？？*</u>）这些限制对所有可能存在的子类都必须总是起作用的。如果这些规则中有任何一条不满足，那么你就必须使用notifyAll而不是 notify

**当 notifyAll因某个特定锁而被调用时，只有等待这个锁的任务才会被唤醒**，也就是说并不是在程序中任何地方，任何处于wait状态中的任务都将被任何对notifyAll的调用唤醒。

## 一个例子

（这个例子可以作为编程的参考，如果你是为了面试而来，可以不看这部分）

``` java
class Car {
  private boolean waxOn = false;
  public synchronized void waxed() {
    waxOn = true; // Ready to buff
    notifyAll();
  }
  public synchronized void buffed() {
    waxOn = false; // Ready for another coat of wax
    notifyAll();
  }
  public synchronized void waitForWaxing()
  throws InterruptedException {
    while(waxOn == false)
      wait();
  }
  public synchronized void waitForBuffing()
  throws InterruptedException {
    while(waxOn == true)
      wait();
  }
}

class WaxOn implements Runnable {
  private Car car;
  public WaxOn(Car c) { car = c; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        printnb("Wax On! ");
        TimeUnit.MILLISECONDS.sleep(200);
        car.waxed();
        car.waitForBuffing();
      }
    } catch(InterruptedException e) {
      print("Exiting via interrupt");
    }
    print("Ending Wax On task");
  }
}

class WaxOff implements Runnable {
  private Car car;
  public WaxOff(Car c) { car = c; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        car.waitForWaxing();
        printnb("Wax Off! ");
        TimeUnit.MILLISECONDS.sleep(200);
        car.buffed();
      }
    } catch(InterruptedException e) {
      print("Exiting via interrupt");
    }
    print("Ending Wax Off task");
  }
}

public class WaxOMatic {
  public static void main(String[] args) throws Exception {
    Car car = new Car();
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new WaxOff(car));
    exec.execute(new WaxOn(car));
    TimeUnit.SECONDS.sleep(5); // Run for a while...
    exec.shutdownNow(); // Interrupt all tasks
  }
} /* Output: (95% match)
Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Wax Off! Wax On! Exiting via interrupt
Ending Wax On task
Exiting via interrupt
Ending Wax Off task
*///:~
```

这里，Car有一个单一的布尔属性 `waxOn`，表示涂蜡抛光处理的状态在 `waitForWaxing`中将检查 `waxOn`标志，如果它为 false，那么这个调用任务将通过调用wait而被挂起。这个行为**发生在 synchronized方法中**这一点很重要，因为在这样的方法中，任务已经获得了锁。

当你调用 wait时，线程被挂起，而锁被释放。锁被释放这一点是本质所在，因为为了安全地改变对象的状态（例如，将 `waxOn`改变为true，如果被挂起的任务要继续执行，就必须执行该动作），其他某个任务就必须能够获得这个锁。在本例中，如果另一个任务调用waxed来表示“是时候该干点什么了”，那么就必须获得这个锁，从而将 `waxOn`改变为true。

之后， waxed()调用 notifyAll，这将唤醒在对wait的调用中被挂起的任务。为了使该任务从wait中唤醒，它必须首先重新获得当它进入wait时释放的锁。在这个锁变得可用之前，这个任务是不会被唤醒的°。

> 🔴**策略：前面的示例强调你必须用一个检查感兴趣的条件的 while循环包围wait.这很重要**，因为：

- 你可能有多个任务出于相同的原因在等待同一个锁，而第一个唤醒任务可能会改变这种状况（即使你没有这么做，有人也会通过继承你的类去这么做）。如果属于这种情况，那么这个任务应该被再次挂起，直至其感兴趣的条件发生变化。
- 在这个任务从其 wait中被唤醒的时刻，有可能会有某个其他的任务已经做出了改变，从而使得这个任务在此时不能执行，或者执行其操作已显得无关紧要。此时，应该通过再次调用wait来将其重新挂起。
- 也有可能某些任务出于不同的原因在等待你的对象上的锁（在这种情况下必须使用notifyAll）。在这种情况下，你需要检查是否已经由正确的原因唤醒，如果不是，就再次调用 wait
- 这种while循环是一种防止错失信号可能性的程序设计

其本质就是要检査所感兴趣的特定条件，并在条件不满足的情况下返回到 wait中。惯用的方法就是使用while来编写这种代码

# await & signal

这与Java并发类库的Condition对象有关。通过在condition上调用`await`来挂起一个任务，调用`signal`来唤醒这个任务，或者调用`signalAll`来唤醒所有在这个condition上被自身挂起的任务。与`notifyAll`相比，`signalAll`是一个更为安全的方式。

# 同步队列

这可以说是生产者消费者问题的解决方案，当然很多问题都是它的变体，所以这样的队列用处很大。

同步队列：`BlockingQueue`、`LinkedBlockingQueue`、`ArrayBlockingQueue`，

如果消费者任务试图从队列获取对象，而该队列此时为空，那么这些队列还可以**挂起消费者任务**，并且当有更多的元素可用时恢复消费者任务。

使用`BlockingQueue`会产生简化：它使显式的wait和notify时存在的类和类之间的耦合被削除了，因为每个类都只和他的`BlockingQueue`通信

这部分请查看并发容器部分。

# 协作的问题

## 错失的信号

错失的信号会造成潜在死锁，这样的情况一般是程序编写逻辑不严谨造成的。

``` java
T1:
synchronized(sharedMonitor){
    <setup condition for T2>
    sharedMonitor.notify();
}
T2:
while(someCondition){
    //point 1
    synchronized(sharedMonitor){
        sharedMonitor.wait();
    }
}
```

**错失信号(死锁情况)**：假设T2对`someCondition`求值发现为true，在point 1线程调度器切换到T1，T1执行设置（即\<setup condition for T2\>），然后掉用notify，然后T2继续执行。此时对于T2来说，时机已经太晚了，以至于不能意识到这个条件已经发生了变化，因此会盲目进入 wait，此时 notify将错失，而T2也将无限地等待这个已经发送过的信号，从而产生死锁。

**解决方案：**(防止在`someCondition`变量上产生竞争条件)

``` java
synchronized(sharedMonitor){
    while(someCondition){
        sharedMonitor.wait();            
    }
}
```

现在：如果T1首先执行，当控制返回T2时，它将发现条件发生了变化，从而不会进入 wait。反过来，如果T2首先执行，那它将进入wait，并且稍后会由T1唤醒。因此，信号不会错失。

## 死锁

- `jps`：确认执行任务的任务号

介绍两种死锁检测工具：

- `jstack`：查看之前`jps`查找的任务号的线程的堆栈信息，例如 `jstack -F 1362`。

  `Jstack`工具可以用于生成java虚拟机当前时刻的线程快照，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源

- `Jconsole`：可以自动检测死锁

从操作系统中学习到的死锁的其他解决办法：

- 死锁预防：以确定的资源顺序获得锁、超时放弃

死锁条件里面的竞争资源，可以是线程池里的线程、网络连接池的连接，数据库中数据引擎提供的锁，等等一切可以被称作竞争资源的东西。

# 参考

> 1. [Java并发编程：深入剖析ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html)