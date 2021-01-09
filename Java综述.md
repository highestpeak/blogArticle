---
title: Java综述
date: 2021-1-9 19:26:00
status: PUBLISHED
comments: true
description: 描述Java过去、现在和未来，详细的从各个角度对"跨平台、面型对象、编译型和解释型语言、Java体系结构"等进行阐述，总结Java值得学习的理由。总结Java学习的内容，并列出一些类库
tags: 
- JAVA
- 编程语言
categories: 
- 面试系列
- 综述
---

> "唯一比一个糟糕的程序员更糟糕的是，一个糟糕的程序员认为他是一个好的程序员。"

# Java 简介

> “Java 编程语言是个简单、面向对象、分布式、解释性、健壮、安全与系统无关、可移植、高性能、多线程和动态的语言” -- Sun 公司

Oak 是 Java 的最初雏形，最初的定位是"家用电器等小型系统的编程语言"，但是由于90年代智能家电没有市场而失败。在90年代之后的时间中，Sun 公司看到了互联网的前景，改造了 Oak，即为 Java。伴随着互联网的迅猛发展而发展，Java 逐渐成为重要的网络编程语言。（时运好）

Java 是一种广泛使用的编程语言，它是具有跨平台、面向对象、多重编程范型的特性的编程语言。Java 在移动应用开发、web 应用开发、大数据等众多领域应用广泛。

- 多重编程范型：面向对象（类别基础）、结构化、命令式、泛型、反射、并发计算
- 从多重范型和面向对象的设计可以看出：Java 可以轻易的实现许许多多的设计模式

Java 受到了很多编程语言的启发: Ada 83、C++、C#、Objective-C 等。同时他也影响了很多语言: Ada 2005、c#、ECMAScript、Groovy、JavaScript、Kotlin、Scala、python 等。

- 更多请查看原页面： [Wikipedia-Java](https://zh.wikipedia.org/wiki/Java)
- 了解这些语言的关联也有助于学习 Java

Java 主要提供三个不同的版本，他们也代表了不同的学习内容。对于我们学习来说，主要是 SE 和 EE ，SE 是最基础内容，而 EE 视根据研究和应用方向来学习。

- Java Platform, Enterprise Edition（Java EE: Java平台企业版）
- Java Platform, Standard Edition（Java SE: Java平台标准版）
- Java Platform, Micro Edition（Java ME: Java平台微型版）

Java 诞生于1990年代初，在如此长的时间内历久弥新，有着强大的持久力。在 Java 发展过程中，JSR规范、JCP组织 的作用尤为重要，他们保证了 Java 保持一定的活力，不断增加新的特性，同时保证语言规范性，也避免了垄断（虽然也带来了臃肿）。时至今日， Java 仍是教育中经常使用的一种语言（如今Python正在取代这种语言）。

## 跨平台

跨平台重要吗？重要！因为可以 write once，run anywhere，因为这样可以减少开发量，这对于个人可以把自己开发的软件快速应用到各类型的设备（mac、win pc、linux）。对于企业来说可以减少开发人员，提升开发速度，降低开发成本。

Java 的跨平台特性与Java虚拟机的存在密不可分，"到处运行" 的关键和前提就是JVM。各种硬件平台上的Java虚拟机就各不相同了。

在早期JVM中，这在一定程度上降低了Java程序的运行效率。但在 J2SE1.4.2 发布后，Java 的运行速度有了大幅提升。

Java 平台通过虚拟机屏蔽了操作系统的底层细节，使得开发者无需过多地关心不同操作系统之间的差异性。c/c++编程是面向操作系统的，需要开发者极大地关心不同操作系统之间的差异性；通过增加一个间接的中间层来进行”解耦“是计算机领域非常常用的一种”艺术手法“，虚拟机是这样，

## 面向对象

Java 似乎被设计为鼓励和驱动尽可能多的OOP。尽管 Java 在面向对象上面做的没有 Ruby 和 Smalltalk 纯粹，但是最新出现的用 Java 实现的语言 Groovy 解决了这些问题。

Java希望面向对象，但实际上它是面向类的

<u>*todo：*</u>

## 编译和解释

Java 即是编译型语言也是解释型语言，但这不同于一般的编译语言或解释型语言。它首先将源代码编译成字节码，再依赖各种不同平台上的虚拟机来解释执行字节码，从而具有“一次编写，到处运行”的跨平台特性。

1. Java 源代码经过 `Javac` 编译成 类文件（.class 文件）

2. 类文件经 JVM 解析或编译运行

   - 解析: 类文件经过JVM内嵌的解析器解析执行

      Java 字节码文件首先被加载到计算机内存中，然后读出一条指令，翻译一条指令，执行一条指令，该过程被称为java语言的解释执行，是由java虚拟机完成的

   - 编译: 存在JIT编译器（Just In Time Compile 即时编译器）把经常运行的代码作为"热点代码"编译与本地平台相关的高性能的本地机器码，并进行各种层次的优化

其他问题和说法：

- 注意：存在两个编译的地方

- Java 是解析运行吗？不正确！

- AOT编译器: Java 9提供的直接将所有代码编译成机器码执行

- 写个程序直接执行字节码就是解释执行。写个程序运行时把字节码动态翻译成机器码就是 JIT 。写个程序把java源代码直接翻译为机器码就是 AOT。造个CPU直接执行字节码，字节码就是机器码

  - JIT 是运行时才做的，需要预热才知道哪些是热点
  - AOT 是编译期，静态的，直接编成类似类库的东西

- 所有的类运行时解析之后，再次运行时还会重复解析吗？如果没有被 JIT 编译，是的

- 只使用解释器的缺点：

  抛弃了JIT可能带来的性能优势。如代码没被JIT编译的话，再次运行时需要重复解析

- 只用 JIT 的缺点：

  - 需要将全部的代码编译成本地机器码。要花更多的时间，JVM启动会变慢非常多；
  - 增加可执行代码的长度（字节码比JIT编译后的机器码小很多），这将导致页面调度，从而降低程序的速度。

## 体系结构

了解一个语言，我觉得了解他的体系结构是很重要的，这样我们能知道他是脚本型的还是编译型的，知道了这个可以让我们在面对不同问题时进行取舍有些依据。（这不仅仅是为了面试的需要、或者为了公司的业务、对于我们个人项目的语言取舍也是很有参考的）

我们经常听到Java的多个术语：JVM、JRE、JDK。他们是什么关系呢？（其实很简单）首先看全称：

- JVM（Java Virtual Machine）
- JRE（Java Runtime Environment）
- JDK（Java Development Kit）

从名字上来看就能一猜一二，程序是需要运行在虚拟机上的，运行环境是依赖于虚拟机的，开发的代码肯定需要编译才能运行，开发环境肯定需要有运行环境才能进行一系列的debug测试。所以包含关系就呼之欲出。

上面只是Java环境的架构

![](E:\_data\博文临时库\博文中的图片\drawio\Java架构图.png)

一些问题/注意：

- java11开始，Oracle和OpenJDK不再有JRE这个单独的文件夹，即 JRE 退出历史舞台
  - <u>*todo： Oracle JDK 和 OpenJDK*</u>
- 如果想要运行一个开发好的Java程序，计算机中只需要安装JRE即可
- Java程序需要运行在虚拟机上，不同的平台有不同的虚拟机，因此Java语言可以实现跨平台，相同平台的虚拟机可以有不同组织的不同的实现

# Java 值得学习嘛

Java值得学习吗？ 答案是明确的"是的"。尽管肯定有人提出反对，Java 绝对可以说是有史以来创建的最好的编程语言之一。让我们学习 Java 的理由有很多：

- **生命力强大**，并有着**继续保持**这样生命力的力量： 

  - 许多有抱负的软件开发人员都想学习最新，最热门的技术。但是一个语言的生命力也同样重要，Java 就具有这样的持久力

  - 只要面向对象还有着自己的地位，Java 就有着自己的位置

  - Java 是 Scala、kotlin、jpython、jRuby 等一众编程语言的父亲，只要这些语言不死，Java 是不需要担心消亡的（总有垫背的）

  - 使用量大，语言生命力就旺盛（比如 JS）

  - Java 有着很好开放性，而这样的开放性保证了生命力

    Java 的标准是[JCP](https://www.jcp.org/en/home/index)定出来的，而 JCP 不是 oracle 的一家之言。这意味着 Java 免费，开源，有足够多的企业在制约这个标准。

- **应用广泛**：

  - Java 广泛用于物联网和API，大数据技术，电子商务网站，高频金融交易平台和科学应用程序中
  - Java 为 Android 提供支持，Android 是地球上使用最广泛的操作系统
  - 就**工作机会**而言，Java 再一次超越所有人

- 基本**完备的**语言：

  - 最大的用户群体，最广泛的社区和厂商，实际开发中语言酷不见得是足够的，很多硬性的能力才是决定因素

- Java 是一种**面向对象**的编程语言，Java 是为数不多的100％OOP编程语言之一

  - 学习设计模式，软件工程的重要语言
  - 是开发大型软件的上佳选择，因为面向对象编程使分工和重用更方便和有条理。

- **JVM**：

  - **平台无关**：这使得我们可以写一个软件，在很多地方使用，这样的话，可以用一个技能做更多的事
  - **关注问题本身**：程序员可以把精力集中在问题本身，JVM 会帮你关注琐碎的底层细节问题。不用时刻担心要写内存分配等等。

- **初学者友好的**

  - 具有丰富的 API ，丰富的类库、开源库，丰富的教程、文档、构建指南（新出现的语言，不活跃的语言就没有这样的好处了），强大的社区支持等等。这些使得 Java 适合普罗大众写出质量可靠的应用
  - 很多使用 Java 程序较少使用语法糖也是它易于学习的原因
  - Java 有着众多强大的开发工具，这样你就不用再费劲去学习一些构建和开发工具
  - 难道它难，咱们就不学了嘛

- 免费的🙃

> 一个有趣的视频

Java的使用如此广泛，以至于创建了一个视频恶作剧，描述了[没有Java的世界](https://www.youtube.com/watch?v=E3418SeWZfQ)的末日场景。这有点愚蠢，但是可以说明Java对我们世界的影响的深度和程度。

<iframe width="560" height="315" src="https://www.youtube.com/embed/E3418SeWZfQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

> 最后
>
> - 但是学习多种语言也是如此。如果您是只能使用一种语言的程序员，那么您不是一个好的程序员，而您永远不会。
> - 请记住：我们不应该只学习一门编程语言，多编程语言编程是有必要的。

## 和C++的比较

Java 的强大"对手" C++，也是需要关注的🙃

- Java 比 C++ 好学，容易上手。但是反过来说一个工具易学称手难道不是优点嘛？

- Java 是纯粹的面向对象语言，所有的对象都继承自 `java.lang.Object`，C++ 为了兼容 C 即支持面向对象也支持面向过程。

- Java 通过虚拟机从而实现跨平台特性，但是 C++ 依赖于特定的平台。

- Java 没有指针，它的引用可以理解为安全指针，而 C++ 具有和 C 一样的指针。

- Java 支持自动垃圾回收，而 C++ 需要手动回收。

  - Java 的垃圾收集能力已经足够了

  - > 为什么有些用C/C++的猿总喜欢用 “我每次拉完屎都有记得擦屁股呦~” 这样的理由来讥诮Java呢？自动冲水不是也很酷吗？有很多人擦不干净呢。而且还不知道是哪里没擦干净? - 知乎某网友

- Java 不支持多重继承，只能通过实现多个接口来达到相同目的，而 C++ 支持多重继承。

- Java 不支持操作符重载，虽然可以对两个 String 对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而 C++ 可以。

- Java 的 `goto` 是保留字，但是不可用，C++ 可以使用 `goto`

## 对Java的看法

> "一个工程师有问题。他决定用 Java 来解决它。现在他有了一个 `ProblemFactory`"
>
> ![](https://i.redd.it/ixv0uh2zzu151.jpg)

- 如果你要写一段简单的 **glue code**，Java就显得太过啰嗦了。这时候就要用 python 这一类的

- 如果从工业化，实用的角度讲，java无疑是一门十分给力的语言；但是从另外一个角度看，Java 就是笼子

- 用Java 编程就好比在肯德基做烹饪。但这也不是贬低 Java ，餐饮也真的需要肯德基这样的快餐模式，软件工程领域也同样需要 Java 这样的一站式语言

- 不利于数据分折和开发算法，因为要从零至使程序可运行须用大量时间和精力，对测试不利（故一般人会在Python、R或MATLAB做研究，到了决定之时便用Java编写软件）

  但是这时候可能你用 Java 用的多，就会觉得不方便

- <u>*todo： 至Java 1.7为止，Java语言不支持闭包（closure）和混入（mixin）特性*</u>

  - <u>*todo: 在当时，甚至可能是现在，普通的程序员都不知道什么是协方差，也不真正理解类型推断，泛型和闭包（pop测验-如果其超类和封闭类都具有一个名为foo的变量，该变量在Java内部类中引用的是foo吗？）（说的就是我）*</u>
  - <u>*todo: hygenic macros, a meta-object protocol, type inference, co/contra-variance, and so on - that were missed out of Java*</u>

- Java希望面向对象，但实际上它是面向类的

- 零经验的"Java程序员":  它是教育中经常使用的一种语言（幸运的是，如今Python正在取代这种语言），但这仅意味着我们拥有许多零经验的"Java程序员"

- JVM本质上本身就是一个操作系统。如果Java重新发明一切，那么一切都会如此美好（固执的向后兼容，长痛和短痛，时机选取）

- Java之所以令人讨厌是因为它取得了令人难以置信的成功。编程语言的成功意味着具有许多不同经验和不同要求的人们的广泛使用，至少其中一些人不喜欢任何给定的语言

- 设计模式和OOP的考量导致Java也显得臃肿

---

这个问题没有标准答案，但是有『好』答案

可以看一下下面几个问题的讨论，这几个问题下的很多答案提供了不同方面的思考：

- [Quora: 为什么有些程序员讨厌Java？](https://www.quora.com/Why-do-some-programmers-hate-Java)
- [Quora: 您如何看待Java？](https://www.quora.com/What-do-you-think-about-Java)

# Java 的学习内容

所以Java到底还包含什么呢？或者说我们需要学习Java的什么呢？下面的每个部分会从不同的角度阐述这些内容（个人观点）

Java平台中有两大核心：

1. Java 语言本身、JDK中所提供的核心类库和相关工具

2. Java 虚拟机以及其他包含的GC

   Java 编译器和虚拟机的不同对 Java 代码的性能影响比语言本身的影响大的多

## 语言本身

就语言本身来讲：首先学习 Java SE ，即掌握 Java 语言本身、Java 核心开发技术以及 Java 标准库的使用。然后可以选择学习Java EE、大数据开发、移动开发（Android）等，根据不同的方向选择不同的技术点学习。

- 从**基础语法**来看：

  每个语言都有他的基础语法，虽然大同小异，但又各不相同，这往往也导致了不同语言的风格不同。语法上来说，学习Java有一点需要注意，就是它的面向对象的特性，Java处处体现着面向对象的设计理念。这一部分我认为有：

  - <u>*基础语法、访问修饰、接口和抽象类的各种写法、内部类*</u>

- 从**类库**角度：

  我之前一个老师说过一句话，他说：“你熟悉一个编程语言，很大程度上也是是否熟悉这个语言的类库和框架”，个人觉得这是很正确的。

  当然，在适应类库的道路上，确实需要花一些时间，但是我们总会找到自己的路的，最终这将是值得的。

  - 基础类库：

    对于最基本的Java来说，这个类库我认为是：<u>*集合框架、并发类库、IO类库、异常机制*</u>。在1.8之后，这个类库我觉得需要包含 <u>*`lambda` 和 `function`*</u> 的操作

    - 在进阶和高级一点来说，这个列表还应当包含：<u>*枚举、泛型、注解和反射*</u>

  - Java EE：

    - 对于Java的扩展，即Java EE来说，我觉得这个列表最少要加入 <u>*Spring、数据库开发、分布式架构*</u>
    - <u>*todo: 其他中间件和类库暂且不表*</u>

  - Android：

    - <u>*todo：*</u>

  - 大数据：

    - Hadoop、Spark、Flink 这些大数据平台

- 从语言本身的很多**陷阱**或者说**特色**：这一部分我认为有：

  - <u>*Object作用和有什么、String的使用、包装类型*</u>

- 从**设计思想**来说：这一部分我认为有：

  - <u>*封装、继承、多态；设计模式；*</u>

这些是基础的知识，就是大家所说的万丈高楼平地起的地基（虽然对这种说法我也有些不耐烦）

### 软件迭代

<u>*todo： 未整理*</u>

由于软件是迭代的，他有很多个版本，所以也需要了解Java个版本的问题、特性，（当然了解最新最广泛使用的就可了，其他的版本的就作为补充吧）

<u>*todo：每个Java版本的重大迭代：*</u>

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

<u>*todo：把Android什么时候放进来也写进去*</u>

- 传统Java是解释型的语言，现在的JIT、AOT，让Java也支持了编译型语言的特性；
- 传统Java是面向对象的语言，JDK8引入Lambda，让Java也有了函数式编程的能力；
- 传统的Java是命令式编程范式，JDK9引入Flow，让Java更好的支持响应式编程范式；

<u>*todo：各个版本要写出发行时间*</u>

## JVM

要写出高效的Java代码，JVM肯定是绕不过的话题。JVM 很强大，在 JVM 上可以运行多种语言写的程序，并且可以和 Java 互操作，比如 Scala、groovy、kotlin 等

围绕虚拟机的效率问题展开，将涉及到一些优化技术，例如：JIT、AOT

- 如果虚拟机加载字节码后，完全进行解释执行，这势必会影响执行效率。所以，对于这个运行环节，虚拟机会进行一些优化处理，例如JIT技术，会将某些运行特别频繁的代码编译成机器码。而AOT技术，是在运行前，通过工具直接将字节码转换为机器码
- 为了学习 JIT 与 AOT 有可以去学习 KVM
- JVM（内存模型/生命周期）、数据结构、操作系统、网络等其他老生常谈的CS基础

为了更好使用 JVM，就需要学习Java相关的工具：各种编译、监控和诊断工具等 

## 前景/生态系统

（由于这个部分涉及到学习的方向之类的内容，故放到这个小节）（我们也需要了解这个语言的活力）

Java在现在（20200804）的主要的研究方向：

- 大数据：SPARK（JAVA在大数据上基本一枝独秀？）、Hadoop、Elasticsearch
- 科学计算
- Spring、中间件、服务器 Netty
- Android
- 嵌入式Java
- UI：JAVA FX
- JVM：
  - JVM实现、JAVA衍生语言（Scale、kotlin、Groovy）、函数式编程语言
  - 虚拟机、编译技术的研究(例如：GC优化、JIT、AOT等)：对效率的追求是人类的另一个天性之一
- Java 语言本身的优化
- 游戏：Minecraft
- 并发编程

### 常见类库/工具

<u>*todo： 继续补充*</u>

1. 工具类
   - 纯工具：Google Guava、Apache common（lang，IO，common）
   - 测试：junit，testng
   - 日志：slf4j + logback
   - 序列化：avro，protobuff
   - JSON处理：Jackson，Gson
   - HTTP：apache http compoent
2. 框架类
   - 应用开发框架：Spring + Spring Boot
   - REST API开发：Swagger Codegen + Swagger UI
   - 网络框架：Netty
   - ORM框架：Hibernate，MyBatis / iBatis
3. 工具
   - maven：构建和打包。
   - git：源代码版本控制
   - IDE
   - Jenkins：自动化持续集成

# 参考

> 1. Java编程思想第四版
> 2. [OnJava8](https://github.com/lingcoder/onJava8/)
> 3. [极客时间: Java核心技术面试精讲](https://time.geekbang.org/column/article/6845)
> 4. [知乎: 如何评价 Java 这门编程语言？](https://www.zhihu.com/question/22600749)
> 5. [知乎: PHP、Java、Python、C、C++ 这几种编程语言都各有什么特点或优点？](https://www.zhihu.com/question/25038841/answer/44396770)
> 6. [维基百科: Java](https://zh.wikipedia.org/wiki/Java)
> 7. [Quora: 为什么有些程序员讨厌Java？](https://www.quora.com/Why-do-some-programmers-hate-Java)
> 8. [Javarevisited: Where is Java used in Real World?](https://javarevisited.blogspot.com/2014/12/where-does-java-used-in-real-world.html#axzz6j28qtSMS)
