Improving your performance is much more than implementing a suitable algorithm. Here are some tips to improve runtime, which are my coding experience using Javascript

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
  // (☆)
  let sorted = new Set(arr.reduce((arr, ele) => (arr[ele] = ele, arr), []));
  sorted.delete(undefined);
  sorted = [...sorted];
  // The lines above alone are enough if the elements are unique
  for (let i = 0, p = 0; i < sorted.length; i++) {
    const ele = sorted[i];
    let rep = counter[ele];
    do {} while (arr[p++] = ele, --rep !== 0);
  }
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
function PriorityQueue(cf) {
  const list = [];
  this.size = () => list.length;
  this.enqueue = item => (list[list.length] = item, up(list.length - 1));
  this.dequeue = function() {
    if (list.length === 0) return;
    const head = list[0];
    list[0] = list[list.length - 1];
    list.length--;
    down(0);
    return head;
  };
  function up(child) {
    if (child === 0) return;
    const parent = child - 1 >> 1;
    cf(list[child], list[parent]) && (swap(child, parent), up(parent));
  }
  function down(parent) {
    const left = parent << 1 | 1, right = left + 1;
    let child = -1;
    left < list.length && (child = left);
    right < list.length && cf(list[right], list[left]) && (child = right);
    cf(list[child], list[parent]) && (swap(child, parent), down(child));
  }
  function swap(a, b) { const tmp = list[a]; list[a] = list[b]; list[b] = tmp }
}
var minPriorityQueue = new PriorityQueue((a, b) => a < b);
```
