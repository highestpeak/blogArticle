这一部分也可以叫做最佳实践

# Iterator & Iterable

- 从高层角度看，要使用容器，必须针对**容器的确切类型**编程，但是迭代器可以改变这个情形
  - 试想原本对着List编码，但是后来发现把相同的代码应用于Set会显得非常方便，如果从头开始，啊，那你真是一个伟大的码农，一个站在天台码农
  - 接收对象容器并传递它，从而在每个对象上都执行操作（？？？什么意思，尚未搞懂这一句话 todo: 懒）
- 迭代器是一个**轻量级对象**：**创建的代价很小**
- JAVA的Iterator只能**单向移动**
- 常用方法：iterator、next、hasNext、remove
- 还可以**移除由next产生的最后一个元素**，即先调用next在调用remove会删除之前的一个元素
- iterator的真正威力：**能够将遍历序列的操作与序列底层的结构分离**，由此而言，**迭代器统一了对容器的访问方式**（例子 todo: 懒）
- ListIterator可以**双向移动**
  - listIterator(n)从索引为n的元素的listIterator，listIterator()为从开始处
- todo: 懒 如果使用栈的行为，使用继承就不合适了，这样会产生具有LinkedList的其他所有方法的类
- todo: 懒 foreach 与 迭代器
  - Iterable接口：任何实现了该接口的类都可以用于foreach，collection也可以应用于foreach，数组也可以应用于foreach，但是数组不是iterable
  - 迭代器也可以重载为向前的方向或向后的方向（例子 todo: 懒）

# Generator

> 从generator创建数组

todo: 懒

> ...创建List

todo: 懒

> ...创建Map

todo: 懒

# ArrayList 循环遍历时删除元素问题

也可以是所有集合的共性问题

https://blog.csdn.net/qq_41936561/article/details/104742305?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1



