不能简单的说是重要接口，因为 list map等等也算是接口，应该说是使用时必须需要实现的接口

（这几个不足以写成一篇文章，可以在“接口”一文中单另一节）



- comparable
- cloneable
- serialable
- Runnable







comparable

cloneable  <u>*todo: 值得注意的是深拷贝和浅拷贝问题*</u>

深浅拷贝见`Java专题-对象拷贝`



在compareT欧中，没有使用“简洁明了”的形势 return i-i2，因为这是一个常见的编程错误，它只有在i和i2都是无符号的int时才能正确工作。对于有符号int就会出错，因为int不够大，不足以表现两个有符号int的查，例如i是一个很大的正整数，j是一个很大的负整数，i-j就会溢出并返回负值，通常会希望compareTo和equals方法一致的自然排序（0值和非0值）。



serialable

transient关键字



**clone() 的替代方案**

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

```
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```





### 序列化

ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。

```java
transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList 实现了 writeObject() 和 readObject() （均为private）来控制只序列化数组中有元素填充那部分内容。

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```

