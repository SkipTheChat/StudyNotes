## 1 HashSet

特点：

1. HashSet是**基于HashMap实现的**，HashSet中的元素都存放在HashMap的key上面，而value中的值都是统一的一个`private static final Object PRESENT = new Object();`
2. 不保证set的迭代顺序，特别是它不保证该顺序恒久不变
3. 不允许有重复元素，有且只允许一个null元素
4. 线程不安全
5. 查询很快，插入速度也很快，但是适用于少量数据的插入操作。数据比较多的时候会涉及到扩容问题，0.75加载因子,所以速度会变慢。





### 3.1 常见属性

```java
//通过HashSet实现的接口可知，其支持所有集合操作，能被克隆，支持序列化
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    //用来作为HashMap中的value值。
    private static final Object PRESENT = new Object();

    ......
}
```



### 3.2 构造函数

```java
//默认初始容量为16，加载因子0.75。
    public HashSet() {
        map = new HashMap<E, Object>();
    }

    //实际底层使用默认的加载因子0.75和足以包含指定collection中所有元素的初始容量来创建一个HashMap。
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<E, Object>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c); 
    }

    //以指定的初始容量和加载因子构造一个空的HashSet
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<E, Object>(initialCapacity, loadFactor);
    }

    //以指定的initialCapacity和默认加载因子0.75构造一个空的HashSet
    public HashSet(int initialCapacity) {
        map = new HashMap<E, Object>(initialCapacity);
    }

    /**
     * 此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持
     * 
     * @param      initialCapacity   初始容量
     * @param      loadFactor        加载因子
     * @param      dummy             标记，用于与其他的构造函数区分（可忽略）
     * @throws     IllegalArgumentException 如果初始容量小于零或加载因子为非正数
     */
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<E, Object>(initialCapacity, loadFactor);
    }
```



### 3.3 查找

```java
 //如果此set包含指定元素，则返回 true
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
```



### 3.4 添加

调用的是底层HashMap中的put(K key, V value)方法，首先判断元素（也就是key）是否存在，如果不存在则插入，如果存在则不插入，这样HashSet中就不存在重复值。 

```java
//如果此set中尚未包含指定元素，则添加指定元素
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

> **注意：**对于HashSet中保存的对象，请注意正确重写其equals和hashCode方法，以保证放入的对象的唯一性。 

### 

### 3.5 迭代

```java
  //返回对此 set中元素进行迭代的迭代器
    public Iterator<E> iterator() {
        return map.keySet().iterator(); //HashMap.keySet()返回<key, value>对中的key集
    }
```



### 3.6 序列化

支持序列化的写入函数writeObject(java.io.ObjectOutputStream s)和读取函数readObject(java.io.ObjectInputStream s)： 



```java
 //java.io.Serializable的写入函数，将HashSet的“总的容量，加载因子，实际容量，所有的元素”都写入到输出流中

    private void writeObject(java.io.ObjectOutputStream s)

        throws java.io.IOException {

        // Write out any hidden serialization magic

        s.defaultWriteObject();

        // Write out HashMap capacity and load factor
    	s.writeInt(map.capacity());
   	 	s.writeFloat(map.loadFactor());

    	// Write out size
   	 	s.writeInt(map.size());

    	// Write out all elements in the proper order.
    	for (E e : map.keySet())
        	s.writeObject(e);
	}

// java.io.Serializable的读取函数，将HashSet的“总的容量，加载因子，实际容量，所有的元素”依次读出
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden serialization magic
    s.defaultReadObject();

    // Read capacity and verify non-negative.
    int capacity = s.readInt();
    if (capacity < 0) {
        throw new InvalidObjectException("Illegal capacity: " +
                                         capacity);
    }

    // Read load factor and verify positive and non NaN.
    float loadFactor = s.readFloat();
    if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
        throw new InvalidObjectException("Illegal load factor: " +
                                         loadFactor);
    }

    // Read size and verify non-negative.
    int size = s.readInt();
    if (size < 0) {
        throw new InvalidObjectException("Illegal size: " +
                                         size);
    }

    // Set the capacity according to the size and load factor ensuring that
    // the HashMap is at least 25% full but clamping to maximum capacity.
    capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
            HashMap.MAXIMUM_CAPACITY);

    // Create backing HashMap
    map = (((HashSet<?>)this) instanceof LinkedHashSet ?
           new LinkedHashMap<E,Object>(capacity, loadFactor) :
           new HashMap<E,Object>(capacity, loadFactor));

    // Read in all elements in the proper order.
    for (int i=0; i<size; i++) {
        @SuppressWarnings("unchecked")
            E e = (E) s.readObject();
        map.put(e, PRESENT);
    }
}
```


## 2 LinkedHashSet

不可重复，有序，线程不安全，允许空值。

LinkedHashSet是**基于LinkedHashMap**实现的。



## 3 TreeSet

**基于TreeMap**（红黑树）实现，线程不安全，不允许空值，不允许重复数据。

以自定义比较器对元素进行排序，或是使用元素的自然顺序。

 TreeSet继承：

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
```

TreeMap继承：

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

属性：

```java
private transient NavigableMap<E,Object> m;
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

//版本号
private static final long serialVersionUID = -2479143000061671589L;
```
构造器：

> 属性是NavigableMap，多态实现

```java
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
```


