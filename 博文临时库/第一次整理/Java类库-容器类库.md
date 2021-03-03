---
title: 
description: 
tags:
- JAVA
create_time: 2020-2-24
---

# 综述

如果一个程序只包含固定数量的并且生命周期都是已知的对象，那么这是一个非常简单的程序。但是在程序中包含固定个数的对象常常来讲是不太可能的（即几个变量搞定），总会有大量的数据在一个地方等着我们把他们放到一个桶里一起带走。在JAVA中这个持有大量对象的桶就是 **数组和容器**。

在学习容器类库过程中，可以深刻体会到 **“面向接口编程而不是面向实现编程”** 这一句话的含义和价值，这也为我们学习设计模式和一些设计思想提供了参考。

关于容器类库，要完整的了解它，需要先了解一些基础知识，然后深入的了解一些较为困难或者说进阶的内容。

---

关于数组：

- 我把数组和容器放在了一起，因为认为数组和容器做的任务基本相同
- 数组基本只在本篇中讲解

**“面向接口编程而不是面向实现编程”**

- Iterable：迭代器提供了一种访问容器的通用方法。Iterable给我们践行“面向接口编程而不是面向实现编程”思想提供了道路
- 尽量使用较高层次的接口，例如List、Map等

关于容器类库，如果要完整的了解它，我认为至少需要了解以下几点：

- 容器继承体系，容器在继承体系中的"地位"
- 容器底层数据结构与实现
  - 一些行为导致的数据结构转化
  - 容器类库只是数据结构在Java中的一小部分体现，诸如图、树等高级一点的数据结构并没有体现
- 容器常见操作的实现细节
  - 例如扩容、map的取key和key重复问题
- 容器之间的关系
- 迭代器
- 线程安全问题，并发场景下应用（安全、线程同步、性能），并发容器
- 泛型容器

更为细致的了解则需要：

- 常用操作方法，操作技巧
  1. 初始化和批量初始化
  2. 迭代顺序（有序、无序）
- 新增API
- Utilities类库
- 重要接口：Iterator、Comparable等
- 流式操作
- 设计模式：
  - 迭代器模式
  - 生成器
  - 适配器
  - 享元模式
- 当存储占用内存很大的大对象的时候，使用 `java.lang.ref` 中类库的类来持有引用
- 当我们在某个场景下选择某个容器后，最好我们去测试一下是否适用

# 数组

**数组**时保存一组对象**最有效**的方式，如果保存一组**基本类型**的数据数组也是合适的选择。但是数组有着固有的缺陷：**固定的尺寸**。

- 数组唯一可以访问的字段或方法是**length**

  - length指标是数组能够容纳的元素，不知道数组中确切的有多少元素
- **对象数组**保存的是**引用**，**基本类型数组**直接保存基本类型的**值**。引用数组的所有引用初始化为**null**，数值型基本类型数组初始化为**0**.
- **多维数组**中，构成矩阵的每个向量都可以具有任意的长度（粗糙数组）

# 容器

大体上容器分为**两大类**：**collection、map**，collection表现起来看着是线性的，map是键值对的对应。更细一点来说，事实上只有**四种容器**：**List、Set、Queue、Map**。

- HashTable、Vector、Stack都是过于遗留下来的类，只是为了保持向后兼容，请不要在新的程序中使用它们，如果需要使用对应的特性，有新的类和新的实现方式

容器直接的直接区别通产归结于什么在背后“支持他们”，也就是说，所使用的接口是由什么**数据结构**实现的。进而使得他们的适用情况不同。这也是为什么需要了解不同容器实现的使用情况（其实了解了数据结构基本优劣，也就很容易明白了）。容器类库重要的原因也通常在于，他是很多常用数据结构的实现，而数据结构是如此的重要。

- 不同类型的Queue只在他们接受和产生数值的方式上有所差异

数据结构的不同也会产生一个容器划分方法，即元素顺序的划分，按照元素顺序划分：

- 无序：（Hash前缀）`HashSet`、`HashMap`等
- 升序：（Tree前缀）`TreeSet`、`TreeMap` 等
- 添加顺序：（Linked前缀）`LinkedHashSet`、`LinkedHashMap` 等

## 容器继承体系

整个继承图其实较为复杂，在这里拆开从几个不同角度去看（可能会越过一层例如几个Abstract），从不同角度来了解整个体系。针对每个例如List、Set、Queue、Map等的具体继承图请到响应的具体的文章去看。

简单的继承图：

![Java容器类库1](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库1.png)

collection的继承图（简单体现常用类）：

![Java容器类库2](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库2.png)

map的继承图（简单体现常用类）：

![Java容器类库3](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库3.png)



## AbstractCollection

AbstractCollection提供类collection的默认实现，使得我们可以创建它的子类型。

AbstractCollection实现了一些方法，例如：`contains`、`toArray`、`remove`、`clear`、`toString`，他们的共同点都是**用`Iterator`来实现逻辑的**，这也告诉我们：`Iterator`提供了访问容器的通用方法，我们可以不用去管他的实现，具体实现交由子类完成，这是一个很好的示例。

- AbstractCollection没有实现的方法：`iterator`、`size`、`add`，都是依赖于具体实现，即依赖于具体的数据结构。其中`add`采用抛出异常的方式来提供默认实现（实际上就是没有实现，这称为可选操作）。

基本上每个容器都有自己的Abstract实现，我们如果需要实现自己的特定容器，则从Abstract继承要比直接实现相应容器接口要方便的多，因为它实现了很多Iterator逻辑，减少了我们的编码量。

# 泛型问题

使用泛型将运行期错误可以提前到编译器错误

## 泛型数组

（🔵泛型数组可能不是经常遇到，你可以选择不去看）

- 不能实例化具有参数化类型的数组，数组必须知道他持有的确切类型。如下非法示例：

  ``` java
  Peel<Banana> peels = new Peel<Banana>[10]; // 注意这里：非法！
  ```

- 但是可以参数化数组本身的类型，如下示例：

  ``` java
  public static <T> T[] f(T[] arg){return arg;} // 注意这里: 可以在这里写更多的数组操作逻辑
  public static void main(String[] args){
      ...;
      Integer[] ints = {1,2,3,4,5,6,7};
      Integer[] result = f(ints); // 注意这里
      ...;
  }
  ```

  这里的泛型是 `T[] array`；上面非法示例是: `Peel<T> array`

- 可以创建非泛型数组，然后将其转型

  ``` java
  List<String>[] ls;
  List[] la = new List[10];
  ls = (List<String>[])la; // "uncheck" warning
  ```

- 一种情形：泛型在类或方法边界处有效，但在类或方法内部，擦除使得泛型不适用，从而不能创建泛型数组（即参数化数组本身的类型）

  ``` java
  public class Example<T>{
      T[] array; // 注意这里,Ok,合法
      public Example(int size){
          // array = new T[size]; // 注意这里：非法
          array = (T[])new Object[size]; // "uncheck" warning
      }
  }
  ```

- 关于泛型数组，可以看一下后面的`Arrays.asList`的一条。`toArray`方法也和这个泛型问题类似

## 泛型容器

- 当对下面的apples**使用get()时，取出的不是我们认为的Apple，而是Object引用，必须转型为Apple**

  ``` java
  ArrayList apples = new ArrayList();
  apples.add(new Apple());
  apples.get(); // Object
  (Apple)apples.get(); // Apple
  ```

- 向上转型适用于泛型容器

# 面向接口编程

> 我们编写的大部分代码都是在**与接口打交道**，尽管并非总是这样，但大部分情况下是如此的

**使用通用的接口，例如List、Set、Queue、Map**：

- 使用如下这句话，ArrayList被**向上转型为List**，使用接口的目的在于决定**修改实现**的时候，只需要在创建处修改。如下所示将ArrayList的实现转换为LinkedList的实现

  ``` java
  List<Apple> apples = new ArrayList<Apple>();
  List<Apple> apples = new LinkedList<Apple>();
  ```

- 如果需要创建一个具体类的对象，将其转型成为接口，在其他地方使用接口。这样做不会损失什么，但可能是一个良好的`编程习惯`的养成

- 如果需要使用某些类的额外功能，那么就不能向上转型为通用的接口。

  - 如`LinkedList`相关函数

# Iterable迭代器

![](E:\_data\博文临时库\博文中的图片\drawio\Java容器类库-迭代器.png)

<u>*todo: 这部分需要解和link: 迭代器模式来进行学习*</u>

迭代器使得我们**不用针对容器的确切类型来编程**，就像上文所说的AbstractCollection是用迭代器来实现的。迭代器能够将遍历的操作与底层数据结构分离，所以我们也说迭代器**统一了对容器的访问方式**。

使用迭代器不仅可以方便的把List实现替换ArrayList为LinkedList，而且可以方便的替换List为Set或为`Map.EntrySet`，这是迭代器的功劳。

- 把List实现替换ArrayList为LinkedList：可以仅仅**面对list接口编程**，而可以不用面对迭代器接口编程（换做set、queue等同理）
- **面对迭代器接口编程**：list甚至set和map都可以提供一个统一的视图

使用迭代器还有一个好处：这是将队列与消费队列的方法连接在一起的耦合度最小的方式。

虽然迭代器有着种种好处，但Java的各种奇怪的限制在迭代器这里也有一条：Iterator只能单向移动。

- 我们可以创建一个双向移动的迭代器
- `ListIterator`可以双向移动

## foreach

foreach可以迭代任何实现Iterable的类或者迭代数组，但是数组不是一个Iterable

Collection**接口**继承了**Iterable**，**Map没有继承Iterable**，并且**Map. Entry也没有继承Iterable**。继承Iterable是一个重要的特性，意味着实现Collection可以被*foreach*遍历，而Map和`Map.Entry`不可以被*foreach*遍历

- entrySet方法可以返回set对象，set实现了Iterable接口进而可以使用Iterable

## Fail-Fast & Fail-Save

> 在系统设计中，快速失效系统一种可以立即报告任何可能表明故障的情况的系统。快速失效系统通常设计用于停止正常操作，而不是试图继续可能存在缺陷的过程。这种设计通常会在操作中的多个点检查系统的状态，因此可以及早检测到任何故障。快速失败模块的职责是检测错误，然后让系统的下一个最高级别处理错误。
>
> 其实，这是一种理念，就是在做系统设计的时候先考虑异常情况，一旦发生异常，直接停止并上报。
>
> <span style="float:right">------From Wikipedia</span>

### fail-fast

Java容器类类库釆用快速报错（fail-fast）机制。通常，当提起Java中的fail-fast机制，默认指的是Java集合的一种错误检测机制。当多个线程对部分集合进行结构上的改变的操作时，有可能会产生fail-fast机制：它会探查容器上的任何除了你的进程所进行的操作以外的所有变化，一旦它发现其他进程修改了容器，就会立刻抛出 `ConcurrentModification Exception`异常。

- “快速报错”的意思是：**不是使用复杂的算法在事后来检查问题**。

- 原理：这个机制用一个计数变量`modCount`来记录对容器的修改，**一旦有其他进程修改，则`modCount`的值会和期望值`expectedModCount`不一致**

  ``` java
  if (modCount != expectedModCount)
      throw new ConcurrentModificationException();
  ```

- Java的fail-fast机制**主要目的是识别并发**，然后通过异常的方式进行通知

- 但是这种异常**并不是只在多线程环境下抛出**，也就是本小节讲的集合类中很多情况也会抛出这种并发有关的异常。

🔴**原理**：

`expectedModCount` 一般是集合实现中Iterator内部类实现中的成员变量（例如ArrayList中的`Itr`）。`expectedModCount`表示这个迭代器预期该集合被修改的次数。其值随着迭代器被创建而初始化。只有通过迭代器对集合进行操作，该值才会改变。

`modCount` 用来记录 ArrayList 结构发生变化的次数，**结构发生变化是指添加或者删除至少一个元素的所有操作**，**或者是调整内部数组的大小**，**仅仅只是设置元素的值不算结构发生变化**。

🔴**fail-fast产生场景**：

1. foreach循环对集合进行remove或add

   在增强for循环中，集合**遍历是通过iterator进行的**，**但是元素的add/remove却是直接使用的集合类自己的方法**，就会触发fail-fast机制，进而抛出`CMException`

   ``` java
   List<String> userNames = new ArrayList<String>() {{
       add("Hollis");
       add("hollis");
       add("HollisChuang");
       add("H");
   }};
   
   for (String userName : userNames) {
       if (userName.equals("Hollis")) {
           userNames.remove(userName);//ConcurrentModificationException
       }
   }
   ```

   ArrayList的`fastRemove`方法只修改了`modCount`，并没有对`expectedModCount`做任何操作。因此触发了异常。

   在增强for循环中，集合遍历是通过iterator进行的，但是元素的add/remove却是直接使用的集合类自己的方法。这就导致iterator在遍历的时候，会发现有一个元素在自己不知不觉的情况下就被删除/添加了，就会抛出一个异常，用来提示可能发生了并发修改。

2. 序列化需检查`modCount`

   在进行序列化或者迭代等操作时，需要比较操作前后 `modCount` 是否改变，如果改变了需要抛出 `ConcurrentModificationException`

3. 并发操作需检查`modCount`

### fail-safe

为了避免触发fail-fast机制，导致异常，我们可以使用Java中提供的一些采用了fail-safe机制的集合类。`java.util.concurrent`包下的容器都是fail-safe的，可以在多线程下并发使用，并发修改。同时也可以在foreach中进行add/remove 。

例如`CopyOnWriteArrayList`就是这样的容器，但是虽然基于拷贝内容的优点是避免了`CMException`，但同样地，迭代器并不能访问到修改后的内容

# 持有的对象对容器影响

1. **equals**：

   进行很多操作的时候都会采用**equals方法**，**必须意识到容器的行为根据持有的对象的equals行为而有所变化！**（在许多次实践吃尽了苦头）对于其他用到equals情况也应当注意。

   - 上述的所说的很多操作例如：确定元素是否属于某个List、发现某个元素的索引、从List中移除一个元素、`retainAll`、`removeAll`等等等

2. **存放的是引用**：容器存放的都是对象的引用，所以容器**不可以使用基本类型**。但是容器可以使用基本类型的包装类，自动包装机制使得容器可以和数组一样方便的使用基本类型。

# 取舍和选择

1. **`栈`**：如果需要栈的行为，就用LinkedList创建一个封装后的类（只暴露一些方法，或者改成更有意义的名字）

2. **`Bitset`**：如果想要高效率地存储大量“开/关”信息， `Bitset`是很好的选择。不过它的效率仅是对空间而言；如果需要高效的访问时间， `BitSet`比本地数组稍慢一点。
   此外， `Bitset`的最小容量是long：64位。如果存储的内容比较小，例如8位，那么 `Bitset`就浪费了一些空间。因此如果空间对你很重要，最好撰写自己的类，或者直接采用数组来存储你的标志信息（只有在创建包含开关信息列表的大量对象，并且促使你做出决定的依据仅仅是性能和其他度量因素时，才属于这种情况。如果你做出这个决定只是因为你认为某些对象太大了，那么你最终会产生不需要的复杂性，并会浪费掉大量的时间

   - `EnumSet`与 `Bitset`：

     如果拥有一个可以命名的固定的标志集合，那么 `EnumSet`与 `Bitset`相比通常是一种更好的选择，因为 `EnumSet`允许你按照名字而不是数字位的位置进行操作，因此可以减少错误。 `EnumSet`还可以防止你因不注意而添加新的标志位置，这种行为能够引发严重的、难以发现的缺陷。你应该使用 `Bitset`而不是 `EnumSet`的理由只包括：只有在运行时才知道需要多少个标志；对标志命名不合理；需要 `Bitset`中的某种特殊操作（查看`Bitset`和 `EnumSet`的JDK文档）。

# 参考

> 1. Java编程思想第四版 第十一、十六、十七章