# [反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/)

### 信息卡片

- 时间： 2020-5-5
- 难度：简单
- 题目描述：

```java
给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

示例 1:

输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 

注意：在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格。
```



### 参考答案

> 代码

```java
class Solution {
     public String reverseWords(String s) {
        char[]  c = s.toCharArray();

        int i = 0;
        int j = 1;
        while(j < c.length){
            if(c[j] == ' '){
                reverse(c,i,j - 1);
                i = j + 1;
            }else if(j == c.length - 1){
                reverse(c,i,j);
            }
            j++;
        }
        return new String(c);
    }

    private void reverse(char[] s, int i, int j) {
        while(i < j){
            char temp = s[i];
            s[i] = s[j];
            s[j] = temp;
            i++;
            j--;
        }
    }

}
```



