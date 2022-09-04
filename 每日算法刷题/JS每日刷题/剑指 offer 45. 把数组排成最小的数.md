# 剑指 offer 45. 把数组排成最小的数

> https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/

## 思路：利用 sort 排序

```javascript
var minNumber = function(nums) {
	return nums.sort((a, b) => {
        return `${a}${b}` - `${b}${a}`;
    }).join('')
};
```

