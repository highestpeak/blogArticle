---
title: JAVA引用对象
description: 
tags:
- JAVA
- 内存管理
- 生命周期
- 引用
- Android
create_time: 2020-3-23 00:19:00
---

为什么需要四种引用类型呢？我认为可能是给我们提供了一种便利。形如ThreadLocal，一个线程会持有一个他自己的独立的一个ThreadLocal变量，例如String，当这个String没有人引用的时候，使用弱引用类型就会自动被GC回收，这样就不用我们每当不去引用String就必须判断String是否有引用然后给它赋值为null，因为我们只需要把自己的引用赋值为null，剩下的由JVM去做



Java中有4种引用类型：强引用，软引用，弱引用，虚引用(其实还有一些其他的引用类型如`FinalReference`)。本文既是介绍这四种引用类型，也相当于介绍了ref包（包含 `Reference` 和 `ReferenceQueue`）。ref包是为了更好更完善的垃圾回收机制、内存管理来进行设计的。

`java.lang.ref` 是 Java 类库中比较特殊的一个包，它提供了与 Java 垃圾回收器密切相关的引用类；**`StrongReference`（强引用）**，**`SoftReference`（软引用）**，**`WeakReference`（弱引用）**，**`PhantomReference`（虚引用）**，这四种引用的强度按照出现的顺序依次减弱。除了强引用之外，其他三种引用都在`java.lang.ref`包中。

1. 强引用（`StrongReference`）
   一般情况下我们使用的都是强引用，如`Object obj = new Object()`的 obj 就是一个强引用。
   当内存不足，JVM 宁愿抛出 `OutOfMemoryError` 错误，使程序异常终止，也不会回收强引用对象来释放内存

   将对象的引用显式地置为null 可帮助垃圾收集器回收此对象

2. 软引用（`SoftReference`）
   GC 发现了只具有软引用的对象并不会立即进行回收，而是让它活的尽可能久一些，在**内存不足**前再进行回收。（**软引用会在内存不足时被回收，内存不足的定义和该引用对象get的时间以及当前堆可用内存大小都有关系**）。

   如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中

   这一点可以很好地用来解决OOM的问题，并且这个特性很适合用来实现缓存

3. 弱引用（`WeakReference`）
   GC 一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存

   被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。
   
4. 虚引用（`PhantomReference`）
   虚引用主要用来跟踪对象被 GC 回收的活动，虚引用必须和引用队列（`ReferenceQueue`）联合使用

   又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。

   为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知。



**注意：**

- > 因为引用对象指向的对象 GC 会自动清理，但是引用对象本身也是对象（是对象就占用一定资源），所以需要我们自己清理。

  举个例子：`SoftReference ss = new SoftReference("abc", queue)`
  其中，`ss` 是一个引用对象（一个普通的对象，本身是强引用），指向`”abc”`对象（该对象只有一个软引用 `ss` 指向它）；`”abc”`对象会在一定时机被 GC 自动清理，但是 `ss` 对象本身的清理工作依赖于 queue，当 `ss` 出现在 queue 中时，说明其指向的对象已经无效，可以放心清理 `ss` 了

- ref包中常用的是**软引用**、**弱引用**

- 

<u>*todo: 阅读源码整理Reference对象的四种状态转换：Active、Pending、Enqueued、Inactive。*</u>

**应用场景：**

- 软引用、弱引用都适合用来实现**内存敏感的高速缓存**
  - 如果只是想避免 `OutOfMemory` 的发生，则可以使用软引用；如果对于性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用；
  - 如果对象可能会经常使用的，就尽量用软引用；如果对象不被使用的可能性更大些（偶尔的使用），就可以用弱引用。

- **ThreadLocal中**，获取到线程私有对象是通过线程持有的一个`ThreadLocalMap`，然后传入ThreadLocal当做key获取到对象的，这时候就有个问题，如果你在使用完ThreadLocal之后，将其置为null，这时候这个对象并不能被回收，因为他还有 `ThreadLocalMap`->entry->key的引用，直到该线程被销毁，但是这个线程很可能会被放到线程池中不会被销毁，这就产生了内存泄露，JDK是通过弱引用来解决的这个问题的，entry中对key的引用是弱引用，当你取消了ThreadLocal的强引用之后，他就只剩下一个弱引用了，所以也会被回收。

- 对于**软引用**关联着的对象，只有在内存不足的时候JVM才会回收该对象。因此，这一点可以很好地用来解决**OOM**的问题，并且这个特性很适合用来**实现缓存**：比如网页缓存、图片缓存

  - 如果为了避免OOM，则可以尽量使用软引用，但也可以通过LRU来避免OOM

- **弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期**。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象

- **虚引用**唯一的用处：能在对象被GC时收到系统通知

- Android中的示例：

  - 软引用：在平常Android开发中,有很多的图片要显示,如果是网络的则通过网络解析获取,如果每次都从网络解析影响体验,那么我们会将其保存到本地,如果每次从本地获取相对于我们将获取后的图片缓存下来直接从内存中获取效率更低.但是因为图片的数量多,消耗内存过大,缓存图片的过程需要大量的内存,内存不够则会OOM,这时便可以采用软引用的技术来解决问题.

  - 弱引用：Handler持有一个Activity的引用,Handler作为一个耗时的异步线程处理,如果在处理过程中把Activity关闭了,因为Handler还持有Activity的引用,而一个异步线程持有Handler引用,那么就将导致内存泄漏。解决方案：

    - 在Activity关闭的地方将**线程停止**以及把Handler的**消息队列的所有消息对象移除**

    - Handler改为内部类

      <u>*todo: remove code here*</u>

      ``` java
      public class WeakReferenceActivity extends AppCompatActivity {
      
          private WeakReference<WeakReferenceActivity> reference;
          private MyHandler myHandler = new MyHandler(this);
      
          @Override
          protected void onCreate(@Nullable Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              myHandler.sendEmptyMessage(1);
          }
      
      
          public static class MyHandler extends Handler {
              private WeakReference<WeakReferenceActivity> reference;
      
              public MyHandler(WeakReferenceActivity activity) {
                  reference = new WeakReference<WeakReferenceActivity>(activity);
              }
      
              @Override
              public void handleMessage(Message msg) {
                  switch (msg.what) {
                      case 1:
                          WeakReferenceActivity weakReferenceActivity = (WeakReferenceActivity) reference.get();
                          if (weakReferenceActivity != null) {
                              System.out.print("WeakReferenceActivity");
                          }
                          break;
      
                      default:
                          break;
                  }
              }
          }
      
          @Override
          protected void onDestroy() {
              super.onDestroy();
              myHandler.removeCallbacksAndMessages(null);
          }
      }
      ```

  - 虚引用：在Java中GC的运行时间是不确定的,在Java里有一个finalize方法,在垃圾回收器准备释放内存的时候，会先调用finalize().但因为内存还没有好耗尽,拟机不能保证在适当的时机调用finalize,因此垃圾回收期与finalize是不可靠的方法.

    这时可以采用虚引用.比如一个固定的内存,在明确知道一个bitmap回收之后会释放一部分内存,新释放开辟的内存就可以让其他bitmap来使用,以此循环来达到内存的稳定性控制.（？？？？？？？？？没看懂）

    用于Android中的`ViewPager`图片的查看？？

# 参考

> 1. [Java引用类型原理源码剖析](https://cloud.tencent.com/developer/article/1484113)
> 2. [分析了Android中的对应例子](https://www.jianshu.com/p/017009abf0cf)
> 3. 

