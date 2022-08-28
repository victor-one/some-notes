# 32 III 从上到下打印二叉树III

## 题目内容

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:
给定二叉树: [3,9,20,null,null,15,7],

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [20,9],
  [15,7]
]
```

**提示：**

节点总数 <= 1000

## 解题方法

### 方法一 层序遍历加双端队列

**算法流程**

1. 和之前的流程类似，变化是在temp存储时采用双端队列来进行存储；
2. 当为奇数层的时候，temp采用append添加元素，当为偶数层的时候采用appendleft进行元素添加；
3. 当temp添加到res中的时候，要注意转化为数组再添加。

**复杂度分析**

* **时间复杂度**为O(N)
* **空间复杂度**为O(N)

**代码**

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        if not root: return []
        res, queue = [], collections.deque()
        queue.append(root)
        while queue:
            temp = collections.deque()
            for _ in range(len(queue)):
                node = queue.popleft()
                if len(res) % 2: temp.appendleft(node.val)
                else: temp.append(node.val)
                if node.left: queue.append(node.left)
                if node.right: queue.append(node.right)
            res.append((list(temp)))
        return res
```

### 方法二 层序遍历加双端队列（奇偶分离）

**算法流程**

1. 此方法主要是消除了奇偶判断的过程。

**复杂度分析**

* **时间复杂度**为O(N)
* **空间复杂度**为O(N)

**代码**

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        if not root: return []
        res, queue = [], collections.deque()
        queue.append(root)
        while queue:
            temp = []
            for _ in range(len(queue)):
                node = queue.popleft()
                temp.append(node.val)
                if node.left: queue.append(node.left)
                if node.right: queue.append(node.right)
            res.append(temp)
            if not queue: break
            temp = []
            for _ in range(len(queue)):
                node = queue.pop()
                temp.append(node.val)
                if node.right: queue.appendleft(node.right)
                if node.left: queue.appendleft(node.left)
            res.append(temp)
        return res
```

### 方法三 层序遍历加倒序

**算法流程**

1. 和昨天的流程是一样的，在添加temp时加上奇偶判定即可。

**复杂度分析**

* **时间复杂度**为O(N)
* **空间复杂度**为O(N)

**代码**

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        if not root: return []
        res, queue = [], collections.deque()
        queue.append(root)
        while queue:
            temp = []
            for _ in range(len(queue)):
                node = queue.popleft()
                temp.append(node.val)
                if node.left: queue.append(node.left)
                if node.right: queue.append(node.right)
            res.append(temp[::-1] if len(res) % 2 else temp)
        return res
```

## 总结

这道题目方法还是挺多的，方法二更加绕一点，但是好处是省了层序判断的过程，因此在实际运行中可以节省更多的时间。
