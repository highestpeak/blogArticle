---
title: Java专题-对象复制/克隆/拷贝
date: 2021-1-14 23:05:00
status: PUBLISHED
comments: true
description: 描述Java过去、现在和未来，从不同角度阐述Java的知识点要点
tags: 
- JAVA
categories: 
- 面试系列
---

拷贝问题，并不是很简单：

**为什么需要克隆对象？直接new一个对象不行吗？**

克隆的对象可能包含一些已经修改过的属性，而new出来的对象的属性都还是初始化时候的值，所以当需要一个新的对象来保存当前对象的“状态”就靠clone方法了。那么我把这个对象的临时属性一个一个的赋值给我新new的对象不也行嘛？可以是可以，但是一来麻烦不说，二来，大家通过上面的源码都发现了clone是一个native方法，就是快啊，在底层实现的。

**什么时候才需要拷贝？**

---

**那拷贝就行了，有什么问题呢？**

假如说你想复制一个简单变量。很简单：

```
int apples = 5;  
int pears = apples;  
```

不仅仅是int类型，其它七种原始数据类型(boolean,char,byte,short,float,double.long)同样适用于该类情况。

但是如果你复制的是一个对象，情况就有些复杂了。

由于对象中含有字段，字段可能是类而不是基本类型，所以涉及到一系列的递归拷贝的问题。这些字段需不需要复制？这是一个问题

这里画上一个两个对象含有相同字段引用的图

由此引入**深拷贝和浅拷贝问题**

---

不同的拷贝方法的操作流程和优缺点

类库帮助：类似lomobok的生成器

implements Cloneable

- 应该避免使用`clone()`和`Cloneable`，因为很难正确实现它，并且很可能会破坏某些内容，尽管不能立即看到。

  https://www.artima.com/intv/bloch.html#part13

- Cloneable正确地进行克隆非常困难，而且付出的努力是不值得的。

- https://stackoverflow.com/a/4081982

- https://stackoverflow.com/a/4081933

- https://stackoverflow.com/questions/4081858/about-java-cloneable/4081895#4081895

序列化成xml json....再从新生成自己需要的对象

- 一种选择是使用序列化。它不是最快的一种，但可能是最安全的一种。

含有相同类型的构造器（这也可以使用lomok生成）

除了显式复制外，另一种方法是使对象不可变（noset或其他mutator方法）。这样就永远不会出现问题。

使用Reflection API

类库帮助：

- apache.commons.lang.SerializationUtils; BeanUtils.clone(..)

  `BeanUtils`方法基于反射，因此（相对）较慢。那么，**推土机**和其他类似的工具。复制构造函数是完成任务的最快方法，

  BeanUtils可以在不同类之间进行属性的复制。Person和PersonDTO 如果字段一致的话，可以使用BeanUtils。话说，Java和C++这种把方法和属性放在一起的设计，真心不如go，js，rust这种语言，dto这种东西根本没有存在的必要--来自掘金 正确性存疑

- 使用`gson`用于复制的对象

https://stackoverflow.com/questions/869033/how-do-i-copy-an-object-in-java

https://en.wikipedia.org/wiki/Object_copying

https://howtodoinjava.com/java/cloning/a-guide-to-object-cloning-in-java/

# 深拷贝与浅拷贝

<u>*todo: 这一小节部分需要结合文章 **JAVA实例创建和参数传递** 来理解更好。*</u> 其实拷贝也是执行赋值运算，区别就是基本类型直接赋值的是数值，对象直接赋值的是地址引用。

并不是说所有的时候都要用深拷贝：（具体取决于实际设计情景、实际需求）

深拷贝相比于浅拷贝速度较慢而且花销大

如果对象的属性全是基本类型的，那么可以使用浅拷贝，但是如果对象有引用属性，那就要基于具体的需求来选择浅拷贝还是深拷贝。我的意思是如果对象引用任何时候都不会被改变，那么没必要使用深拷贝，只需要使用浅拷贝就行了。如果对象引用经常改变，那么就要使用深拷贝。



https://blog.csdn.net/justloveyou_/article/details/60983034

# clone() & Cloneable  

- 在派生类中覆盖基类的clone()方法，并声明为public。

- 在派生类的clone()方法中，**调用`super.clone()`**。

- 在派生类中**实现Cloneable接口**。



<u>*todo: 编写习惯，编写常识套路*</u>

xxx



# Serializable 序列化对象

存储到文件，（编程思想IO一章）

xxx

类序列化时类的版本号的用途，如果没有指定一个版本号，系统是怎么处理的？如果加了字段会怎么样？



Serializable接口中为什么需要定义serialVersionUID变量？

序列化操作的时候系统会把当前类的serialVersionUID写入到序列化文件中，当反序列化时系统会去检测文件中的serialVersionUID，判断它是否与当前类的serialVersionUID一致，如果一致就说明序列化类的版本与当前类版本是一样的，可以反序列化成功，否则失败。

serialVersionUID有两种显示的生成方式： 
一是默认的1L，比如：private static final long serialVersionUID = 1L; 
二是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，



序列化时不想让某些属性参加序列化,transient

# xml持久化

可以在网络上传输？（编程思想IO一章）

xxx



# 容器拷贝

`Arrays.copyOf`

集合的拷贝

