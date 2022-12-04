# 归并排序和快速排序

## 简言

* 应用分治思想的基础排序，包括归并排序和快速排序。

> 学习曲线：★★★

## 归并排序（Merge Sort）

### 归并排序的核心思想

* **归并排序的核心思想就是，排序一个数组，先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分有序地合并在一起，这样整个数组就会变得有序。**

![[归并排序分解图.png]]

* **归并排序使用的就是分治思想——将大问题拆解成小问题，小问题解决之后从而解决大问题。**
* 分治和递归这两个概念很相似。可以这样理解，**分治是一种解决问题的核心思想理念，而递归是一种编程技巧**，后者更偏向具体实现。这二者并不冲突。
* 分治思想后续会通过专门的一节来讨论学习，这里我们把重点放回到归并排序上。
* 根据归并的定义，先来尝试写出递归公式：

```typescript
// 递推公式：
merge_sort(p…r) = merge(merge_sort(p…q), merge_sort(q+1…r))

// 终止条件：
p >= r 不用再继续分解
```

* 相当于我们把数组拆分成两部分，对这两部分分别执行排序逻辑，再将其合并。

![[归并排序详细.png]]

### 归并排序的实现

* 由上述理解我们知道，归并排序有两个方法。
* 下面是两个方法的各自实现，**前者是负责并（合并），后者负责归（递归）**：

```typescript
/**
 * @description: 归并排序，合并逻辑
 * @param {number[]} left
 * @param {number[]} right
 * @return {number[]}
 */
function merge(left: number[], right: number[]): number[] {
	// 注意这里借助了额外的空间，所以不是原地排序算法
  let res: number[] = [];
  while (left.length && right.length) {
    if (left[0] <= right[0]) {
      res.push(left.shift());
    } else {
      res.push(right.shift());
    }
  }

  // 注意这里要将其合并，连接；而且其实 left 和 right 中一定有一个是空的数组
  return res.concat(left).concat(right);
  // return [...res, ...left, ...right];
}

/**
 * @description: 归并排序，分治逻辑，递归逻辑
 * @param {number[]} arr
 * @return {number[]}
 */
function mergeSort(arr: number[]): number[] {
  if (arr.length <= 1) return arr;
  const midIndex: number = Math.floor(arr.length / 2);
  const left: number[] = arr.slice(0, midIndex);
  const right: number[] = arr.slice(midIndex);

  return merge(mergeSort(left), mergeSort(right));
}

console.info(mergeSort([3, 4, 1, 2]));
```

* 将两个方法整合在一起，可以这样写：

```typescript
/**
 * @description: 归并排序
 * @param {number[]} arr
 * @return {number[]}
 */
function mergeSort(arr: number[]): number[] {
  if (arr.length <= 1) {
    return arr;
  }

  const midIndex: number = Math.floor(arr.length / 2);
  // 在这里调用递归函数
  const left: number[] = mergeSort(arr.slice(0, midIndex));
  const right: number[] = mergeSort(arr.slice(midIndex));
  const res: number[] = [];

  while (left.length && right.length) {
    // 重点是将每次的第一个元素拿出来比较，并放入结果内
    if (left[0] <= right[0]) {
      res.push(left.shift());
    } else {
      res.push(right.shift());
    }
  }

  // 下面两者等同，注意一定要将其联结
  // return [...res, ...left, ...right];
  return res.concat(left).concat(right);
}

console.log(mergeSort([7, 4, 5, 3, 2, 1, 6]));
```

* 我们给原来是排序实现加上打印日志，方便我们理解归并排序的过程。而且由于函数调用是递归的，这里我们用字符的多少和缩进来标识调用栈的变化，让它更直观地输出。
* 下面是添加打印日志后的具体实现。

```typescript
/**
 * @description: 归并排序，打印执行日志
 * @param {number[]} arr
 * @param {string} indent 这里我们用这个字符的缩进来标识调用栈，缩进越多，说明调用栈越深
 * @return {number[]}
 */
function mergeSort(arr: number[], indent = '----') {
  if (arr.length <= 1) return arr;

  const baseIndex = Math.floor(arr.length  / 2);
  console.log(`${indent} 当前数组 [${arr}], 从索引 ${baseIndex} 开始分割，左右数组分别为 [${arr.slice(0, baseIndex)}], [${arr.slice(baseIndex)}]`);
  // 递归在这里调用
  const left = mergeSort(arr.slice(0, baseIndex), indent + '----');
  const right = mergeSort(arr.slice(baseIndex), indent + '----');
  console.log(`${indent} 归并后左右数组分别为 [${left}], [${right}]`);
  const res = [];
  
  while (left.length && right.length) {
    if (left[0] < right[0]) {
      res.push(left.shift());
    } else {
      res.push(right.shift());
    }
  }
  
  return res.concat(left).concat(right);
}
console.table(mergeSort([5, 4, 9, 3, 7, 2, 8, 1]));
```

* 输出日志。

```shell
---- 当前数组 [5,4,9,3,7,2,8,1], 从索引 4 开始分割，左右数组分别为 [5,4,9,3], [7,2,8,1]
-------- 当前数组 [5,4,9,3], 从索引 2 开始分割，左右数组分别为 [5,4], [9,3]
------------ 当前数组 [5,4], 从索引 1 开始分割，左右数组分别为 [5], [4]
------------ 归并后左右数组分别为 [5], [4]
------------ 当前数组 [9,3], 从索引 1 开始分割，左右数组分别为 [9], [3]
------------ 归并后左右数组分别为 [9], [3]
-------- 归并后左右数组分别为 [4,5], [3,9]
-------- 当前数组 [7,2,8,1], 从索引 2 开始分割，左右数组分别为 [7,2], [8,1]
------------ 当前数组 [7,2], 从索引 1 开始分割，左右数组分别为 [7], [2]
------------ 归并后左右数组分别为 [7], [2]
------------ 当前数组 [8,1], 从索引 1 开始分割，左右数组分别为 [8], [1]
------------ 归并后左右数组分别为 [8], [1]
-------- 归并后左右数组分别为 [2,7], [1,8]
---- 归并后左右数组分别为 [3,4,5,9], [1,2,7,8]
┌─────────┬────────┐
│ (index) │ Values │
├─────────┼────────┤
│    0    │   1    │
│    1    │   2    │
│    2    │   3    │
│    3    │   4    │
│    4    │   5    │
│    5    │   7    │
│    6    │   8    │
│    7    │   9    │
└─────────┴────────┘
```

### 归并排序的性能分析（三问）

#### 1. 归并排序是稳定的算法吗

* 因为判断逻辑在 `merge` 函数中是可控的，我们可以在比较的时候按顺序放入临时数组中。
* 所以***归并排序是稳定的算法。***

#### 2. 如何分析归并排序的时间复杂度

* **分析归并排序的时间复杂度可以看成是分析递归代码的时间复杂度。**
* 而从递归的角度分析时间复杂度，就是当前问题等于子问题耗时之和加上子问题合并的时间。
* 简单概括：`T(a) = T(b) + T(c) + k`
	* *k 就是合并子问题的时间。*
* 假设对 n 个元素进行归并排序需要的时间是 `T(n)`，拆分成两个子数组后所需时间就是 `T(n / 2)`.
* 而 `merge` 函数合并两个有序子数组的时间复杂度是 `O(n)` —— 合并过程就是经历一次遍历。
* 所以套用前面的公式，归并排序的时间复杂度的计算公式如下：

```typescript
T(1) = C；  // n = 1 时，只需要常量级的执行时间，所以表示为 C。
T(n) = 2 * T(n / 2) + n； // n > 1 // ?对于这里把 n 都作为系数而存疑，个人理解为什么不是 + O(n)? 可能是把时间复杂度都抽象出来了
```

* 拆分成两个的结果可能不够直观，我们可以将其拆分成更小的。

```typescript
T(n) = 2*T(n/2) + n
     = 2*(2*T(n/4) + n/2) + n = 4*T(n/4) + 2*n
     = 4*(2*T(n/8) + n/4) + 2*n = 8*T(n/8) + 3*n
     = 8*(2*T(n/16) + n/8) + 3*n = 16*T(n/16) + 4*n
     ......
     = 2^k * T(n/2^k) + k * n
     ......
```

* 通过分解推导，公式可以简化成 `T(n) = 2^k * T(n / 2^k) + k * n`.
	* 当 `T(n / 2^k) = T(1)` 时，也就是：`n / 2^k = 1`.
	* 求解得到 `k = log₂n`. 把这个值代入上面的公式：
		* `T(n) = Cn + nlog₂n`.
		* 用大 O 标记法来表示，省略系数就是 `O(nlogn)`。
* **归并排序的时间复杂度就是 `O(nlogn)`。**
* 归并排序的时间复杂度和原始数据的有序度无关。所以时间复杂度也是非常稳定的，**最好、最坏和平均情况时间复杂度都是 `O(nlogn)`.**

#### 3. 如何分析归并排序的空间复杂度

* **归并排序的时间复杂度任何情况下都是 `O(nlogn)`.**
* 但是*归并排序的最大弱点是它不是原地排序算法*；这使得它的应用没有快速排序那么广泛。尽管快排的最坏时间复杂度在 `O(n²)`，比归并排序弱一些。
* **根本原因是归并排序在将两个数合并成一个数组的时候，需要借助额外的存储空间，使得它并不是原地排序算法。**
* 这个额外的空间在每次合并的时候创建，合并完成后释放。而每次只会有一次合并同时进行，CPU 只会有一个函数在执行，也就是*始终*只会有一个临时的内存空间在使用，因此临时内存空间最大不会超过 n，***归并排序的空间复杂度是 `O(n)`***.

### 归并排序的视频讲解

* 如果通过文字描述理解不够的话，可以通过下面这个视频来理解归并排序的原理。

<iframe src='https://player.bilibili.com/player.html?bvid=BV1Ax411U7Xx&cid=16503123&page=1&share_source=copy_web' scrolling='no' border='0' frameborder='no' framespacing='0' allowfullscreen='true' width="100%" height="520px"></iframe>

## 快速排序（Quick Sort）

* 快速排序简称快排也是使用到了分治思想。

### 快速排序的核心

* **快排会选择任意一个数据值作为分区点 `pivot`。再定义两个数组，分别保存比分区点的值更小的数据，和比分区点的值更大的数据。我们遍历整个数组，把数据放入相应的数组内，这就是核心的逻辑。再递归复用这样的逻辑进行处理，直到两个数组的区间个数缩小为 1. 递归结束。** 这样数据就是有序的。

![[快排核心.png]]

* 核心概括递推公式，就是以下内容：

```typescript
quick_sort(p…r) = quick_sort(p…q-1) + quick_sort(q+1…r)

// 终止条件：
p >= r
```

* 注意上文的终止条件——不是数组的长度为 1，而是左边的索引大于等于右边的索引。
* 再将递推公式实现成代码的形式，以下是伪代码，主要是方便理解：

```typescript
// 快速排序，A是数组，n表示数组的大小
quick_sort(A, n) {
  quick_sort_c(A, 0, n-1)
}
// 快速排序递归函数，p,r为下标
quick_sort_c(A, p, r) {
  if p >= r then return

  q = partition(A, p, r) // 获取分区点
  quick_sort_c(A, p, q-1)
  quick_sort_c(A, q+1, r)
}
```

* 归并函数中有 `merge` 函数，对应到这里也有 `partition` 分区函数。
* **分区函数所做的事情就是随机挑选一个数据作为分区点。**
* 不考虑空间消耗的话，它本身的实现非常简单，通常在 js 的实现里，取中间值作为分区点。于是就有：

```typescript
/**
 * @description: 分区函数，获取中间值作为分区点
 * @param {number[]} arr
 * @return {number}
 */
function partition(arr: number[]): number {
  const midIndex = Math.floor(arr.length / 2);
  const [pivot] = arr.splice(midIndex, 1);

  return pivot;
}
```

* 用图来示意就是下面的效果，这是从末尾取分区点的逻辑：

![[快速排序分区点.png]]

* 通过这种方式，就会使用额外的内存空间，也就是临时数组 X 和 数组 Y 来保存比分区值更大和更小的区间。这样就不属于原地排序算法。
* 想要让快排是原地排序算法，那么就要让空间复杂度为 `O(1)`.
* 空间复杂度为 `O(1)` 的方式，我们可以借鉴上一节几种排序的方式，通过交换让数组元素在原地完成分区操作。从而实现原地排序。
* 参照选择排序，我们尝试实现分区函数。

```typescript
/**
 * @description: 支持原地排序的分区函数，空间复杂度(O(1))
 * @param {number[]} arr
 * @param {number} start
 * @param {number} end
 * @return {number}
 */
function partition(arr: number[], start: number, end: number): number {
  const pivotValue = arr[end]; // 默认从末尾取分区点
  let pivotIndex = start; // 从起始位置取索引

  for (let i = start; i < end; i++) {
    // 如果当前值小于分区值，就交换到前面的位置
    if (arr[i] < pivotValue) {
      [arr[i], arr[pivotIndex]] = [arr[pivotIndex], arr[i]];
      // 更新分区值的索引，移动到下一个元素
      pivotIndex++;
    }
  }
  // 把基准值放在中间
  [arr[pivotIndex], arr[end]] = [arr[end], arr[pivotIndex]];

  return pivotIndex;
}
```

* 核心是选择分区值，再比较大小去交换位置，而交换过程是原地进行的，类似冒泡的交换逻辑。
* 可以观察下面的过程示意图来理解：

![[快速排序示意图.png]]

* 以上的图是对 `[6, 3, 11, 9, 8]` 进行快速排序的**第一轮遍历**的过程。
	* 图示中部分数据交换但实际数组未改变，是因为交换两者的索引值相同。
* 排序过程中，这里面相同的元素在经过分区操作后可能会发生交换，所以快速排序并不是稳定的排序算法。

### 快速排序的实现

#### 非原地排序

* 下面的排序实现是**不考虑空间复杂度的实现**（**非原地排序**）：

```typescript
/**
 * @description: 快速排序，同样使用分治的思想
 * @param {number[]} arr
 * @return {number[]}
 */
function quickSort(arr: number[]): number[] {
  if (arr.length <= 1) return arr;

  const midIndex = Math.floor(arr.length / 2); // 分区点索引
  // 将数组分割成三个区间
  const [baseValue] = arr.splice(midIndex, 1); // 分区点的值
  const left: number[] = []; // 存放比分区点小的值的数组
  const right: number[] = []; // 存放比分区点大的值的数组

  for (const ele of arr) {
    // 比较大小，放入对应的数组内
    if (ele <= baseValue) {
      left.push(ele);
    } else {
      right.push(ele);
    }
  }

  // 注意最后要将三分的数组再连接合并
  return quickSort(left).concat(baseValue).concat(quickSort(right));
}

console.log(quickSort([9, 8, 6, 1, 3, 2, 5]));
```

* 类似上文的归并排序，这里我们也加上日志，来观察排序的整个过程。
* 由于加日志的地方较少，下面直接列出差异的代码。

```diff
+ console.log(`${indent} 当前的左右数组是 [${left}] 和 [${right}] 和分区值 ${povit}`);
- return quickSort(left).concat(baseValue).concat(quickSort(right));
+ const sorted = quickSort(left, indent + '----').concat([povit]).concat(quickSort(right, indent + '----'));
+ console.log(`${indent} 当前分区排序后的数组为 [${sorted}]`);
+ return sorted;
```

* 打印日志如下。

```shell
---- 当前的左右数组是 [5,4,3,2,1] 和 [9,8] 和分区值 7
-------- 当前的左右数组是 [2,1] 和 [5,4] 和分区值 3
------------ 当前的左右数组是 [] 和 [2] 和分区值 1
------------ 当前分区排序后的数组为 [1,2]
------------ 当前的左右数组是 [] 和 [5] 和分区值 4
------------ 当前分区排序后的数组为 [4,5]
-------- 当前分区排序后的数组为 [1,2,3,4,5]
-------- 当前的左右数组是 [] 和 [9] 和分区值 8
-------- 当前分区排序后的数组为 [8,9]
---- 当前分区排序后的数组为 [1,2,3,4,5,7,8,9]
```

#### 原地排序

* **考虑空间复杂度的实现**（**原地排序**）：

```typescript
/**
 * @description: 支持原地排序的分区函数，空间复杂度(O(1))
 * @param {number[]} arr
 * @param {number} start
 * @param {number} end
 * @return {number}
 */
function partition(arr: number[], start: number, end: number): number {
  const pivotValue = arr[end]; // 默认从末尾取分支点
  let pivotIndex = start; // 从起始位置取索引

  for (let i = start; i < end; i++) {
    // 如果当前值小于分区值，就交换到前面的位置
    if (arr[i] < pivotValue) {
      [arr[i], arr[pivotIndex]] = [arr[pivotIndex], arr[i]];
      // 更新分区值的索引，移动到下一个元素
      pivotIndex++;
    }
  }
  // !把分区的基准值放在中间，使得数组左中右三部分整体上是有序的
  [arr[pivotIndex], arr[end]] = [arr[end], arr[pivotIndex]];

  return pivotIndex;
}

/**
 * @description: 空间复杂度(O(1))的快速排序
 * 支持原地排序，所以这里直接打印原数组即可查看效果
 * @param {number[]} arr
 * @param {number} start
 * @param {number} end
 * @return {void}
 */
function quickSortRecursive(arr: number[], start = 0, end = arr.length - 1): void {
  // 终止条件是索引的叠加
  if (start >= end) return;

  const index = partition(arr, start, end);
  // 注意分割的索引位置，分别在分区点的前后
  quickSortRecursive(arr, start, index - 1);
  quickSortRecursive(arr, index + 1, end);
}

const arr1 = [5, 7, 23, 8, 1, 2, 34, 0];
quickSortRecursive(arr1);
console.table(arr1);
```

* 加上打印日志。添加的代码较多，整理所有代码如下。

```typescript
function partition(start: number, end: number, arr: number[], indent: string): number {
  const pivotValue = arr[end];
  let pivotIndex = start;
  console.log(`${indent} 本轮分区开始的数组 [${arr}] ${indent}`);
  console.log(`${indent} 进入循环： 本轮分区值 ${pivotValue}, 初始分区索引 ${pivotIndex} ${indent}`);
  
  for (let i = start; i < end; i++) {
    if (arr[i] < pivotValue) {
      console.log(`${indent} {{ 当前比较值 ${arr[i]} 和索引对应值 ${arr[pivotIndex]} 交换, 分区索引更新后为 ${pivotIndex + 1} }} ${indent}`);
      [arr[i], arr[pivotIndex]] = [arr[pivotIndex], arr[i]];
      console.log(`${indent} {{ 交换后的数组 [${arr}] }} ${indent}`);
      pivotIndex++;
    }
  }
  
  console.log(`${indent} 结束循环： 本轮最终分区索引 ${pivotIndex}， ${arr[pivotIndex]} 与 ${arr[end]} 交换 ${indent}`);
  [arr[pivotIndex], arr[end]] = [arr[end], arr[pivotIndex]];
  console.log(`${indent} 本轮分区结束的数组 [${arr}] ${indent} \n`);
  return pivotIndex;
}

function quickSort(arr: number[], start: number = 0, end: number = arr.length - 1, indent: string = '----'): void {
  if (start >= end) return;
  const index = partition(start, end, arr, indent);
  console.log(`${indent} [${arr}] 对应的分区索引是 ${index}, 起始为 ${start}, 结束为 ${end} ${indent}`);

  quickSort(arr, start, index - 1, indent + '----');
  quickSort(arr, index + 1, end, indent + '----');
}

const arr = [9, 8, 6, 1, 3, 2, 5];
quickSort(arr);
console.table(arr);
```

* 打印出的日志如下：

```shell
---- 本轮分区开始的数组 [9,8,6,1,3,2,5] ----
---- 进入循环： 本轮分区值 5, 初始分区索引 0 ----
---- {{ 当前比较值 1 和索引对应值 9 交换, 分区索引更新后为 1 }} ----
---- {{ 交换后的数组 [1,8,6,9,3,2,5] }} ----
---- {{ 当前比较值 3 和索引对应值 8 交换, 分区索引更新后为 2 }} ----
---- {{ 交换后的数组 [1,3,6,9,8,2,5] }} ----
---- {{ 当前比较值 2 和索引对应值 6 交换, 分区索引更新后为 3 }} ----
---- {{ 交换后的数组 [1,3,2,9,8,6,5] }} ----
---- 结束循环： 本轮最终分区索引 3， 9 与 5 交换 ----
---- 本轮分区结束的数组 [1,3,2,5,8,6,9] ---- 

---- [1,3,2,5,8,6,9] 对应的分区索引是 3, 起始为 0, 结束为 6 ----
-------- 本轮分区开始的数组 [1,3,2,5,8,6,9] --------
-------- 进入循环： 本轮分区值 2, 初始分区索引 0 --------
-------- {{ 当前比较值 1 和索引对应值 1 交换, 分区索引更新后为 1 }} --------
-------- {{ 交换后的数组 [1,3,2,5,8,6,9] }} --------
-------- 结束循环： 本轮最终分区索引 1， 3 与 2 交换 --------
-------- 本轮分区结束的数组 [1,2,3,5,8,6,9] -------- 

-------- [1,2,3,5,8,6,9] 对应的分区索引是 1, 起始为 0, 结束为 2 --------
-------- 本轮分区开始的数组 [1,2,3,5,8,6,9] --------
-------- 进入循环： 本轮分区值 9, 初始分区索引 4 --------
-------- {{ 当前比较值 8 和索引对应值 8 交换, 分区索引更新后为 5 }} --------
-------- {{ 交换后的数组 [1,2,3,5,8,6,9] }} --------
-------- {{ 当前比较值 6 和索引对应值 6 交换, 分区索引更新后为 6 }} --------
-------- {{ 交换后的数组 [1,2,3,5,8,6,9] }} --------
-------- 结束循环： 本轮最终分区索引 6， 9 与 9 交换 --------
-------- 本轮分区结束的数组 [1,2,3,5,8,6,9] -------- 

-------- [1,2,3,5,8,6,9] 对应的分区索引是 6, 起始为 4, 结束为 6 --------
------------ 本轮分区开始的数组 [1,2,3,5,8,6,9] ------------
------------ 进入循环： 本轮分区值 6, 初始分区索引 4 ------------
------------ 结束循环： 本轮最终分区索引 4， 8 与 6 交换 ------------
------------ 本轮分区结束的数组 [1,2,3,5,6,8,9] ------------ 

------------ [1,2,3,5,6,8,9] 对应的分区索引是 4, 起始为 4, 结束为 5 ------------
┌─────────┬────────┐
│ (index) │ Values │
├─────────┼────────┤
│    0    │   1    │
│    1    │   2    │
│    2    │   3    │
│    3    │   5    │
│    4    │   6    │
│    5    │   8    │
│    6    │   9    │
└─────────┴────────┘
```

#### 两种排序的耗时比较

* 可以简单对这二者进行比较，第二种方式节约了空间，但或许会有时间损失。这里我们存疑，用案例来验证。
* 还是和上期类似，我们先生成一个拥有 10000 随机数值的元素组成的数组，将其排序。观察两种实现的差别。
* 生成函数复用上期实现，这里不再赘述。

```typescript
const randomArr = generateArr(10000);

console.time('quickSort');
quickSort(randomArr);
console.timeEnd('quickSort');
console.log('quickSort ===>', randomArr[0], randomArr[1]);

console.time('quickSort in place');
quickSortRecursive(randomArr);
console.timeEnd('quickSort in place');
console.log('quickSort in place ===>', randomArr[0], randomArr[1]);
```

* 结果如下：

```shell
quickSort: 48.80419921875 ms
quickSort ===> 2088 9392
quickSort in place: 17.260986328125 ms
quickSort in place ===> 0 6
```

* 两次运行，第一种实现并不会修改原始数组的内容，数组仍然是无序的，也就是它会额外消耗内存空间。打印结果显示数组内容不变。
* 第二种实现是直接修改了原数组，打印结果验证了前面的说法。原始数组最终也变得有序。
* 甚至我们可以发现第二种原地排序的实现不仅空间上节约，时间上也更有效。
* 而且所需时间明显优于上一期的运行结果。也可以验证它的时间复杂度是优于上期三种基础排序的。

### 快速排序的性能分析（三问）

#### 1. 快速排序是种怎样的算法

* **快速排序是一种不稳定的原地排序算法。**
* 前面我们已经说明。

#### 2. 如何分析快速排序的时间复杂度

* 分析快排的时间复杂度其实和归并排序类似。因为它也是递归实现的。
* 如果每次分区恰好能将数组分成大小相近的两个区间，那么快排的时间复杂度的递推求解公式和归并是相同的。
* 意味着时间复杂度也是 `O(nlogn)`.

```typescript
T(1) = C; n=1时，只需要常量级的执行时间，所以表示为C。
T(n) = 2 * T(n / 2) + n; n > 1
```

* 但是，前面也提到，这个结果的前提是每次分区恰好要让原始区间接近一分为二。实际上这很难实现。
* 比如一个原本有序的数组，取末位数作为分区点，那么第一次分区的结果就是极其不相等的，大大影响了排序效率。这种时候的时间复杂度就变成了 `O(n²)`.
* 这里分析递归的时间复杂度的更好方式是递归树，这部分会在后面的树章节进行学习掌握，现在先不深入。可以先记住结论：`T(n)` 在多数时候的时间复杂度都是 `O(nlogn)`. 极端情况下才会退化到 `O(n²)`. 剩下的后面章节会继续深入。

#### 3. 如何分析快速排序的空间复杂度

* 前面已经分析过，我们可以实现成 `O(1)` 复杂度的方式。而非原地排序实现的方式应该是 `O(n)`.

### 快排的应用（查找数组第 k 大的元素）

* [**Leetcode-215**](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

#### 常规解法

* 通常的实现方式：

```typescript
/**
 * @description: 从大到小排序，并取出相应的值
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
function findKthLargest(nums: number[], k: number): number {
  nums.sort((a, b) => b - a );

  return nums[k - 1];
};
```

#### 快排求解的思路

* 我们选择数组区间 `A[0...n-1]` 的最后一个元素 `A[n-1]` 作为 `pivot`，对数组`A[0...n-1]` 原地分区，这样数组就分成了三部分，`A[0...p-1]`、`A[p]`、`A[p+1...n-1]`。
* 如果 `p + 1 = K`，那 `A[p]` 就是要求解的元素；如果 `K > p + 1`, 说明第 K 大元素出现在 `A[p+1...n-1]` 区间，我们再按照上面的思路递归地在 `A[p+1...n-1]` 这个区间内查找。同理，如果 `K < p + 1`，那我们就在 `A[0...p-1]` 区间查找。

![[快排查找第k大元素.png]]

* 下面是快排的实现（分区函数和上文一样，核心关注**查找函数**）

```typescript
/**
 * @description: 支持原地排序的分区函数，空间复杂度(O(1))
 * @param {number[]} arr
 * @param {number} start
 * @param {number} end
 * @return {number}
 */
function partition(arr: number[], start: number, end: number): number {
  const pivotValue = arr[end]; // 默认从末尾取分支点
  let pivotIndex = start;

  // 注意其实位置不是从 0 开始，而是从 start 开始
  for (let i = start; i < end; i++) {
    // 如果当前值小区分区值，就交换到前面的位置
    if (arr[i] < pivotValue) {
      [arr[i], arr[pivotIndex]] = [arr[pivotIndex], arr[i]];
      // 更新分区值的索引，移动到下一个元素
      pivotIndex++;
    }
  }
  // 把基准值放在中间
  [arr[pivotIndex], arr[end]] = [arr[end], arr[pivotIndex]];

  return pivotIndex;
}

/**
 * @description: 使用快排实现寻找第 k 大元素
 * 时间复杂度 O(n)
 * @param {number} nums
 * @param {number} k
 * @return {number}
 */
function findKthLargest(nums: number[], k: number): number {
  let left = 0;
  let right = nums.length - 1;
  // 数组中第 k 大的数等于排序之后从前往后第 len - k 个数，注意数组从前往后是以 0 起始的，所以可以直接相减
  const n = nums.length - k;

  while (true) {
    // 寻找分区点（实际在这个过程中，数组也是在排序的）
    const pivotIndex = partition(nums, left, right);
    if (pivotIndex === n) {
      return nums[pivotIndex];
      // 如果分区点索引小于要求解的值，则继续进行分区
    } else if (pivotIndex < n) {
      // 起始值置为分区值加 1
      left = pivotIndex + 1;
    } else {
      right = pivotIndex - 1;
    }
  }
}
```

* **注意分区函数的执行，在这个过程中数组本身就是在排序的（原地排序）。**
* 分区的过程中，k 所在的区间一直在排序，一直在变得更有序。

##### 分析快排求解的时间复杂度

* 快排求解的时间复杂度是 `O(n)`。
* 第一次分区查找，我们需要对大小为 n 的数组执行分区操作，需要遍历 n 个元素。第二次分区查找，只需要对大小为 `n / 2` 的数组执行分区操作，需要遍历 `n / 2` 个元素。依次类推，分区遍历元素的个数分别为 `n / 2`、`n / 4`、`n / 8`、`n / 16`.……直到区间缩小为 1。
* 如果把每次分区遍历的元素个数加起来，就是：`n + n / 2 + n / 4 + n / 8 + ...+ 1`。这其实是一个等比数列求和，最后的和等于 `2n - 1`。这样的系数是可以忽略不计的。所以上述解决思路的时间复杂度就为 `O(n)`。

#### 其他解法

* 我们也可以遍历数组，每次取最小值，直到取到 k 为止。这样的算法的时间复杂度是 `O(k * n)`. 而这里的系数 k 不能省略，因为 k 是不确定的。当 k 较大时，例如 `k = n / 2` 或者 `k = n`，那其实这种算法的时间复杂度就是 `O(n²)` ，所以它也不是最优的。

## 小结两种排序

### 两种排序的异同点

* 归并排序和快速排序是两种稍微复杂的排序算法，它们用的都是分治的思想，代码都通过递归来实现，过程非常相似。
* 它们的区别在于：
	* **归并排序的过程是自下而上的，先处理子问题，然后再合并。拆分的时候不排序的，真正排序过程先从子问题开始，再而向上合并。**
	* **而快速排序的过程是自上而下的，先排序分区，再处理子问题。**
* 观察下面的图来更好地理解：

![[归并排序和快速排序.png]]

* 它们的平均时间复杂度都是 `O(nlogn)`. 快速排序的最坏时间复杂度是 `O(n²)`. 但可以通过合理选择分区点来避免这种情况发生。
* 归并排序算法是一种在任何情况下时间复杂度都比较稳定的排序算法，这也使得它存在致命的缺点——归并排序不是原地排序算法，空间复杂度比较高，是 `O(n)`。

#ALG
