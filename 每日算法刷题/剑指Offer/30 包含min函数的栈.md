# 30 包含min函数的栈

## 题目内容

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

**示例:**

```
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.min();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.min();   --> 返回 -2.
```

**提示：**

各函数的调用总次数不超过 20000 次

## 解题方法

### 辅助栈

**算法流程**

1. 初始化两个栈A与B，A用于正常存储元素，B用于按照非严格降序存储元素；
2. push操作时，A正常操作，而B只有当其中元素为空或最上方元素大于等于要压入的元素时才进行操作；
3. pop操作时，A正常操作，而B只有当其中的元素最上方的元素等于A中出栈的元素才进行出栈操作；
4. top操作，直接返回A最上面的元素；
5. min操作，直接返回B最上面的元素。

**复杂度分析**

* **时间复杂度**为O(1)
* **空间复杂度**为O(N)

**代码**

```python
class MinStack:

    def __init__(self):
        """
        initialize your data structure here.
        """
        self.A, self.B = [], []


    def push(self, x: int) -> None:
        self.A.append(x)
        if not self.B or self.B[-1] >= x:
            self.B.append(x) 


    def pop(self) -> None:
        if self.A.pop() == self.B[-1]:
            self.B.pop()


    def top(self) -> int:
        return self.A[-1]



    def min(self) -> int:
        return self.B[-1]



# Your MinStack object will be instantiated and called as such:
# obj = MinStack()
# obj.push(x)
# obj.pop()
# param_3 = obj.top()
# param_4 = obj.min()
```

## 总结

由于要考虑重复性的问题，因此在B入栈操作时要选择大于等于，否则会影响其在min的时候的表现。