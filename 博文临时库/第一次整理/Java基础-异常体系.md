---
title: 
description: 
tags:
- JAVA
create_time: 2020-8-6
---

<u>*advice：这部分感觉题目为 ”JAVA防御式编程 “ 较为好一点，但现在主要记录异常，暂时使用这个名称，之后添加的时候和整理的时候在进行更改吧。*</u>

# 异常综述

我们肯定想创建健壮的系统，而错误恢复机制就可以提高代码健壮性，Java中异常就是它的错误恢复机制。另外Java的主要目标之一就是创建供他人使用的程序构件。

除了健壮性以外，异常往往能**降低**错误处理代码的**复杂度**，因为它把**“描述在正常执行过程中做什么事”**的代码和**“出了问题怎么办”**的代码相分离，也就是正常**执行路径**和异常执行路径的分离。**异常最重要的方面之一就是如果发生问题，它们将不允许程序沿着其正常的路径继续走下去**。

另外不应把 Java 的异常处理机制当成是单一用途的工具。是的，它被设计用来处理一些烦人的运行时错误，这些错误往往是由代码控制能力之外的因素导致的；然而，它对于发现某些编译器无法检测到的编程错误，也是非常重要的。

- 注意：异常处理是 Java 中**唯一官方**的错误报告机制

发现错误的**理想时机是在编译阶段**。然而，**编译期间并不能找出所有的错误**，余下的问题必须在运行期间解决。这就需要错误源能通过某种方式，把适当的信息传递给某个接收者——该接收者将知道如何正确处理这个问题。

**C 以及其他早期语言**常常具有多种错误处理模式，这些模式往往建立在约定俗成的基础之上，而并**不属于语言的一部分**。

异常情形与普通问题：

把异常情形与普通问题相区分很重要，所谓的普通问题是指，在当前环境下能得到足够的信息，总能处理这个错误。而对于异常情形，就不能继续下去了，因为在当前环境下无法获得必要的信息来解决问题。例如：除数有可能为 0，所以先进行检查很有必要。但除数为 0 代表的究竟是什么意思呢？更进一步：如果这是一个意料之外的值，你也不清楚该如何处理，那就要抛出异常，而不是顺着原来的路径继续执行下去。

# 继承体系

> 异常除了名字不一样，实际上都差不多

异常在继承体系上可分类为：Error、Exception，它们都是Throwable。

- Error：程序中**无法处理**的错误，表示运行应用程序中出现了严重的错误。用来表示编译时和系统错误。一般表示代码运行时 JVM 出现问题，此类错误发生时，JVM 将终止线程。（除特殊情况外，此类错误一般不用你关心）
- Exception：程序本身**可以捕获**并且**可以处理**的异常。异常又分为两类：运行时异常和编译时异常.

其中Exception又可按照是否受到编译器检查上分类为：**受检异常和非受检异常**。

- `RuntimeException` 及其子类 都属于非受检异常

- 除 `RuntimeException` 及其子类，其他的 Exception 异常都属于受检异常

  在程序中，通常不会自定义该类异常，而是直接使用系统提供的异常类。该异常我们必须手动在代码里添加捕获语句来处理该异常。但是受检异常有时候也可以给我们提供一些便利，所以 `通常不会自定义该类异常` 这个说法不是很恰当。

- 有人说受检异常可以从异常中恢复，而非受检异常是导致程序崩溃并且不能恢复。这样的说法有失妥当，因为我们完全可以catch这些异常，并且处理它们。另外，使用前确保不会出现非受检异常（例如确保对象不是null）也是程序员的责任，也即非受检异常本身就不应当抛出。

- 非受检异常一般是由**程序逻辑错误**引起的，在程序中可以选择捕获处理，也可以不处理。此类异常的出现绝大数情况是代码本身有问题应该从逻辑上去解决并改进代码。

Exception中除去`RuntimeException`以外都是会在编译时检查的，即：

- Java 编译器会检查它。如果程序中出现此类异常，要么通过**throws**进行声明抛出，要么通过**try-catch**进行捕获处理，否则不能通过编译。
- **throw 和 throws 的区别**（数量和位置上）：throw 关键字用在方法内部，只能用于抛出一种异常；throws 关键字用在方法声明上，可以抛出多个异常，用来标识该方法可能抛出的异常列表。

![](E:\_data\博文临时库\博文中的图片\drawio\JavaException.png)

（在这个图中同时给出了比较常见的Exception和Error）

# 异常处理流程

当抛出异常后，首先，同 Java 中其他对象的创建一样，将**使用 new 在堆上创建异常对象**。然后，当前的**执行路径被终止**（它不能继续下去了），并且从当前环境中弹出对异常对象的引用。此时，异常处理机制接管程序，并**开始寻找一个恰当的地方来继续执行**程序。这个恰当的地方就是异常处理程序，它的任务是将程序从错误状态中恢复，以使程序能换一种方式运行。

## 异常匹配

- 按照代码的书写顺序找出“最近”的处理程序
- 并不要求抛出的异常同处理程序所声明的异常完全匹配。派生类的对象也可以匹配其基类的处理程序
  - 在捕获上，把继承体系上相对顶级的异常如Exception放到相对靠后的catch语句。否则编译器会发现某些 catch 子句永远得不到执行，因此它会向你报告错误

例如：类 `ExampleA` 继承 Exception，类 `ExampleB` 继承`ExampleA`

``` java
try {
	throw new ExampleB("b")
} catch（ExampleA e）{
	System.out.println("ExampleA");
} catch（Exception e）{
	System.out.println("Exception");
}
// 输出：ExampleA
```

## 异常处理

在捕获到异常后，对于**异常的处理通常有两种模型**，即中止模型和恢复模型。

- 中止模型：假设错误非常严重，以至于程序无法返回到异常发生的地方继续执行，程序将终止（可能不是整个程序中止，而是产生异常的地方中止）

- 恢复模型：异常处理程序的工作是修正错误，然后重新尝试调用出问题的方法

  虽然恢复模型开始显得很吸引人，但不是很实用。其中的主要原因可能是它所导致的耦合：恢复性的处理程序需要了解异常抛出的地点，这势必要包含依赖于抛出位置的非通用性代码

Java支持的是中止模型，但也有方式使得它支持恢复模型：

1. 遇见错误时就不能抛出异常，而是调用方法来修正该错误

2. 把 try 块放在 while 循环里，这样就不断地进入 try 块，直到得到满意的结果

   如果把 try 块放在循环里，就建立了一个“程序继续执行之前必须要达到”的条件。还可以加入一个 static 类型的计数器或者别的装置，使循环在放弃以前能尝试一定的次数。这将使程序的健壮性更上一个台阶。

## 其他问题

1. **catch和switch**：

   try-catch中的catch在某些程度上有点像switch，但是一旦 catch 子句结束，则处理程序的查找过程结束。这与 switch 语句不同，switch 语句需要在每一个 case 后面跟一个 break，以避免执行后续的 case 子句。

   异常和程序返回值、throw和return的比较：

   异常对象的类型通常与方法设计的返回类型不同，但从效果上看，它就像是从方法“返回”的。但是要注意：异常返回的“地点”与普通方法调用返回的“地点”完全不同。

   - 异常将在一个恰当的异常处理程序中得到解决，它的**位置可能离异常被抛出的地方很远**，也可能会跨越方法调用栈的许多层级。

2. 异常对象的**清理回收**：

   永远不必为清理前一个异常对象而担心，或者说为异常对象的清理而担心。它们都是用 new 在堆上创建的对象，所以垃圾回收器会自动把它们清理掉

# finally块

无论异常是否被抛出，finally 子句总能被执行。finally 子句在多数情况下用于内存释放。

## 关于内存释放

- 对于没有垃圾回收和析构函数自动调用机制的语言来说，finally 非常重要。它能使程序员保证：无论 try 块里发生了什么，内存总能得到释放。但 Java 有垃圾回收机制，所以**内存释放**不再是问题。而且，Java 也没有析构函数可供调用。那么，Java 在什么情况下才能用到 finally 呢？

- 当要**把除内存之外的资源恢复到它们的初始状态**时，就要用到 finally 子句。

  这也是 Java 的缺陷：除了内存的清理之外，所有的清理都不会自动发生。这一点也体现在了Object类中的  finalize()  方法。

## try-finally

（没有catch）在异常没有被当前的异常处理程序捕获的情况下，异常处理机制也会在跳到更高一层的异常处理程序之前，执行 finally 子句，如下所示

``` java
public static void main(String[] args) {
    System.out.println("Entering first try block");
    try {
        System.out.println("Entering second try block");
        try {
            throw new FourException();
        } finally { // 注意这里没有捕获FourException，而是会继续抛出
            System.out.println("finally in 2nd try block");
        }
    } catch(FourException e) {
        System.out.println(
            "Caught FourException in 1st try block");
    } finally {
        System.out.println("finally in 1st try block");
    }
}
/* 输出为
Entering first try block
Entering second try block
finally in 2nd try block
Caught FourException in 1st try block
finally in 1st try block
*/
```

- 这种通用的清理惯用法在构造器不抛出任何异常时也应该运用，其基本规则是：在创建需要清理的对象之后，立即进入一个 try-finally 语句块

## return & finally

即使在try语句中有return，finally也会执行。但是有时也会有某些人在finally中写return（我真的觉得这些人很可恶🙃）

在 finally 中改变返回值的做法是不好的，因为如果存在 finally 代码块，try中的 return 语句不会立马返回调用者，而是记录下返回值待 finally 代码块执行完毕之后再向调用者返回其值，然而如果在 finally 中修改了返回值，就会返回修改后的值。显然，在 finally 中返回或者修改返回值会对程序造成很大的困扰

1. 第一个示例：

   ``` java
   public static int getInt() {
       int a = 10;
       try {
           System.out.println(a / 0);
           a = 20;
       } catch (ArithmeticException e) {
           a = 30;
           return a;
           /*
            * return a 在程序执行到这一步的时候，这里不是return a 而是 return 30；这个返回路径就形成了
            * 但是呢，它发现后面还有finally，所以继续执行finally的内容，a=40
            * 再次回到以前的路径,继续走return 30，形成返回路径之后，这里的a就不是a变量了，而是常量30
            */
       } finally {
           a = 40;
       }
   	return a;
   }
   ```

2. 第二个示例：

   ``` java
   public static int getInt() {
       int a = 10;
       try {
           System.out.println(a / 0);
           a = 20;
       } catch (ArithmeticException e) {
           a = 30;
           return a;
       } finally {
           a = 40;
           //如果这样，就又重新形成了一条返回路径，由于只能通过1个return返回，所以这里直接返回40
           return a; 
       }
   }
   
   ```

## 异常丢失

1. finally 中抛出异常

   ``` java
   public class LostMessage {
       void f() throws VeryImportantException {
           throw new VeryImportantException();
       }
       void dispose() throws HoHumException {
           throw new HoHumException();
       }
       public static void main(String[] args) {
           try {
               LostMessage lm = new LostMessage();
               try {
                   lm.f(); // 注意这里
               } finally {
                   lm.dispose(); // 注意这里
               }
           } catch(VeryImportantException | HoHumException e) { // 注意这里
               System.out.println(e);
           }
       }
   }
   /* 输出为
   A trivial exception
   */
   ```

   从输出中可以看到，VeryImportantException 不见了，它被 finally 子句里的 HoHumException 所取代。这是相当严重的缺陷，也许在 Java 的未来版本中会修正这个问题。

   对于这个问题，我们需要把所有抛出异常的方法，如上例中的 dispose() 方法，全部打包放到 try-catch 子句里面。

2. finally 中返回：

   ``` java
   // exceptions/ExceptionSilencer.java
   public class ExceptionSilencer {
       public static void main(String[] args) {
           try {
               throw new RuntimeException();
           } finally {
               // Using 'return' inside the finally block
               // will silence any thrown exception.
               return;
           }
       }
   }
   ```

   如果运行这个程序，就会看到即使方法里抛出了异常，它也不会产生任何输出。

---

finally的另类用法：

- 把 finally 子句和带标签的 break 及 continue 配合使用，在 Java 里就没必要使用 `goto` 语句了

# 构造器中的异常

🔴**构造器会把对象设置成安全的初始状态**，但还会有别的动作，**比如打开一个文件，这样的动作只有在对象使用完毕并且用户调用了特殊的清理方法之后才能得以清理**。如果在构造器内抛出了异常，这些清理行为也许就不能正常工作了。这意味着在编写构造器时要格外细心。

🔴你也许会认为使用 finally 就可以解决问题。但问题并非如此简单，因为 finally 清理代码。如果构造器在其执行过程中半途而废，**也许该对象的某些部分还没有被成功创建，而这些部分在 finally 子句中却是要被清理的**。

看下面一个例子：

``` java
// exceptions/InputFile.java
// Paying attention to exceptions in constructors
import java.io.*;
public class InputFile {
    private BufferedReader in;
    public InputFile(String fname) throws Exception {
        try {
            in = new BufferedReader(new FileReader(fname));
            // Other code that might throw exceptions
        } catch(FileNotFoundException e) {
            System.out.println("Could not open " + fname);
            // Wasn't open, so don't close it
            throw e;
        } catch(Exception e) {
            // All other exceptions must close it
            try {
                in.close();
            } catch(IOException e2) {
                System.out.println("in.close() unsuccessful");
            }
            throw e; // Rethrow
        } finally {
        // Don't close it here!!!
        }
    }
    public String getLine() {
        String s;
        try {
            s = in.readLine();
        } catch(IOException e) {
            throw new RuntimeException("readLine() failed");
        }
        return s;
    }
    public void dispose() {
        try {
            in.close();
            System.out.println("dispose() successful");
        } catch(IOException e2) {
            throw new RuntimeException("in.close() failed");
        }
    }
}
```

- 如果 `FileReader` 的构造器失败了，将抛出 `FileNotFoundException` 异常。如果还有其他方法能抛出 `FileNotFoundException`，这个方法就显得有些投机取巧了。这时，通常必须把这些方法分别放到各自的 try 块里

- 🔴在本地做完处理之后，异常被重新抛出，对于构造器而言这么做是很合适的，因为你总**不希望去误导调用方，让他认为“这个对象已经创建完毕，可以使用了”**

- 对于在构造阶段可能会抛出异常，并且要求清理的类，最安全的使用方式是使用嵌套的 try 子句：

  ``` java
  public static void main(String[] args) {
      try {
          InputFile in = new InputFile("Cleanup.java"); // 注意这里
          try {
              String s;
              int i = 1;
              while((s = in.getLine()) != null)
                  ; // Perform line-by-line processing here...
          } catch(Exception e) {
              System.out.println("Caught Exception in main");
              e.printStackTrace(System.out);
          } finally {
              in.dispose();  // 注意这里
          }
      } catch(Exception e) {  // 注意这里
          System.out.println(
              "InputFile construction failed");
      }
  }
  ```

  🔴请仔细观察这里的逻辑：对 `InputFile` 对象的构造在其自己的 try 语句块中有效，如果构造失败，将进入外部的 catch 子句，而 dispose() 方法不会被调用。但是，如果构造成功，我们肯定想确保对象能够被清理，因此在构造之后立即创建了一个新的 try 语句块。执行清理的 finally 与内部的 try 语句块相关联。在这种方式中，finally 子句在构造失败时是不会执行的，而在构造成功时将总是执行。

# Try-With-Resources

上面的构造器的内容可能让人有些头疼。在考虑所有可能失败的方法时，找出放置所有 try-catch-finally 块的位置变得令人生畏。确保没有任何故障路径，使系统远离不稳定状态，这非常具有挑战性。

当 finally 子句有自己的 try 块时，感觉事情变得过于复杂，有时候我们会遇到这样的情况：

``` java
public static void main(String[] args) {
    InputStream in = null;
    try {
        in = new FileInputStream(
            new File("MessyExceptions.java"));
        int contents = in.read();
        // Process contents
    } catch(IOException e) {
        // Handle the error
    } finally {
        if(in != null) { // 注意这里
            try {
                in.close();
            } catch(IOException e) {
                // Handle the close() error
            }
        }
    }
}
```

幸运的是，Java 7 引入了 try-with-resources 语法，它可以非常清楚地简化上面的代码：

```java
// exceptions/TryWithResources.java
import java.io.*;
public class TryWithResources {
    public static void main(String[] args) {
        try(
                InputStream in = new FileInputStream(
                        new File("TryWithResources.java"))
        ) {// 注意这里
            int contents = in.read();
            // Process contents
        } catch(IOException e) {
            // Handle the error
        }
    }
}
```

括号内的部分称为**资源规范头**（resource specification header）。规范头中定义的每个对象都会在 try 语句块运行结束之后调用 close() 方法。

## 运作原理和特殊情况

**运作原理、如何编写**：

- 在 try-with-resources 定义子句中创建的对象（在括号内）必须实现 `java.lang.AutoCloseable` 接口，这个接口有一个方法：`close()`。Closeable 继承了 `AutoCloseable` 接口，所以所有实现了 Closeable 接口的对象，都支持了  try-with-resources 特性。

- 退出 try 块会调用对象的 close() 方法，并以与创建顺序相反的顺序关闭它们。顺序很重要，因为在此配置中，Second 对象可能依赖于 First 对象，因此如果 First 在第 Second 关闭时已经关闭。 Second 的 close() 方法可能会尝试访问 First 中不再可用的某些功能。

**特殊情况**：

- 如果在规范头中的一个**构造函数抛出异常**怎么办？如下：

  ``` java
  class CE extends Exception {}
  class SecondExcept extends Reporter {
      SecondExcept() throws CE {
          super();
          throw new CE(); // 注意这里
      }
  }
  public class ConstructorException {
      public static void main(String[] args) {
          try(
                  First f = new First();
                  SecondExcept s = new SecondExcept(); // 注意这里
                  Second s2 = new Second()
          ) {
              System.out.println("In body");
          } catch(CE e) { // 注意这里
              System.out.println("Caught: " + e);
          }
      }
  }
  ```

  `SecondExcept` 在创建期间抛出异常。请注意，不会为 `SecondExcept` 调用 close()，因为如果构造函数失败，则无法假设你可以安全地对该对象执行任何操作，包括关闭它。由于 `SecondExcept` 的异常，Second 对象实例 s2 不会被创建，因此也不会有清除事件发生。

- 如果在有对象在try{}中而不是try()中被定义，那么它就有可能不会被保护（这其实就是**一般情况**，但是可能会被**混用在 try-with-resources 中**）如下：

  ``` java
  public static void main(String[] args) {
      try(
          First f = new First();
          Second s2 = new Second()
      ) {
          System.out.println("In body");
          Third t = new Third();
          new SecondExcept();
          System.out.println("End of body");
      } catch(CE e) {
          System.out.println("Caught: " + e);
      }
  }
  ```

  请注意，第 3 个对象永远不会被清除。那是因为它不是在资源规范头中创建的，所以它没有被保护。因此，如果可以，你应当始终使用 try-with-resources。这个特性有助于生成更简洁，更易于理解的代码。

## try()

 `try-with-resources` 里面的 try 语句块可以不包含 catch 或者 finally 语句而独立存在，例如：

``` java
try(
    Stream<String> in = Files.lines(
        Paths.get("StreamsAreAutoCloseable.java"));
    PrintWriter outfile = new PrintWriter(
        "Results.txt"); // 这里是分号分割，最后一个分号可以省略
) {
    in.skip(5)
        .limit(1)
        .map(String::toLowerCase)
        .forEachOrdered(outfile::println);
}
```

# 常见异常

## RuntimeException

RuntimeException 是“不受检查异常”。这种异常属于错误，将被自动捕获，不用亲自动手。要是自己去检查 RuntimeException 的话，代码就显得太混乱了。但是尽管通常不用捕获 RuntimeException 异常，还是可以在代码中抛出 RuntimeException 类型的异常，也可以在一个地方捕获它。（有无必要和能不能）

RuntimeException 代表的是编程错误：

1. 无法预料的错误。比如从你控制范围之外传递进来的 null 引用。
2. 作为程序员，应该在代码中进行检查的错误。（比如对于 `ArrayIndexOutOfBoundsException`，就得注意一下数组的大小了。）在一个地方发生的异常，常常会在另一个地方导致错误。

## `NoClassDefFoundError` & `ClassNotFoundException` 

`NoClassDefFoundError` 是一个 Error 类型的异常，是由 JVM 引起的，不应该尝试捕获这个异常

引起该异常的原因是 JVM 或 `ClassLoader` 尝试加载某类时在内存中找不到该类的定义，该动作发生在运行期间，即编译时该类存在，但是在运行时却找不到了，可能是变异后被删除了等原因导致；

`ClassNotFoundException` 是一个受检异常，需要显式地使用 try-catch 对其进行捕获和处理，或在方法签名中用 throws 关键字进行声明。当使用 `Class.forName`, `ClassLoader.loadClass` 或 `ClassLoader.findSystemClass` 动态加载类到内存的时候，通过传入的类路径参数没有找到该类，就会抛出该异常；另一种抛出该异常的可能原因是某个类已经由一个类加载器加载至内存中，另一个加载器又尝试去加载它。

# 性能

（对于性能这一点，我存疑，因为很多情况下，异常带来的好处要比这个问题的坏处多，当然了，如果在性能关键处就应该减少异常使用）

异常处理的性能成本非常高。创建一个异常非常慢，抛出一个异常又会消耗时间，当一个异常在应用的多个层级之间传递时，会拖累整个应用的性能。

- 仅在异常情况下使用异常；
- 在可恢复的异常情况下使用异常；

尽管使用异常有利于 Java 开发，但是在应用中最好不要捕获太多的调用栈，因为在很多情况下都不需要打印调用栈就知道哪里出错了。因此，异常消息应该提供恰到好处的信息

# 一些写法

对于try、catch、finally的写法：

- try()
- try-catch
  - try()-catch
- try-catch-finally
  - try()-catch-finally
- try-finally
  - try()-finally

也就是除了try都可以省略不写

## 自定义异常

异常也是对象的一种，所以可以继续修改这个异常类，以得到更强的功能。但要记住，使用程序包的客户端程序员可能仅仅只是查看一下抛出的异常类型（也就是类名），其他的就不管了（大多数 Java 库里的异常都是这么用的），所以对异常所添加的其他功能也许根本用不上。

所有标准异常类都有**两个构造器：一个是无参构造器；另一个是接受字符串作为参数**，以便能把相关信息放入异常对象的构造器。但其实对异常来说，最重要的部分就是类名。

- 只有三种基本的异常类提供了带 cause 参数的构造器。它们是 Error、Exception 、 RuntimeException

**自定义异常**：要自己定义异常类，必须从已有的异常类继承，最好是选择意思相近的异常类继承（不过这样的异常并不容易找）

## 常用方法

- `printStackTrace`

  ``` java
  void printStackTrace()
  void printStackTrace(PrintStream)
  void printStackTrace(java.io.PrintWriter)
  ```

- `getMessage`

- `toString`

- `fillInStackTrace`

- `initCause`

- `getStackTrace` 见下面的遍历栈轨迹

- `getClass`

  对于异常来说，`getClass` 也许是个很好用的方法，它将返回一个表示此对象类型的对象。然后可以使用 `getName` 方法查询这个 Class 对象包含包信息的名称，或者使用只产生类名称的 `getSimpleName` 方法

**`fillInStackTrace`**：

调用 `filInStackTrace` 方法，将返回一个 Throwable 对象，它是通过把当前调用栈信息填入原来那个异常对象而建立的，就像这样：

``` java
void h() throws Exception {
    try {
        f();
    } catch(Exception e) {
        System.out.println(
            "Inside h(), e.printStackTrace()");
        e.printStackTrace(System.out);
        throw (Exception)e.fillInStackTrace();
    }
}
```

当然也可以在捕获异常之后抛出另一种异常（`throw new Exception()`）。这么做的话，得到的效果类似于使用 `filInStackTrace`，有关原来异常发生点的信息会丢失，剩下的是与新的抛出点有关的信息

 **`initCause`**：

`initCause` 方法和带 cause 参数的构造器的作用基本相同，但是如果要把其他类型的异常链接起来，应该使用 `initCause`方法而不是构造器：

``` java
DynamicFieldsException dfe =
    new DynamicFieldsException();
dfe.initCause(new NullPointerException());
throw dfe;

// ---------------------

try {
    result = getField(id); // Get old value
} catch(NoSuchFieldException e) {
    // Use constructor that takes "cause":
    throw new RuntimeException(e);
}
```

## 声明将抛出异常，但不抛出

声明方法将抛出异常，实际上却不抛出。编译器相信了这个声明，并强制此方法的用户像真的抛出异常那样使用这个方法。这样做的好处是，为异常先占个位子，以后就可以抛出这种异常而不用修改已有的代码。在定义抽象基类和接口时这种能力很重要，这样派生类或接口实现就能够抛出这些预先声明的异常。

- 其实在线代IDE帮助下，修改方法上抛出的异常变得更加容易，虽然往往也会出现错误。这么做的目的，很大程度上也是来**表达设计**，或者某种程度上的超前设计。

## 多重捕获

通过 Java 7 的多重捕获机制，你可以使用“或”将不同类型的异常组合起来，只需要一行 catch 语句：

``` java
void f() {
    try {
        x();
    } catch(Except1 | Except2 e) {
        process1();
    } catch(Except3 | Except4 e) {
        process2();
    }
}
```

## 遍历栈轨迹

(这可以在一个地方集中处理异常来发挥作用)

`getStackTrace`的数组中的最后一个元素和栈底是调用序列中的第一个方法调用。下面这个程序只打印了方法名，但实际上还可以打印整个 `StackTraceElement`，它包含其他附加的信息。

``` java
 void f() {
// Generate an exception to fill in the stack trace
     try {
         throw new Exception();
     } catch(Exception e) {
         for(StackTraceElement ste : e.getStackTrace())
             System.out.println(ste.getMethodName());
     }
 }
```



## “受检异常”转为“非受检异常”

把“被检查的异常”这种功能“屏蔽”掉的话，这看上去像是一个好办法。不用“吞下”异常，也不必把它放到方法的异常说明里面，而异常链还能保证你不会丢失任何原始异常的信息。

这种做法有几个用处：

- 不想显式的抛出异常，即不想在方法上写throws
- 我们想要不写 try-catch 子句和/或异常说明，直接忽略异常，使代码看起来“好看”
- 之前的接口设计没有写throws，现在的一些接口实现需要抛出异常，则这个时候可以进行这样的转化，之后再取出来

这种转化有几种写法：

- 创建自己的 RuntimeException 的子类
- 把异常传入带 cause 参数的构造器的RuntimeException异常： `throw new RuntimeException(e);`
- 使用`initCause`方法

转化为非受检异常后，它自己会沿着调用栈往上“冒泡”，同时，还可以用 `getCause()` 捕获并处理特定的异常，就像这样：

``` java
class WrapCheckedException {
    void throwRuntimeException(int type) {
        try {
            switch(type) {
                case 0: throw new FileNotFoundException();
                case 1: throw new IOException();
                case 2: throw new
                        RuntimeException("Where am I?");
                default: return;
            }
        } catch(IOException | RuntimeException e) {
            // Adapt to unchecked:
            throw new RuntimeException(e);
        }
    }
}
class SomeOtherException extends Exception {}
public class TurnOffChecking {
    public static void main(String[] args) {
        WrapCheckedException wce =
                new WrapCheckedException();
        // You can call throwRuntimeException() without
        // a try block, and let RuntimeExceptions
        // leave the method:
        wce.throwRuntimeException(3);
        // Or you can choose to catch exceptions:
        for(int i = 0; i < 4; i++)
            try {
                if(i < 3)
                    wce.throwRuntimeException(i);
                else
                    throw new SomeOtherException();
            } catch(SomeOtherException e) {
                System.out.println(
                        "SomeOtherException: " + e);
            } catch(RuntimeException re) {
                try {
                    throw re.getCause();
                } catch(FileNotFoundException e) {
                    System.out.println(
                            "FileNotFoundException: " + e);
                } catch(IOException e) {
                    System.out.println("IOException: " + e);
                } catch(Throwable e) {
                    System.out.println("Throwable: " + e);
                }
            }
    }
}
```



## 某些难以查找的问题

🔴程序员们只做最简单的事情，常常是无意中"吞食”了异常，然而一旦这么做，虽然能通过编译，但除非你记得复查并改正代码，否则异常将会丢失。异常确实发生了，但“吞食”后它却完全消失了。因为编译器强迫你立刻写代码来处理异常，所以这种看起来最简单的方法，却可能是最糟糕的做法。

这个话题看起来简单，但实际上它不仅复杂，更重要的是还非常多变。总有人会顽固地坚持自己的立场，声称正确答案（也是他们的答案）是显而易见的。

`OutOfMemoryError`、`NullPointerException` 什么情况下会碰到？如何避免？

## 异常指南

应该参照下列情况处理异常：

1. 尽可能使用 try-with-resource
2. 在恰当的级别处理问题（在知道该如何处理的情况下才捕获异常。）
3. 解决问题并且重新调用产生异常的方法
4. 进行少许修补，然后绕过异常发生的地方继续执行
5. 用别的数据进行计算，以代替方法预计会返回的值
6. 把当前运行环境下能做的事情尽量做完，然后把相同的异常重抛到更高层
7. 把当前运行环境下能做的事情尽量做完，然后把不同的异常抛到更高层
8. 终止程序
9. 进行简化（如果你的异常模式使问题变得太复杂，那用起来会非常痛苦也很烦人。）
10. 让类库和程序更安全（这既是在为调试做短期投资，也是在为程序的健壮性做长期投资。）

对异常的处理：

1. 尽可能使用 try-with-resource
2. finally中清理资源
3. 明确鲜明的自定义异常类名
4. 优先捕获更为具体的异常
5. 不忽略异常
6. 不在finally中使用return

有两种方式转换异常：

- 非受检异常转为受检异常
- 在finally里重新抛出新的类型异常（也叫异常屏蔽）

# 总结

可以回答一些问题：

- 抛出什么、在哪里抛出、为什么抛出
- 什么异常需要捕获，什么异常不必要捕获
- 常见的Exception和Error以及抛出原因

# 参考

1. Java编程思想，On Java 8
2.  [部分例子参考](https://blog.csdn.net/ThinkWon/article/details/101681073)