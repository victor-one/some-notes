# LeetCode1030. 距离顺序排列矩阵单元格

> 力扣题1030 **距离顺序排列矩阵单元格**
>
> 地址：https://leetcode.cn/problems/matrix-cells-in-distance-order/

## 思路1：利用哈希表（桶排序）

**整体思路**：利用一个哈希表进行存储，哈希表的键为长度，对应值为存储位置的数组。

**代码**：

```javascript
/**
 * @param {number} rows
 * @param {number} cols
 * @param {number} rCenter
 * @param {number} cCenter
 * @return {number[][]}
 */
var allCellsDistOrder = function(rows, cols, rCenter, cCenter) {
  const hashmap = {};
  const res = [];
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      const len = Math.abs(i - rCenter) + Math.abs(j - cCenter);
      if (!hashmap[len]) {
        hashmap[len] = []
      }
      hashmap[len].push([i, j]);
    }
  }
  for (let i = 0; i < rows + cols; i++) {
    if (!hashmap[i]) {
      continue
    }
    for(arr of hashmap[i]){
      res.push(arr);
    }
  }
  return res;
};
```

发现执行用时较高，进行**优化**，能够优化的原因是因为 JS 对象中属性是按顺序排列的：

```JavaScript
/**
 * @param {number} rows
 * @param {number} cols
 * @param {number} rCenter
 * @param {number} cCenter
 * @return {number[][]}
 */
var allCellsDistOrder = function(rows, cols, rCenter, cCenter) {
  const hashmap = {};
  const res = [];
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      const len = Math.abs(i - rCenter) + Math.abs(j - cCenter);
      if (!hashmap[len]) {
        hashmap[len] = []
      }
      hashmap[len].push([i, j]);
    }
  }
  for (key in hashmap) {
    for(arr of hashmap[key]){
      res.push(arr);
    }
  }
  return res;
};
```

## 思路2：利用sort方法进行排序

```JavaScript
/**
 * @param {number} rows
 * @param {number} cols
 * @param {number} rCenter
 * @param {number} cCenter
 * @return {number[][]}
 */
var allCellsDistOrder = function(rows, cols, rCenter, cCenter) {
  let res = [];
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      res.push([i, j]);
    }
  }
  res.sort((a,b) => {
    const len1 = Math.abs(a[0] - rCenter) + Math.abs(a[1] - cCenter);
    const len2 = Math.abs(b[0] - rCenter) + Math.abs(b[1] - cCenter);
    return len1 - len2;
  })
  return res;
};
```

----

以下是通过看别人的解析获取的思路：

## 思路3：BFS 遍历

```JavaScript
/**
 * @param {number} rows
 * @param {number} cols
 * @param {number} rCenter
 * @param {number} cCenter
 * @return {number[][]}
 */
var allCellsDistOrder = function(rows, cols, rCenter, cCenter) {
  const res = [];
  const visited = new Array(rows);
  for (let i = 0; i < rows; i++) {
    visited[i] = new Array(cols).fill(false);
  }
  const q = [[rCenter, cCenter]];
  visited[rCenter][cCenter] = true;
  while(q.length) {
    const cur = q.shift();
    const [x, y] = [cur[0], cur[1]];
    res.push(cur);
    if(x + 1 < rows && !visited[x + 1][y]) {
      q.push([x + 1, y]);
      visited[x + 1][y] = true;
    }
    if(x - 1 >= 0 && !visited[x - 1][y]) {
      q.push([x - 1, y]);
      visited[x - 1][y] = true;
    }
    if(y + 1 < cols && !visited[x][y + 1]) {
      q.push([x, y + 1]);
      visited[x][y + 1] = true;
    }
    if(y - 1 >= 0 && !visited[x][y - 1]) {
      q.push([x, y - 1]);
      visited[x][y - 1] = true;
    }
  }
  return res;
};
```

