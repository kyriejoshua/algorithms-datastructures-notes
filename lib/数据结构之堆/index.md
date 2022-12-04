# 堆和堆排序

## 简言

* 堆排序和快速的排序的时间复杂度是一样的，都是 `O(nlogn)`. 但在实际的开发中，快速排序的性能会比堆排序好。这是为什么呢？
* 本节主要关注堆这种数据结构和堆排序的实现。

> 学习曲线：★★★☆

## 如何理解堆

### 堆的定义

* **堆是完全二叉树。**
	* 除了最后一层，其他层的节点个数都是满的，而最后一层的节点靠左排列。
* **堆中所有的节点值必须大于等于（或小于等于）当前节点的左右子节点的值。**
	* 堆中的每个节点的值必须大于等于（或小于等于）其子树中每个节点的值。
	* **对于每个节点的值都大于等于其子节点的值的堆，称为大顶堆；反之称之为小顶堆。**
* 观察下面的树，判断这些树是否是堆。

![[堆的示意图.png]]

* 上图四个树分别是：大顶堆，大顶堆，小顶堆，不是堆。
* 其实对于同一组数据，也可以形成不同的堆。

### 如何实现一个堆？

* 了解如何实现堆之前，我们先了解如何来存储堆这种数据结构。

#### 如何存储堆

* 前面的章节提到过，**完全二叉树适合用数组来存储**，因为不需要存储左右子节点的指针。仅通过下标，就可以找到一个节点的左右子节点和父节点。
* 下面就是用数组来存储一个堆的示例。

![[堆的存储.png]]

* 从图中可以发现，下标 i 的左子节点就是下标为 `2 * i` 的节点；下标 i 的右子节点就是下标为 `2 * i + 1` 的节点。
* 父节点就是下标为 `Math.floor(i / 2)` 的节点。

### 堆的核心操作

* 堆的核心操作这里主要介绍两种：
	* 往堆中插入一个元素；
	* 删除堆顶元素。

#### 1. 往堆中插入一个元素

* 往堆中插入一个元素后，需要继续满足堆的两个特性。
* 如果直接把新插入的元素放到堆的末尾，如下图。就会导致原来的大顶堆不满足条件了。
* 我们需要对其进行调整，这个调整的过程称之为**堆化(heapify)**.

![[堆的插入.png]]

* 堆化实际上有两种，**从下往上和从上往下**。这里先介绍从下往上的堆化方法。
* **堆化的本质比较简单，就是顺着节点的路径，向上或者向下比较节点值的大小，然后进行交换。**
* 两种堆化的定义：
	* **自下而上堆化：将节点与其父节点比较，如果节点大于父节点（大顶堆）或者节点小于父节点（小顶堆），则节点与父节点交换位置。**
	* **自上而下堆化：将节点与其左右子节点比较，如果存在左右子节点大于当前节点（大顶堆）或左右子节点小于当前节点（小顶堆），则将当前节点的最大值（大顶堆）或最小值（小顶堆）与当前节点交换位置。**
* 下面是堆化的分解图，*让新插入的节点和父节点比较大小，如果不满足子节点小于等于父节点值的大小关系，就互换这两个节点。重复这个过程，直到父子节点之间满足大小关系。*

![[堆化示意图.png]]

* 可以结合着代码来理解。
* 下面是两种堆化的实现。**从下往上**堆化形成小顶堆：

```typescript
/**
 * @description: 从下往上堆化，交换当前节点与父节点
 * 这里的方向指的是比较的方向，将当前节点与父节点比较
 * @param {number[]} arr
 * @param {number} i
 * @return {void}
 */
function heapify(arr: number[], i: number): void {
  while (Math.floor(i / 2) > 0 && arr[Math.floor(i / 2)] > arr[i]) {
    swap(arr, i, Math.floor(i / 2));
    i = Math.floor(i / 2);
  }
}
```

* 这是**从上往下**堆化的实现，构建的是小顶堆。

```typescript
/**
 * @description: 自上往下的堆化，交换当前节点和左右子节点之一
 * 这里的方向指的是比较的方向，将当前节点与左右子节点比较
 * 这是堆化小顶堆的写法，大顶堆也类似
 * @param {number[]} arr
 * @param {number} heapSize
 * @param {number} i
 * @return {void}
 */
function heapify(arr: number[], heapSize: number, i: number): void {
  let minIndex = i;
  
  while (true) {
    // 下面两个判断找出左右子节点中的较小值，将其与当前节点交换
    if (2 * i <= heapSize && arr[minIndex] > arr[2 * i]) minIndex = 2 * i;
    if ((2 * i + 1) <= heapSize && arr[minIndex] > arr[2 * i +1]) minIndex = 2 * i + 1;
    // 相等时说明已经堆化到了最后
    if (minIndex === i) break;
    swap(arr, i, minIndex);
    i = minIndex;
  }
}
```

* 下面是堆的 TS 的简单实现，含插入操作。

```typescript
class Heap {
  private arr: any[]; // 使用数组表示堆
  private max: number; // 堆可以存储的最大数量
  private count: number; // 堆中已经存在的数据

  /**
   * @description: 初始化堆 
   * @param {number} capacity
   * @return {void}
   */
  constructor(capacity: number) {
    this.arr = new Array(capacity + 1);
    this.max = capacity;
    this.count = 0;
  }

  /**
   * @description: 交换数组中对应索引的位置
   * @param {any[]} arr
   * @param {number} i
   * @param {number} j
   * @return {void}
   */
  private swap(arr: any[], i: number, j: number): void {
    [arr[j], arr[i]] = [arr[i], arr[j]];
  }

  /**
   * @description: 自下往上堆化
   * @param {any} data
   * @return {boolean}
   */
  public insert(data: any): boolean {
    if (this.count > this.max) return false;
    this.count++;
    this.arr[this.count] = data;
    let i = this.count;

    // 索引从后往前
    while (Math.floor(i / 2) > 0 && this.arr[i] > this.arr[Math.floor(i / 2)]) {
      // 交换位置
      this.swap(this.arr, i, Math.floor(i/2));
      i /= 2;
      i = Math.floor(i);
    }

    return true;
  }
}

const heap = new Heap(8);
heap.insert('hello world');
console.log('heap ===>', heap);
```

* 原文中的 Java 实现，以供参考。

```java
public class Heap {
  private int[] a; // 数组，从下标1开始存储数据
  private int n;  // 堆可以存储的最大数据个数
  private int count; // 堆中已经存储的数据个数

  public Heap(int capacity) {
    a = new int[capacity + 1];
    n = capacity;
    count = 0;
  }

  public void insert(int data) {
    if (count >= n) return; // 堆满了
    ++count;
    a[count] = data;
    int i = count;
    while (i/2 > 0 && a[i] > a[i/2]) { // 自下往上堆化
      swap(a, i, i/2); // swap()函数作用：交换下标为i和i/2的两个元素
      i = i/2;
    }
  }
 }
```

#### 2. 删除堆顶元素

* 从堆的定义中可以发现，**堆顶元素存储的就是堆中数据的最大值或者最小值。** 根据大顶堆或小顶堆而表现不同。
* 假设构造一个大顶堆，堆顶元素就是最大的元素。当删除堆顶元素之后，就需要把第二大的元素放到堆顶，那么第二大元素肯定会出现在左右子节点中。然后再继续迭代地删除第二大节点，依次类推，直到叶子节点删除。
* 下面是这种方法的分解图，但这种方法也有一个问题，就是最后堆化出来的堆并不满足完全二叉树的特性。

![[删除堆顶元素错误示意图.png]]

* 对于这个问题，可以稍微改变一下思路来解决。
	* **删除堆顶元素后，把最后一个节点放到堆顶，然后利用同样的父子节点对比方法，对于不满足父子节点大小关系的，互换两个节点，并且重复这个过程，直到父子节点之间满足大小关系为止。**
	* **这就是从上往下的堆化方法。**
* 因为移除的是数组的最后一个元素，而在堆化的过程中，又都是交换操作，所以不会出现数组中的空洞，也就是这种方法堆化之后的结果仍然是满足二叉树的特性的。

![[删除堆顶元素示意图.png]]

### 堆的实现

* 下面是含堆化和删除堆顶元素的完整的堆的实现，也涵盖了插入操作。
	* 包含插入和删除的示例操作和效果。

```typescript
class Heap {
  private arr: any[]; // 使用数组表示堆
  private max: number; // 堆可以存储的最大数量
  private count: number; // 堆中已经存在的数据数量

  /**
   * @description: 初始化堆 
   * @param {number} capacity
   * @return {void}
   */
  constructor(capacity: number) {
    this.arr = new Array(capacity + 1);
    this.max = capacity;
    this.count = 0;
  }

  /**
   * @description: 交换数组中对应索引的位置
   * @param {any[]} arr
   * @param {number} i
   * @param {number} j
   * @return {void}
   */
  private swap(arr: any[], i: number, j: number): void {
    [arr[j], arr[i]] = [arr[i], arr[j]];
  }

  /**
   * @description: 堆化
   * 按照大顶堆的方式堆化
   * @param {number[]} arr
   * @param {number} heapSize
   * @param {number} i
   * @return {void}
   */
  private heapify(arr: number[], heapSize: number, i: number): void {
    let maxIndex = i;

    while (true) {
      // 保存较大值的索引，分别与左右子节点比较
      if (2 * i <= heapSize && arr[maxIndex] < arr[2 * i]) maxIndex = 2 * i;
      if ((2 * i + 1) <= heapSize && arr[maxIndex] < arr[2 * i + 1]) maxIndex = 2 * i + 1;
      if (maxIndex === i) break;
      this.swap(arr, i, maxIndex);
      i = maxIndex;
    }
  }

  /**
   * @description: 插入一个元素
   * 自下往上堆化
   * @param {any} data
   * @return {boolean}
   */
  public insert(data: any): boolean {
    if (this.count > this.max) return false;
    this.count++;
    this.arr[this.count] = data;
    let i = this.count;

    // 索引从后往前
    while (Math.floor(i / 2) > 0 && this.arr[i] > this.arr[Math.floor(i / 2)]) {
      // 交换位置
      this.swap(this.arr, i, Math.floor(i / 2));
      i /= 2;
      i = Math.floor(i);
    }

    return true;
  }

  /**
   * @description: 删除堆顶元素
   * 交换堆顶元素到末尾，再从上往下进行堆化
   * @return {void}
   */
  public removeMax() {
    // 当前堆中没有数据
    if (this.count === 0) return -1;
    // 把末尾元素交换到堆顶点
    this.arr[1] = this.arr[this.count];
    this.arr.pop(); // 利用 js 的方法直接剔除最后一个元素
    // 剔除最后一个节点
    this.count--;
    // 自上往下堆化
    this.heapify(this.arr, this.count, 1);
  }
}

// 建立一个有着四个元素的堆，分别插入四个元素，再删除堆顶元素，观察其变化
const heap = new Heap(4);
heap.insert(4);
heap.insert(7);
heap.insert(2);
heap.insert(3);
console.log('heap before deleting ==>', heap);
heap.removeMax();
console.log('heap after deleting ==>', heap);
```

* 这是删除过程的 Java 代码片段，以供参考。

```java
public void removeMax() {
  if (count == 0) return -1; // 堆中没有数据
  a[1] = a[count];
  --count;
  heapify(a, count, 1);
}

private void heapify(int[] a, int n, int i) { // 自上往下堆化
  while (true) {
    int maxPos = i;
    if (i*2 <= n && a[i] < a[i*2]) maxPos = i*2;
    if (i*2+1 <= n && a[maxPos] < a[i*2+1]) maxPos = i*2+1;
    if (maxPos == i) break;
    swap(a, i, maxPos);
    i = maxPos;
  }
}
```

* 由之前的学习所知，一个包含 n 个节点的完全二叉树，树的高度不会超过 `log₂n`。堆化的过程是沿着节点所在路径进行比较交换的，所以堆化的时间复杂度就和树的复杂度成正比，也就是 `O(logn)`.
* **插入数据和删除堆顶元素的主要逻辑就是堆化，所以插入一个元素和删除堆顶元素的时间复杂度都是 `O(logn)`.**

### 如何基于堆实现排序

* 前面的章节提到过很多种算法，有时间复杂度是 `O(n^2)` 的冒泡排序、插入排序、选择排序，也有时间复杂度是 `O(nlogn)` 的归并排序、快速排序和线性排序等。
* 现在我们借助堆这种数据结构实现的算法，就是堆排序。这种排序方法的时间复杂度也比较稳定，就是 `O(nlogn)`. 而且它是原地排序算法。
* 可以把堆排序的过程大致分解成两个大步骤，**建堆和排序**。

#### 建堆

* 首先是把数组原地建成一个堆。原地的意思就是不借助外部数据，直接在原数组上操作。建堆的过程也有两种思路。
* 第一种是借助前面讲的在堆中插入一个元素的思路。尽管现在的数组中包含 n 个数据，但是可以假设起初堆中只包含一个数据就是下标为 1 的数据。再调用插入操作，把下标从 2 到 n 的数据依次插入到堆中。这样就把包含了 n 个数据的数据组织成了堆。
	* **自下而上堆化**
* 第二种实现思路和第一种相反。第一种建堆思路是从前往后处理数组数据，并且每个数据插入堆中时，都是从下往上堆化。第二种实现思路就是从后往前处理数组，并且每个数据是从上往下堆化的。
	* **自上而下堆化**
* 下面是第二种实现思路的过程，建堆分解步骤图。
	* 因为叶子节点往下堆化只能和自己比较，所以我们直接从最后一个非叶子节点开始，依次堆化即可。

![[建堆示意图一.png]]
![[建堆示意图二.png]]

#### 自下而上堆化构建小顶堆

* 第一种建堆过程及应用，这里建堆的是小顶堆，从前往后，堆化沿用上文中的从下到上的方式：

```typescript
/**
 * @description: 建堆
 * 从前往后构建堆
 * @param {number[]} arr
 * @param {number} heapSize
 * @return {void}
 */
function buildHeap(arr: number[], heapSize: number): void {
  while(heapSize < arr.length - 1) {
    heapSize++;
    heapify(arr, heapSize);
  }
}

const items = [undefined, 5, 2, 3, 4, 1];
// 初始有效序列长度为 1
buildHeap(items, 1);
console.log(items);
```

##### 自上而下堆化构建大顶堆

* 下面是第二种实现的 TS 代码，从后往前建堆，从上往下堆化。

```typescript
/**
 * @description: 堆化
 * 按照大顶堆的方式堆化
 * @param {number[]} arr
 * @param {number} heapSize
 * @param {number} i
 * @return {void}
 */
function heapify(arr: number[], heapSize: number, i: number): void {
  let maxIndex = i;

  while (true) {
    // 保存较大值的索引，分别与左右子节点比较
    if (2 * i <= heapSize && arr[maxIndex] < arr[2 * i]) maxIndex = 2 * i;
    if ((2 * i + 1) <= heapSize && arr[maxIndex] < arr[2 * i + 1]) maxIndex = 2 * i + 1;
    if (maxIndex === i) break;
    swap(arr, i, maxIndex);
    i = maxIndex;
  }
}

/**
 * @description: 建堆
 * 这里从后往前建立大顶堆
 * @param {number[]} arr
 * @param {number} heapSize
 * @return {void}
 */
function buildHeap(arr: number[], heapSize: number): void {
  // 从后往前从最后的非叶子节点开始
  for (let i = Math.floor(heapSize / 2); i > 0; i--) {
    heapify(arr, heapSize, i);
  }
}

// 验证构建效果
const items = [undefined, 1, 2, 3, 4, 5];
// 初始有效序列长度为 1
buildHeap(items, items.length - 1);
console.log(items);
```

* 原文中的 Java 代码，是第二种，以供参考。

```java
private static void buildHeap(int[] a, int n) {
  for (int i = n/2; i >= 1; --i) {
    heapify(a, n, i);
  }
}

private static void heapify(int[] a, int n, int i) {
  while (true) {
    int maxPos = i;
    if (i*2 <= n && a[i] < a[i*2]) maxPos = i*2;
    if (i*2+1 <= n && a[maxPos] < a[i*2+1]) maxPos = i*2+1;
    if (maxPos == i) break;
    swap(a, i, maxPos);
    i = maxPos;
  }
}
```

* 从代码中可以发现，只需要对 1 到 `n / 2` 的节点进行堆化。剩下的节点是叶子节点，不需要堆化。
	* 实际上在完全二叉树中，下标 `n / 2 + 1` 到 n 的节点都是叶子节点。

##### 建堆的时间复杂度

* 每个节点堆化的时间复杂度是 `O(logn)`. 但 `n / 2 + 1` 个节点加起来的时间复杂度并不是  `O(nlogn)`，这个结果其实不够精确。
* 实际上，**堆排序建堆过程的时间复杂度是 `O(n)`.**
* 叶子节点不需要堆化，所以需要堆化的节点从完全二叉树的倒数第二层开始。每个节点堆化的过程中，需要比较和交换的节点个数，与这个节点的高度 k 成正比。
* 下面是每一层节点的个数和对应的高度都画了出来，可以观察发现，只需要将每个节点的高度求和，得出的就是建堆的时间复杂度。

![[建堆的节点示意图.png]]

* 把每个非叶子节点的高度求和，就是这个公式:

![[建堆的时间复杂度计算公式.png]]

* 求解这个公式的方式比较取巧，在高中数学有学过：就是把公式左右都乘以 2，得到另一个公式，再把两个公式错位对齐，并且用 S2 减去 S1，可以得到结果 S。

![[建堆的时间复杂度公式求解过程.png]]

* S 的中间部分是一个等比数列，所以可以使用等比数列的求和公式来计算。最终结果如下：

![[建堆的时间复杂度计算最终公式.png]]

* 因为 `h = log₂n`，代入公式 S，就能得到 `S = O(n)`. 因此，建堆的时间复杂度就是 `O(n)`.

#### 排序

* 成功的建堆之后，数组中的数据应当是按照大顶堆的特性来组织的。数组中第一个元素是堆顶，也就是最大的元素，我们把这个元素和最后一个元素交换，也就是把最大的元素放到了末尾下标为 n 的位置。
* 这个过程有些类似上文中的删除堆顶元素的操作，**当堆顶元素移除之后，把下标为 n 的元素放到堆顶，再通过堆化的方法把剩下的 n - 1 个元素重新构建成堆；堆化完成之后，再去堆顶的元素，放到下标为 n - 1 的位置，一直重复这个过程，直到堆中只剩下下标为 1 的一个元素，排序就完成了。**

![[堆排序示意图.png]]

#### 堆排序的实现

* 堆排序的 TS 实现，堆化和建堆复用上文（自上往下式）的方法，但要注意把比较值改成最大值让数组有序且是顺序的。下面是交换和堆排序的实现。

```typescript
function heapify(arr: number[], heapSize: number, i: number): void {
  let maxIndex = i;

  /* A infinite loop. */
  // eslint-disable-next-line no-constant-condition
  while (true) {
    // 下面两个判断找出左右子节点中的较大值，将其与当前节点交换
    if (2 * i <= heapSize && arr[maxIndex] < arr[2 * i]) maxIndex = 2 * i;
    if (2 * i + 1 <= heapSize && arr[maxIndex] < arr[2 * i + 1]) maxIndex = 2 * i + 1;
    // 相等时说明已经堆化到了最后
    if (maxIndex === i) break;
    swap(arr, i, maxIndex);
    i = maxIndex;
  }
}

/**
 * @description: 数组元素的交换逻辑
 * @param {any[]} arr
 * @param {number} i
 * @param {number} j
 * @return {void}
 */
function swap(arr: any[], i: number, j: number): void {
  [arr[j], arr[i]] = [arr[i], arr[j]];
}

/**
 * @description: 堆排序
 * 1. 建堆，从小到大排序，建立大顶堆
 * 2. 从最后节点开始，和根节点交换
 * 3. 交换后对根节点进行堆化
 * 4. 堆化后再次交换到根节点，并继续对根节点堆化
 * @param {number[]} arr
 * @return {number[]}
 */
function heapSort(arr: number[]): number[] {
  let heapSize: number = arr.length - 1;
  buildHeap(arr, heapSize);
  // !从后往前比较，把当前遍历的节点值交换到根节点,把根节点（也就是最大值）放到当前树（子树）的末尾
  for (let index = arr.length - 1; index > 1; index--) {
    // 把当前遍历的节点交换到根节点的位置
    swap(arr, 1, index);
    heapSize--;
    // 对根节点进行堆化
    heapify(arr, heapSize, 1);
  }
  return arr;
}

// 使用堆排序，数组第一个元素为空
const arrs = [undefined, 3, 2, 4, 1, 5, 6];
heapSort(arrs);
console.log(arrs, 'arrs');
```

* 这里的排序默认会忽略数组的第一位元素，因此比较是从索引 1 开始的而不是索引 0；更具体的实现可以参见[代码](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Heap/heap.ts)。
* 原文中堆排序的 Java 实现如下。

```java
// n表示数据的个数，数组a中的数据从下标1到n的位置。
public static void sort(int[] a, int n) {
  buildHeap(a, n);
  int k = n;
  while (k > 1) {
    swap(a, 1, k);
    --k;
    heapify(a, k, 1);
  }
}
```

#### 堆排序的时间复杂度，空间复杂度及稳定性分析

* 堆排序的过程只需要极个别的临时存储空间，所以堆排序是**原地排序算法**。
* 堆排序包括建堆和排序两个操作；
	* 建堆过程的时间复杂度是 `O(n)`；
	* 堆化的时间复杂度是 `O(logn)`;
	* 由上面的代码两层循环可以得到循环过程的时间复杂度是 `O(n) * O(logn)`.
	* 因此排序过程的时间复杂度是 O(nlogn).
* **堆排序的时间复杂度是 `O(nlogn)`.**
* **堆排序不是稳定的排序算法**，因为在排序过程中，存在把堆的最后一个节点和堆顶元素互换的操作，因此就有可能改变值相同数据的原始相对顺序。

##### 堆存储的说明

* 这里要对前面的内容做一个解释，**前文假设堆中的数据是从数组下标为 1 的位置开始存储的。如果从 0 开始存储，实际上处理思路也完全一样。**
	* 如果节点的下标是 i，左子节点的下标就是 `2 * i + 1`，右子节点的下标就是 `2 * i + 2`，父节点的下标就是 `(i - 1) / 2`.
	* 如上所言，**使用下标为 0 的方式开始存储，在每次计算左子节点的时候会需要多一次加法运行**。相对来说是内存是多耗费的。

### 为何实际开发中快速排序的性能会比堆排序更好？

* 作者觉得主要有以下两方面原因造成：
	* 堆排序数据的访问的方式没有快排友好；
	* 同样的数据，排序过程中，堆排序算法的数据交换次数多于快速排序。

#### 堆排序数据的访问的方式没有快速排序友好

* **对于快排来说，数据是顺序访问的。而对于堆排序来说，数据是跳着访问的。**
	* 例如堆排序中最重要的操作之一，数据的堆化。对堆顶点进行堆化，会依次访问下标为 1，2，4，8 的元素，而不是像快速排序一样局部访问。
	* **这样的访问对 CPU 缓存是不友好的。**

![[堆排序下标访问示意图.png]]

#### 同样的数据，排序过程中，堆排序算法的数据交换次数多于快速排序

* 前面的章节提到过有序度和逆序度的概念。**对于基于比较的排序算法来说，整个排序过程就是由两个基本的操作组成的——比较和交换（移动）。**
	* 快速排序数据交换的次数不会比逆序度多。
	* 然而堆排序的第一步是建堆，**建堆的过程会打乱数据原有的相对先后顺序，导致原数据的有序度降低。**
		* 比如，一组已经有序的数据经过建堆之后，*数据反而变得无序了*。

![[堆排序堆化过程示意图.png]]

* 对于这一点，可以使用代码实践。用一个变量来记录交换次数，每次交换，对变量加一，排序完成之后，变量大小就是总的交换次数。比较两者的变量大小来验证堆排序的交换是否比快速排序交换次数更多。
* 下面我们用实际例子来验证。
* 首先，生成一个长度为 20 的随机排列的数组。

```typescript
/**
 * @description: 生成限制最大值和最小值以及长度的随机数的数组
 * @param {number} len
 * @param {number} min
 * @param {number} max
 * @return {number[]}
 */
function randomArray(len: number, min: number, max: number): number[] {
  return Array.from({length: len}, () => Math.floor(Math.random()*(max-min)+min));
}
randomArray(20, 1, 20);
```

* 分别执行堆排序和快速排序。并且在交换的地方记录次数。

```typescript
const arr = [18, 3, 7, 14, 19, 8, 1, 15, 4, 16, 17, 10, 5, 2, 6, 13, 12, 11, 9, 20];
let heapSwapCount = 0;

function swap(arr: any[], i: number, j: number): void {
	heapSwapCount++;
  [arr[j], arr[i]] = [arr[i], arr[j]];
}

/**
 * @description: 堆排序
 * 1. 建堆，从小到大排序，建立大顶堆
 * 2. 从最后节点开始，和根节点交换
 * 3. 交换后对根节点进行堆化
 * 4. 堆化后再次交换到根节点，并继续对根节点堆化
 * @param {number[]} arr
 * @return {number[]}
 */
function heapSort(arr: number[]): number[] {
  buildHeap(arr, arr.length - 1);
  let heapSize: number = arr.length - 1;
  for (let index = arr.length - 1; index > 1; index--) {
    // 把当前遍历的节点交换到根节点的位置
    swap(arr, 1, index);
    heapSize--;
    // 对根节点进行堆化
    heapify(arr, heapSize, 1);
  }
  return arr;
}

// 使用堆排序，数组第一个元素为空
const arrs = [undefined, 3, 2 , 4, 1, 5, 6];
console.log(heapSort([undefined, ...arr].slice()));
console.log(heapSwapCount); // 65
```

* 注意这里的快速排序是使用原地排序的实现方式。

```typescript
const arr = [18, 3, 7, 14, 19, 8, 1, 15, 4, 16, 17, 10, 5, 2, 6, 13, 12, 11, 9, 20];
let quickSwapCount = 0;

function partition(arr: number[], start: number, end: number): number {
  const pivotValue = arr[end]; // 默认从末尾取分支点
  let pivotIndex = start; // 从起始位置取索引

  for (let i = start; i < end; i++) {
    if (arr[i] < pivotValue) {
      quickSwapCount++;
      [arr[i], arr[pivotIndex]] = [arr[pivotIndex], arr[i]];
      pivotIndex++;
    }
  }
  // !把分区的基准值放在中间，使得数组左中右三部分整体上是有序的
  [arr[pivotIndex], arr[end]] = [arr[end], arr[pivotIndex]];

  return pivotIndex;
}

function quickSortRecursive(arr: number[], start = 0, end = arr.length - 1): number[] {
  // 终止条件是索引的叠加
  if (start >= end) return;

  const index = partition(arr, start, end);
  // 注意分割的索引位置，分别在分区点的前后
  quickSortRecursive(arr, start, index - 1);
  quickSortRecursive(arr, index + 1, end);

  return arr;
}

console.log(quickSortRecursive(arr));
console.log(quickSwapCount); // 32
```

* 从这个可能并不算经典的例子对比来看，原地排序的快排的交换次数比堆排序少得明显，分别为 51 次和 73 次。
* 让我们把随机数组的长度拉大到 100，然后直接运行得到结果。

```typescript
const bigArr = [
  54, 13, 70, 48, 19, 29, 81, 27, 12, 58, 28, 37, 48, 71, 34, 87, 79, 91, 65, 60, 72, 77, 34, 54, 75, 79, 87, 30, 66,
  61, 37, 59, 84, 84, 87, 29, 75, 70, 50, 18, 53, 50, 59, 83, 36, 84, 21, 63, 52, 32, 44, 64, 64, 61, 29, 99, 90, 93,
  37, 19, 81, 95, 45, 48, 56, 23, 39, 68, 21, 63, 33, 91, 23, 73, 54, 92, 89, 35, 44, 68, 15, 71, 6, 49, 59, 49, 44, 66,
  72, 7, 20, 34, 6, 2, 68, 70, 78, 58, 28, 85,
];

console.log(heapSort([undefined, ...bigArr].slice()));
console.log(count); // 582

console.log(quickSortRecursive(bigArr));
console.log(quickSwapCount); // 341
```

* 可以看到差距仍然是十分显著的 `582-341`。

## 小结

* 堆是一种完全二叉树。
* 特性：每个节点的值都大于等于或小于等于其子树节点的值，堆因此分成大顶堆和小顶堆。
* 堆中比较重要的操作是插入一个数据和删除堆顶元素。这两个操作都会用到堆化。
	* 插入一个数据时，新插入的数据先放到数组最后，然后从下往上堆化；
	* 删除堆顶数据的时候，把数组中最后一个元素放到堆顶，然后从上往下堆化。
	* 这两个操作的时间复杂度都是 `O(logn)`.
* 堆的经典应用包括堆排序。堆排序包括两个过程，建堆和排序。
	* 把下标 n / 2 到 1 的节点，依次进行从上到下的堆化，然后就可把数组中的数据组织成堆这种数据结构。
	* 再迭代地把堆顶元素和堆的末尾元素进行交换，堆的大小减一，再从上到下堆化，重复此过程直到堆中只剩下一个元素，这时堆排序就已经完成，数组变成有序。

## 扩展

* 为何完全二叉树中，下标 n / 2 到 n 的节点都是叶子节点，这是如何推导的？
	* 💭 可以使用反证法证明。
* 堆的应用除了堆排序还有哪些内容？
	* 详见下一章节 [[Tech/技术笔记/《算法与数据结构》学习笔记/数据结构之堆的应用/index|堆的应用]]。

#ALG #ADT 