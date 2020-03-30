# [实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

### 信息卡片

- 时间： 2019-3-29
- 难度：中等
- 题目描述：

```
实现一个 Trie (前缀树)，包含 insert, search, 和 startsWith 这三个操作。

示例:

Trie trie = new Trie();

trie.insert("apple");
trie.search("apple");   // 返回 true
trie.search("app");     // 返回 false
trie.startsWith("app"); // 返回 true
trie.insert("app");   
trie.search("app");     // 返回 true

说明:

    你可以假设所有的输入都是由小写字母 a-z 构成的。
    保证所有输入均为非空字符串。
```



### 参考答案

> 思路

定义：

Trie 是一颗非典型的多叉树模型，多叉好理解，即每个结点的分支数量可能为多个。 

为什么说非典型呢？因为它和一般的多叉树不一样，尤其在结点的数据结构设计上，比如一般的多叉树的结点是这样的： 

```c
struct TreeNode {
    VALUETYPE value;    //结点值
    TreeNode* children[NUM];    //指向孩子结点
};
```

而 Trie 的结点是这样的(假设只包含'a'~'z'中的字符)： 

```c
struct TrieNode {
    bool isEnd; //该结点是否是一个串的结束
    TrieNode* next[26]; //字母映射表
};
```

我们可以看到`TrieNode`结点中并没有直接保存字符值的数据成员 ,`TrieNode* next[26]`中保存了对当前结点而言下一个可能出现的所有字符的链接，因此我们可以通过一个父结点来预知它所有子结点的值.



举例：想象以下，包含三个单词"sea","sells","she"的 Trie 会长啥样呢？ 

![](https://pic.leetcode-cn.com/3a0be6938b0a5945695fcddd29c74aacc7ac30f040f5078feefab65339176058-file_1575215106942)

红色代表是单词的末尾。



> 代码

可通过运行的代码

```java
public class Trie {
    private boolean is_last=false;  //标记是否是末尾值，是单词末尾的话为true
    private Trie next[]=new Trie[26]; //保存当前节点下一个可能出现的字母集合

    public Trie(){}

    public void insert(String word){//插入单词 时间&空间复杂度 : O(m)。
        Trie root=this;
        char w[]=word.toCharArray();
        for(int i=0;i<w.length;++i){
            if(root.next[w[i]-'a']==null)root.next[w[i]-'a']=new Trie();
            root=root.next[w[i]-'a'];
        }
        root.is_last=true;
    }

    public boolean search(String word){//查找单词  时间复杂度 : O(m)。
        Trie root=this;
        char w[]=word.toCharArray();
        for(int i=0;i<w.length;++i){
            if(root.next[w[i]-'a']==null)return false;
            root=root.next[w[i]-'a'];
        }
        return root.is_last;
    }
    
    public boolean startsWith(String prefix){//查找前缀  时间复杂度 : O(m)。
        Trie root=this;
        char p[]=prefix.toCharArray();
        for(int i=0;i<p.length;++i){
            if(root.next[p[i]-'a']==null)return false;
            root=root.next[p[i]-'a'];
        }
        return true;
    }
}

```



> 可读性较强的代码

```java

/*
class TrieNode {

    // R links to node children
    private TrieNode[] links;

    private final int R = 26;

    private boolean isEnd;

    public TrieNode() {
        links = new TrieNode[R];
    }

    public boolean containsKey(char ch) {
        return links[ch -'a'] != null;
    }
    public TrieNode get(char ch) {
        return links[ch -'a'];
    }
    public void put(char ch, TrieNode node) {
        links[ch -'a'] = node;
    }
    public void setEnd() {
        isEnd = true;
    }
    public boolean isEnd() {
        return isEnd;
    }
}
*/


class Trie {
    private TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    public void insert(String word) {	//插入单词 时间&空间复杂度 : O(m)。
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char currentChar = word.charAt(i);
            if (!node.containsKey(currentChar)) {
                node.put(currentChar, new TrieNode());
            }
            node = node.get(currentChar);
        }
        node.setEnd();
    }
    

    public boolean search(String word) {	//查找单词  时间复杂度 : O(m)。
       TrieNode node = searchPrefix(word);
       return node != null && node.isEnd();
    }
    
    private TrieNode searchPrefix(String word) {	
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char curLetter = word.charAt(i);
            if (node.containsKey(curLetter)) {
                node = node.get(curLetter);
            } else {
                return null;
            }
        }
        return node;
    }

    
    
     public boolean startsWith(String prefix) {	//查找前缀  时间复杂度 : O(m)。
        TrieNode node = searchPrefix(prefix);
        return node != null;
    }
}
```

