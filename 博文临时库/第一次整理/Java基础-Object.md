---
title: JAVA-Object
description: 
tags:
- JAVA
- Object
create_time: 2020-3-15 19:26:00
---

Object类是所有类的基类，了解他很是重要。分类来看Object类的方法，其中主要的有两组重要的方法：第一和第二组，分别和类相等比较、并发问题息息相关。剩下的方法分散在各个用途。

1. hashCode()和equals(Object obj)
2. notify()、notifyAll()、wait(long timeout)、wait(long timeout, int nanos)、wait()
3. toString()
4. getClass() （本文不做阐述）
5. clone()
6. finalize()

# hashCode & equals & ==

equals是一个很重要的方法，它不仅仅是可以让我们在if等条件判断可以用上。在很多方面，他也是一种逻辑上的需要，我们需要在很多地方与equals的结果保持意义上的一致性。例如我们即将说的hashCode和equals要一致，此外还有equals会影响容器类库的表现，compareTo方法希望产生与equals一致的自然排序等（相等0值和不相等非0值）。

## equals & ==

== 和 equals 的区别

- ==比较对象地址，equals默认使用==比较
- 可以重写equals（是重写不是重载）

特殊需要注意的/类库中需要注意的：

- String中的equals方法是被重写过的
- 传入equals的参数为null，则返回的是false

关于equals和==有一个有趣的问题：[StackOverflow : Could someone explain me the .getClass() method in java](https://stackoverflow.com/questions/36233435/could-someone-explain-me-the-getclass-method-in-java)

**类相等：**

- 两个类相等，需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间。
- 这里的相等，包括类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、isInstance() 方法的返回结果为 true，也包括使用 instanceof 关键字做对象所属关系判定结果为 true。

## hashCode & equals 

首先知道两点：

- hashCode方法默认由native方法底层实现
- 同一个对象(如果该对象没有被修改)：那么重复调用hashCode()或者equals返回的值是相同的）

🔴 🔴 🔴 两者联系，注意几点：

- 🔴 hashcode相同，equals不一定相同
  - 这要从哈希算法的性质入手
- 🔴equals相同，hashcode一定相同
  - 这要从equals的意义入手
- 重写equals时必须重写hashCode方法
  - 这是由于上面的第一点导致的做法
- 对象相等，则equals必定相等。反之也成立
  - 这要从equals的意义入手

由于计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。即由于**hash算法**的原因可能会产生相同的hash值。所以导致了第一点的现象。

此外，由于我们需要程序的执行符合我们的想法，符合实际情况。例如：我们希望如果两个对象"相等"的话，那么他们equals必须相同。这是我们赋予的**equals的意义**。

那为什么equals和hashCode又有关联呢。这主要是和map这种结构有关（当然可能也有别的地方用到了hash算法），我们希望如果相同的对象在**map**中只有一个实例（如果这个对象被当作值key）：这里的相同使用的是我们的equals方法，而map这种结构是使用了hashcode来”散列“（以保证均匀和足够的空间利用），这个时候就看出来我们需要同时重写两个方法。

可能你写的hashcode是合法的，但是他其实不正确或不符合逻辑。记住：hashCode并不需要总是能够返回唯一的标识码，但是equals方法必须严格的判断两个对象是否相同

设计hashCode的一个重要的规则是：无论何时，对同一个对象调用hashCode都应该生成同样的值。所以，如果你的hashCode方法依赖于对象中一边的数据，用户就要当心了，因为此数据发生变化是，hashCode就会产生一个不同的散列码相当于产生了一个不同的键。也不应该是hashCode依赖于具有唯一性的对象的值，尤其是this的值（某些情况下使用id也是同理），因为这样很难产生重复的键，

要使得hashCode实用，他必须速度快，并且必须有意义，他必须基于对象内容生成散列码。

散列码不必是独一无二的（应该更关注速度，而不是唯一性），但是通过hashCode和equals必须能够完全确定对象的身份。

好的hashCode应该产生分布均匀的散列码



## 如何重写/重写模板

理想的哈希函数应当具有**均匀性**，即不相等的对象应当均匀分布到所有可能的哈希值上。可以将对象的每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。

R 一般取 31：

- 因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位，最左边的位丢失。

- 一个数与 31 相乘可以转换成移位和减法：`31*x == (x<<5)-x`，编译器会自动进行这个优化。

  来源：[CS-NOTES](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%9F%BA%E7%A1%80.md#hashcode)

```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```



编写像样的hashCode的基本指导（线代IDE基本都提供了自动补充hashCode的方法）：

1. 给int变量的result赋予非零值常量：例如17

2. 给对象每个有意义的域f（即每个可以做equals操作的域，每个被equals操作的域）计算出一个int散列码c

   | 域类型                 | 计算                                                      |
   | ---------------------- | --------------------------------------------------------- |
   | boolean                | c=(f?0:1)                                                 |
   | byte、char、short、int | c=(int)f                                                  |
   | long                   | c=(int)(f^(f>>>32))                                       |
   | float                  | c=Float.floatToIntBits(f)                                 |
   | double                 | long l =Double.doubleToLongBits(f)<br>c=(int)(l^(l>>>32)) |
   | Object                 | c=f.hashCode(0)                                           |
   | 数组                   | 每个元素应用如上                                          |

   

3. 合并计算得到的散列码

   result=37*result+c

4. 返回result

5. 检查结果，确保相同对象有相同的散列码

# notify & wait

这是一个方法簇，是一系列方法：notify()、notifyAll()、wait(long timeout)、wait(long timeout, int nanos)、wait()

调用这些方法必须持有锁，即必须在synchronized修饰的块中调用，否则会抛出异常。

- 注意，被ReentrantLock包围的块也不能调用这几个方法。ReentrantLock可以借助Condition对象来进行await等操作

调用wait()的线程会释放掉锁。导致wait()的线程被唤醒可以有以下几种情况

- 该线程被中断

- wait()时间到了

- 被notify()/notifyAll()唤醒



notify()唤醒的是在等待队列的某个线程(不确定会唤醒哪个)，notifyAll()唤醒的是等待队列所有线程。notifyAll()唤醒所有进程后，哪个线程接下来能够获取到锁要具体依赖于操作系统的调度。被唤醒之后不意味着立刻就能获得到锁。

- Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权。主要的区别在于**锁释放**上面：Object.wait()在释放CPU同时，释放了**对象锁的控制**。而Thread.sleep()没有对锁释放控制
- wait和notify方法均可释放对象的锁，但wait同时释放**CPU控制权**，即它后面的代码停止执行，线程进入阻塞状态，而notify方法不立刻释放CPU控制权，而是在相应的synchronized(){}语句块执行结束，再自动释放锁
- notify方法调用后，被唤醒的线程不会立马获得到锁对象。而是等待notify的同步代码块执行完之后才会获得锁对象（这和上面一点其实相同）

要注意这些方法在并发上的作用，可以看其他篇介绍并发库的文章，

# 其他

> toString()

```java
public class Example {}
```

默认返回 Example@5564617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

> clone

``` java
Address address = new Address(1, 2, 3);
address.clone(); //java.lang.CloneNotSupportedException
```

上面的代码不能执行成功，因为clone在Object中是protected方法，而Address和Object不在同一包之下。

如果要调用clone方法，则对象要实现Cloneable接口(该接口没有任何方法)，重写clone方法。

> finalize

finalize()在垃圾回收器清除对象之前调用，一般不会重写。**如非必要，尽量避免使用finalize**。

即使Override了finalize，事情也没有那么简单，首先，override了finalize的object并不是和其他情况下一样直接释放内存，也不是直接调用finalize，而是放到finalizing queue，由JVM上一个单独的thread来进行处理，同时你要当心在finalize里面不小心调用了code，<u>*把object自己重新reference到*</u>——这个时候它就复活了，可能事情做到一半，又从finalizing queue里面拿出来了，造成不稳定的状态——这种风险是不值得绝大多数情况下的应用场景的。

执行流程：

- finalize**在对象被垃圾收集器析构(回收)之前调用**，finalize()方法由GC**至多执行一次**（用户当然可以手动调用对象的finalize()方法，但并不影响GC对finalize()的行为）。

- 当对象变成(GC Roots)不可达时，GC会判断该对象是否覆盖了finalize方法，若未覆盖，则直接将其回收。否则，若对象未执行过finalize方法，将其放入F-Queue队列，由一低优先级线程执行该队列中对象的finalize方法。执行finalize方法完毕后，GC会再次判断该对象是否可达，若不可达，则进行回收，否则，对象“复活”。

finalize清理可以用于：

1. 清理本地对象(通过JNI创建的对象)；

   Java的垃圾回收器只会释放由我们new出来的内存堆块，那些不是由new出来的“特殊内存”，垃圾回收器是不会管理的。这些所谓的特殊内存指通过JNI用C/C++向系统申请的内存，这些内存如果不手动去清除就会一直占据在内存中。

2. 确保某些非内存资源(如Socket、数据库、文件等)的释放：在finalize()方法中显式调用其他资源释放方法。

   finalize不应该用来管理socket, file handle, database connection等昂贵资源，因为finalize是在object被GC的时候才调用——如果内存足够，它可能永远都不会被调用。

如果调用则需要注意执行 super.finalize(); 

