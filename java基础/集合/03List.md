# 2 List

### 2.1 Vector类（类数组）

Vector类方法使用synchronized修饰的，也就是说是线程安全的。底层其实就是一个Object数组。

##### 2.1.1 原理

通过源码分析，发现在 Vector 类中有一个 **Object[] 类型数组**。

1. 表面上把数组存储到 Vector 对象中，其实底层依然是把数据存储到 Object 数组中。该数组的元素类型是 Object 类型，即是该集合中能且只能存储任意类型的对象。
2. **该集合中只能存储对象，不能存储基本数据类型的值**。
3. 在 Java5 之前，必须对基本数据类型进行手动装箱，**从 Java5 开始支持自动装箱**。
4. 集合类中存储的对象，实际上**存储的是对象的引用，而不是对象本身**。 



对第四点的证明：集合类中存储的对象都是对象的引用而不是本身

```java
Vector v = new Vector();
StringBuilder s = new StringBuilder("ABC");
v.addElement(s);
s.append("123");
System.out.println(v);
```



输出：

```
ABC123
```



##### 2.1.2 扩容

Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在扩容时使容量 capacity 增长 capacityIncrement。如果这个参数的值小于等于 0，扩容时每次都令 capacity 为原来的两倍。

```java
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

调用没有 capacityIncrement 的构造函数时，capacityIncrement 值被设置为 0，也就是说默认情况下 Vector 每次扩容时容量都会翻倍。

```java
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}

public Vector() {
    this(10);
}
```



##### 2.1.3 与 ArrayList 的比较

- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；
- Vector 每次扩容请求其大小的 2 倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍。







### 2.2 ArrayList（Vector的升级版）

ArrayList类是Java集合框架出现之后用来取代Vector类的，二者底层原理都是基于数组的算法，一般我们把它认为是可以自增扩容的数组。



##### 2.2.1 与Vector的区别

Vector所有的方法都使用了sychronized修饰，ArrayList全部方法都没有使用sychronized。



##### 2.2.2 如何保证线程安全

但是即便是在多线程环境下，也不使用Vector，我们可以用两种方法来保证线程安全：

可以使用 `Collections.synchronizedList();` 得到一个线程安全的 ArrayList。

```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
```

也可以使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

```java
List<String> list = new CopyOnWriteArrayList<>();
```



##### 2.2.3 初始构造器

*源码：jdk1.8*

ArrayList一共提供了三个初始化的方法： 

```java
public ArrayList()  // 无参

public ArrayList(Collection<? extends E> c)  //传入集合

public ArrayList(int initialCapacity)；//传入长度

```



1.无参构造方法的实现 

注释表示他会默认提供容量为10的数组，但是实际并不是在这一步实现。 (transient是序列化)

所以这一步实际上只是将elementData指向一个空数组而已。 

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.6.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.7.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.5.png)



2.带参数的构造方法

这个方法是直接将一个集合作为ArrayList的元素，此时elementData即为集合c转为的数组，size即为elementData的长度。这里size是ArrayList的一个int型私有变量，用于记录该list集合中当前元素的数量，**注意不是容量**。 

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.8.png)



3.初始化容量的构造方法 

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.9.png)



##### 2.2.4 add方法的实现

add方法：调用了ensureCapacityInternal保证容量问题

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.10.png)



ensureCapacityInternal方法：如果传入初始化长度<10，那么会默认长度为10，反之取传入的长度

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.12.png)



![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.11.png)



ensureExplicitCapacity方法：

可以看到modCount++，这个参数主要是用在集合的Fail-Fast机制(即快速失败机制)的判断中使用的。
在这个方法里进行判断，新增元素后的大小minCapacity是否超过当前集合的容量elementData.length，如果超过，则调用grow方法进行扩容。

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.13.png)



grow方法：

在这里可以很清楚的看到扩容容量的计算:int newCapacity = oldCapacity + (oldCapacity >> 1),其中oldCapacity是原来的容量大小，oldCapacity >> 1 为位运算的右移操作，右移一位相当于除以2，所以这句代码就等于int newCapacity = oldCapacity + oldCapacity / 2；即容量扩大为原来的1.5倍(注意我这里使用的是jdk1.8，没记错的话1.7也是一样的)，获取newCapacity后再对newCapacity的大小进行判断，如果仍然小于minCapacity，则直接让newCapacity 等于minCapacity，而不再计算1.5倍的扩容。然后还要再进行一步判断，即判断当前新容量是否超过最大的容量 if (newCapacity - MAX_ARRAY_SIZE > 0)，如果超过，则调用hugeCapacity方法，传进去的是minCapacity，即新增元素后需要的最小容量。

这里Arrays.copyof方法实际是调用System.arraycopy方法。 

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.16.png)



![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.14.png)



hugeCapacity方法：如果minCapacity大于MAX_ARRAY_SIZE，则返回Integer的最大值。否则返回MAX_ARRAY_SIZE。 

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.15.png)



总结：与Vector不同的是，Vector每次扩容容量是翻倍，即为原来的2倍，而ArrayList是1.5倍。 



##### 2.2.5 get方法的实现

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.17.png)



##### 2.2.6 remove

删除元素时需要调用 System.arraycopy() 对元素进行复制，因此删除操作成本很高。

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```



##### 2.2.7 size&modCount

size记录了ArrayList中元素的数量，modCount记录的是关于元素的数目被修改的次数，每次元素数量改变一次，modCount就要++，修改不算。



##### 2.2.8 序列化

ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。

```java
transient Object[] elementData;
```

ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而  writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject()  来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```





##### 2.2.9 总结

总之，ArrayList默认容量是10，如果初始化时一开始指定了容量，或者通过集合作为元素，则容量为指定的大小或参数集合的大小。每次扩容为原来的1.5倍，如果新增后超过这个容量，则容量为新增后所需的最小容量。如果增加0.5倍后的新容量超过限制的容量，则用所需的最小容量与限制的容量进行判断，超过则指定为Integer的最大值，否则指定为限制容量大小。然后通过数组的复制将原数据复制到一个更大(新的容量大小)的数组。



### 2.3 CopyOnWriteArrayList

##### 2.3.1 读写分离

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

写操作需要加锁，防止并发写入时导致写入数据丢失。

写操作结束之后需要把原始数组指向新的复制数组。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;
}
```

```java
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

###  

##### 2.3.2 适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。



### 2.4 LinkedList

线程不安全。是双向链表，单向队列，双向队列，栈的实现类。

LinkedList 底层的数据结构是基于双向循环链表的，且头结点中不存放数据,数据添加删除效率高，只需要改变指针指向即可，但是访问数据的平均效率低，需要对链表进行遍历。 

基于双向链表实现，使用 Node 存储链表节点信息。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

每个链表存储了 first 和 last 指针：

```java
transient Node<E> first;
transient Node<E> last;
```



##### 2.4.1 如何保证线程安全

```java
List<String> list = Collections.synchronizedList(new LinkedList<String>());
```

```
LinkedList换成ConcurrentLinkedQueue 
```

建议第二种



##### 2.4.2 与 ArrayList 的比较

ArrayList 基于动态数组实现，LinkedList 基于双向链表实现。ArrayList 和 LinkedList 的区别可以归结为数组和链表的区别：

- 数组支持随机访问，但插入删除的代价很高，需要移动大量元素；
- 链表不支持随机访问，但插入删除只需要改变指针。