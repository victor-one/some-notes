# Leetcode 646. 最长数对链

> https://leetcode.cn/problems/maximum-length-of-pair-chain/

## 方法一 动态规划

```JavaScript
var findLongestChain = function(pairs) {
	pairs.sort((a, b) => {
    return a[0] - b[0];
  });
  const n = pairs.length;
  const dp = new Array(n).fill(1);
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (pairs[j][1] < pairs[i][0]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }
  return dp[n-1];
};
```

## 方法二 贪心+二分查找

```javascript
var findLongestChain = function(pairs) {
	pairs.sort((a,b) => a[0] - b[0]);
  const arr = [];
  for (const p of pairs) {
    const a = p[0], b = p[1];
    if (!arr.length || a > arr[arr.length - 1]) {
      arr.push(b);
    } else {
      const index = binarySearch(arr, a);
      arr[index] = Math.min(arr[index], b);
    }
  }
  return arr.length;
};
const binarySearch = (arr, a) => {
  let left = 0, right = arr.length - 1;
  while(left < right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] >= a) {
      right = mid;
    }else {
      left = mid + 1;
    }
  }
  return left;
}
```

## 方法三 贪心

```javascript
var findLongestChain = function(pairs) {
	pairs.sort((a, b) => a[1] - b[1]);
  let cur = -Number.MAX_VALUE, res = 0;
  for(const p of pairs) {
    if(p[0] > cur) {
      cur = p[1];
      res += 1;
    }
  }
  return res;
};
```

