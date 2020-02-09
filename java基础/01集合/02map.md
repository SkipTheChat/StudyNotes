# 1 map

HashMap,LinkedHashMap,TreeMap都不是线程安全的，性能较高。

- Hashtable 是早期Java类库提供的一个哈希表实现，本身是同步的，不支持 null 键和值，由于同步导致的性能开销，所以已经很少被推荐使用。
- HashMap与 HashTable主要区别在于 HashMap 不是同步的，支持 null 键和值等。通常情况下，HashMap 进行 put 或者 get 操作，可以达到常数时间的性能，所以它是绝大部分利用键值对存取场景的首选。
- LinkedHashMap有顺序不重复。
- TreeMap 则是基于红黑树的一种提供顺序访问的 Map，和 HashMap 不同，它的 get、put、remove 之类操作都是 O（log(n)）的时间复杂度，具体顺序可以由指定的 Comparator 来决定，或者根据键的自然顺序来判断。



## 1.1 Hashmap

[推荐博客](https://www.jianshu.com/p/ee0de4c99f87)

### 1.1.1 结构

HashMap是数组+链表+红黑树 。

数组被分为一个个桶（bucket），每个桶存储有一个或多个Entry对象，每个Entry对象包含三部分key（键）、value（值），next(指向下一个Entry），通过哈希值决定了Entry对象在这个数组的寻址；哈希值相同的Entry对象（键值对），则以链表形式存储。如果链表大小超过树形转换的阈值（TREEIFY_THRESHOLD= 8），链表就会被改造为树形结构。
![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.29.png)



查询时间复杂度：HashMap的本质可以认为是一个数组，数组的每个索引被称为桶，每个桶里放着一个单链表，一个节点连着一个节点。很明显通过下标来检索数组元素时间复杂度为O(1)，而且遍历链表的时间复杂度是O(n)，所以在链表长度尽可能短的前提下，HashMap的查询复杂度接近O(1)

- 数组：存储区间连续，占用内存严重，寻址容易，插入删除困难；  
- 链表：存储区间离散，占用内存比较宽松，寻址困难，插入删除容易；  

Hashmap综合应用了这两种数据结构，实现了寻址容易，插入删除也容易。 



### 1.1.2 拉链法的工作原理

```java
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```

- 新建一个 HashMap，默认大小为 16；
- 插入 <K1,V1> 键值对，先计算 K1 的 hashCode 为 115，使用除留余数法得到所在的桶下标 115%16=3。
- 插入 <K2,V2> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。
- 插入 <K3,V3> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 <K2,V2> 前面。

应该注意到链表的插入是以**头插法方式**进行的，例如上面的 <K3,V3> 不是插在 <K2,V2> 后面，而是插入在链表头部。

> 1.7使用头插法，1.8使用尾插法

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.30.png)



查找需要分成两步进行：

- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。



### 1.1.3 属性&类

属性：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //序列号，序列化的时候使用。
    private static final long serialVersionUID = 362498820763181265L;
    /**默认容量，1向左移位4个，00000001变成00010000，也就是2的4次方为16。**/
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    
    //最大容量，2的30次方。
    static final int MAXIMUM_CAPACITY = 1 << 30;
    
    //加载因子，用于扩容使用。
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    //当某个桶节点数量大于8时，会转换为红黑树。
    static final int TREEIFY_THRESHOLD = 8;
    
    //当某个桶节点数量小于6时，会转换为链表，前提是它当前是红黑树结构。
    static final int UNTREEIFY_THRESHOLD = 6;
    
    //当整个hashMap中元素数量大于64时，也会进行转为红黑树结构。
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    //存储元素的数组，transient关键字表示该属性不能被序列化
    transient Node<K,V>[] table;
    
    //将数据转换成set的另一种存储形式，这个变量主要用于迭代功能。
    transient Set<Map.Entry<K,V>> entrySet;
    
    //元素数量
    transient int size;
    
    //统计该map修改的次数
    transient int modCount;
    
    //临界值，也就是元素数量达到临界值时，会进行扩容。
    int threshold;
    
    //也是加载因子，只不过这个是变量。
    final float loadFactor;  
```



几个常用内部类：

使用静态内部类，是为了方便调用，而不用每次调用里面的属性或者方法都需要new一个对象。这是一个红黑树的结构。

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
}
```

```java
 static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
 
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
}
```

里面还包含了一个结点内部类，是一个单向链表。上面这两个内部类再加上之前的Node<K,V>[] table属性，组成了hashMap的结构，哈希桶。 



### 1.1.4 初始化

共四种构造器：

```java
    public HashMap() {
        //默认的加载因子0.75
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
    }
 
 
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
 
 
//构造一个初始容量为initialCapacity,负载因子为loadFactor的HashMap
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        //threshold临界值，如果达到临界值需要扩容
        this.threshold = tableSizeFor(initialCapacity);
    }
```

第四个构造器：

   该构造函数，传入一个Map，然后把该Map转为hashMap，负载因子为默认的0.75

loadFactor：负载因子，0.75

threshold：临界值，临界值是判断数组是否要扩容的依据

tableSizeFor：用于计算临界值的方法

resize：扩容方法

```java
    public HashMap(Map<? extends K, ? extends V> m) {
        //负载因子，默认0.75
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
 
 
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            //table:存储元素的数组。判断table是否初始化，如果没有初始化
            if (table == null) { // pre-size
          	//根据插入的map的size计算要创建的HashMap的容量
                float ft = ((float)s / loadFactor) + 1.0F;
                //判断该容量大小是否超出上限。
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
               //threshold代表临界值，判断t是否大于临界值，如果就对threshold进行重新计算赋值得出新的临界值
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            //如果table已经初始化，并且大于当前临界值，则进行扩容操作，resize()就是扩容。
            else if (s > threshold)
                resize();
            //遍历，把map中的数据转到hashMap中。
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```



`tableSizeFor(int cap)` ：

```java
//Returns a power of two size for the given target capacity.
   static final int tableSizeFor(int cap) {
       int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
       return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

tableSizeFor代码解释：以65举例子， >>>表示右移

```java
n-=1 // n=1000000(二进制)
n |= n >>> 16;	//1000000 | 0000000 = 1000000 = 64
n |= n >>> 8;	//1000000 | 0000000 = 1000000 = 64
n |= n >>> 4;	//1000000 | 0000100 = 1000100 = 68 
n |= n >>> 2;	//1000100 | 0010001 = 1010101 = 85
n |= n >>> 1;	//1010101 | 0101010 = 1111111 = 127

return n + 1;
```



putVal(hash(key))方法中的`hash(key)`：

```java
static final hash(Object key){
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

 `(h = key.hashCode()) ^ (h >>> 16)`：异或运算

原来的hashCode：

1111 1111 1111 1111 0100 1100 0000 1010

向右移16位的hashCode：

0000 0000 0000 0000 1111 1111 1111 1111

进行异或运算结果：

1111 1111 1111 1111 1011 0011 1111 0101

好处：生成的hash值的随机性会增大



### 1.1.5 扩容 resize()

![](../assets/2.1.png)

> 因为从 JDK 1.8 开始引入了红黑树，扩容操作较为复杂。jdk1.7是没有用红黑树的。

|    参数     | 含义                                                         |
| :---------: | :----------------------------------------------------------- |
|  capacity   | table 的容量大小，默认为 16，需要注意的是 capacity 必须保证为 2 的次方。 |
|    size     | table 的实际使用量。                                         |
|  threshold  | size 的临界值，size 必须小于 threshold，如果大于等于，就必须进行扩容操作。 |
| load_factor | table 能够使用的比例，threshold = capacity * load_factor。   |

扩容使用 resize() 实现，需要注意的是，扩容操作同样需要把旧 table 的所有键值对重新插入新的 table 中，因此这一步是很费时的。

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;//将当前table暂存到oldtab来操作
    	//保存当前table的容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
    	//保存当前临界阈值
        int oldThr = threshold;
        int newCap, newThr = 0;
    
    	//1.当table已初始化时
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {//如果老容量大于最大容量
                threshold = Integer.MAX_VALUE;//阈值直接设置为Integer的最大值
                return oldTab;//直接返回不用扩容
            }
            //使新的容量翻倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // 阈值翻倍
        }
    
    
     //2.当table未初始化时，让新的容量为老的阈值
    else if (oldThr > 0){
        newCap = oldThr;
       }
    
   
    //3.这种情况是初始化HashMap时啥参数都没加
    else { 
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
    
    
    //当新的阈值为0，重新计算，新的阈值= 新的容量*0.75
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;//把新的阈值赋给当前table
    
    
        @SuppressWarnings({"rawtypes","unchecked"})
    	//初始化table
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            //对老table进行遍历
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {//遍历到的赋给e进行暂存，同时将老table对应项赋值为null
                    oldTab[j] = null;
                    if (e.next == null)//将不为空的元素复制到新table中
                        newTab[e.hash & (newCap - 1)] = e;//等于是创建一个新的空table然后重新进行元素的put，这里的table长度是原table的两倍
                    else if (e instanceof TreeNode)//如果e是红黑树的元素，要进行红黑树的rehash操作
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { //如果是链表，进行链表的rehash操作
                        Node<K,V> loHead = null, loTail = null;//用于保存put后不移位的链表
                        Node<K,V> hiHead = null, hiTail = null;//用于保存put后移位的链表
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {//如果与的结果为0，表示不移位，将桶中的头结点添加到lohead和lotail中，往后如果桶中还有不移位的结点，就向tail继续添加
                                if (loTail == null)//在后面遍历lohead和lotail保存到table中时，lohead用于保存头结点的位置，lotail用于判断是否到了末尾
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            
                            else {//这是添加移位的结点，与不移位的类似
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        
                        
                        if (loTail != null) {//把不移位的结点添加到对应的链表数组中去
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        
                        if (hiTail != null) {//把移位的结点添加到对应的链表数组中去
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```





### 1.1.6 putVal()

put:

1.如果数组未进行初始化或者为空，则通过resize()初始化,并返回数组的长度

2.用hash值计算应该存放在数组中的位置，如果此位置为空，则新的节点放在这个位置。

如果不为空，继续判断。

3.如果hash值与数组中节点的hash值相等，key也相等，则直接取代。如果是红黑树节点，putTreeVal将值加入红黑树节点。如果是链表节点，放入链表，放入后计算节点是否达到阈值，达到了就转为一颗红黑树。

4.最后判断数组的长度是否超过了threshold，超过了需要进行扩容。



```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        
        // 如果table为空或者长度为0，则resize()扩容
        if ((tab = table) == null || (n = tab.length) == 0) 
            n = (tab = resize()).length;
        
        // 确定插入table的位置，算法是(n - 1) & hash，相当于取模操作
        //这个表达式中可以看出为什么数组的扩容的长度为什么要是2的n次幂的原因了 
        // 确定后的在数组中的该位置为空，则新的节点放在这个位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        
        // 以下的else中的逻辑表示确定的该位置不是空
        else {
            Node<K,V> e; K k;
            //如果hash值相同并且key相同，或者key是equals的，直接替换这个节点
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            
            // 如果是红黑树节点
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            
            // 如果是链表节点，遍历链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    //如果在尾端都没有找到和key值相同的节点，则生成一个新的Node插入尾端
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        
                        // 节点到达阈值，转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果有hash，key相同的节点或者equals的系欸但，直接替换
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            
            //如果e不为空就替换旧的oldValue值
            if (e != null) { 
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        
        // 如果数组中的元素个数超过阈值，则进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

>注：hash 冲突发生的几种情况：
>
>1. 两节点key 值相同（hash值一定相同），导致冲突；
>2. 两节点key 值不同，由于 hash 函数的局限性导致hash 值相同，冲突；
>3. 两节点key 值不同，hash 值不同，但 hash 值对数组长度取模后相同，冲突；

##### 



### 1.1.7 null值

HashMap 允许有一个 Node 的 Key 为 null，该 Node 一定会放在第 0 个桶的位置，因为这个 Key 无法计算 hashCode()，因此只能规定一个桶让它存放。



### 1.1.8 1.7和1.8的HashMap的不同点

（1）JDK1.7用的是头插法，而JDK1.8及之后使用的都是尾插法，那么为什么要这样做呢？因为JDK1.7是用单链表进行的纵向延伸，当采用头插法就是能够提高插入的效率，但是也会容易出现逆序且环形链表死循环问题。但是在JDK1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题。

（2）扩容后数据存储位置的计算方式也不一样：

在JDK1.7的时候是直接用hash值和需要扩容的二进制数进行&amp;（这里就是为什么扩容的时候为啥一定必须是2的多少次幂的原因所在，因为如果只有2的n次幂的情况时最后一位二进制数才一定是1，这样能最大程度减少hash碰撞）（hash值 &amp; length-1） 。
而在JDK1.8的时候直接用了JDK1.7的时候计算的规律，也就是扩容前的原始位置+扩容的大小值=JDK1.8的计算方式，而不再是JDK1.7的那种异或的方法。但是这种方式就相当于只需要判断Hash值的新增参与运算的位是0还是1就直接迅速计算出了扩容后的储存方式。

（3）JDK1.7的时候使用的是数组+ 单链表的数据结构。但是在JDK1.8及之后时，使用的是数组+链表+红黑树的数据结构（当链表的深度达到8的时候，也就是默认阈值，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O（N）变成O（logN）提高了效率）。





### 4.1.9 HashMap为什么是线程不安全的？

HashMap 在并发时可能出现的问题主要是两方面：

1. **put的时候导致的多线程数据不一致**

   比如有两个线程A和B，首先A希望插入一个key-value对到HashMap中，首先计算记录所要落到的 hash桶的索引坐标，然后获取到该桶里面的链表头结点，此时线程A的时间片用完了，而此时线程B被调度得以执行，和线程A一样执行，只不过线程B成功将记录插到了桶里面，假设线程A插入的记录计算出来的 hash桶索引和线程B要插入的记录计算出来的 hash桶索引是一样的，那么当线程B成功插入之后，线程A再次被调度运行时，它依然持有过期的链表头但是它对此一无所知，以至于它认为它应该这样做，如此一来就覆盖了线程B插入的记录，这样线程B插入的记录就凭空消失了，造成了数据不一致的行为。

2. **resize而引起死循环**

   这种情况发生在HashMap自动扩容时，当2个线程同时检测到元素个数超过 数组大小 × 负载因子。此时2个线程会在put()方法中调用了resize()，**两个线程同时修改一个链表结构会产生一个循环链表**（JDK1.7中，会出现resize前后元素顺序倒置的情况）。接下来再想通过get()获取某一个元素，就会出现死循环。



### 4.1.10 与 HashTable 的区别

HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：**线程安全性，同步，以及速度。**

1. HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，Hashtable是synchronized。
2. HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行
3. 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
4. 在单线程环境下Hashtable比HashMap要慢。
5. HashMap不能保证随着时间的推移Map中的元素次序是不变的。

> java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。





## 2.2 LinkedHashMap

保持了插入顺序的HashMap。

##### 4.2.1 存储结构

继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序。

```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
```

accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。java

```java
final boolean accessOrder;
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

###  

##### 4.2.2 afterNodeAccess()

当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

###  

##### 4.2.3 afterNodeInsertion()

在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。

evict 只有在构建 Map 的时候才为 false，在这里为 true。

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap  并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

###  

##### 4.2.4 LRU 缓存

以下是使用 LinkedHashMap 实现的一个 LRU 缓存：

- 设定最大缓存空间 MAX_ENTRIES  为 3；
- 使用 LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启 LRU 顺序；
- 覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final int MAX_ENTRIES = 3;

    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LRUCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }
}
public static void main(String[] args) {
    LRUCache<Integer, String> cache = new LRUCache<>();
    cache.put(1, "a");
    cache.put(2, "b");
    cache.put(3, "c");
    cache.get(1);
    cache.put(4, "d");
    System.out.println(cache.keySet());
}
[3, 1, 4]
```







### 4.3 TreeMap

底层基于红黑树，是一个有序的hashmap。



### 4.4 三者区别

1.HashMap中k的值没有顺序，常用来做统计。

2.LinkedHashMap吧。它内部有一个链表，保持Key插入的顺序。迭代的时候，也是按照插入顺序迭代，而且迭代比HashMap快。

3.TreeMap的顺序是Key的自然顺序（如整数从小到大），也可以指定比较函数。但不是插入的顺序。Hashtable、HashMap、TreeMap都实现了Map接口，使用键值对的形式存储数据和操作数据。

典型回答：

- Hashtable是java早期提供的，方法是同步的（加了synchronized）。key和value都不能是null值。
- HashMap的方法不是同步的，支持key和value为null的情况。行为上基本和Hashtable一致。由于Hashtable是同步的，性能开销比较大，一般不推荐使用Hashtable。通常会选择使用HashMap。HashMap进行put和get操作，基本上可以达到常数时间的性能。
- TreeMap是基于红黑树的一种提供顺序访问的Map，和HashMap不同，它的get或put操作的时间复杂度是O(log(n))。具体的顺序由指定的Comparator来决定，或者根据键key的具体顺序来决定。

