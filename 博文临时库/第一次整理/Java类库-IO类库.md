# 综述

I/O 问题是任何编程语言都无法回避的问题，可以说 I/O 问题是整个人机交互的核心问题，因为 I/O 是机器获取和交换信息的主要渠道。I/O 问题尤其突出，很容易成为一个性能瓶颈。正因如此，所以 Java 在 I/O 上也一直在做持续的优化，如从 1.4 开始引入了 NIO，提升了 I/O 的性能。后续引入了AIO等。

> Java的IO

Java 语言中的BIO、NIO 和 AIO 是对操作系统的各种 IO 模型的封装。想要更深入的了解这些IO模型，了解操作系统IO模型的各种实现也尤为重要。

Java 的 I/O 操作类在包 java.io 下，大概有将近 80 个类，但是这些类大概可以分成四组，分别是：

1. （数据格式）基于字节操作的 I/O 接口：`InputStream` 和 `OutputStream`
2. （数据格式）基于字符操作的 I/O 接口：Writer 和 Reader
3. （传输方式）基于磁盘操作的 I/O 接口：File
4. （传输方式）基于网络操作的 I/O 接口：Socket

详细的操作系统的IO原理请查看介绍操作系统IO的文章。

# JAVA的IO模型

Java 语言中的BIO、NIO 和 AIO 是对操作系统的各种 IO 模型的封装。

java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。

## BIO

同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

在`while(true)` 循环中服务端会调用 `accept()` 方法等待接收客户端的连接的方式监听请求，让 BIO 通信模型能够同时处理多个客户端请求，就必须使用**多线程**（主要原因是`socket.accept()`、`socket.read()`、`socket.write()` 涉及的三个主要函数都是同步阻塞的），典型的**一请求一应答**通信模型。

- 可以通过**线程池机制**改善（即下面的所谓的伪异步IO模型图）

<img src="E:\_data\博文临时库\博文中的图片\BIO通信模型图.png" style="zoom: 67%;" />

<img src="E:\_data\博文临时库\博文中的图片\伪异步IO模型图.png" style="zoom:67%;" />

## NIO

NIO（Non-blocking I/O，在Java领域，也称为New I/O）是一种同步非阻塞的I/O模型，也是I/O多路复用的基础。

- 多线程和NIO：如果使用多线程和BIO，线程池中的线程仍然是堵塞的。

NIO由原来的阻塞读写（占用线程）变成了单线程轮询事件，原来使用BIO进行多个IO操作需要开多个线程，现在只需要一个线程就可以处理多个IO操作。

- 除了事件的轮询是阻塞的（没有可干的事情必须要阻塞），剩余的I/O操作都是纯CPU操作。

 NIO 框架，提供了 **Channel , Selector，Buffer**等抽象。它支持面向**缓冲**的，基于**通道**的I/O操作方法。

-  NIO提供了与传统BIO模型中的 Socket 和 `ServerSocket` 相对应的 `SocketChannel` 和 `ServerSocketChannel` 两种不同的套接字通道实现，两种通道都支持阻塞和非阻塞两种模式。

---

NIO带来了很多便利性：

- 事件驱动模型
- 避免多线程：单线程处理多任务
- 非阻塞I/O，I/O读写不再阻塞，而是返回0
- 基于block的传输，通常比基于流的传输更高效
- 更高级的IO函数，zero-copy
- IO多路复用大大提高了Java网络应用的可伸缩性和实用性

---

NIO与多线程：

- 单线程处理I/O的效率确实非常高，没有线程切换，只是拼命的读、写、选择事件。但现在的服务器，一般都是多核处理器，如果能够利用多核心进行I/O，无疑对效率会有更大的提高。
- 仔细分析一下我们需要的线程，其实主要包括以下几种： 
  1. 事件分发器，单线程选择就绪的事件。 
  2. I/O处理器，包括connect、read、write等，这种纯CPU操作，一般开启CPU核心个线程就可以。 
  3. 业务线程，在处理完I/O后，业务一般还会有自己的业务逻辑，有的还会有其他的阻塞I/O，如DB操作，RPC等。只要有阻塞，就需要单独的线程。
- 连接的处理和读写的处理通常可以选择分开，这样对于海量连接的注册和读写就可以分发。

## AIO

全称是 Asynchronous I/O。

在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

## 其他问题

> NIO的特性/NIO与BIO区别

- BIO
  - 一个线程管理一个通道；其中的流数据的读取是阻塞的；
  - 阻塞，面向流(Stream)，
  - 流是单向的，Stream只能进行单向操作，通过一个Stream只能进行读或者写
    - 比如`InputStream`只能进行读取操作，`OutputStream`只能进行写操作

- NIO
  - 一个线程管理多个通道；但是数据的处理将会变得复杂；
  - 不阻塞，面向缓冲区(Buffer)
  - 通道双向，一个Channel既可以进行读，也可以进行写；

> 三种IO的对比

以`socket.read()`为例子：

- 传统的BIO里面`socket.read()`，如果TCP `RecvBuffer`里没有数据，函数会一直阻塞，直到收到数据，返回读到的数据。

- 对于NIO，如果TCP `RecvBuffer`有数据，就把数据从网卡读到内存，并且返回给用户；反之则直接返回0，永远不会阻塞。

- 最新的AIO(`Async I/O`)里面会更进一步：不但等待就绪是非阻塞的，就连数据从网卡到内存的过程也是异步的。

换句话说，BIO里用户最关心“我要读”，NIO里用户最关心”我可以读了”，在AIO模型里用户更需要关注的是“读完了”。

NIO一个重要的特点是：socket主要的读、写、注册和接收函数，在等待就绪阶段（等待内核空间数据就绪）都是非阻塞的，真正的I/O操作（数据从内核空间拷贝到用户空间）是同步阻塞的（消耗CPU但性能非常高）。

> 如何选用

- 对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；
- 对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

如果需要维持不太多的打开的连接到其他计算机，并且这些连接会发送大量的数据，（如P2P网络），使用一个单独的线程来管理你所有出站连接，可能是一个优势。（NIO）

<img src="E:\_data\博文临时库\博文中的图片\NIO对比BIO一个线程一个连接.png" style="zoom:75%;" />

如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。（NIO）

<img src="E:\_data\博文临时库\博文中的图片\NIO管理多个连接的单个线程.png" style="zoom:75%;" />

---

注意：

- 使用NIO != 高性能，当连接数<1000，并发程度不高或者局域网环境下NIO并没有显著的性能优势。

  NIO并没有完全屏蔽平台差异，它仍然是基于各个操作系统的I/O系统实现的，差异仍然存在。使用NIO做网络编程构建事件驱动模型并不容易，陷阱重重。

- `Netty`的出现很大程度上改善了 JDK 原生 NIO 所存在的一些让人难以忍受的问题（API复杂、多线程难点、使用NIO可能写出很多bug）

  所以应当尽可能使用成熟的NIO框架，如`Netty`，`MINA`等。解决了很多NIO的陷阱，并屏蔽了操作系统的差异，有较好的性能和编程模型。

# NIO

几个核心的组件：Buffer(缓冲区)、Channel(通道)、Selector(选择器)

## Buffer

缓存的使用可以使用`DirectByteBuffer`和`HeapByteBuffer`：

- `DirectByteBuffer`：一般来说可以减少一次系统空间到用户空间的拷贝。但Buffer创建和销毁的成本更高，更不宜维护
- 如果数据量比较小的中小应用情况下，可以考虑使用`heapBuffer`；反之可以用`directBuffer`。
- `DirectByteBuffer` 指向堆外内存：直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 `DirectByteBuffer` 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆之间来回复制数据。

其他：

- `buf.flip();`切换读写状态
- 可以通过调用缓冲区的 `asReadOnlyBuffer()` 方法，将任何常规缓冲区转换为只读缓冲区，这个方法返回一个与原缓冲区完全相同的缓冲区(并与其共享数据)，只不过它是只读的。
- `slice()` 方法根据现有的缓冲区创建一种 *子缓冲区* 。也就是说，它创建一个新的缓冲区，新缓冲区与原来的缓冲区的一部分共享数据。

**缓冲区状态变量：**

- capacity：最大容量；
- position：当前已经读写的字节数；
- limit：还可以读写的字节数。

状态变量的改变过程举例：

1.  新建一个大小为 8 个字节的缓冲区，此时 position 为 0，而 limit = capacity = 8。capacity 变量不会改变，下面的讨论会忽略它。

   ![](E:\_data\博文临时库\博文中的图片\NIO缓冲区状态变量1.png)

2. 从输入通道中读取 5 个字节数据写入缓冲区中，此时 position 为 5，limit 保持不变。

   ![](E:\_data\博文临时库\博文中的图片\NIO缓冲区状态变量2.png)

3. 在将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，这个方法将 limit 设置为当前 position，并将 position 设置为 0。

   ![](E:\_data\博文临时库\博文中的图片\NIO缓冲区状态变量3.png)

4. 从缓冲区中取 4 个字节到输出缓冲中，此时 position 设为 4。

   ![](E:\_data\博文临时库\博文中的图片\NIO缓冲区状态变量4.png)

5. 最后需要调用 clear() 方法来清空缓冲区，此时 position 和 limit 都被设置为最初位置。

   ![](E:\_data\博文临时库\博文中的图片\NIO缓冲区状态变量5.png)

   

## Channel

![](E:\_data\博文临时库\博文中的图片\NIO通道和缓冲区.png)

NIO 通过Channel（通道） 进行读写。

- NIO的通道和BIO的流：通道是双向的，可读也可写，而流的读写是单向的。
- 无论读写，通道只能和Buffer交互，通道可以异步地读写。

## Selector

<img src="E:\_data\博文临时库\博文中的图片\NIO选择器.png" style="zoom:75%;" />

Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，则获取事件。然后针对每个事件进行相应的响应处理。

这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销

- `select()`方法：该方法将一直阻塞，直到有一个事件可用于已注册的频道之一。方法返回后，线程即可处理这些事件。

一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情（<u>*todo：？？*</u>）。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道

> **`Selector.wakeup()`**

- 解除阻塞在`Selector.select()`/`select(long)`上的线程，立即返回。
- 两次成功的select之间多次调用wakeup等价于一次调用。
- 如果当前没有阻塞在select上，则本次wakeup调用将作用于下一次select——“记忆”作用。

**为什么要唤醒？**

- 注册了新的channel或者事件
- channel关闭，取消注册
- 优先级更高的事件触发（如定时器事件），希望及时处理

**原理**

<u>*todo：原理我写的看不懂了？我复制的看不懂了？fun🙃*</u>

Linux上利用pipe调用创建一个管道，Windows上则是一个loopback的TCP连接。这是因为win32的管道无法加入select的`fd set`，将管道或者TCP连接加入`select fd set`。

wakeup往管道或者连接写入一个字节，阻塞的select因为有I/O事件就绪，立即返回。可见，wakeup的调用开销不可忽视。



---

## 其他

- **Pipe：**

  Java NIO Pipe是两个线程之间的单向数据连接。A `Pipe` 具有源通道和宿通道。您将数据写入接收器通道。然后可以从源通道读取此数据。

  <img src="E:\_data\博文临时库\博文中的图片\NIO管道.png" style="zoom:75%;" />

## 分散聚集 Scatter/Gather

Java NIO带有内置的分散/收集支持。分散/收集是在读取和写入通道中使用的概念。

- 对通道数据的分散是将数据写入到多个缓冲区中的操作。即通道将数据从通道“分散”到多个缓冲区中。
- 对通道的聚集写入是一种将来自多个缓冲区的数据读取到单个通道的操作。即通道将来自多个缓冲区的数据“收集”到一个通道中。

<img src="E:\_data\博文临时库\博文中的图片\NIO分散.png" alt="NIO分散" style="zoom:75%;" /><img src="E:\_data\博文临时库\博文中的图片\NIO聚集.png" style="zoom:75%;" />



在需要分别处理传输数据的各个部分的情况下，散布/收集可能非常有用。例如，如果消息由标头和正文组成，则可以将标头和正文保留在单独的缓冲区中。这样做可以使您更轻松地分别使用标题和正文。

``` java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

- 注意如何将缓冲区首先插入到数组中，然后将数组作为参数传递给 `channel.read()`方法。`read()`然后，该方法按照缓冲区在数组中出现的顺序从通道写入数据。一旦缓冲区已满，通道将继续填充下一个缓冲区。

分散读取会先填充一个缓冲区，然后再移动到另一个缓冲区，这一事实表明它不适合动态调整大小。换句话说，如果您有标头和正文，并且标头是固定大小（例如128字节），则分散读取效果很好。

``` java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

- 缓冲区数组被传递到`write()`方法中，该方法按照缓冲区在数组中遇到的顺序写入缓冲区的内容。仅写入缓冲区的位置和极限之间的数据。因此，如果缓冲区的容量为128个字节，但仅包含58个字节，则只有58个字节从该缓冲区写入通道。因此，与分散读取相比，聚集写入对于动态大小的消息部分可以很好地工作。



# API

装饰着模式

# 参考

> 1. [JavaGuide-BIO-NIO-AIO](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/BIO-NIO-AIO.md)
> 2. [Java NIO浅析 - 美团技术团队](https://tech.meituan.com/2016/11/04/nio.html)
> 3. [Java NIO 底层原理](https://www.cnblogs.com/crazymakercircle/p/10225159.html)
> 4. [StackOverflow同步异步的一个问题](https://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean)
> 5. [Java NIO Tutorial](http://tutorials.jenkov.com/java-nio/index.html)

