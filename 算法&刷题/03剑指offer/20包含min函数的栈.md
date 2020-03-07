# 包含min函数的栈 

### 信息卡片 

- 时间： 2019-1-24
- 题目描述：

```
定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
```



### 参考答案

> 思路

用两个栈，一个栈去保存正常的入栈出栈的值，另一个栈去存最小值 

```
入栈 3 
|   |    |   |
|   |    |   |
|_3_|    |_3_|
stack  minStack

入栈 5 
|   |    |   |
| 5 |    | 3 |
|_3_|    |_3_|
stack  minStack

入栈 2 
| 2 |    | 2 |
| 5 |    | 3 |
|_3_|    |_3_|
stack  minStack

出栈 2
|   |    |   |
| 5 |    | 3 |
|_3_|    |_3_|
stack  minStack

出栈 5
|   |    |   |
|   |    |   |
|_3_|    |_3_|
stack  minStack

出栈 3
|   |    |   |
|   |    |   |
|_ _|    |_ _|
stack  minStack

```






> 代码

```java
private Stack<Integer> dataStack = new Stack<>();
private Stack<Integer> minStack = new Stack<>();

public void push(int node) {
    dataStack.push(node);
    minStack.push(minStack.isEmpty() ? node : Math.min(minStack.peek(), node));
}

public void pop() {
    dataStack.pop();
    minStack.pop();
}

public int top() {
    return dataStack.peek();
}

public int min() {
    return minStack.peek();
}
```


