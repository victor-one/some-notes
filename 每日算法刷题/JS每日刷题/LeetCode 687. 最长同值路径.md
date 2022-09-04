# LeetCode 687. 最长同值路径

> 力扣题 687 **最长同值路径**
>
> 地址：https://leetcode.cn/problems/longest-univalue-path/

## 思路：DFS 后序遍历

代码：

```JavaScript
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
var longestUnivaluePath = function(root) {
  let res = 0;
  const maxL = (root) => {
    if (!root) {
      return 0;
    }
    const left = maxL(root.left), right = maxL(root.right);
    let leftL = 0, rightL = 0;
    if (root.left && root.left.val === root.val) {
      leftL = left + 1;
    }
    if (root.right && root.right.val === root.val) {
      rightL = right + 1;
    }
    res = Math.max(res, leftL + rightL);
    return Math.max(leftL, rightL);
  }
  maxL(root);
  return res;
};
```

