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
```
const quickSort = (arr, left = 0, right = arr.length - 1) => {
  if (arr.length <= 1) return;
  let pivot = arr[right + left >> 1];
  let i = left, j = right, tmp;
  while (i <= j) {
    while (arr[i] < pivot) i++; while (arr[j] > pivot) j--;
	// (☆)
    i <= j && (tmp = arr[i], arr[i++] = arr[j], arr[j--] = tmp);
  }
  left < i - 1 && quickSort(arr, left, i - 1);
  i < right && quickSort(arr, i, right);
};
```
Furthermore, you can sort **multiple** same-lengthed arrays simultaneously. E.g.
```
const scores = [5, 2, 5, 8, 4];
const indexes = Array(scores.length).fill().map((_, i) => i);
const quickSort = (arr, dependence, left, right) => {
    // (☆)
	i <= j && (swap(arr, i, j), swap(dependence, i, j));
};
quickSort(scores, indexes);
```
Besides, the trick below will increase your sorting speed by 4 times so long as the elements of your array are all numbers of unsigned 32-bit integers:
```
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
