---
title: Java综述
description: 描述Java过去、现在和未来，从不同角度阐述Java，阐述如何学习和深入Java。
tags:
- JAVA
- 编程语言
create_time: 2020-8-4
---



在开始之前应当阅读以下几个部分，即[Java](https://zh.wikipedia.org/wiki/Java)和[Java EE](https://zh.wikipedia.org/zh-hans/Jakarta_EE)，在维基中的内容不会再次说明，文章仅做其他的补充性质的，讨论性质的内容。

所以除了维基中的向外行人介绍的说辞，Java到底还包含什么呢？或者说我们需要学习Java的什么呢？下面的每个部分可能会从不同的角度阐述这些内容

---



> 从基础语法来看：

每个语言都有他的基础语法，虽然大同小异，但又各不相同，这往往也导致了不同语言的风格不同

基础语法、访问修饰、接口和抽象类的各种写法、内部类。

> 从类库角度

我之前一个老师说过一句话，他说：“你熟悉一个编程语言，很大程度上也是是否熟悉这个语言的类库和框架”，个人觉得这是没问题的。在适应类库的道路上，确实需要花一些时间，但是总是会找到自己的路，最终这将是值得的。

对于最基本的Java来说，这个类库我认为是：集合框架、并发类库、IO类库、异常机制。在1.8之后，这个类库我觉得需要包含lambda和function的操作

- 在进阶和高级一点来说，这个列表还应当包含：枚举、泛型、注解和反射

对于Java的扩展，即Java EE来说，我觉得这个列表可以添加进去这么些东西：Spring。这些是基础的知识，就是你很厌烦的大家所说的万丈高楼平地起的地基

> 从语言本身的很多陷阱或者说特色

Object作用和有什么、String的使用、包装类型

> 从设计思想来说

封装、继承、多态；设计模式；

> 从原理来说

JVM（内存模型/生命周期）、数据结构、操作系统、网络等其他老生常谈的CS基础



---

了解一个语言，我觉得了解他的体系结构是很重要的，这样我们能知道他是脚本型的还是编译型的，知道了这个可以让我们在面对不同问题时进行取舍有些依据。（这不仅仅是为了面试的需要、或者为了公司的业务、对于我们个人项目的语言取舍也是很有参考的）

我们经常听到Java的多个术语：JVM、JRE、JDK。他们是什么关系呢？（其实很简单）首先看全称：

- JVM（Java Virtual Machine）
- JRE（Java Runtime Environment）
- JDK（Java Development Kit）

从名字上来看就能一猜一二，程序是需要运行在虚拟机上的，运行环境是依赖于虚拟机的，开发的代码肯定需要编译才能运行，开发环境肯定需要有运行环境才能进行一系列的debug测试。所以包含关系就呼之欲出。

上面只是Java环境的架构

![](E:\_data\博文临时库\博文中的图片\drawio\Java架构图.png)

（todo：图）

一些简单的问题：

- 如果想要运行一个开发好的Java程序，计算机中只需要安装JRE即可
- Java程序需要运行在虚拟机上，不同的平台有不同的虚拟机，因此Java语言可以实现跨平台，相同平台的虚拟机可以有不同组织的不同的实现
- Oracle JDK 和 OpenJDK

Java不同于一般的编译语言或解释型语言。它首先将源代码编译成字节码，再依赖各种不同平台上的虚拟机来解释执行字节码，从而具有“一次编写，到处运行”的跨平台特性。

---

<u>*todo: 对于基本的语法建议采用wiki形式，构建速查表*</u>

---

我们也需要了解这个语言的活力，目前状况和前景。

Java在现在（20200804）的主要的研究方向：

- 大数据：SPARK（JAVA在大数据上基本一枝独秀？）、Hadoop、Elasticsearch
- 科学计算
- Spring、中间件、服务器 Netty
- Android
- 嵌入式Java
- UI：JAVA FX
- JVM：JVM实现、JAVA衍生语言（Scale、kotlin、Groovy）、函数式编程语言
- 游戏：Minecraft

---

由于软件是迭代的，他有很多个版本，所以也需要了解Java个版本的问题、特性，（当然了解最新最广泛使用的就可了，其他的版本的就作为补充吧）

**New highlights in Java SE 8**

1. Lambda Expressions
2. Pipelines and Streams
3. Date and Time API
4. Default Methods
5. Type Annotations
6. Nashhorn JavaScript Engine
7. Concurrent Accumulators
8. Parallel operations
9. PermGen Error Removed

---

Java的强大对手C++，也是需要关注的🙃

Java 与 C++ 的区别

- Java 是纯粹的面向对象语言，所有的对象都继承自 `java.lang.Object`，C++ 为了兼容 C 即支持面向对象也支持面向过程。
- Java 通过虚拟机从而实现跨平台特性，但是 C++ 依赖于特定的平台。
- Java 没有指针，它的引用可以理解为安全指针，而 C++ 具有和 C 一样的指针。
- Java 支持自动垃圾回收，而 C++ 需要手动回收。
- Java 不支持多重继承，只能通过实现多个接口来达到相同目的，而 C++ 支持多重继承。
- Java 不支持操作符重载，虽然可以对两个 String 对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而 C++ 可以。
- Java 的 goto 是保留字，但是不可用，C++ 可以使用 goto

---

参考：

1. Java编程思想、[OnJava8](https://github.com/lingcoder/onJava8/)

