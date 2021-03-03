---
title: Java基础Wiki
description: 描述了Java的基础语法，含有一些速查表，作为wiki而不是文章提供一种字典式的参考
tags:
- JAVA
- 速查表
- wiki
create_time: 2020-8-4
---

# 数据类型

- 基本数据类型
  - 数值型
    - 整数类型(byte,short,int,long)
    - 浮点类型(float,double)
  - 字符型(char)
  - 布尔型(boolean)
- 引用数据类型
  - 类(class)
  - 接口(interface)
  - 数组([])

| 数据类型/字节 | 范围                                       |
| :------------ | ------------------------------------------ |
| byte/8        | [ -128 (-2^7) ,127 (2^7-1) ]               |
| char/16       | [ '\u0000' , '\uffff' ] 或者说 [ 0,65535 ] |
| short/16      | [ -32768 (-2^15)  , 32767 (2^15 - 1) ]     |
| int/32        | [ -2^31 , 2^31 - 1 ]                       |
| float/32      |                                            |
| long/64       |                                            |
| double/64     |                                            |
| boolean/~     | true 和 false                              |

$$
65535=2^{15}-1
$$

注：

- boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的
- float和double赋值

  - :heavy_check_mark:`float f =(float)3.4;`

  - :heavy_check_mark:`float f =3.4F;`
  - :heavy_check_mark:`double d = 3.4;`
  - ❌`float f=3.4;`
- char是Unicode编码

## double比较

经常遇到两个浮点类型比较大小的问题,要注意的是浮点数只是一个近似值。如果是我们显示指定的double的确定的值是可以用 `==` 比较的，如果是运算的结果，则不能使用 `==`

- 禁止使用“==”比较两个浮点数是否相等，如果场景允许，优先使用“<” 或 “>” 比较符，如果确实要比较两个浮点数是否相等，请采用以下两种方式：
  - 使用阈值的方式即 `Math.abs(f1 - f2) < THRESHOLD`
  - 使用BigDecimal的方式

  ``` java
  //Method 1
  BigDecimal f1 = new BigDecimal("0.0");
  BigDecimal pointOne = new BigDecimal("0.1");
  for (int i = 1; i <= 11; i++) {
      f1 = f1.add(pointOne);
  }
  
  //Method 2
  BigDecimal f2 = new BigDecimal("0.1");
  BigDecimal eleven = new BigDecimal("11");
  f2 = f2.multiply(eleven);
  
  System.out.println("f1 = " + f1);
  System.out.println("f2 = " + f2);
  
  if (f1.compareTo(f2) == 0)
      System.out.println("f1 and f2 are equal using BigDecimal\n");
  else
      System.out.println("f1 and f2 are not equal using BigDecimal\n");
  ```

  

  

## 隐式类型转换

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型。

```java
short s1 = 1;
// s1 = s1 + 1;
```

但是使用 += 或者 ++ 运算符会执行隐式类型转换。

```java
s1 += 1;
s1++;
```

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

```java
s1 = (short) (s1 + 1);
```

[StackOverflow : Why don't Java's +=, -=, *=, /= compound assignment operators require casting?](https://stackoverflow.com/questions/8710619/why-dont-javas-compound-assignment-operators-require-casting)

> A compound assignment expression of the form `E1 op= E2` is equivalent to `E1 = (T)((E1) op (E2))`, where `T` is the type of `E1`, except that `E1` is evaluated only once.
>
> the following code is correct:
>
> ```java
> short x = 3;
> x += 4.6;
> ```
>
> and results in x having the value 7 because it is equivalent to:
>
> ```java
> short x = 3;
> x = (short)(x + 4.6);
> ```

# 访问修饰符


| Modifier    | Class | Package | Subclass<br>(same Package) | Subclass<br/>(diff Package) | World<br>(same Package) |
| ----------- | ----- | ------- | -------------------------- | --------------------------- | ----------------------- |
| public      | +     | +       | +                          | +                           | +                       |
| protected   | +     | +       | +                          | +                           |                         |
| no modifier | +     | +       | +                          |                             |                         |
| private     | +     |         |                            |                             |                         |

- `+` : accessible	;	`blank` : not accessible;

规则:
1. private：只能在同一个类中访问
2. 没有修饰符：只能在同一包内访问
3. protected：同一包内所有类，其他包中子类
4. public: 所有同一个module的类

( public>protected>no modifier>private )

规则：
1. `Principle of Least Knowledge`: 只允许所需的最小可见度
2. Java8 引入了接口的 default ，这为接口中写模板方法提供了便利
3. 访问修饰的继承：
   - 父类中声明为 public 的方法在子类中也必须为 public。
   - 父类中声明为 protected 的方法在子类中要么声明为 protected，要么声明为 public，不能声明为 private。
   - 父类中声明为 private 的方法，不能够被继承。

注意：
1. 表格告诉我们允许访问与否，但什么是构成访问？访问实际上以复杂的方式与内部类和继承进行着交互
2. 类方法、内部类、静态方法是如何进行访问控制的？

## 示例/易于误解的protected

``` java
package fatherpackage;

public class Father
{
	protected void foo(){}
}

// ------------------------------------------

package sonpackage;

public class Son extends Father
{

}
```

`foo()`可以在3个上下文访问：

1. 和`foo()`定义在同一个包中的类，可以访问`foo()`

   ``` java
   package fatherpackage;
   
   public class SomeClass
   {
       public void someMethod(Father f, Son s)
       {
           f.foo();
           s.foo();
       }
   }
   
   // ------------------------------------------
   
   package fatherpackage;
   
   public class Son extends Father
   {
       public void sonMethod(Father f)
       {
           f.foo();
       }
   }
   ```

2. 在子类内部，可以通过 this 和 super 访问

   ``` java
   package sonpackage;
   
   public class Son extends Father
   {
       public void sonMethod()
       {
           this.foo();
           super.foo();
       }
   }
   ```

3. 在类型相同的引用上，注意这和this和super访问不同（重点在类型相同？）

   ``` java
   package fatherpackage;
   
   public class Father
   {
       public void fatherMethod(Father f)
       {
           f.foo(); // valid even if foo() is private
       }
   }
   
   // ------------------------------------------
   
   package sonpackage;
   
   public class Son extends Father
   {
       public void sonMethod(Son s)
       {
           s.foo();
       }
   }
   ```

在以下情况下不能访问：

1. 类型为父类，但是和`foo()`定义不在一个包中

   ``` java
   package sonpackage;
   
   public class Son extends Father
   {
       public void sonMethod(Father f)
       {
           f.foo(); // compilation error
       }
   }
   ```

2. 子类的同一个包内，但是不是子类（子类从其父类继承受保护的成员，并使它们对非子类私有）

   ``` java
   package sonpackage;
   
   public class SomeClass
   {
       public void someMethod(Son s) throws Exception
       {
           s.foo(); // compilation error
       }
   }
   ```

注意区分上面这5种情况中的以下三种：

``` java
package fatherpackage;

public class Son extends Father
{
    public void sonMethod(Father f)
    {
        f.foo(); // ok
    }
}

// ------------------------------------------

package sonpackage;

public class Son extends Father
{
    public void sonMethod(Son s)
    {
        s.foo(); // ok
    }
}

// ------------------------------------------

package sonpackage;

public class SomeClass
{
    public void someMethod(Son s) throws Exception
    {
        s.foo(); // compilation error
    }
}
```

<u>*todo: 当访问修饰符结合上了子类、内部类、接口的时候，进而结合到static、final的时候，情况就变得愈发复杂，这种情况下如何取舍*</u>

参考：

- [What is the difference between public, protected, package-private and private in Java?](https://stackoverflow.com/questions/215497/what-is-the-difference-between-public-protected-package-private-and-private-in)

# 小问题/技巧

- switch条件表达式：
  switch(expr)中，expr可以是 byte、short、char、int、枚举、字符串 
  
  [StackOverflow : Why can't your switch statement data type be long, Java?](https://stackoverflow.com/questions/2676210/why-cant-your-switch-statement-data-type-be-long-java)
  
  - 注意没有long
  
- 乘除2的幂次
  
  - 计算 2 乘以 8：`2 << 3`
  
- 四舍五入的原理是在参数上加 0.5 然后进行下取整
  - `Math.round(11.5)` = 12
  - `Math.round(-11.5)` = -11
  
- Java没有 `goto` 语句，但是 `goto` 是关键字，这个作用是防止 `goto` 这个单词作为变量名、方法名

  [StackOverflow : Is there a goto statement in Java?](https://stackoverflow.com/questions/2545103/is-there-a-goto-statement-in-java)

- 值传递和引用传递

  Java是值传递，Java的对象引用也是按值传递的

  - 值传递是拷贝变量的值，引用传递是把形参的地址和传入变量的地址赋为相同

  - ``` java
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Student s1 = new Student();
        Student s2 = new Student();
        Test.swap(s1, s2);
    }
    
    public static void swap(Student x, Student y) {
        Student temp = x;
        x = y;
        y = temp;
    }
    ```

    如果是值传递：s1最后还是s1，s2最后还是s2

    如果是引用传递：s1最后是s2，s2最后是s1
    
  - 调用方法时发生了什么？====>>参数传递基本上就是**赋值操作**
  
    **（重要）注意下面的第三个和第四个例子，同样是传入对象，什么改变了什么没改变，什么能改变什么不能改变**
  
    ``` java
    第一个例子：基本类型
    void foo(int value) {
        value = 100;
    }
    foo(num); // num 没有被改变
    
    第二个例子：没有提供改变自身方法的引用类型
    void foo(String text) {
        text = "windows";
    }
    foo(str); // str 也没有被改变
    
    第三个例子：提供了改变自身方法的引用类型
    StringBuilder sb = new StringBuilder("iphone");
    void foo(StringBuilder builder) {
        builder.append("4");
    }
    foo(sb); // sb 被改变了，变成了"iphone4"。
    
    第四个例子：提供了改变自身方法的引用类型，但是不使用，而是使用赋值运算符。
    StringBuilder sb = new StringBuilder("iphone");
    void foo(StringBuilder builder) {
        builder = new StringBuilder("ipad");
    }
    foo(sb); // sb 没有被改变，还是 "iphone"。
    ```
  
  - [Java 到底是值传递还是引用传递？ 知乎 - Intopass 回答](https://www.zhihu.com/question/31203609/answer/50992895)
  
- int型的变量，如何将它转成String，`Integer.toString(xx)`，`(new Integer(xx)).toString()`这两个方法有什么区别吗？

  第一个是静态方法，第二个要实例化对象

