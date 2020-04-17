# [Excel表列序号](https://leetcode-cn.com/problems/excel-sheet-column-number/)

### 信息卡片

- 时间： 2019-4-15
- 难度：简单
- 题目描述：

```java
给定一个Excel表格中的列名称，返回其相应的列序号。

例如，

    A -> 1
    B -> 2
    C -> 3
    ...
    Z -> 26
    AA -> 27
    AB -> 28 
    ...

示例 1:

输入: "A"
输出: 1

示例 2:

输入: "AB"
输出: 28

示例 3:

输入: "ZY"
输出: 701
```



### 参考答案

> 思路

- 初始化结果ans = 0，遍历时将每个字母与A做减法，因为A表示1，所以减法后需要每个数加1，计算其代表的数值num = 字母 - ‘A’ + 1
- 因为有26个字母，所以相当于26进制，每26个数则向前进一位
- 所以每遍历一位则ans = ans * 26 + num
- 以 ZY 为例，Z 的值为 26 ，Y 的值为 25 ，则结果为 26 * 26 + 25 = 701



> 代码

```java
class Solution {
    public int titleToNumber(String s) {
        int ans = 0;
        for(int i=0;i<s.length();i++) {
            int num = s.charAt(i) - 'A' + 1;
            ans = ans * 26 + num;
        }
        return ans;
    }
}
```
