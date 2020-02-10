# 3 set

### 3.1 HashSet

不允许元素重复，不会记录元素的先后添加顺序。

hashSet专门对快速查找进行了优化。



##### 3.1.1 原理总结

1.基于HashMap实现的，默认构造函数是构建一个初始容量为16，负载因子为0.75 的HashMap。封装了一个 HashMap 对象来存储所有的集合元素，所有放入 HashSet 中的集合元素实际上由 HashMap 的 key 来保存，而 HashMap 的 value 则存储了一个 PRESENT，它是一个静态的 Object 对象。因为HashSet用不到value,他们的value都是一样的指向同一个地方。 

2.HashSet的其他操作都是基于HashMap的。

3.查询很快，插入速度也很快，但是适用于少量数据的插入操作。数据比较多的时候会涉及到扩容问题，0.75加载因子,所以速度会变慢。



##### 3.1.2 hashCode&equals

对象的hashCode值决定了再哈希表中的存储位置。

当往hashSet集合中添加新的对象的时候，先会判断该对象和集合对象中的hashCode值：

- 不等：直接存入hashCode指定位置。
- 相等：再继续判断新对象和集合对象中的equals。
  - true：则视为同一个对象，不保存。
  - false：非常麻烦，存储在之前对象同槽位的链表上。

结论：存储在哈希表中的对象，都应该覆盖equals和hashCode方法，并且保证equals相等的时候，hashCode也相等。

每一个存储到hash表中的对象，都得提供hashCode和equals方法，用来判断是否是同一个对象。



##### 3.1.3 源码

```java
import java.util.AbstractSet;
import java.util.Collection;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Set;

import javax.swing.text.html.HTMLDocument.Iterator;

public class HashSet<E>  
          extends AbstractSet<E>  
          implements Set<E>, Cloneable, java.io.Serializable  
{  
static final long serialVersionUID = -5024744406713321676L;  

// 底层使用HashMap来保存HashSet中所有元素。  
private transient HashMap<E,Object> map;  
  
// 定义一个虚拟的Object对象作为HashMap的value，将此对象定义为static final。  
private static final Object PRESENT = new Object();  

 
//  默认的无参构造器，构造一个空的HashSet。 
//   
//  实际底层会初始化一个空的HashMap，并使用默认初始容量为16和加载因子0.75。 
  
public HashSet() {  
map = new HashMap<E,Object>();  
}  

 
//  构造一个包含指定collection中的元素的新set。 
//  
//  实际底层使用默认的加载因子0.75和足以包含指定 
//  collection中所有元素的初始容量来创建一个HashMap。 
//  @param c 其中的元素将存放在此set中的collection。 
 
public HashSet(Collection<? extends E> c) {  
map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));  
addAll(c);  
}  

 
//  以指定的initialCapacity和loadFactor构造一个空的HashSet。 
//  
//  实际底层以相应的参数构造一个空的HashMap。 
// @param initialCapacity 初始容量。 
//  @param loadFactor 加载因子。 
   
public HashSet(int initialCapacity, float loadFactor) {  
map = new HashMap<E,Object>(initialCapacity, loadFactor);  
}  

 
//  以指定的initialCapacity构造一个空的HashSet。 
//  
//  实际底层以相应的参数及加载因子loadFactor为0.75构造一个空的HashMap。 
//  @param initialCapacity 初始容量。 
 
public HashSet(int initialCapacity) {  
map = new HashMap<E,Object>(initialCapacity);  
}  


//  以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合。 
//  此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持。 
//  
//  实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。 
//  @param initialCapacity 初始容量。 
//  @param loadFactor 加载因子。 
//  @param dummy 标记。 
   
HashSet(int initialCapacity, float loadFactor, boolean dummy) {  
map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);  
}  

 
//  返回对此set中元素进行迭代的迭代器。返回元素的顺序并不是特定的。 
//  
// 底层实际调用底层HashMap的keySet来返回所有的key。 
//  可见HashSet中的元素，只是存放在了底层HashMap的key上， 
//  value使用一个static final的Object对象标识。 
//  @return 对此set中元素进行迭代的Iterator。 

 public Iterator<E> iterator() {  
     return map.keySet().iterator();  

}  
 
// 返回此set中的元素的数量（set的容量）。 
//  
//  底层实际调用HashMap的size()方法返回Entry的数量，就得到该Set中元素的个数。 
//  @return 此set中的元素的数量（set的容量）。 
  
public int size() {  
return map.size();  
}  

 
//  如果此set不包含任何元素，则返回true。 
//  
//  底层实际调用HashMap的isEmpty()判断该HashSet是否为空。 
//  @return 如果此set不包含任何元素，则返回true。 
 
public boolean isEmpty() {  
return map.isEmpty();  
}  

 
//  如果此set包含指定元素，则返回true。 
//  更确切地讲，当且仅当此set包含一个满足(o==null ? e==null : o.equals(e)) 
//  的e元素时，返回true。 
// 
//  底层实际调用HashMap的containsKey判断是否包含指定key。 
// @param o 在此set中的存在已得到测试的元素。 
// @return 如果此set包含指定元素，则返回true。 
  
public boolean contains(Object o) {  
return map.containsKey(o);  
}  


// 如果此set中尚未包含指定元素，则添加指定元素。 
// 更确切地讲，如果此 set 没有包含满足(e==null ? e2==null : e.equals(e2)) 
// 的元素e2，则向此set 添加指定的元素e。 
// 如果此set已包含该元素，则该调用不更改set并返回false。 
// 
// 底层实际将将该元素作为key放入HashMap。 
// 由于HashMap的put()方法添加key-value对时，当新放入HashMap的Entry中key 
//与集合中原有Entry的key相同（hashCode()返回值相等，通过equals比较也返回true）， 
//新添加的Entry的value会将覆盖原来Entry的value，但key不会有任何改变， 
//  因此如果向HashSet中添加一个已经存在的元素时，新添加的集合元素将不会被放入HashMap中， 
//  原来的元素也不会有任何改变，这也就满足了Set中元素不重复的特性。 
//  @param e 将添加到此set中的元素。 
//  @return 如果此set尚未包含指定元素，则返回true。 
 
public boolean add(E e) {  
       return map.put(e, PRESENT)==null;  
}  

//  如果指定元素存在于此set中，则将其移除。 
//  更确切地讲，如果此set包含一个满足(o==null ? e==null : o.equals(e))的元素e， 
//  则将其移除。如果此set已包含该元素，则返回true 
// （或者：如果此set因调用而发生更改，则返回true）。（一旦调用返回，则此set不再包含该元素）。 
//  
//  底层实际调用HashMap的remove方法删除指定Entry。 
//  @param o 如果存在于此set中则需要将其移除的对象。 
//  @return 如果set包含指定元素，则返回true。 
 
public boolean remove(Object o) {  
return map.remove(o)==PRESENT;  
}  

 
//  从此set中移除所有元素。此调用返回后，该set将为空。 
//  
//  底层实际调用HashMap的clear方法清空Entry中所有元素。 
 
public void clear() {  
map.clear();  
}  
 
//  返回此HashSet实例的浅表副本：并没有复制这些元素本身。 

//  底层实际调用HashMap的clone()方法，获取HashMap的浅表副本，并设置到HashSet中。 
 
public Object clone() {  
    try {  
        HashSet<E> newSet = (HashSet<E>) super.clone();  
        newSet.map = (HashMap<E, Object>) map.clone();  
        return newSet;  
    } catch (CloneNotSupportedException e) {  
        throw new InternalError();  
    }  
}  
}

```



1. 实现原理，基于哈希表（hashmap）实现。
2. 不允许重复键存在，但可以有null值。
3. 哈希表存储是无序的。
4. 添加元素时把元素当作hashmap的key存储，HashMap的value是存储的一个固定值object
5. 排除重复元素是通过equals检查对象是否相同。
6. 判断2个对象是否相同，先根据2个对象的hashcode比较是否相等（如果两个对象的hashcode相同，它们也不一定是同一个对象，如果不同，那一定不是同一个对象）如果不同，则两个对象不是同一个对象，如果相同，在将2个对象进行equals检查来判断是否相同，如果相同则是同一个对象，不同则不是同一个对象。
7. 如果要完全判断自定义对象是否有重复值，这个时候需要将自定义对象重写对象所在类的hashcode和equals方法来解决。
8. 哈希表的存储结构就是：数组+链表，数组的每个元素都是以链表的形式存储的。





### 3.2 LinkedHashSet

不可重复，保证插入顺序，线程不安全。

底层基于LinkedHashMap实现，是一个哈希表和双向链表的结合。（链表可以保证插入的顺序，哈希表保证取数据的快速）





##### 3.2.1 构造器

初始化容量为16，加载因子为0.75 。

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.18.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.19.png)



##### 3.2.2 add方法

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.20.png)





### 3.3 TreeSet

底层红黑树算法，会对存储的元素默认使用自然排序（从小到大），基于TreeMap实现，线程不安全。

注意：必须保证TreeSet集合中的元素对象是相同的数据类型，否则报错。



##### 3.3.1 特性总结

1.TreeSet()是使用二叉树的原理对新add()的对象按照指定的顺序排序，每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。

2.Integer和String对象都可以进行默认的TreeSet排序，而自定义类的对象是不可以的，自己定义的类必须实现Comparable接口，并且覆写相应的compareTo()函数，才可以正常使用。

3.在覆写compare()函数时，要返回相应的值才能使TreeSet按照一定的规则来排序（升序，this.对象 < 指定对象的条件下返回-1）

（降序，this.对象 < 指定对象的条件下返回-1）升序是：比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数。
eg:

```java
public int compareTo(Object obj){
		Persontest per = (Persontest) obj;
		//升序
		if(per.age < this.age){
			return -1;
		}
		if(per.age > this.age){
			return 1;
		}
		if(per.age == this.age){
			return this.name.compareTo(per.name);
		}
		return 0;
	}
```





##### 3.3.2 初始化

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.21.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.22.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.23.png)



comparator：用于维护元素的比较器。如果为null，则keys用的是自然排序。 

创建一个TreeSet，等于创建一个空的自然排序的TreeMap容器。 



##### 3.3.3 add

容器提供了两种排序的方法，一种自然排序当comparator为空的时候，构建无参构造函数的时候默认的一种排序方式 。

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.24.png)



![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.25.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.26.png)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/java%E5%9F%BA%E7%A1%80/assets/1.27.png)





