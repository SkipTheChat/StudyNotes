# [路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

### 信息卡片

- 时间： 2020-3-24
- 难度：简单
- 题目描述：

```
给定一个二叉树，它的每个结点都存放着一个整数值。

找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

示例：

root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8

      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

返回 3。和等于 8 的路径有:

1.  5 -> 3
2.  5 -> 2 -> 1
3.  -3 -> 11
```



### 参考答案

> 思路

以当前结点为最终叶子结点向上追溯，路径上的任一结点为根节点到当前结点的路径和为sum的路径个数。 

用一个数组保存一路的路径，每次节点通过n + n1 + n2计算出当前路径数

> 代码

```js
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
      public int pathSum(TreeNode root, int sum) {
        return pathSum(root, sum, new int[1000], 0);
    }

    public int pathSum(TreeNode root, int sum, int[] array/*保存一路的路径节点*/, int p/*当前节点索引*/) {
        if (root == null) {
            return 0;
        }
        int tmp = root.val;
        int n = root.val == sum ? 1 : 0; //判断当前值是否等于sum
        for (int i = p - 1; i >= 0; i--) { //判断从当前开始往上追寻是否有符合的路径
            tmp += array[i];
            if (tmp == sum) {
                n++;
            }
        }
        array[p] = root.val;
        int n1 = pathSum(root.left, sum, array, p + 1); //寻找左树的所有路径
        int n2 = pathSum(root.right, sum, array, p + 1);//寻找右树的所有路径
        return n + n1 + n2;
    }
}
```



### 其他优秀解答
