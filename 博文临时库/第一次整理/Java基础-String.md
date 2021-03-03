---
title: JAVA字符串
description: 
tags:
- JAVA
- 字符串
- JVM
create_time: 2020-2-23 19:26:00
---

> 本文所有内容基于以下几个要点

- String不可变
- String重载了"+"
- 对象进行字符串相加会调用toString方法
- StringBuilder 和 StringBuffer
- JVM 常量池

# String

String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）

在 Java 8 中，String 内部使用 char 数组存储数据。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char[] value;
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

- value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。
- 如果`String`仅包含ASCII字符，则每个`byte`存储一个字符，否则，每两个`byte`存储一个字符，这样做的目的是为了节省内存，因为大量的长度较短的`String`通常仅包含ASCII字符

## 常用方法

- 看其名知其意的方法：
  - length、charAt、equals、indexOf、replace、substring、toLowerCase、toUpperCase
- 其他方法：
  - trim()：去除字符串两端空白
  - split()：分割字符串，返回一个分割后的字符串数组。
  - getBytes()：返回字符串的 byte 类型数组

# 不可变String

平时操作String对象，都是直接把操作后的结果传过去，这看起来就像修改原对象了一样，例如System.out.println(“abcdefghijkl”.toUpperCase());但实际是没改动的

- String类中每一个特看起来会修改String值的方法，实际上都是创建了一个全新的String对象，而最初的String对象**丝毫未动**
  - 这意味着String是**只读的**，指向String的任何**引用**都不会改变它的值
  - 不可变性会带来**效率**问题(字符串相加的中间对象)
- String中值的保存的源码，**final**字符数组意味着value的不可变

``` java
    /** The value is used for character storage. */
    private final char value[];
```

- 字符串常量池
  - 每个字符序列如"123"会生成一个实例放在常量池里，这个实例是不在堆里的，也不会被GC
  - 我们在使用诸如String str = "abc"的格式定义类时，总是想当然地认为，创建了String类的对象str。但是由于字符串常量池的存在，**担心陷阱！对象可能并没有被创建！而可能只是指向一个先前已经创建的对象。** 只有通过*new()方法才能保证每次都创建一个新的对象*，因此在使用 "==" 进行对象比较时需要注意这个问题
  - JVM中的常量池在内存当中是以表（hashtable）的形式存在的， 对于String类型，有一张固定长度的CONSTANT_String_info表用来存储文字字符串值(注意：该表只存储文字字符串值，不存储符号引用。
     常量池中保存着很多String对象，并且可以被**共享使用**，因此它**提高了效率**)。
  - String类有一个**intern()**方法，它可以访问**字符串池**。是扩充常量池的 一个方法。当一个String实例str调用intern()方法时，Java 查找常量池中 是否有相同的字符串常量，**如果有，则返回其的引用，如果没有，则在常量池中增加一个等于str的字符串并返回它的引用；**
  
- 使用new和直接赋值String

  - String str1="ABC" 可能创建一个对象或者不创建对象

  - String str2 = new String("ABC") 至少创建一个对象，也可能两个。

    它首先会在heap创建一个 str2 的String 对象，value 是 "ABC"。同时，如果"ABC"这个字符串在java String池里不存在，会在java String池创建这个一个String对象("ABC")

---

**好处：**

1. 可以缓存 hash 值

   因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

2. 节约内存：String Pool的需要

   如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

3. 安全性

   一些重要参数例如：网络连接地址URL、文件路径path、反射机制所需要的String参数、String作为HashMap的值等，在这些参数里的String使用前可能会经过一系列的合法性校验，如果String可变那么可能刚刚校验正确的参数就被修改成错误的了，这样的情况是不合理的。

4. 线程安全

   String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

   [Program Creek : Why String is immutable in Java?](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)

# StringBuilder & StringBuffer

- StringBuilder允许**预先指定大小**，如果知道最终字符串大小有多长，那么预先指定StringBuilder的大小会**避免**多次重新**分配缓冲**

- StringBuilder常用的方法：append()、toString()、delete()

- StringBuffer是**线程安全**的，但是**开销大**

- StringBuilder和StringBuffer均继承自**AbstractStringBuilder**类

  - AbstractStringBuilder类值的保存的源码，可见对象是可变的
    ``` java
    	/**
         * The value is used for character storage.
         */
        char[] value;
    ```


# String重载了“+”

- String的“+”和“+=”是Java中仅有的两个重载过的操作符，<u>*java不允许程序员重载任何操作符*</u>

- 当有字符串常量进行相加时，会产生大量的**中间对象**，所有的这些字符串常量都会保存到 JVM **常量区**，此方式产生了大量需要垃圾回收的中间对象

  - “abc”和mango生成一个中间变量x，x和“def”生成一个新的中间变量，以此类推

  ``` java
  String mango = "mango";
  String s = "abc" + mango + "def" + 47;
  System.out.println(s);
  ```
  
- 编译器会对字符串相加自动引入StringBuilder类，因为他更加高效。（详细的分析可以在这篇文章找到 [String concatenation with Java 8](http://www.pellegrino.link/2015/08/22/string-concatenation-with-java-8.html)）

  - 但是编译器并不知道需要创建几个StringBuilder，在循环中进行字符串相加会创建**大量的StringBuilder对象**，这同样需要**大量的GC**

    ``` java
    String result = "";
    for(int i=0;i<fields.length;i++){ // String[] fields ....
        result += fields[i]; // 每一次循环会创建一个新的StringBuilder
    }
    ```

    （此处需要依次javap反编译查看JVM字节码）

    但是手动创建StringBuilder却只需要**一个对象**就可以完成操作
    
    ``` java
    StringBuilder result = new StringBuilder();
    for(int i=0;i<fields.length;i++){
        result.append(fields[i]); // 仅有一个StringBuilder对象
    }
result.toString() // get target string
    ```
    
    （此处需要依次javap反编译查看JVM字节码）

# toString & 无意识递归

- 无意识**递归**问题

  - 发生示例如下。this在这里会被进行**类型转换**，由A类转换成String，但转换正是通过**toString**来进行的，于是发生递归调用。

    ``` java
    public class A{
        ...
        public String toString(){
            return "toString memory address: "+ this;
        }  
        ...
    }
    
    ```
	如果期望**打印对象地址**应当使调用Object.toString()用 **super.toString()**而不是this.

# 何时使用"+"和StringBuilder

- 使用 “+” 连接字符串
  - 少量数据
  - 没有循环
- 使用StringBuilder
  - 单线程
  - 大量数据相加
  - 大量循环
- 使用StringBuffer
  - 多线程
  - 同StringBuilder

# 更加深入的了解

- 需要了解JVM内存模型

- 需要查看String、StringBuilder、StringBuffer源码

- 需要学习一些和字符串常见的组会使用的类，例如：

  正则相关、格式化相关、编码相关

- 可能需要做一些习题

# 可供练习的习题

- 习题来源于  [深入理解Java：String](https://www.cnblogs.com/ITtangtang/p/3976820.html)

``` java
public static void main(String[] args) {  
    /** 
         * 情景一：字符串池 
         * JAVA虚拟机(JVM)中存在着一个字符串池，其中保存着很多String对象; 
         * 并且可以被共享使用，因此它提高了效率。 
         * 由于String类是final的，它的值一经创建就不可改变。 
         * 字符串池由String类维护，我们可以调用intern()方法来访问字符串池。  
         */  
    String s1 = "abc";     
    //↑ 在字符串池创建了一个对象  
    String s2 = "abc";     
    //↑ 字符串pool已经存在对象“abc”(共享),所以创建0个对象，累计创建一个对象  
    System.out.println("s1 == s2 : "+(s1==s2));    
    //↑ true 指向同一个对象，  
    System.out.println("s1.equals(s2) : " + (s1.equals(s2)));    
    //↑ true  值相等  
    //↑------------------------------------------------------over  
    /** 
         * 情景二：关于new String("") 
         *  
         */  
    String s3 = new String("abc");  
    //↑ 创建了两个对象，一个存放在字符串池中，一个存在与堆区中；  
    //↑ 还有一个对象引用s3存放在栈中  
    String s4 = new String("abc");  
    //↑ 字符串池中已经存在“abc”对象，所以只在堆中创建了一个对象  
    System.out.println("s3 == s4 : "+(s3==s4));  
    //↑false   s3和s4栈区的地址不同，指向堆区的不同地址；  
    System.out.println("s3.equals(s4) : "+(s3.equals(s4)));  
    //↑true  s3和s4的值相同  
    System.out.println("s1 == s3 : "+(s1==s3));  
    //↑false 存放的地区多不同，一个栈区，一个堆区  
    System.out.println("s1.equals(s3) : "+(s1.equals(s3)));  
    //↑true  值相同  
    //↑------------------------------------------------------over  
    /** 
         * 情景三：  
         * 由于常量的值在编译的时候就被确定(优化)了。 
         * 在这里，"ab"和"cd"都是常量，因此变量str3的值在编译时就可以确定。 
         * 这行代码编译后的效果等同于： String str3 = "abcd"; 
         */  
    String str1 = "ab" + "cd";  //1个对象  
    String str11 = "abcd";   
    System.out.println("str1 = str11 : "+ (str1 == str11));  
    //↑------------------------------------------------------over  
    /** 
         * 情景四：  
         * 局部变量str2,str3存储的是存储两个拘留字符串对象(intern字符串对象)的地址。 
         *  
         * 第三行代码原理(str2+str3)： 
         * 运行期JVM首先会在堆中创建一个StringBuilder类， 
         * 同时用str2指向的拘留字符串对象完成初始化， 
         * 然后调用append方法完成对str3所指向的拘留字符串的合并， 
         * 接着调用StringBuilder的toString()方法在堆中创建一个String对象， 
         * 最后将刚生成的String对象的堆地址存放在局部变量str3中。 
         *  
         * 而str5存储的是字符串池中"abcd"所对应的拘留字符串对象的地址。 
         * str4与str5地址当然不一样了。 
         *  
         * 内存中实际上有五个字符串对象： 
         *       三个拘留字符串对象、一个String对象和一个StringBuilder对象。 
         */  
    String str2 = "ab";  //1个对象  
    String str3 = "cd";  //1个对象                                         
    String str4 = str2+str3;                                        
    String str5 = "abcd";    
    System.out.println("str4 = str5 : " + (str4==str5)); // false  
    //↑------------------------------------------------------over  
    /** 
         * 情景五： 
         *  JAVA编译器对string + 基本类型/常量 是当成常量表达式直接求值来优化的。 
         *  运行期的两个string相加，会产生新的对象的，存储在堆(heap)中 
         */  
    String str6 = "b";  
    String str7 = "a" + str6;  
    String str67 = "ab";  
    System.out.println("str7 = str67 : "+ (str7 == str67));  
    //↑str6为变量，在运行期才会被解析。  
    final String str8 = "b";  
    String str9 = "a" + str8;  
    String str89 = "ab";  
    System.out.println("str9 = str89 : "+ (str9 == str89));  
    //↑str8为常量变量，编译期会被优化  
    //↑------------------------------------------------------over  
}
```

<u>*[todo：善用intern和慎用split](https://segmentfault.com/a/1190000022509965)*</u>

<u>*todo：编码问题*</u>

# 参考

> 1. Think in java 第四版 第十三章
> 2. [String concatenation with Java 8：和 thinking in java 有着大体相同阐述的文章，提供了数学分析和数据](http://www.pellegrino.link/2015/08/22/string-concatenation-with-java-8.html)
> 3. [深入理解Java：String：提供了五个String比较的情景](https://www.cnblogs.com/ITtangtang/p/3976820.html)
> 4. [CS-Notes-Java 基础.md](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%9F%BA%E7%A1%80.md)

