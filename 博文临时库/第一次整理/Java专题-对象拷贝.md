

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

