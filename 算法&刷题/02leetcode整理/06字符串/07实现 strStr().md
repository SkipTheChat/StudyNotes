# [实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

### 信息卡片

- 时间： 2019-2-5
- 难度：简单
- 题目描述：

```
实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:

输入: haystack = "hello", needle = "ll"
输出: 2

示例 2:

输入: haystack = "aaaaa", needle = "bba"
输出: -1

说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。
```



### 参考答案

> 思路

KMP算法！没错，很难....


```js
class Solution {
    //KMP算法。比较难。
     public int strStr(String S, String T) {
        if (T == null || T.length() == 0) return 0;
        int[] next = new int[T.length()];
        getNext(T, next);
        int i = 0;
        int j = 0;
        while (i < S.length() && j < T.length()) {
            if (j == -1 || S.charAt(i) == T.charAt(j)) {
                i++;
                j++;
            } else j = next[j];
        }
        if (j == T.length()) return i - j;
        return -1;


    }

    private void getNext(String t, int[] next) {
        next[0] = -1;
        int i = 0;
        int j = -1;
        while (i < t.length() - 1) {
            if (j == -1 || t.charAt(i) == t.charAt(j)) {
                i++;
                j++;
                next[i] = j;
            } else {
                j = next[j];
            }
        }
    }
}
```






