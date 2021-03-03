---
title: 
description: 
tags:
- 
create_time: 
---

# static

static相关的内容，多在类加载时执行，因此会提高性能

在Jdk8之前，static变量存储在`PermGen`中，即方法区中。从Jdk8开始，方法去被移除，元空间仅存储类元数据，即static现在存在堆中。

[StackOverflow: Where are static methods and static variables stored in Java?](https://stackoverflow.com/questions/8387989/where-are-static-methods-and-static-variables-stored-in-java)

[StackOverflow: static allocation in java - heap, stack and permanent generation](https://stackoverflow.com/questions/3849634/static-allocation-in-java-heap-stack-and-permanent-generation/3849819#3849819)

## 可应用场所

按照应用的场所不同，有以下几种情况，其中以前三种最为常用。（静态导包虽然可以简化代码但是会破坏可读性，所以不是常用）

- 静态变量
- 静态方法
- 静态语句块
- 静态内部类
- 静态导包

**1. 静态变量**

- 静态变量/类变量：
  - 变量是属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它
  - 静态变量在内存中只存在一份
- 实例变量：
  - 每创建一个实例就会产生一个实例变量，它与该实例同生共死

**2. 静态方法**

静态方法：

- 在**类加载**时就存在了，它不依赖于任何实例
- 静态方法**必须有实现**，它不能是抽象方法
- 只能访问所属类的静态字段和静态方法，方法中**不能用 this 和 super** 关键字，因为这两个关键字与具体对象关联。

**3. 静态语句块**

静态语句块：

- 在类初始化时运行一次。

```java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
123
```

**4. 静态内部类**

静态内部类：

- 不能访问外部类的非静态的变量和方法

**非**静态内部类：

- 必须现有外部类实例，才能用这个实例去创建非静态内部类

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

**5. 静态导包**

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低

```java
import static com.xxx.ClassName.*
```

## 何时使用静态方法

**设计考虑**：

- 当您知道某些事不会在实例之间改变时，“静态”通常很有价值。如果是这样的话，我觉得应该考虑“单一职责”，这意味着一个类应只有一个责任，因此只有一个改变的理由。我觉得应该考虑将 ` ConvertMpgToKpl（double mpg）` 功能和类似方法移到自己的类中。汽车对象的目的是允许汽车实例化，而不是提供它们之间的比较。这些应该在类外部。

  上面是一个关于是否在一个对象中使用static方法的观点，我同意这个观点，并在这里列出。

- 静态方法的基本问题是他是过程性代码，和面向对象理念有所冲突

**仅在以下情况下定义静态方法**：

1. 如果您正在编写实用程序类，则不应更改它们。
2. 如果该方法未使用任何实例变量。
3. 如果有任何操作不依赖于实例创建。
4. 如果某些代码可以被所有实例方法轻松共享，则将该代码提取到静态方法中。
5. 如果您确定该方法的定义将永远不会更改或覆盖。由于静态方法不能被覆盖。

事实上不少IDE也会依据上面的情况来建议把一些方法更改为static方法（对，我说的就是IDEA）

**一些其他理由**：

- 性能：如果要运行一些代码，并且不想实例化一个额外的对象来这样做，请将其放入静态方法中。
  - JVM可以对静态方法优化很多
- 实用类/Util类
- 某种测试性的需要，（我个人认为static方法是可以测试的）



1. [Java: when to use static methods](https://stackoverflow.com/questions/2671496/java-when-to-use-static-methods)

## 静态语句块的执行时机

（类加载的初始化阶段）（这个问题综合考虑了类加载和类实例化过程）

**🔴先静态后普通、先父后子、构造函数在最后**

- **静态先于其他**：静态变量和静态语句块优先于实例变量和普通语句块
- 遵守在代码中的顺序：静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序
- 最后是构造函数的初始化

```java
public static String staticField = "静态变量";
static {
    System.out.println("静态语句块");
}
public String field = "实例变量";
{
    System.out.println("普通语句块");
}
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

- 🔵**存在继承**的情况下，初始化顺序为：
  - 父类（静态变量、静态语句块）`<clinit>`
  - 子类（静态变量、静态语句块）`<clinit>`
  - 父类 `<init>` 方法
    - 父类（实例变量、普通语句块）
    - 父类（构造函数）
  - 子类 `<init>` 方法
    - 子类（实例变量、普通语句块）
    - 子类（构造函数）

## 多线程环境下

这个环境下令人迷惑的地方主要为：

- static变量在多线程中使用
- static方法在多线程中使用
- 静态内部类内存溢出

**static变量和方法**：

首先给定一些总结：

- static变量：线程非安全
- static方法：
  - 使用实例变量/类变量/static变量：线程非安全
  - 否则/仅操作局部变量和形参：线程安全

看下面一个例子：

```java
public class DateUtils {
    public static Date getNormalizeDate(Date date) {
        // some operations
    }   
}
```

在这个例子中date是一个可变对象（意为date内部的值可能被他人修改），由于他是可变对象，所以我们需要确保其他线程不在别的地方去修改它。但是这和该方法的线程安全无关，它完全是另一回事。

这个方法本身是线程安全的：因为这个方法不维护状态（不操作任何实例变量/类变量）、**仅操作自己的参数（局部变量和形参）**。

如果说他不安全，这个说法是错误的。只能说调用他的方法不是线程安全的。例如，“日期不是线程安全的”，如果调用代码读取了另一个线程写入的日期，则必须在“日期”写入和读取代码中使用适当的同步。

**静态内部类**：

在多线程场景下，非静态内部类可能会引发内存溢出的问题。下面这个例子中，thread持有着外部类的引用，可能导致外部类的实例迟迟不会被回收，如果`AExample`创建了很多实例，会导致内存溢出

``` java
public class AExample {
    public void functionA() {
        // 匿名内部类, 会持有外部实例的引用
        new Thread() {
            public void run() {
                //...耗时操作
            };
        }.start();   
    }
}
```

所以有人认为应该像下面这样写：

``` java
public class AExample {
    public void functionA() {
        InnerClass innerClass = new InnerClass();
        innerClass.start();
    }
    static class InnerClass extends Thread {
         @Override
         public void run() {
             //...耗时操作   
         }
    }
}
```

我认为这样写仍然会有溢出问题，应该加一个线程池，或者所有外部类都用一些公用的`InnerClass`线程 ? 但其实这么想是不对的。因为这个想法忘记了`static`的修饰。因为静态类是定义一次性的。所以这样写确实有好处。（这是我的想法）

## 争议性问题

> Java充满了奇怪的局限性

这些问题含有争议：

- 为什么不允许**覆盖**静态方法
- 为什么静态方法不能是**抽象**的

**不允许覆盖静态方法**：

这些争议通常是在设计上的，编程思想上的，OO上的：

静态方法被JVM视为全局方法，根本没有绑定到对象实例。

**覆盖**取决于拥有类的实例。**多态性**的关键是可以对一个类进行子类化，并且实现那些子类的对象对于在超类中定义的**相同方法将具有不同的行为**（在子类中被重写）。静态方法未与类的任何实例关联，因此该概念不适用。

不能覆盖和不能是抽象的好像在一定程度上可以提高性能？

很多人认为这是Java设计中的缺陷，在这个问题中 [StackOverflow : Why doesn't Java allow overriding of static methods?](https://stackoverflow.com/questions/2223386/why-doesnt-java-allow-overriding-of-static-methods)有一个`BigDecimal` 的示例，感觉有点道理（虽然我觉得这个例子的设计也有问题）。

在上面这个链接中也有人指出了利用反射达到覆盖的目的，but，我想说这样做很“作弊”

**静态方法不能是抽象的**：

“抽象”的意思是：“不实现任何功能”，而“静态”的意思是：“即使没有对象实例也有功能”。这是一个逻辑上的矛盾

同上面一样很多人也认为这是Java设计中的缺陷🙃

1. [StackOverflow : Why can't static methods be abstract in Java?](https://stackoverflow.com/questions/370962/why-cant-static-methods-be-abstract-in-java)



一些其他资料：

1. [StackOverflow : Difference between Static methods and Instance methods](https://stackoverflow.com/questions/11993077/difference-between-static-methods-and-instance-methods)

2. [StackOverflow : Static Classes In Java](https://stackoverflow.com/questions/7486012/static-classes-in-java)

# final

可应用场所：

1. 变量
2. 方法
3. 类

**1.变量**

声明为**常量**，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使**数值**不变；
- 对于引用类型，final 使**引用**不变，也就不能引用其它对象，**但是被引用的对象本身是可以修改的**

```
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

**2. 方法**

声明方法**不能被子类重写**。

- private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。
- final修饰的方法可以被重载 但不能被重写

**3. 类**

声明类不允许被继承



final和static的区别：

下面这个类中的`i`在每个实例都可能不一样，而`j`必定一样。final是用来保证变量不可变，static是来保证类变量

```java
class MyClass {
  public final double i = Math.random();
  public static double j = Math.random();
}
```



匿名内部类中使用的外部局部变量只能是final变量：

为编译器编译的时候其实悄悄对函数做了手脚，偷偷把外部环境方法的x和y局部变量，拷贝了一份到匿名内部类里

https://www.zhihu.com/question/21395848

``` java
public AnnoInner getAnnoInner(final int x){ 
    final int y=100; 
    return new AnnoInner(){ 
        int z=100; 
        public int addXYZ(){
            return x+y+z;
        } 
        //public void changeY(){y+=1;} //这个函数无法修改外部环境中的自由变量y。 
    };
}

// 编译器实际上相当于做了这样的一个工作：
public AnnoInner getAnnoInner(final int x){ 
    final int y=100; 
    return new AnnoInner(){ 
        int copyX=x; // 注意这里
        int copyY=y; // 注意这里
        int z=100; 
        public int addXYZ(){
            return x+y+z;
        } 
        //public void changeY(){y+=1;} //这个函数无法修改外部环境中的自由变量y。 
    };
}

```

