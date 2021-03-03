# 工具类库

对于工具类库，可以进一步看一看Guava的工具类库

<u>*什么时候用`Arrays.copy`，底层原理，`Collections.sort`底层原理*</u>

Arrays.fill

Arrays.copy

Arrays.equal

comparable接口

`Arrays.copyOf()` 

Arrays.asList方法接受一个数组或是一个用都好分割的元素列表，并将其转化成一个List对象

Collections.addAll接受一个Collection对象、一个数组或是一个逗号分割的列表

这些在util工具类库文章中有

Collections.nCopies()

Collection.fill：他只能替换List中已经存在的元素，而不能添加新的元素？剩余容量也不行吗？

Collections. unmodifiableCollection(Collection c) 方法来创建一个只读集合

#### 数组和 List 之间的转换？

- 数组转 List：使用 Arrays. asList(array) 进行转换。
- List 转数组：使用 List 自带的 toArray() 方法。

ArrayList 不是线程安全的，如果遇到多线程场景，可以通过 Collections 的 synchronizedList 方法将其转换成线程安全的容器后再使用。

# Arrays.copy & System.arraycopy

Arrays.copy就是用System.arraycopy实现的。

[StackOverflow上的讲解](https://stackoverflow.com/questions/2589741/what-is-more-efficient-system-arraycopy-or-arrays-copyof)

The difference is that `Arrays.copyOf` does not only copy elements, it also creates a new array. `System.arraycopy` copies into an existing array.

Here is the source for `Arrays.copyOf`, as you can see it uses `System.arraycopy` internally to fill up the new array:

``` java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

The other point is that `Arrays.copyOf` uses `System.arraycopy` under the hood. Therefore `System.arraycopy` is *on the face of it* should not be slower2 than `Arrays.copyOf`. But you can see (from the code quoted [above](https://stackoverflow.com/a/2589797/139985)) that `Arrays.copyOf` will in some cases use reflection to create the new array. So the performance comparison is not straightforward.

There are a couple of flaws in this analysis.

1. We are looking at the implementation code from a specific version of Java. These methods may change, invalidating previous assumptions about efficiency.
2. We are ignoring the possibility that the JIT compiler could do some clever special case optimization for these methods. And it apparently this does happen with `Arrays.copyOf`; see [Why is Arrays.copyOf 2 times faster than System.arraycopy for small arrays?](https://stackoverflow.com/questions/44487304/why-is-arrays-copyof-2-times-faster-than-system-arraycopy-for-small-arrays). This method is "intrinsic" in current-generation Java implementations, which means that the JIT compiler will ignore what is in the Java source code!



Collections.sort底层原理

如果没有传递comparable接口实现，那么就会最终调用一个ComparableTimSort进行归并排序？好像是这样的。



# 初始化 & 填充(添加元素)

java.util.**Arrays**和java.util.**Collection**、java.util.**Collections**在类库中提供了方便的添加元素的方法。另外基于设计模式的基本思想可以创建Generator解决方案。

## java.util.*

> 添加一组元素用于初始化

- Arrays.asList()方法接收一个数组或是一个逗号分隔的元素列表(可变参数列表)，并转换成为一个List对象

- Collections.addAll()方法接收一个Collection对象，以及一个数组或都好分割的列表(可变参数列表)

- Collection.addAll()方法接受一个Collection对象

- 三个方法的进一步分析

  - collection的构造器可以接收另一个Collection来初始化，但是Collection.addAll()运行起来**快**得多，**构造一个空构造器然后调用Collection.addAll()方法**很方便，是首选的方式
- Collection.addAll()不如另外两者更加灵活，因为他们接收**可变参数列表**
- 其他填充方式例如：Collections.nCopies()、Collection.fill()；但是注意两者创建的**引用指向**的是**相同的对象**，**fill()**方法更为有限，只能**替换**已经在List中存在的**元素**，不能添加新的元素

> Arrays.asList()

- 如果直接使用**Arrays.asList()输出的List**，那么由于这个List底层**是数组**，所以**不能改变这个list的尺寸，不能add或者delete**

- **传递基本类型数组的注意事项！**需要注意的是，Arrays.asList()接收的是一个参数列表，但并不是意味着可以把一个数组对象当作是它的参数。如下示例（虽然在诸多文章中雷同的出现，本处加以引用），示例中传入参数列表的是一个int数组，因此myList持有的是一个数组的列表，每一个元素都是一个数组。故而size是1。

  ``` java
  int[] myArray = { 1, 2, 3 };
  List myList = Arrays.asList(myArray);
  System.out.println(myList.size());
  //......
  //output size is 1
  ```

  需要注意的是这是**基本数据类型数组**，基本数据类型不是对象，如果使用Integer则可以size为3，即如下所示，因为Integer数组可以转化成为一组Integer的对象（这么说好像也并不是很通顺，暂时先这样吧）（关于可变参数可以自行Google）

  ``` java
  Integer[] myArray = { 1, 2, 3 };
  List myList = Arrays.asList(myArray);
  System.out.println(myList.size());
  //......
  //output size is 3
  ```

  （这种问题，在查看并简单的分析asList源码即可解决，这也是一个可以不用Google就可以快速解决问题的例子，值得在自己能力低下的时候警惕。）

  Arrays.asList()源码如下，可以注意到泛型T，传入的每一个参数都是T类型，传入一个数组并且数组内不能转化成为对象，则T自然是一个数组类型。

  ``` java
  @SafeVarargs
  @SuppressWarnings("varargs")
  public static <T> List<T> asList(T... a) {
      return new ArrayList<>(a);
  }
  ```
  
  Arrays.asList可以插入泛型例如：`Arrays.<Snow>asList` ，这样写告诉编译器对于asList产生的List类型，实际 的目标类型是什么，（这个对数组的介绍那里有点有用）
  
  toArray和这个泛型问题类似

# 数组比较、排序、查找

- equals用于比较两个数组是否相等，**`deepEquals`用于多维数组**

- 使用标准类库**`System.arraycopy`**方法**复制数组比**用**for循环**复制**快得多**

- 复制对象数组知识复制了对象的引用，而不是对象的拷贝，是**浅拷贝**

- 数组相等条件：元素**个数相等**、**对应位置元素相等**，这需要调用每一个元素的**equals**，因此需要注意equals方法的编写

- 使用**内置的排序方法**可以对任意**基本类型**、实现了**Comparable接口**的对象、具有关联的Comparator的对象进行排序
  - 如果数组**已经排好序**了，可以使用**`Arrays.binarySearch()`**执行快速查找，对未排序的数组调用结果未知。

