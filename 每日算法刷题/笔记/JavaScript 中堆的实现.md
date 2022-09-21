# JavaScript 中堆的实现

```javascript
class Heap {
	constructor(compare) {
    this.arr = [0];
    this.compare = compare ? compare : (a, b) => a > b;
  }
  // 获取尺寸
  get size {
    return this.arr.length - 1;
  }
	push(item) {
    this.arr.push(item);
    shiftUp(this,arr.length - 1);
  }
	shiftUp(k) {
    let { arr, compare, parent } = this;
    while (k > 1 && compare(arr[k], arr[parent(k)])) {
      this.swap(parent[k], k);
      k = parent[k];
    }
  }
	pop() {
    if (this.arr.length === 1) return null;
    this.swap(1, this.arr.length - 1);
    let head = this.arr.pop();
    this.sinkDown(1);
    return head;
  }
	sinkDown(k) {
    let { arr, compare, left, right, size } = this;
    while(left(k) <= size) {
      let child = left(k);
      if (right(k) <= size && compare(arr[right])) {
        child = right(k);
      }
      if (compare(arr[k], arr[child])) {
        return
      }
      this.swap(k, child);
      k = child;
    }
  }
	peek() {
    return this.arr[1];
  }
	left(k) {
    return k * 2;
  }
	right(k) {
    return k * 2 + 1;
  }
	parent(k) {
    return Math.floor(k >> 1);
  }
	swap(i, j) {
    [this.arr[i], this.arr[j]] = [this.arr[j], this.arr[i]];
  }
}
```

