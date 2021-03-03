---
title: 
description: 
tags:
- JAVA
create_time: 2020-2-24
---



# 综述

标准Java类库有几个Map的基本实现：`HashMap`、`TreeMap`、`LinkedHashMap`、`WeakHashMap`、`ConcurrentHashMap`、`IdentityHashMap`。

他们都有同样的基本接口Map，但是行为特性各不相同，这表现在：**效率**、**键值对的保存和呈现次序**、**对象的保存周期**、**映射表如何在多线程中工作**、**判定“键”的等价策略**等方面。

除了`IdentityHashMap`，所有的Map实现的插入操作都会随着Map尺寸的变大而明显变慢（<u>*todo：这个明显的阈值是什么？*</u>）但是，查找的代价通常比插入要小的多

`TreeMap`通常要比HashMap慢，`LinkedHashMap`在插入时比`HashMap`慢一点，因为他需要额外维护链表以保持插入顺序。正是由于这个列表，使得他的迭代速度更快。（<u>*todo ： why？为什么？*</u>）

Map接口中包含了Entry接口：

``` java
public interface Map<K,V> {
    ...;
	interface Entry<K,V> {...;}
    ...;
}
```

从数据结构角度看，常用Map的区别如下：

- HashMap：数组+链表==链表长度大于8==>数组+红黑树
- LinkedHashMap：数组+链表+双向链表（这个链表和HashMap的链表有所不同）
- TreeMap：

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-map.png)

- TreeMap是`sortedMap`现在的的唯一实现

- 注意这样一个设计：在自己实现Map接口的同时其实也需要同时实现`Map.Entry`接口。所以了解Map接口就要了解`Map.Entry`接口。同样的，这样的设计在List的各个实现中也有体现：实现接口的同时要提供迭代器的实现。
- 为什么引入红黑树：即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。
- 数组是主体，链表是为了解决hash冲突存在

既然都用了数组，那么就需要明白哈希桶数组大小的影响：

- 如果哈希桶数组很大，即使较差的Hash算法也会比较分散，如果哈希桶数组数组很小，即使好的Hash算法也会出现较多碰撞，所以就需要在空间成本和时间成本之间权衡，其实就是在根据实际情况确定哈希桶数组的大小，并在此基础上设计好的hash算法减少Hash碰撞。

那么通过什么方式来控制map使得Hash碰撞的概率又小，哈希桶数组（Node[] table）占用空间又少呢？答案就是好的Hash算法和扩容机制。

<u>*todo：环形链表？？？*</u>

> 环形链表

由于 `JDK 1.8` 转移数据操作为 **按旧链表的正序遍历链表、在新链表的尾部依次插入**，所以不会出现链表 **逆序、倒置**的情况，故不容易出现环形链表的情况。

## key & hashCode

无论何时，对同一个对象调用hashCode都应该产生同样的值。

Java的散列函数都使用2的整数次方，对于处理器来说，除法和取余是很慢的操作，使用2的整数次方长度的散列表，可以用掩码代替除法，因为get是使用最多的操作，对余数的%操作是开销最大的部分，使用2的整数次可以削除此开销

- String、Integer 这样的包装类适合作为 key 键
  - String、Integer等包装类的特性，保证了Hash值的不可更改性 & 计算准确性
    - （因为是final类型所以具有不可变性）
  - 有效减少了Hash碰撞的机率：因为这是JDK开发人员写的，规范遵守的应该比较好
- Object作为key的步骤：
  - 重写hashCode
  - 重写equals
- 为什么把hashCode放到了根类Object里：
  - 每个类都有产生hashCode的需求

## 性能

（减少空间占用，减少重新散列次数，减少查找时间即链表长度，减少Hash碰撞）

（map的一系列类的写法可以看出，map这一系列的作者适合其他类的作者风格迥异的，大概是搞算法很厉害的，也正是这样说明了map的重要之处？）

提高Map的性能：

- 构建专门针对特定类型的Map，这样可以避免与Object之间的类型转换操作
- 用数组代替溢出桶，这样可以针对磁盘存储方式做出优化、在创建和回收时可以节约时间

提高HashMap性能可以调整的参数：

- 容量、初始容量、尺寸、负载因子、
  - 创建一个具有恰当大小的初始容量的map可以避免自动再散列的开销

避免Hash碰撞采取的办法有：

1. 好的hash算法：
   - 好的hashCode实现
2. 扩容机制

链表变红黑树：

即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。

<u>*todo: 为什么不直接使用红黑树而是链表长度大于8才采用红黑树*</u>

猜测应该是这个是计算耗时的

# HashMap

从结构实现来讲，HashMap是数组+链表+红黑树。

内部包含了一个 **Entry** 类型的**数组 table**。Entry 存储着键值对。它包含了**四个字段**，从 next 字段我们可以看出 **Entry** 是一个**链表**。即数组中的每个位置被当成一个桶，**一个桶存放一个链表**。

``` java
transient Entry[] table;
......
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node
}
```

- 我们无法控制数组下标值的产生，这个值依赖于HashMap的容量，容量的改变与容器的充满程度和负载因子有关。

冲突由链表处理，同一个链表中存放哈希值和散列桶取模运算结果相同的 Entry。当链表长度大于8时转换成红黑树。

数组并不直接保存值，而是保存值的list。虽然应用equals在list中进行线性查找会比较慢。但是如果散列函数好的话，数组的每个位置就只有比较少的值。因此不是查询整个list，而是快速地跳到数组地某个位置，只对很少地元素进行比较。（桶数组只保存每一段地第一个元素，所以说是“跳到”）

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-map-hashMap结构.png)

- 需要明白数组底层具体存储的是什么：`Node`，即`Map.Entry`

## 构造函数

``` java
public HashMap(int initialCapacity, float loadFactor) {...;}
```

从HashMap的构造函数源码可知，构造函数就是对下面几个事关map存储容量的字段进行初始化。HashMap也可以接收一个其他的Map作为构造参数。

（注意之后的术语：capacity是容量，意为最多存多少键值对，但是不一定存满；size意为实际存在的键值对数量；这里没有使用length这个词。）

- `loadFactor`：负载因子，容量为capacity的容器最多存`capacity*loadFactor`个元素

  - 默认的负载因子0.75f，这是对空间和时间效率的一个平衡选择，建议不要修改

  - 除非在时间和空间比较特殊的情况下，可以修改：
    - 如果内存**空间很多**而又对**时间**效率**要求很高**，可以**降低**负载因子Load factor的值；
    - 如果内存**空间紧张**而对**时间**效率**要求不高**，可以**增加**负载因子`loadFactor`的值，这个值**可以大于1**。
    
  - `loadFactor`有什么用？调整了之后有什么影响

    - `loadFactor`如果很小，虽然导致冲突减少，但是会频繁扩容，导致不能充分利用空间。如果很大就会很难触发扩容，越来越多的冲突就会发生，就会在链表或者红黑树上去查找，这样就不是常数时间找到目标了。

    [Stackoverflow: What is the significance of load factor in HashMap?]( https://stackoverflow.com/questions/10901752/what-is-the-significance-of-load-factor-in-hashmap)

- table的容量capacity：

  - 大小必须为**2的n次方**(一定是**合数**)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数，具体证明可以参考[这篇文章](https://blog.csdn.net/liuqiyao_01/article/details/14475159)

    采用这种非常规设计，主要是为了在**取模和扩容**时做**优化**，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了**高位参与运算**的过程。
  
- threshold：在此`loadFactor`和capacity对应下允许的最大元素数目

  - size超过这个threshold数目就重新resize(扩容)，扩容后的HashMap容量是之前容量的两倍。
  - 传入的`initialCapacity`会通过`tableSizeFor` 方法调整为**2的N次幂**
  - threshold = capacity* Load factor

有时候就会有人问这样的问题：

- **准备用HashMap存1w条数据，构造时传10000还会触发扩容吗？**  

  **不会**，从threshold、`initialCapacity`、`tableSizeFor`来回答即可。
  
  在创建的时候会把size调整为2的次幂大小，所以算最后threshold的时候需要用调整后的大小。

## hash

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。而我们的hash方法应当尽量使每个位置上的元素数量只有一个。

- 有时两个key会定位到相同的位置，表示发生了Hash碰撞。如果Hash算法计算结果越分散均匀，Hash碰撞的概率就越小，map的存取效率就会越高。

Hash算法本质上就是三步：**取key的hashCode值、高位运算、取模运算**。

- 这三步扰动处理，加大了随机性，使得数据分布更为均匀，尽量减少了hash冲突。

``` java
方法一：
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
方法二：
static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
     //当length总是2的n次方时，h& (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率
}
```

<img src="E:\_data\博文临时库\博文中的图片\HashMap的hash过程.png" style="zoom:50%;" />

## put & get

**put:**

put调用的是`putVal`，下面主要的逻辑都是`putVal`的。

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-map-hashMap-put方法.png)

- hash：找到桶下标的过程就是用到了上面的hash方法
- 头插入：是作为链表头插入，头插法不是尾插法，也就是新的键值对插在链表的头部，而不是链表的尾部
- 插入null值：允许插入键为 null 的键值对，但是因为无法调用 null 的 hashCode() 方，使用第 0 个桶存放键为 null 的键值对
- 新节点替换旧节点：如果已经包含有key所关联的键值对，那么将会替换掉原来的那一对键值对，然后返回旧的值。
- 如果是替换则不会触发扩容，如果是是插入则在put方法最后会判断是否就行扩容
- <u>*todo：这个图感觉不是很好了*</u>

> 判断key是否重复的代码：

（先比较key的地址，再调用equals方法）（先判断对应桶数组是否为null，再判断对应位置上链表第一个节点是否和key相同/即用下面这行代码，再在链表或红黑树中逐个比较key是否相同，最后插入）

```java
// putVal方法，putVal值得一读
((k = e.key) == key || (key != null && key.equals(k))))
```

**get:**

get调用了`getNode`方法，这个方法是查找的核心（除get系列之外还有contains系列、replace系列方法用到了`getNode`）

- 首先计算出hash，找到目标位置
- 然后判断是不是第一个位置
- 如果不是第一个位置就判断是链表还是红黑树，然后执行各自的查找
- 如果没找到返回null

### 扩容

（首先要明确扩容时机，然后明确再散列的操作过程。）

size超过这个threshold数目就重新resize(扩容)，扩容后的HashMap容量是之前容量的**两倍**，之后使用新的数组代替已有的容量小的数组。在旧数组中同一条Entry链上的元素，通过重新计算索引位置后，有可能被放到了新数组的不同位置上。

- 扩容时机：所有键值对的实际数目 size 大于 table的实际大小（`loadFactor*capacity`即`threshold`）时进行扩容
- 扩容过程：
  - 计算每个node的新下标，然后进行头插入
  - 使用新的Entry数组代替旧的数组

举个例子说明下扩容过程：

假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。其中的哈希桶数组table的size=2， 所以key = 3、7、5，put顺序依次为 5、7、3。在mod 2以后都冲突在table[1]这里了。这里**假设**负载因子`loadFactor`为1，即当键值对的实际大小size 大于 table的实际大小时进行扩容（**这是扩容时机**）。接下来的三个步骤是哈希桶数组 resize成4，然后所有的Node重新rehash的过程。

- 在扩容转移Node到新的数组和链表中的时候，注意在新链表中是头插入的

<img src="E:\_data\博文临时库\博文中的图片\HashMap扩容过程简单举例.png" style="zoom:75%;" />

**JDK1.8做了哪些优化：**

<u>*todo：这部分内容暂未整理*</u>

下面讲解下JDK1.8做了哪些优化。经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。

看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![](E:\_data\博文临时库\博文中的图片\HashMap在jdk1.8扩容过程.png)

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![](E:\_data\博文临时库\博文中的图片\HashMap在jdk1.8扩容过程2.png)

因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成原索引加`oldCap`，可以看看下图为16扩充为32的resize示意图：

<img src="E:\_data\博文临时库\博文中的图片\HashMap在jdk1.8扩容过程3.png" style="zoom:50%;" />

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。这一块就是JDK1.8新增的优化点。有一点注意区别，JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，JDK1.8不会倒置。可以研究下JDK1.8的resize源码，写的很好。

### 死循环

```java
do {
    Entry<K,V> next = e.next; // <--假设线程一执行到这里就被调度挂起了
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i]; // 注意这里的newTable[i]，很让我迷惑，其实他就是一个节点，这三步就是链表头插入。newTable[i]就是表头。
    newTable[i] = e;
    e = next;
} while (e != null);
```

https://coolshell.cn/articles/9606.html

https://zhuanlan.zhihu.com/p/67915754

https://juejin.im/post/6844903554264596487的评论

## keySet & values & iterator

对map的修改会反映在keySet和values中，对keySet和values的修改也会反映到map中。

- 不能对values、keySet、entrySet进行添加元素。
- 可以对`EntrySet`、values、`KeySet`执行remove、clear

它们的迭代器和回调类似，是用的HashMap的方法来访问元素，只不过包装了一下

## 1.8 优化内容

<u>*todo：这部分内容可以放到wiki里，其实放到这里不是很合适，文章里讲现在的原理就行了，如果一味的讲解不同版本的反而容易弄混，适当即可，不要过度*</u>

减少 Hash冲突 & 提高哈希表的存、取效率。

数据结构上：

![](E:\_data\博文临时库\博文中的图片\HashMap在jdk1.8和jdk1.7数据结构的变化.png)

存放/获取数据时：

![](E:\_data\博文临时库\博文中的图片\HashMap在jdk1.8和jdk1.7存取数据变化.png)

扩容机制：

![](E:\_data\博文临时库\博文中的图片\HashMap在jdk1.8和jdk1.7扩容机制的变化.png)

# LinkedHashMap

``` java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>{
	...;
    final boolean accessOrder;
    ...;
}
```

- LinkedHashMap继承自HashMap，实现了map接口

  **继承自 HashMap**，因此具有和 HashMap 一样的快速查找特性。内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序

- `accessOrder` 决定了顺序，默认为 false，此时维护的是插入顺序。`accessOrder` 为true可以实现LRUMap（LRU算法）。

  迭代次序是LRU次序或者插入次序。在迭代访问时比HashMap要快，因为它使用了链表维护内部次序

下面是`LinkedHashMap`的键值对的结构，可以发现继承自`HashMap.Node`因此具有`HashMap.Node`的特点，另外添加了before, after两个指针，进而实现了双向链表记录了访问顺序。

``` java
	/**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-map-LinkedHashMap结构.png)

- <u>*todo: 注意，很多笔记说它是循环链表，我不那么认为，我认为他不是循环链表，即不是双向循环链表，只是一个双向链表*</u>
- <u>*todo：这个图可能有点乱，但是应该表达了我的想法*</u>

## put & get

在HashMap中最常用的两个操作就是：put(Key,Value) 和 get(Key)。同样地，在 LinkedHashMap 中最常用的也是这两个操作。

对于put(Key,Value)方法而言，LinkedHashMap完全继承了HashMap的 put(Key,Value) 方法，没有任何修改。

对于get(Key)方法而言，LinkedHashMap则直接对它进行了重写。在LinkedHashMap的get方法中，通过HashMap中的`getNode`方法获取Entry对象。并在之后添加调用`afterNodeAccess`的步骤。

``` java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

``` java
void afterNodeAccess(Node<K,V> p) {...}
void afterNodeInsertion(boolean evict) {...}
void afterNodeRemoval(Node<K,V> e) {...}
```

这两个函数在HashMap中是空的(如下)，在HashMap中响应方法调用没有效果，但是在LinkedHashMap中调用由于重写就会有了效果，这是**模板方法**设计模式。

``` java
    // Callbacks to allow LinkedHashMap post-actions
    void afterNodeAccess(Node<K,V> p) { }
    void afterNodeInsertion(boolean evict) { }
    void afterNodeRemoval(Node<K,V> p) { }
```

- `afterNodeAccess`

  当一个节点被访问时，如果 `accessOrder` 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

- `afterNodeInsertion`

  在 put 等操作之后执行，当 `removeEldestEntry`方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。

  evict 只有在构建 Map 的时候才为 false，在这里为 true。

  ``` java
  void afterNodeInsertion(boolean evict) { // possibly remove eldest
      LinkedHashMap.Entry<K,V> first;
      if (evict && (first = head) != null && removeEldestEntry(first)) {
          K key = first.key;
          removeNode(hash(key), key, null, false, true);
      }
  }
  ```

  `removeEldestEntry`默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

  ``` java
  protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
      return false;
  }
  ```

## LRU 缓存

以下是使用 LinkedHashMap 实现的一个 LRU 缓存：

- 设定最大缓存空间 MAX_ENTRIES 为 3；
- 使用 LinkedHashMap 的构造函数将 `accessOrder` 设置为 true，开启 LRU 顺序；
- 覆盖 `removeEldestEntry`方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final int MAX_ENTRIES = 3;

    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LRUCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }
}
```

# TreeMap

TreeMap本身即是一棵红黑树，不具备Hash的查找性能。TreeMap为增、删、改、查这些操作提供了log(N)的时间开销，从存储角度而言，这比HashMap与LinkedHashMap的O(1)时间复杂度要差些；但是在统计性能上，TreeMap同样可以保证log(N)的时间开销，这又比HashMap与LinkedHashMap的O(N)时间复杂度好不少。

唯一带有`subMap`的Map，可以返回一个子树。

- 注意：TreeMap没有继承自HashMap。
- 可以传入自己的Comparator，但是只能修改对key的排序。
- 如果想让TreeMap按照values排序，则应该获取`EntrySet`然后对其排序。即用自己的Comparator接口

# 其他问题

1. 大数据量处理：

   例如：如果问：把200W个对象放到HashMap中应该注意哪些细节：

   - 不使用`hashmap`，换用其他框架Redis等、中间件
   - 写出更好的`hashcode`避免碰撞
   - 提前设定好容量和负载因子，避免频繁扩容

2. 并发Map不在这里讲解

3. HashMap的key值要是为类对象则该类需要满足什么条件？

   需要同时重写该类的hashCode()方法和它的equals()方法，并保证实现正确

# 参考

> 1. [Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)
> 2. [Java 容器](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%AE%B9%E5%99%A8.md)

