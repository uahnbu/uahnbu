Improving your performance is much more than implementing a suitable algorithm. Here are some tips to improve runtime, which are my coding experience using Javascript.

# Bitwise
* `(n & 1) === 0` &xlarr; `n % 2 === 0`
Note that the operation is wrapped in **brackets**, since bitwise operators have lower precedence than the equal operator. Furthermore, omitting the equal operator is **inadvisable**.
* `n >> 1` &xlarr; `Math.floor(n / 2)`
* `n / 3 - 1 | 0` &xlarr; `Math.floor(n / 3 - 1)`
Note that the brackets is **omitted**, since bitwise operators have higher precedence than arithmetic operators. However, in `(n / 3 | 0) - 1`, the bracket is **essential**.
* `n *= 2` &xlarr; `n <<= 1`
* `true && (n = 2);` &xlarr; `if (true) n = 2;`
* `true && (n = 2) || (n = 3);` &xlarr; `if (true) n =2; else n = 3;`
Note that `n = 2` must return an **equivalence** of `true`,
`true && (n = 0) || (n = 3)` will behave improperly.
Instead, use `true && (n = 0, true) || (n = 3)`.
Besides, similar to the above mentioned, the **brackets** are necessary when assigning values.
Use `if` only when **reserved** keywords are accompanied.
`if (true) break; if (true) return;`
# Loop
* **Increment for-loop** is faster than decrement for-loop.
```
for (let i = 1; i < s.length >> 1; i++);
```
is faster than
```
for (let i = (s.length >> 1) - 1; i > 0; i--);
```
although you may consider the second one to be faster due to fewer calculations each iteration.
* It's faster if you place all the expressions **inside** condition block.
```
let i = 0; for (; tmp[i] = arr[i], arr[i] >>= 1, i++ < arr.length ;);
```
is faster than
```
for (let i = 0; i < arr.length; i++) { tmp[i] = arr[i]; arr[i] >>= 1 }
```
* **do-while** is faster than while and for.
```
let i = 0; do {} while (tmp[i] = arr[i], arr[i] >>= 1, i++ < arr.length);
```
is faster than
```
let i = 0; for (; tmp[i] = arr[i], arr[i] >>= 1, i++ < arr.length ;);
```

# Sort
A manual quick sort function is faster than the built-in one.
```javascript
const quickSort = (arr, left = 0, right = arr.length - 1) => {
  if (arr.length <= 1) return;
  let pivot = arr[right + left >> 1];
  let i = left, j = right, tmp;
  while (i <= j) {
    while (arr[i] < pivot) i++; while (arr[j] > pivot) j--;
    i <= j && (tmp = arr[i], arr[i++] = arr[j], arr[j--] = tmp);
  }
  left < i - 1 && quickSort(arr, left, i - 1);
  i < right && quickSort(arr, i, right);
};
```
Furthermore, you can sort **multiple** same-lengthed arrays simultaneously. E.g.
```javascript
const scores = [5, 2, 5, 8, 4];
const indexes = Array(scores.length).fill().map((_, i) => i);
const quickSort = (arr, dep, left, right, dir) => {
  if (arr.length <= 1) return;
  let pivot = arr[right + left >> 1];
  let i = left, j = right, tmp;
  while (i <= j) {
    while (arr[i] * dir < pivot * dir) i++;
    while (arr[j] * dir > pivot * dir) j--;
    i <= j && (
      dep && (tmp = dep[i], dep[i] = dep[j], dep[j] = tmp),
      tmp = arr[i], arr[i++] = arr[j], arr[j--] = tmp
    );
  }
  left < i - 1 && quickSort(arr, dep, left, i - 1, dir);
  i < right && quickSort(arr, dep, i, right, dir);
};
quickSort(scores, indexes, 0, scores.length - 1, -1);
```
Besides, the trick below will increase your sorting speed by 4 times so long as the elements of your array are all numbers of unsigned 32-bit integers:
```javascript
function cheatSort(arr) {
  const counter = arr.reduce((cnt, ele) => (cnt[ele] = (cnt[ele] || 0) + 1, cnt), {});
  const sorted = new Set(arr.reduce((arr, ele) => (arr[ele] = ele, arr), []));
  let i = 0;
  sorted.delete(undefined);
  sorted.forEach(e => { do {} while (arr[i++] = e, --counter[e] !== 0) });
  // If the elements are unique just return [...sorted]
}
```

# Other
* `Array(n).fill()` &xlarr; `Array.from({length: n})` or `[...Array(n)]`
* `story = stories.concat.apply([], stories)` &xlarr; `story = stories.flat()`
* Storing array length for 1 loop is **unnecessary**.
Storing the number of rows and columns for 2d-array is advisable.
* Working with an array of **characters** is faster than working with a string.
* `str.slice(1).reduce(() => {})` &xlarr; `for (let i = 1; i < str.length; i++) {}`
* **Set** is faster than Array, **Object** is faster than Map. Use Map when you work with multiple `add` and `delete`.
---
# Frequently used functions
```javascript
function Queue(mylist = []) {
  const list = [...mylist];
  let index = 0;
  Object.defineProperties(this, {
    size: { get: () => list.length - index },
    first: { get: () => index < list.length ? list[index] : null },
    last: { get: () => index < list.length ? list[~-list.length] : null }
  });
  this.dequeue = () => index < list.length ? list[index++] : null;
  this.enqueue = function(item) {
    list[list.length] = item;
    return this;
  };
  this.pop = function() {
    if (index >= list.length) return null;
    const val = list[~-list.length];
    --list.length;
    return val;
  };
};
```
```javascript
function PriorityQueue(cf) {
  const list = [];
  Object.defineProperty(this, 'size', { get: () => list.length });
  this.enqueue = item => (list[list.length] = item, up(~-list.length));
  this.dequeue = function() {
    if (list.length === 0) return null;
    const head = list[0];
    list[0] = list[~-list.length];
    --list.length, down(0);
    return head;
  };
  function up(child) {
    if (child === 0) return;
    const parent = ~-child >> 1;
    cf(list[child], list[parent]) && (swap(child, parent), up(parent));
  }
  function down(parent) {
    const left = parent << 1 | 1, right = -~left;
    let child = -1;
    left < list.length && (child = left);
    right < list.length && cf(list[right], list[left]) && (child = right);
    cf(list[child], list[parent]) && (swap(child, parent), down(child));
  }
  function swap(a, b) { const tmp = list[a]; list[a] = list[b]; list[b] = tmp }
}

const maxPriorityQueue = new PriorityQueue((a, b) => a > b);
const minPriorityQueue = new PriorityQueue((a, b) => a < b);
```
```javascript
function FastPriorityQueue(cp, list = []) {
  let size = list.length;
  Object.defineProperty(this, 'size', { get: () => size });
  Object.defineProperty(this, 'list', { get: () => list });
  this.clone = () => new PriorityQueue(cp, list.slice(0, size));
  this.heapify = arr => {
    size = list.length = arr.length;
    for (let i = 0; i < arr.length; ++i) list[i] = arr[i];
    for (let i = size >> 1; i >= 0; --i) percolateDown(i);
  };
  this.push = val => void (list[size] = val, percolateUp(size++));
  this.pop = () => {
    if (size === 0) return;
    const ans = list[0];
    if (--size === 0) return ans;
    list[0] = list[size], percolateDown(0);
    return ans;
  };
  this.kTop = k => {
    if (size === 0) return [];
    k = Math.min(size, k);
    const newSize = Math.min((k > 0 ? 2 ** ~-k : 0) + 1, size);
    const pq = new PriorityQueue(cp, list.slice(0, newSize));
    const podium = Array(k).fill().map(() => pq.pop());
    return podium;
  };
  this.peek = () => size === 0 ? void 0 : list[0];
  this.trim = () => void(list = list.slice(0, size));
  this.isEmpty = () => size === 0;
  this.forEach = cb => {
    if (size === 0 || typeof cb !== "function") return;
    const pq = new PriorityQueue(cp, list.slice(0, size));
    for (let i = 0; i < size; ++i) cb(pq.pop(), i);
  };
  this.replaceTop = val => {
    if (size === 0) return;
    const ans = list[0];
    return list[0] = val, percolateDown(0), ans;
  };
  this.remove = val => {
    for (let i = 0; i < size; ++i) {
      if (cp(list[i], val) || cp(val, list[i])) continue;
      return removeAt(i), true;
    }
    return false;
  };
  this.removeMatch = cb => {
    if (typeof cb !== 'function') return;
    for (var i = 0; i < size; ++i) {
      if (cb(list[i])) return removeAt(i);
    }
  };
  this.removeMatchAll = (cb, limit) => {
    if (size === 0 || typeof cb !== "function") return [];
    limit = Math.min(limit || Infinity, size);
    const res = Array(limit), tmp = Array(size);
    let resSize = 0, tmpSize = 0;
    while (size !== 0 && resSize < limit) {
      const item = this.pop();
      if (cb(item)) res[resSize++] = item;
      else tmp[tmpSize++] = item;
    }
    res.length = resSize;
    for (let i = 0; i < tmpSize; ++i) this.push(tmp[i]);
    return res;
  };
  const removeAt = i => {
    if (i > size - 1 || i < 0) return;
    return percolateUp(i, true), this.pop();
  };
  const percolateUp = (i, forced) => {
    let val = list[i], p, ap;
    while (i > 0) {
      ap = list[p = ~-i >> 1];
      if (!forced && !cp(val, ap)) break;
      list[i] = ap, i = p;
    }
    list[i] = val;
  };
  const percolateDown = i => {
    const hsize = size >>> 1, ai = list[i];
    let l, r, bestc;
    while (i < hsize) {
      bestc = list[l = i << 1 | 1], r = l + 1;
      if (r < size && cp(list[r], bestc)) bestc = list[l = r];
      if (!cp(bestc, ai)) break;
      list[i] = bestc, i = l;
    }
    list[i] = ai;
  };
}

const maxPriorityQueue = new FastPriorityQueue((a, b) => a > b);
const minPriorityQueue = new FastPriorityQueue((a, b) => a < b);
```
```javascript
function kSum(nums, target, k, start = 0) {
  const tuples = [];
  if (k === 2) {
    let lo = start, hi = ~-nums.length, sum;
    while (lo < hi) {
      sum = nums[lo] + nums[hi];
      if (sum > target) --hi;
      else if (sum < target) ++lo;
      else {
        tuples[tuples.length] = [nums[lo], nums[hi]];
        while (nums[lo] === nums[-~lo]) ++lo;
        while (nums[hi] === nums[~-hi]) --hi;
        ++lo;
      }
    }
    return tuples;
  }
  for (let i = start; i < nums.length; ++i) {
    if (k * nums[i] > target) break;
    if (i > start && nums[i] === nums[~-i]) continue;
    const iSum = kSum(nums, target - nums[i], ~-k, -~i);
    for (let j = 0; j < iSum.length; ++j) {
      iSum[j].unshift(nums[i]);
      tuples[tuples.length] = iSum[j];
    }
  }
  return tuples;
};

const fourSum = (nums, target) => kSum(nums.sort((a, b) => a - b), target, 4);
```
```javascript
function Trie() {
  const trie = {};
  this.insert = function(key, val) {
    let node = trie;
    for (let i = 0; i < key.length; ++i) {
      node = node[key[i]] ||= {};
    }
    node.val = val;
  };
  this.search = function(key) {
    let node = trie;
    for (let i = 0; i < key.length; ++i) {
      if (!(node = node[key[i]])) return null;
    }
    return node.val ?? null;
  };
  this.remove = function(key) {
    let node = trie;
    for (let i = 0; i < key.length; ++i) {
      if (!(node = node[key[i]])) return false;
    }
    return delete node.val;
  };
  this.searchPrefix = function(prefix) {
    let node = trie;
    for (let i = 0; i < prefix.length; ++i) {
      if (!(node = node[prefix[i]])) return {};
    }
    const matches = {};
    (function dfs(node, key) {
      node.val != null && (matches[key] = node.val);
      for (const prop in node) {
        prop !== 'val' && dfs(node[prop], key + prop);
      }
    })(node, prefix);
    return matches;
  };
}
```
