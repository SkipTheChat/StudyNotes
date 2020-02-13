# List

## 1 Vector类（类数组）

Vector类方法使用synchronized修饰的，也就是说是线程安全的。底层其实就是一个Object数组。

### 1.1 原理

通过源码分析，发现在 Vector 类中有一个 **Object[] 类型数组**。

1. 其底层把数据存储到 Object 数组中。该数组的元素类型是 Object 类型，即是该集合中能且只能存储任意类型的对象。
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



### 1.1.2 扩容

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



### 1.1.3 与 ArrayList 的比较

- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；
- Vector 每次扩容请求其大小的 2 倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍。





## 2 ArrayList（Vector的升级版）

ArrayList 的底层是数组队列，相当于自增扩容的动态数组。在添加大量元素前，应用程序可以使用`ensureCapacity`操作来增加 ArrayList 实例的容量。

   它继承于 **AbstractList**，实现了 **List**, **RandomAccess**, **Cloneable**, **java.io.Serializable** 这些接口。

- ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
- ArrayList 实现了**RandomAccess 接口**， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
- ArrayList 实现了**Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。
- ArrayList 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。

和 Vector 不同，**ArrayList 中的操作不是线程安全的，Vector所有的方法都使用了sychronized修饰**，所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 **Vector 或者  CopyOnWriteArrayList**。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```



### 2.1 属性

```java
  private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空数组（用于空实例）。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};


     //用于默认大小空实例的共享空数组实例。
      //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};


	/**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; 


    /**
     * ArrayList 所包含的元素个数
     */
    private int size;
```



### 2.2 构造器

-  `public ArrayList()` ：初始其实是空数组，当添加第一个元素的时候数组容量才变成10，避免了空间浪费。
- `public ArrayList(int initialCapacity)`：带初始容量参数的构造函数（用户自己指定容量）
- `ArrayList(Collection<? extends E> c)`： 先将集合转为数组，然后判断数组是否是Object[].class类型，如果不是的话，就进行转换，其实就是数组拷贝 `elementData = Arrays.copyOf(elementData, size, Object[].class);`

```java
  /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     *默认构造函数，DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        //
        elementData = c.toArray();
        //如果指定集合元素个数不为0
        if ((size = elementData.length) != 0) {
            // c.toArray 可能返回的不是Object类型的数组所以加上下面的语句用于判断，
            //这里用到了反射里面的getClass()方法
            if (elementData.getClass() != Object[].class)
            //数组拷贝，转为了Object[].class类型
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

```



### 2.3 扩容

1. `ensureCapacityInternal(minCapacity )`：数组如果为空，设置minCapacity = max（10，minCapacity）
2. 接着调用 `ensureExplicitCapacity(minCapacity)`：判断如果minCapacity还是大于当前数组实际长度，就调用grow(minCapacity)
3. `grow(minCapacity)`：int newCapacity = oldCapacity + (oldCapacity >> 1)，扩容1.5倍。
   * `newCapacity < minCapacity：`直接newCapacity = minCapacity
   * `newCapacity > MAX_ARRAY_SIZE：` newCapacity = hugeCapacity(minCapacity);
   * 最后elementData = Arrays.copyOf(elementData, newCapacity);完成
4.  `hugeCapacity(minCapacity)`： 

```java
return (minCapacity > MAX_ARRAY_SIZE) ?

	 Integer.MAX_VALUE :  MAX_ARRAY_SIZE;
```

> 注：MAX_ARRAY_SIZE的大小为 Integer.MAX_VALUE - 8;



```java
 /**
     * 修改这个ArrayList实例的容量是列表的当前大小。 size是elementData实际元素个数
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

//下面是ArrayList的扩容机制
//ArrayList的扩容机制提高了性能，插入数据会导致数组拷贝，扩容避免了频繁的拷贝
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0 : DEFAULT_CAPACITY;
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        //数组如果为空，设置max（10，minCapacity）
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
		//扩容
        ensureExplicitCapacity(minCapacity);
    }

  //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
		//如果minCapacity还是大于当前长度，就调用grow
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，相当于newCapacity = 1.5*oldCapacity
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
      
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    //比较minCapacity和 MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

```



### 2.4 add()

1. 调用ensureCapacityInternal方法
2. 设置值

```java
  /**
     * 将指定的元素追加到此列表的末尾。 
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    public void add(int index, E element) {
        //边界检查
        rangeCheckForAdd(index);
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        
        //将从index开始之后的所有成员后移一个位置
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

```



### 2.5 remove()

```java
  /**
     * 删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。 
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
      //从列表中删除的元素 
        return oldValue;
    }

    /**
     * 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。
     *返回true，如果此列表包含指定的元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

   /**
     * 从此列表中删除指定集合中包含的所有元素。 
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        //如果此列表被修改则返回true
        return batchRemove(c, false);
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

```



### 2.6 set() & get()

```java
    /**
     * 返回此列表中指定位置的元素。
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此列表中指定位置的元素。 
     */
    public E set(int index, E element) {
        //对index进行界限检查
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        //返回原来在这个位置的元素
        return oldValue;
    }
```



### 2.7 检查边界方法

```java
   /**
     * 从此列表中删除所有索引为fromIndex （含）和toIndex之间的元素。
     *将任何后续元素移动到左侧（减少其索引）。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内。
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add和addAll使用的rangeCheck的一个版本
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 返回IndexOutOfBoundsException细节信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

 

```



### 2.8clone()&toArray()

```java

    /**
     * 返回此ArrayList实例的浅拷贝。 （元素本身不被复制。） 
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。 
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 
     *返回的数组的运行时类型是指定数组的运行时类型。 如果列表适合指定的数组，则返回其中。 
     *否则，将为指定数组的运行时类型和此列表的大小分配一个新数组。 
     *如果列表适用于指定的数组，其余空间（即数组的列表数量多于此元素），则紧跟在集合结束后的数组中的元素设置为null 。
     *（这仅在调用者知道列表不包含任何空元素的情况下才能确定列表的长度。） 
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 新建一个运行时类型的数组，但是ArrayList数组的内容
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
            //调用System提供的arraycopy()方法实现数组之间的复制
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

```



### 2.9 其他方法

```java
    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

  
    public boolean contains(Object o) {
        //indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1 
        return indexOf(o) >= 0;
    }

    /**
     *返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1 
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //equals()方法比较
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1。.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

  
    /**
     * 从列表中删除所有元素。 
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

 
    /**
     * 仅保留此列表中包含在指定集合中的元素。
     *换句话说，从此列表中删除其中不包含在指定集合中的所有元素。 
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     *指定的索引表示初始调用将返回的第一个元素为next 。 初始调用previous将返回指定索引减1的元素。 
     *返回的列表迭代器是fail-fast 。 
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     *返回列表中的列表迭代器（按适当的顺序）。 
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     *以正确的顺序返回该列表中的元素的迭代器。 
     *返回的迭代器是fail-fast 。 
     */
    public Iterator<E> iterator() {
        return new Itr();

```



### 2.10 内部类

```java
private class Itr implements Iterator<E>  
private class ListItr extends Itr implements ListIterator<E>  
private class SubList extends AbstractList<E> implements RandomAccess  
static final class ArrayListSpliterator<E> implements Spliterator<E>  
```

　ArrayList有四个内部类，其中的**Itr是实现了Iterator接口**，同时重写了里面的**hasNext()**， **next()**， **remove()** 等方法；其中的**ListItr** 继承 **Itr**，实现了**ListIterator接口**，同时重写了**hasPrevious()**， **nextIndex()**， **previousIndex()**， **previous()**， **set(E e)**， **add(E e)** 等方法，所以这也可以看出了 **Iterator和ListIterator的区别:** ListIterator在Iterator的基础上增加了添加对象，修改对象，逆向遍历等方法，这些是Iterator不能实现的。 



### 2.11 如何保证线程安全

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



### 2.12 Arrays.copyOf()&System.arraycopy()

两者的区别在于，Arrays.copyOf()不仅仅只是拷贝数组中的元素，在拷贝元素时，会创建一个新的数组对象。而System.arrayCopy只拷贝已经存在数组元素。

**如果我们看过Arrays.copyOf()的源码就会知道，该方法的底层还是调用了System.arrayCopyOf()方法。**

```java
public static int[] copyOf(int[] original, int newLength) { 
   int[] copy = new int[newLength]; 
   System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength)); 
   return copy; 
}
```

**System.arrayCopyOf()是一个native本地方法：**

```java
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```







### 2.13 modCount->fail-fast机制

[fail-fast机制](https://blog.csdn.net/chenssy/article/details/38151189)

modCount记录的是关于元素的数目被修改的次数，每次元素数量改变一次，modCount就要++。

> 如add、remove、clear方法，不包括修改



ArrayList中迭代器的源代码 :

可以看到  checkForComodification中的判断：  `if (ArrayList.this.modCount == this.expectedModCount)`

迭代器中，remove或者next都需要进行checkForComodification()

```java
  private class Itr implements Iterator<E> {

        int cursor;

        int lastRet = -1;

        int expectedModCount = ArrayList.this.modCount;

      public boolean hasNext() {
        return (this.cursor != ArrayList.this.size);
    }
 
    public E next() {
        checkForComodification();
        /** 省略此处代码 */
    }
 
    public void remove() {
        if (this.lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();
        /** 省略此处代码 */
    }
 
    final void checkForComodification() {
        if (ArrayList.this.modCount == this.expectedModCount)
            return;
        throw new ConcurrentModificationException();
    }
}
```


避免fast-fail的方法：

- 方案一：在遍历过程中所有涉及到改变modCount值得地方全部加上synchronized或者直接使用Collections.synchronizedList，这样就可以解决。但是不推荐，因为增删造成的同步锁可能会阻塞遍历操作。
-  方案二：使用CopyOnWriteArrayList来替换ArrayList。推荐使用该方案。



### 2.14 序列化

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





## 3 CopyOnWriteArrayList-写入时复制

[博客](https://www.cnblogs.com/leesf456/p/5547853.html)

是对List进行操作的一个线程安全的类，使用ReentrantLock锁

每次修改时，都会创建并生成一个新的不可变的对象，写操作在复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

写操作需要加锁，防止并发写入时导致写入数据丢失。

写操作结束之后需要把原始数组指向新的复制数组。

### 3.1 常用属性

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable 
```

属性中有一个可重入锁，用来保证线程安全访问，还有一个Object类型的数组，用来存放具体的元素。当然，也使用到了反射机制和CAS来保证原子性的修改lock域。 

```java
    // 版本序列号
    private static final long serialVersionUID = 8673264195747942595L;
    // 可重入锁
    final transient ReentrantLock lock = new ReentrantLock();
    // 对象数组，用于存放元素
    private transient volatile Object[] array;
    // 反射机制
    private static final sun.misc.Unsafe UNSAFE;
    // lock域的内存偏移量
    private static final long lockOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = CopyOnWriteArrayList.class;
            lockOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("lock"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
```



### 3.2 构造类

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

```

继承结构比较常规，可以随机访问(RandomAccess)，可以复制(Cloneable)，可以序列化(Serializable )，然后来看下构造方法： 

```java
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}

public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    // 类型是否相同
    if (c.getClass() == CopyOnWriteArrayList.class)
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        // 不同，先转为数组，然后判断是否为Object数组类型
        elements = c.toArray();
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    // 设置数组
    setArray(elements);
}

public CopyOnWriteArrayList(E[] toCopyIn) {
    // 转化为Object数组类型，然后设置数组
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}


final void setArray(Object[] a) {
    array = a;
}

```



### 3.3 迭代器

该类用于表示迭代器，提供了一个基础数组array的快照，该迭代器保留了一个指向底层基础数组的引用，该数组不会被修改，因此在对其进行操作的时候只需确保数组内容的可见性，这样多个线程可以同时对这个容器进行迭代，而不会发生冲突。

```java
static final class COWIterator<E> implements ListIterator<E> {
    /** 基础数组array的快照 */
    private final Object[] snapshot;
    /** Index of element to be returned by subsequent call to next.  */
    /** 可以翻译为游标 */
    private int cursor;

    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements;
    }

    public boolean hasNext() {
        return cursor < snapshot.length;
    }

    public boolean hasPrevious() {
        return cursor > 0;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        if (! hasNext())
            throw new NoSuchElementException();
        return (E) snapshot[cursor++];
    }

    @SuppressWarnings("unchecked")
    public E previous() {
        if (! hasPrevious())
            throw new NoSuchElementException();
        return (E) snapshot[--cursor];
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    /** 不支持add,set,remove操作 */
    public void remove() {
        throw new UnsupportedOperationException();
    }
    public void set(E e) {
        throw new UnsupportedOperationException();
    }
    public void add(E e) {
        throw new UnsupportedOperationException();
    }

    @Override
    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        Object[] elements = snapshot;
        final int size = elements.length;
        for (int i = cursor; i < size; i++) {
            @SuppressWarnings("unchecked") E e = (E) elements[i];
            action.accept(e);
        }
        cursor = size;
    }
}

```

 

### 3.4 add()

##### 3.4.1 add(E e)

add方法很简单：

- 首先，获取锁，获取数组及数组长度；
- 其次，在原数组基础上复制一个新的数组，新的数组比原数组长度多1；
- 然后把新的元素添加到数组的尾部；
- 最后把该数组赋值给基础数组array

**可以看到并没有使用扩容机制，每次add都会引起底层的数组复制**

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



##### 3.4.2 add(int index, E element)

而add的另一个重载方法就稍微复杂一点点 ：

主要是加锁，判断了下index的大小，然后根据index的大小进行相应的复制操作：

- 如果index>len或者<0,报错
- 如果index = len，`newElements = Arrays.copyOf(elements, len + 1);`
- 如果计算的要插入位置在之间，就进行两次复制，空出index的位置

最后unlock()解锁

```java
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 判断索引是否合法
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        // 构造新数组
        Object[] newElements;
        // 计算元素要保存的位置的下标值
        int numMoved = len - index;
        // 如果元素是数组最后一个元素，操作和上面描述的一个参数的add方法操作相同
        if (numMoved == 0)
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            // 否则，构造新的数组，长度是 基础数组长度加1
            newElements = new Object[len + 1];
            // 进行两次复制，先复制索引前的元素，再复制索引后的元素
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        // 将新元素存进去，并更新基础数组
        newElements[index] = element;
        setArray(newElements);
    } finally {
        lock.unlock();
    }
}

```



##### 3.4.3 addIfAbsent(E e) 

该方法表示添加的时候如果数组中不存在，则进行添加；如果存在，则不添加，直接返回

流程：

- 首先还是相同的操作，获取锁，获取当前数组，获取数组长度；
- 然后判断原先的快照数组和当前的数组是否相等，不相等则说明数组发生了修改；
- 这个时候取数组长度较小的值，进行遍历操作；
- 如果当前数组元素与快照数组元素不相等，并且要添加的元素与当前数组元素相等，说明快照与当前数组current之间，数组发生了修改，并且设置了数组某一元素为e，说明已经存在，直接返回；
- 如果上述条件没有发生，则从下标common开始再次进行查找操作；
- 如果查找不到，进行后续的添加操作；

```java
public boolean addIfAbsent(E e) {
    Object[] snapshot = getArray();
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
        addIfAbsent(e, snapshot);
}

private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        // 如果快照和当前数组不相等，说明数组发生了修改
        if (snapshot != current) {
            // Optimize for lost race to another addXXX operation
            // 这个时候取数组长度较小的值，这里是一个优化操作
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                // 如果当前数组元素与快照数组元素不相等，并且要添加的元素与当前数组元素相等
                // 说明快照与当前current之间数组发生了修改，并且设置了数组某一元素为e，已经存在，直接返回
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            // 在当前数组中查找元素
            if (indexOf(e, current, common, len) >= 0)
                    return false;
        }
        // 进行后续的添加操作
        Object[] newElements = Arrays.copyOf(current, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

```





### 3.5 indexOf()

可以看到读操作是不加锁的

```java
public int indexOf(Object o) {
    Object[] elements = getArray();
    return indexOf(o, elements, 0, elements.length);
}

private static int indexOf(Object o, Object[] elements,
                           int index, int fence) {
    // 如果要查找的元素是null
    if (o == null) {
        for (int i = index; i < fence; i++)
            // 如果有值是null的直接返回
            if (elements[i] == null)
                return i;
    } else {
        // 遍历，根据equals方法进行判断
        for (int i = index; i < fence; i++)
            if (o.equals(elements[i]))
                return i;
    }
    return -1;
}

```



### 3.6 set()

```java
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        // 先获取index处的元素值
        E oldValue = get(elements, index);
        // 比较下旧的值和新的值是否相同
        if (oldValue != element) {
            // 不相同，复制一个新的数组，并设置对应index处的值为新的值
            // 再更新数组
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len);
            newElements[index] = element;
            setArray(newElements);
        } else {
            setArray(elements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}

```



### 3.7 Array.copyof方法

CopyOnWriteArrayList这个类的操作基本都是通过`Array.copyof`方法来实现的，我们顺便来看下这个方法的源码：

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        // 确定copy的类型（将newType转化为Object类型，将Object[].class转化为Object类型，判断两者是否相等，若相等，则生成指定长度的Object数组
        // 否则,生成指定长度的新类型的数组）
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        // 将original数组从下标0开始，复制长度为(original.length和newLength的较小者),复制到copy数组中（也从下标0开始）
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```



### 3.8 CopyOnWriteArraySet

而CopyOnWriteArraySet实现元素的不重复，则是借助于CopyOnWriteArrayList中的`addIfAbsent`和`addAllAbsent`方法来实现的。比如： 

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    public CopyOnWriteArraySet(Collection<? extends E> c) {
        if (c.getClass() == CopyOnWriteArraySet.class) {
            @SuppressWarnings("unchecked") CopyOnWriteArraySet<E> cc =
                (CopyOnWriteArraySet<E>)c;
            al = new CopyOnWriteArrayList<E>(cc.al);
        }
        else {
            al = new CopyOnWriteArrayList<E>();
            al.addAllAbsent(c);
        }
    }

    public boolean add(E e) {
        return al.addIfAbsent(e);
    }
}
```



### 3.9 适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。



## 4 LinkedList

[博客](http://www.tianxiaobo.com/2018/01/31/LinkedList-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-JDK-1-8/)

（1）LinkedList是一个实现了List接口和Deque接口，所以队列、双端队列、栈的特性；

（3）LinkedList在队列首尾添加、删除元素非常高效，时间复杂度为O(1)；

（4）LinkedList在中间添加、删除元素比较低效，时间复杂度为O(n)；

（5）LinkedList不支持随机访问，所以访问非队列首尾的元素比较低效；

（6）LinkedList在功能上等于ArrayList + ArrayDeque；

（7） LinkedList 基于链表实现，存储元素过程中，无需像 ArrayList  那样进行扩容。但有得必有失，LinkedList 存储元素的节点需要额外的空间存储前驱和后继的引用。

（8）LinkedList 是非线程安全的集合类



### 4.1 内部结构

基于双向链表实现，使用 Node 存储链表节点信息。

```java
private static class Node<E> {
        E item;//节点值
        Node<E> next;//后继节点
        Node<E> prev;//前驱节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```



### 4.2 主要属性

```java
// 元素个数
transient int size = 0;
// 链表首节点
transient Node<E> first;
// 链表尾节点
transient Node<E> last;

```



### 4.3 主要构造方法

```java

```



### 4.4 查找

LinkedList 底层基于链表结构，无法向 ArrayList 那样随机访问指定位置的元素，需要从链表头结点（或尾节点）向后查找，时间复杂度为 O(N)。相关源码如下：

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

Node<E> node(int index) {
    /*
     * 则从头节点开始查找，否则从尾节点查找
     * 查找位置 index 如果小于节点数量的一半，
     */    
    if (index < (size >> 1)) {
        Node<E> x = first;
        // 循环向后查找，直至 i == index
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```



### 4.5 遍历

链表的遍历过程也很简单，和上面查找过程类似，我们从头节点往后遍历就行了。

但对于 LinkedList 的遍历还是需要注意一些，不然可能会导致代码效率低下。通常情况下，我们会使用 `foreach` 遍历 LinkedList，**而 foreach 最终转换成迭代器形式**。所以分析 LinkedList 的遍历的核心就是它的迭代器实现，相关代码如下：

```java
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}

private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned;
    private Node<E> next;
    private int nextIndex;
    private int expectedModCount = modCount;

    /** 构造方法将 next 引用指向指定位置的节点 */
    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }

    public E next() {
        checkForComodification();
        if (!hasNext())
            throw new NoSuchElementException();

        lastReturned = next;
        next = next.next;    // 调用 next 方法后，next 引用都会指向他的后继节点
        nextIndex++;
        return lastReturned.item;
    }
    
    // 省略部分方法
}
```



以下是一个典型的性能很差的例子： **LinkedList 不擅长随机位置访问，如果大家用随机访问的方式遍历 LinkedList，效率会很差** 

```java
List<Integet> list = new LinkedList<>();
list.add(1)
list.add(2)
......
for (int i = 0; i < list.size(); i++) {
    Integet item = list.get(i);
    // do something
}
```



### 4.6 添加元素

作为一个双端队列，添加元素主要有两种，一种是在队列尾部添加元素，一种是在队列首部添加元素。

```java
// 从队列首添加元素
private void linkFirst(E e) {
    final Node<E> f = first;
    // 创建新节点，新节点的next是首节点
    final Node<E> newNode = new Node<>(null, e, f);
    // 让新节点作为新的首节点
    first = newNode;
    // 判断是不是第一个添加的元素
    // 如果是就把last也置为新节点
    // 否则把原首节点的prev指针置为新节点
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    // 元素个数加1
    size++;
    // 修改次数加1，说明这是一个支持fail-fast的集合
    modCount++;
}

// 从队列尾添加元素
void linkLast(E e) {
    final Node<E> l = last;
    // 创建新节点，新节点的prev是尾节点
    final Node<E> newNode = new Node<>(l, e, null);
    // 让新节点成为新的尾节点
    last = newNode;
    // 判断是不是第一个添加的元素
    // 如果是就把first也置为新节点
    // 否则把原尾节点的next指针置为新节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    // 元素个数加1
    size++;
    // 修改次数加1
    modCount++;
}

public void addFirst(E e) {
    linkFirst(e);
}

public void addLast(E e) {
    linkLast(e);
}

// 作为无界队列，添加元素总是会成功的
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}

```



作为List，是要支持在中间添加元素的，主要是通过下面这个方法实现的。 

```java
// 在节点succ之前添加元素
void linkBefore(E e, Node<E> succ) {
    // succ是待添加节点的后继节点
    // 找到待添加节点的前置节点
    final Node<E> pred = succ.prev;
    // 在其前置节点和后继节点之间创建一个新节点
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 修改后继节点的前置指针指向新节点
    succ.prev = newNode;
    // 判断前置节点是否为空
    // 如果为空，说明是第一个添加的元素，修改first指针
    // 否则修改前置节点的next为新节点
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    // 修改元素个数
    size++;
    // 修改次数加1
    modCount++;
}

// 寻找index位置的节点
Node<E> node(int index) {
    // 因为是双链表
    // 所以根据index是在前半段还是后半段决定从前遍历还是从后遍历
    // 这样index在后半段的时候可以少遍历一半的元素
    if (index < (size >> 1)) {
        // 如果是在前半段
        // 就从前遍历
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        // 如果是在后半段
        // 就从后遍历
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

// 在指定index位置处添加元素
public void add(int index, E element) {
    // 判断是否越界
    checkPositionIndex(index);
    // 如果index是在队列尾节点之后的一个位置
    // 把新节点直接添加到尾节点之后
    // 否则调用linkBefore()方法在中间添加节点
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

```



### 4.7 删除元素

作为双端队列，删除元素也有两种方式，一种是队列首删除元素，一种是队列尾删除元素。

作为List，又要支持中间删除元素，所以删除元素一个有三个方法，分别如下。

```java
// 删除首节点
private E unlinkFirst(Node<E> f) {
    // 首节点的元素值
    final E element = f.item;
    // 首节点的next指针
    final Node<E> next = f.next;
    // 添加首节点的内容，协助GC
    f.item = null;
    f.next = null; // help GC
    // 把首节点的next作为新的首节点
    first = next;
    // 如果只有一个元素，删除了，把last也置为空
    // 否则把next的前置指针置为空
    if (next == null)
        last = null;
    else
        next.prev = null;
    // 元素个数减1
    size--;
    // 修改次数加1
    modCount++;
    // 返回删除的元素
    return element;
}
// 删除尾节点
private E unlinkLast(Node<E> l) {
    // 尾节点的元素值
    final E element = l.item;
    // 尾节点的前置指针
    final Node<E> prev = l.prev;
    // 清空尾节点的内容，协助GC
    l.item = null;
    l.prev = null; // help GC
    // 让前置节点成为新的尾节点
    last = prev;
    // 如果只有一个元素，删除了把first置为空
    // 否则把前置节点的next置为空
    if (prev == null)
        first = null;
    else
        prev.next = null;
    // 元素个数减1
    size--;
    // 修改次数加1
    modCount++;
    // 返回删除的元素
    return element;
}
// 删除指定节点x
E unlink(Node<E> x) {
    // x的元素值
    final E element = x.item;
    // x的前置节点
    final Node<E> next = x.next;
    // x的后置节点
    final Node<E> prev = x.prev;
    
    // 如果前置节点为空
    // 说明是首节点，让first指向x的后置节点
    // 否则修改前置节点的next为x的后置节点
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    // 如果后置节点为空
    // 说明是尾节点，让last指向x的前置节点
    // 否则修改后置节点的prev为x的前置节点
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    // 清空x的元素值，协助GC
    x.item = null;
    // 元素个数减1
    size--;
    // 修改次数加1
    modCount++;
    // 返回删除的元素
    return element;
}
// remove的时候如果没有元素抛出异常
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
// remove的时候如果没有元素抛出异常
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
// poll的时候如果没有元素返回null
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
// poll的时候如果没有元素返回null
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
// 删除中间节点
public E remove(int index) {
    // 检查是否越界
    checkElementIndex(index);
    // 删除指定index位置的节点
    return unlink(node(index));
}

```





### 4.8 栈

可以作为栈使用

```java
public void push(E e) {
    addFirst(e);
}

public E pop() {
    return removeFirst();
}

```





### 4.9 如何保证线程安全

方法一：

> List<String> list = Collections.synchronizedList(new LinkedList<String>());

方法二：

>  LinkedList换成ConcurrentLinkedQueue 

**建议第二种**



### 4.10 与 ArrayList 的比较

ArrayList 基于动态数组实现，LinkedList 基于双向链表实现。ArrayList 和 LinkedList 的区别可以归结为数组和链表的区别：

- 数组支持随机访问，但插入删除的代价很高，需要移动大量元素；
- 链表不支持随机访问，但插入删除只需要改变指针。
- LinkedList 因为是双端队列，添加元素可以认为总是成功的，无需扩容机制。ArrayList 是数组，需要扩容机制。

