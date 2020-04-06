# [ LRU缓存机制](https://leetcode-cn.com/problems/lru-cache/)

### 信息卡片

- 时间： 2019-3-29
- 难度：中等
- 题目描述：

```
运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

进阶:

你是否可以在 O(1) 时间复杂度内完成这两种操作？

示例:

LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```



### 参考答案

> 思路

哈希表 + 双向链表

![](https://pic.leetcode-cn.com/815038bb44b7f15f1f32f31d40e75c250cec3c5c42b95175ec012c00a0243833-146-1.png)



> 代码

```java
import java.util.Hashtable;
public class LRUCache {

    //定义双端链表
    class DLinkedNode {
        int key;
        int value;
        DLinkedNode prev;
        DLinkedNode next;
    }

    private void addNode(DLinkedNode node) {  //插进头节点后边
        node.prev = head;
        node.next = head.next;

        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(DLinkedNode node){
        DLinkedNode prev = node.prev; //获取node前一个节点
        DLinkedNode next = node.next; //获取node后一个节点

        prev.next = next;
        next.prev = prev;
    }

    private void moveToHead(DLinkedNode node){ //将node移到head后面
        removeNode(node);
        addNode(node);
    }

    private DLinkedNode popTail() {	//删除最后一个节点
        DLinkedNode res = tail.prev;
        removeNode(res);
        return res;
    }

    private Hashtable<Integer, DLinkedNode> cache = new Hashtable<Integer, DLinkedNode>();
    private int size; //元素数量
    private int capacity; //容量
    private DLinkedNode head, tail;

    public LRUCache(int capacity) { //初始化
        this.size = 0;
        this.capacity = capacity;

        head = new DLinkedNode();
        // head.prev = null;

        tail = new DLinkedNode();
        // tail.next = null;

        head.next = tail;
        tail.prev = head;
    }

    //1.get
    public int get(int key) {
        DLinkedNode node = cache.get(key);
        if (node == null) return -1;

        moveToHead(node); //移到队头

        return node.value;
    }

    //2.put
    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);


        if(node == null) { //node == null,放进hash表
            DLinkedNode newNode = new DLinkedNode();
            newNode.key = key;
            newNode.value = value;

            cache.put(key, newNode);
            addNode(newNode);

            ++size;

            //超出容量的话，就删除队尾元素
            if(size > capacity) {
                // pop the tail
                DLinkedNode tail = popTail();
                cache.remove(tail.key);
                --size;
            }
        } else { //node！=null，更新元素并且移到队头
            node.value = value;
            moveToHead(node);
        }
    }
}

```



