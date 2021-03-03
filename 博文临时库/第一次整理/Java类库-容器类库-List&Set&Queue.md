---
title: 
description: 
tags:
- JAVA
create_time: 2020-2-24
---



# List

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-list.png)

有三种基本的List：**ArrayList、LinkedList、Vector**（注意：🔴**不建议继续使用Vector**）。这几个List的区别也可以堪称数据结构上的区别：

- **ArrayList**：数组
- **LinkedList**：链表
- **Vector**：线程安全

其中区别就是数据结构的区别，即：

- 数组/ArrayList：擅长随机访问元素，插入和删除元素慢，代价高昂；
- 链表/LinkedList：在随机访问上慢，但插入和删除代价较低；
  - LinkedList在顺序访问有一定优化
- Vector：是提供了线程安全的List，但同时增加了加锁的开销，并且是老旧的类

他们的数据结构实现在代码中的体现也很容发现：

1. **ArrayList**：

   **RandomAccess** 接口标识着该类支持快速随机访问，**elementData**也标识了数据结构

   ``` java
   public class ArrayList<E> extends AbstractList<E>
           implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
       ...;
       transient Object[] elementData;
       ...;
       private static final int DEFAULT_CAPACITY = 10;
   }
   ```

   代码中也表现了数组的默认大小：即10

   ![](E:\_data\博文临时库\博文中的图片\ArrayList数据结构.png)

2. **LinkedList**：

   Node的定义标明了是**双向链表**（**非循环**），使用 **Node** 存储链表节点信息。LinkedList中有两个域**first和last**记录链表头尾。

   ``` java
   transient Node<E> first;
   transient Node<E> last;
   
   ...;
   
   private static class Node<E> {
       E item;
       Node<E> next;
       Node<E> prev;
   }
   ```

   ![](E:\_data\博文临时库\博文中的图片\LinkedList数据结构.png)

## ArrayList

（**添加和删除元素**）

有两个类型的add方法，在末尾和在指定index进行add操作；另外有add一个元素和多个元素的addAll方法。

对于remove，同样有按index进行remove和查找object进行remove，两者均只删除一个元素。

添加操作都调用了`ensureCapacityInternal`最终它调用了 `System.arraycopy` 方法，删除操作调用了`System.arraycopy` 方法。

（**查找元素**）

与查找相关联的是contains方法和indexOf方法，contains是调用index实现的。indexOf方法源码如下，暂不解释

``` java
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

- 🔴List的 indexof、remove、retainAll求交集、（contains调用的是indexof）都会用到equals方法，必须认识到List的行为依据Equals的行为而有所变化。**用索引值来移除元素，与通过对象引用来移除元素相比，显得更加直观并且不必担心equals的行为**

（其他方法）

ArrayList还有些其他的方法：

- retainAll可以求两个list的交集，相应的removeAll也可以用作求差集操作

### 扩容

添加元素时使用 `ensureCapacityInternal()` 方法来保证容量足够，如果不够时，需要使用 `grow()` 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 1.5 倍。

- `ensureCapacityInternal` --> `grow` 

 `grow()` 方法的扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，最终 `Arrays.copyOf()`调用了`System.arraycopy`方法。

🔴**最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。**

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-list-ArrayList-扩容.png)

### SubList

> Returns a view of the portion of this list between the specified fromIndex , inclusive, and toIndex, exclusive. 

``` java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}
```

这个方法返回了一个SubList，SubList类是ArrayList中的一个内部类。

SubList并没有重新创建一个List，而是直接引用了原有的List（返回了父List的视图），只是指定了一下他要使用的元素的范围而已（范围：`[fromIndex,toIndex)`）。subList 返回一个视图，对于subList的所有操作都会最终反映到源列表上。（持有引用和源对象的问题）

- 如果将subList方法产生的片段用来传递给较大列表的containsAll()方法，会得到true，即使这个片段上调用了shuffle也会返回true，因为这**与顺序无关**，

- 这些问题同样可以从源码反应出来,下面以ArrayList的subList源码反应,可以看到内部类**SubList持有一个较大列表的引用，并保存一个offset值**，很明显这样就能回答上面的问题

我们不能把subList方法返回的List强制转换成ArrayList等类，因为他们之间没有继承关系。subList只是持有了外部类ArrayList的引用：

``` java
private class SubList extends AbstractList<E> implements RandomAccess {
    private final AbstractList<E> parent;
    private final int parentOffset;
    private final int offset;
    int size;
    ...;
}
```

**🔴对父子List的修改**：

1. 对父(sourceList)子(subList)List做的非结构性修改（non-structural changes），都会影响到彼此。

2. 对子List做结构性修改，操作同样会反映到父List上，但不一定会抛出异常。

   - 子List的添加不会扩容，初始 `size=toIndex-fromIndex` 是多少就是多少，因为在子List的add之前会检查是否在子List的size内

3. **对父List做结构性修改，会抛出异常ConcurrentModificationException**

   （反过来不成立）
   
   ``` java
   public static void main(String[] args) {
       List<String> sourceList = new ArrayList<String>() {{
           add("H");
           add("O");
           add("L");
           add("L");
           add("I");
           add("S");
       }};
   
       List subList = sourceList.subList(2, 5);
   
       System.out.println("sourceList ： " + sourceList);
       System.out.println("sourceList.subList(2, 5) 得到List ：");
       System.out.println("subList ： " + subList);
   
       sourceList.add("666");
   
       System.out.println("sourceList.add(666) 得到List ：");
       System.out.println("sourceList ： " + sourceList);
       System.out.println("subList ： " + subList);
   
   }
   //output
   //sourceList ： [H, O, L, L, I, S]
   //sourceList.subList(2, 5) 得到List ：
   //subList ： [L, L, I]
   //sourceList.add(666) 得到List ：
   //sourceList ： [H, O, L, L, I, S, 666]
   //Exception in thread "main" java.util.ConcurrentModificationException
   ```

只需要记住

1. 对于subList的所有操作都会最终反映到源列表上；
2. 现在多了一个需要注意`ConcurrentModificationException`的地方台：对原集合元素的增加和删除，均会导致子列表的遍历、增加、删除产生`ConcurrentModificationException`

<u>🔴注意：collection.shuffle方法和这个类似，他没有影响原来的数组，而只是打乱了shuffled中的引用，之所以这样，是因为randomized方法用一个ArrayList将Arrays.asList方法的结果包装了起来*</u>

<u>*todo：把上面这个注意添加到某些聚合文章中*</u>

### batchRemove

求差集removeAll和求交集retainAll均调用了该方法(注意是removeAll不是remove)，batchRemove是批量删除元素的方法，他避免了移动数组尾部元素。<u>*todo：这一部分的源码值得一读*</u>。

- 求差集就是把不包含在c中的元素放到elementData前面，维护一个w指针，从0到w都是不包含在c中的。求交集就是相反。

这个方法使用复制的方法，而不是调用remove()的方式，减少了元素的重复复制。最后将newSize之外的元素引用置空，防止内存泄漏。

``` java
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}
```

在batchRemove中使用了`try-finally`，在finally中有下面一段，这段代码确保了如果try被终止的话，能够把r到size的未处理的元素复制到w后面，之后更新w的值作为新的边界。然后清理掉w之后空间即赋为null

```java
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

## LinkedList

需要清楚一点：尽管LinkedList是基于链表实现的，但是对外部的视图仍然是List，外部的可见性和ArrayList在List接口下是一致的。

### node()

``` java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

以中间index为界限，减少了一半的搜索量。

### 其它封装用途

LinkedList除了实现了List接口也实现了Deque接口，这意味着它可以作为队列和双向队列，另外由于JDK内置的stack类实现的不太好，我们也可以用LinkedList封装一个栈。

- 如果从LinkedList继承，出现一个新的类Stack，这样的继承是不太合理的，因为这样会产生具有LinkedList所有其他方法的类，Java内置的Stack就是犯了这个错误，所以我们应该在一个类内部持有LinkedList，添加一些stack的方法来封装一下
- Deque = new LinkedList可以不用看底层实现，而只面对Deque接口

``` java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

# Queue

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-queue.png)

Queue有两个常用实现 ：

- LinkedList：可以用它来实现双向队列
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列

## Deque

Deque = new LinkedList可以不用看底层实现，从而只面对Deque接口

Deque接口提供的部分方法（这几个表格是用来对照一下，因为名字很迷惑人）

|      |     头部      |     头部      |    尾部     |     尾部     |
| :--- | :-----------: | :-----------: | :---------: | :----------: |
| 插入 |  addFirst(e)  | offerFirst(e) | addLast(e)  | offerLast(e) |
| 移除 | removeFirst() |  pollFirst()  | remveLast() |   pollLast   |
| 获取 |  getFirst()   |  peekFirst()  |  getLast()  |   peekLast   |

> stack

|          | Stack   | Deque         |
| -------- | ------- | ------------- |
|          | push(e) | addFist(e)    |
|          | pop()   | removeFirst() |
| 查询栈顶 | peek()  | peekFirst()   |

> Queue

| Queue     | Deque         |
| --------- | ------------- |
| add(e)    | addLast()     |
| offer(e)  | offerLast()   |
| remove()  | removeFirst() |
| poll()    | pollFirst()   |
| element() | getFirst()    |
| peek()    | peekFirst()   |

- `offer` 和 `add` 都是在队列中插入一个元素，具体区别：

  对于一些 Queue 的实现的队列是有大小限制的，因此如果想在一个满的队列中加入一个新项，多出的项就会被拒绝

   **`add()`**方法会**抛出异常**，而 **`offer()`** 只是**返回 false**

- `remove()` 和 `poll()` 方法都是从队列中删除第一个元素

  **`remove()`**将**抛出异常**，而 **`poll()`** 则会**返回 null**

- `element()` 和 `peek()` 用于在队列的头部查询元素

  在队列为空时， `element()` **抛出异常**，而 `peek()` **返回 null**

## PriorityQueue

<u>*todo:优先队列声明下一个弹出的元素是最需要的元素（具有最高优先级的元素）*</u>

优先队列常用于操作系统的任务调度，也是贪心算法的重要组成部分

# Set

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-set.png)

*<u>todo：</u>*

Set接口的方法很少，他没有get类型的方法，对于元素的获取，只有contains和iterator遍历。此外也有一些常用的方法。

- HashSet（无序，唯一）：基于 HashMap 实现，采用 HashMap 来保存元素

  - HashSet的**值存放于HashMap的key**上，HashMap的value统一为PRESENT

    （<u>*todo：thinking：这也给我们一个使用HashMap的新思路：只使用HashMap的Key，value存储为一样的值，但是这么做不就是HashSet了么？*</u>）

    ``` java
        // Dummy value to associate with an Object in the backing Map
        private static final Object PRESENT = new Object();
    ```

    内部持有一个HashMap

    ``` java
    private transient HashMap<E,Object> map;
    ```

  - 向HashSet添加元素调用的是HashMap的put，不重复性是由HashMap的键不重复来实现的，添加相同的元素（equals相同）的行为和HashMap相同

- LinkedHashSet： LinkedHashSet 继承于 HashSet，其内部通过 LinkedHashMap 来实现。

- TreeSet（有序，唯一）： 红黑树(自平衡的排序二叉树)

一些简单取舍：

- <u>*todo: hashset在添加和查询时性能基本总是比TreeSet好*</u>
- <u>*todo: treeSet在迭代时通常比hashSet更快？为什么？上面刚说了查询性能不好？*</u>

# 选择取舍

## ArrayList & LinkedList

如Think in Java书中所讲，"**最好的策略是置之不理，直到需要担心这个问题。**如果开始在某个ArrayList中间执行过多插入操作，程序开始变慢，那么List的实现采用ArrayList很可能是罪魁祸首"。在编写之初过多的思考反而不利于开发，不知道在哪里看到这种思想，但是实际编程和很多地方看到这样的说法也确实说明有道理，优化问题提前过多的考量是不是也算在了超前设计里面。

LinkedList的插入删除速度更快，但也占用更大的存储空间，因为每个节点都保有前后节点的引用

总之，两者不同主要体现在：

1. 底层实现不同：数组和链表
2. 访问、插入、删除的效率不同：数组适合查找，链表适合增删（一般情况）
3. 存储所占内存大小不同：链表有前后node引用

**ArrayList增删真比LinkedList慢吗**：

这个问题需要考虑的地方主要有几个方面：**增删位置、数据量**。

<u>*todo: 这里没有进行实验说明，实验代码应当在单独的文章系列中进行讲解。由于很多地方都是会编写测试和实验代码的，把实验代码和测试代码放到一个地方也有助于总结编写实验代码的注意事项和技巧*</u>

对于**尾部插入**而言，ArrayList 与 LinkedList 的性能是**几乎一致**的。

对于**头部插入**而言，LinkedList 时间复杂度是 O(1)，而对于**ArrayList**来说，头部的每一次插入都需要移动 size-1 个元素，**效率很低**。

对于**中间插入**而言，在**大数据量**情况下**ArrayList** 速度比 LinkedList 的速度**快**不少，在**小数据量**情况下还是应该使用**LinkedList** 。

大数据量要考虑是否使用ArrayList而小数据量要考虑是否使用LinkedList。在指定容量情况下，ArrayList不进行频繁的扩容，也可以提供很好的查找性能。LinkedList适合进行小数量频繁的插入和删除操作。

# 其他问题

## 排序List

todo:https://stackoverflow.com/questions/8725387/why-is-there-no-sortedlist-in-java?

todo:https://stackoverflow.com/questions/416266/sorted-collection-in-java



List可以插入多个null元素

Set只允许存入一个null元素

# 参考

> 1. 
> 2. 
