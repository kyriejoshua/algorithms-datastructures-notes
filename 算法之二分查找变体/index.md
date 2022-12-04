# 二分查找的变体

> 学习曲线：★★★

## 二分查找变体

* 前面我们所提到的都是值唯一的有序数据。
* 下面的几种变体都是基于存在重复项的有序数据作为条件的。
* 因此我们需要做一些改进来适配当前的具体场景。

![[二分查找的变体.png]]

### 变体一：查找第一个等于给定值的元素

* 核心在于控制判断条件。

```typescript
/**
 * @description: 二分查找第一个等于给定值的索引
 * @param {number[]} arr
 * @param {number} value
 * @return {number}
 */
function binarySearchFirst(arr: number[], value: number): number {
  let low = 0;
  let high = arr.length - 1;
  while (low <= high) {
    // 使用位运算时注意优先级，需要括号包裹
    const mid = ~~(low + ((high - low) >> 1));
    if (arr[mid] < value) {
      low = mid + 1;
    } else if (arr[mid] > value) {
      high = mid - 1;
    } else {
      // !核心在于判断相等之后，还需要确认是否是数组的第一个位置，或者是否是重复的位置
      // 如果是重复的位置，则需要继续查找
      if (mid === 0 || arr[mid - 1] !== value) return mid;
      high = mid - 1;
    }
  }

  return -1;
}

const list = [1, 2, 3, 4, 5, 6, 7, 7, 7, 7, 9];
console.log(binarySearchFirst(list, 7)); // 6
```

### 变体二：查找最后一个等于给定值的元素

* 观察判断条件相较于是上一个实现的变化。

```typescript
/**
 * @description: 二分查找最后一个等于给定值的索引
 * @param {number[]} arr
 * @param {number} value
 * @return {number}
 */
function binarySearchLast(arr: number[], value: number): number {
  let low = 0;
  let high = arr.length - 1;
  while (low <= high) {
    const mid = ~~(low + ((high - low) >> 1));
    if (arr[mid] < value) {
      low = mid + 1;
    } else if (arr[mid] > value) {
      high = mid - 1;
    } else {
      if (mid === arr.length - 1 || arr[mid + 1] !== value) return mid;
      // 寻找最后的数，所以向后
      low = mid + 1;
    }
  }
  return -1;
}

const list = [1, 2, 3, 4, 5, 6, 7, 7, 7, 7, 9];
console.log(binarySearchFirst(list, 7)); // 9
```

### 变体三：查找第一个大于等于给定值的元素

```typescript
/**
 * @description: 二分查找第一个大于等于给定值的索引
 * @param {number[]} arr
 * @param {number} value
 * @return {number}
 */
function binarySearchFirstBigger(arr: number[], value: number): number {
  let low = 0;
  let high = arr.length - 1;
  while (low <= high) {
    // 使用位运算时注意优先级，需要括号包裹
    const mid = ~~(low + ((high - low) >> 1));
    if (arr[mid] >= value) {
			// 核心在于判断相等之后，还需要确认是否是数组的第一个位置，或者是否是重复的位置
      // 如果是重复的位置，则需要继续查找
      if (mid === 0 || arr[mid - 1] < value) return mid;
      high = mid - 1;
    } else ( {
			low = mid + 1;
    }
  }

  return -1;
}

const list = [1, 2, 3, 4, 5, 6, 7, 7, 7, 7, 9];
console.log(binarySearchFirstBigger(list, 7)); // 6
```

#### LeetCode

* 这题 [LeetCode-278](https://leetcode.cn/problems/first-bad-version/) 解决思路是非常类似的。
* 通过二分查找，找到第一个出错的版本。

```typescript
var solution = function(isBadVersion: any) {
  return function(n: number): number {
    let min = 1;
    let max = n;

    while (min <= max) {
      const mid = Math.floor(min + ((max - min) >> 1));

      if (isBadVersion(mid)) {
        if (!isBadVersion(mid - 1)) return mid;
        max = mid - 1;
      } else {
        min = mid + 1;
      }
    }
  };
};
```

### 变体四：查找最后一个小于等于给定值的元素

```typescript
/**
 * @description: 二分查找最后一个等于给定值的索引
 * @param {number[]} arr
 * @param {number} value
 * @return {number}
 */
function binarySearchLastSmaller(arr: number[], value: number): number {
  let low = 0;
  let high = arr.length - 1;
  while (low <= high) {
    const mid = ~~(low + ((high - low) >> 1));
    if (arr[mid] <= value) {
			if (mid === arr.length - 1 || arr[mid + 1] !== value) return mid;
      // 寻找最后的数，所以向后
      low = mid + 1;
    } else {
			high = mid - 1;
    }
  }
  return -1;
}

const list = [1, 2, 3, 4, 5, 6, 7, 7, 7, 7, 9];
console.log(binarySearchFirst(list, 7)); // 9
```

## 二分查找变体的实际应用

### 快速查询 IP 归属地

* **在 IP 地址库中， IP 地址的范围区间和归属地存在一一对应的关系。**
* 查找 IP 归属地的场景实际就是上述第四种变体的应用。
	* 在有序数组中，查找最后一个小于等于某个给定值的元素。

#### 解决思路

* 首先处理 IP 数据，十几万条数据按照起始 IP 从小到大排序。
* 排序的过程：
	* IP 地址可以转化为 32 位的整型数，借助这点我们可以把起始地址按照对应的整型值大小关系进行从小到大的排序。
* 使用二分查找找到最后一个起始 IP 小于等于当前 IP 的 IP 区间。
* 然后检查 IP 是否存在这个区间范围内，
	* 如果存在，就取出对应的归属地显示；
	* 如果不存在，就返回未查找到。

## 小结

* 实际应用场景中，很多时候二分查找会被散列表和二叉树所取代。
* 但是在上述几种变体的场景里，例如大于等于、小于等于给定值，这些场景会比查找等于给定值更为常见，实际是适合使用二分查找的。
	* **二分查找更适合用在查找近似值的场景。**
* 变体的二分查找更容易出错，需要重点关注的几点包括：**终止条件、区间上下界的更新方法、返回值的选择。**


## 扩展

### 在循环有序数组中使用二分查找

* 对应题目： [LeetCode-33](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)
* 这道题至少有两个注意点：
	* 一是解题思路，始终在有序的数组中查找，如果不在当前的有序数组中，就对半分另一侧的数组，使其有序，始终保持这一逻辑；
	* 二是判断值的大小比较，每个边界都要确认清楚，是 **`>=`** (大于等于)还是 **`>`** (大于)，是 **`<=`** (小于等于)还是 **`<`** (小于)，这会影响整个执行逻辑。

```typescript
/**
 * @description: 在循环有序数组中使用二分查找
 * @param {number[]} arr
 * @param {number} value
 * @return {number}
 */
function binarySearchForCircleArray(arr: number[], value: number): number {
  if (!arr.length) return -1;
  if (arr.length === 1) {
    return arr[0] === value ? 0 : -1;
  }
  let low = 0;
  let high = arr.length - 1;

  // !核心在于确认这里的判断条件，对半分之后一定有一半是有序的，始终在有序的这半边里面进行查找
  while (low <= high) {
    const mid = ~~(low + ((high - low) >> 1));
    if (arr[mid] === value) return mid;
    // 对半分之后，左边是有序数组
    if (arr[low] <= arr[mid]) {
      // 左侧有序的查找
      if (arr[low] <= value && value < arr[mid]) {
        high = mid - 1;
      } else {
        low = mid + 1;
      }
    } else { // 对半分之后，右边是有序数组
      if (arr[mid] < value && value <= arr[high]) {
        low = mid + 1;
      } else {
        high = mid - 1;
      }
    }
  }

  return -1;
}
const list = [4,5,6,7,0,1,2], value = 2;
console.log(binarySearchForCircleArray(list, value)); // 6
```

#### 执行过程

* 个人觉得这道题目比较经典，所以我们还是加上几条打印日志，来帮助我们理解这里二分查找一个旋转后的数组里的值的执行过程。
	* 在循环中加入以下几句代码，以及传入下面的数组。

```typescript
console.log(`第 ${idx} 次对半分的数组是 [${arr.slice(low, high)}] 中的左侧 [${arr.slice(low, mid)}]`);
console.log(`第 ${idx} 次对半分的数组是 [${arr.slice(low, high)}] 中的右侧 [${arr.slice(mid, high)}]`);

const list = [4, 5, 6, 7, 8, 9, 0, 1, 2, 3], value = 2;
console.log(binarySearchForCircleArray(list, value)); // 8
```

* 打印出的日志如下，最终结果是索引 8。

```shell
第 1 次二分的索引是 4
第 1 次对半分的数组是 [4,5,6,7,8,9,0,1,2] 中的左侧 [4,5,6,7]
第 2 次二分的索引是 7
第 2 次对半分的数组是 [9,0,1,2] 中的右侧 [1,2]
第 3 次二分的索引是 8
```

#ALG