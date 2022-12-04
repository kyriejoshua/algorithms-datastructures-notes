# 初识动态规划

## 简言

* 淘宝双十一通常会有满 200 减 40 的促销活动。假设购物车中有 n 个(n>100) 想买的商品，我们希望从中选出任意个，在凑够满减条件的前提条件下，让选出来的商品总和最大程度的接近满减条件(200元）。
	* 解决这类问题的核心思想，就是动态规划。
* 关于动态规划，我们会用三个篇章来学习。这节主要通过几个经典案例来了解动态规划，下一节则是学习动态规划的理论，以及最后一节的动态规划实战。
* 这一节是通过案例了解为什么需要动态规划，动态规划的题解是如何演变的。
* 下一节是动态规划适合解决的问题模型和它的解决思路，以及对比贪心，分治，回溯等算法思想。
* 实战的章节是运用动态规划理论知识，来解决三个非常经典的动态规划问题。

> 学习曲线：★★★★☆

## 动态规划

* **动态规划（`Dynamic Programming`）适合用来求解最优问题，可以用来求最大值或最小值。而且它可以显著地降低时间复杂度，提高代码的执行效率。**

## 优化的回溯算法

### 0-1 背包问题

* 对上一节的 0-1 背包我们再次分析，把它代入具体的场景。比如有五个物品 2、2、4、6、3，背包承受的最大重量是 9，把整个求解过程中涉及的值都列出来，可以得到下面这颗二叉树。
* 递归树中的每个节点表示一种状态，我们可以用 `(i, currentWeight)` 来表示。i 表示决策第几个物品是否装入背包，`currentWeight` 表示当前背包中的物品的总重量。

![[01背包二叉树示意图.png]]

* 通过上图的二叉树演示可以看出，随着树的分叉的增长，0-1 背包问题的时间复杂度是==指数级别==的。而且观察节点内容可以发现，其实有很多子问题的求解是重复的，有时候我们已经求出其中的解，就没必要重复计算。于是可以自然地想到把前面子问题的解都先缓存起来，如果后面计算的时候发现有相同的，就直接取出这个解使用。
	* 例如上面的 `f(2, 2)` 和 `f(3, 2)` 都有重复求解的过程。
* 我们使用一个对象数组来保存缓存。这里的物品和重量与上文略有不同，主要是方便演示执行时间。新增缓存后的实现如下：

```typescript
const weightList = [43, 9, 20, 8, 83, 32, 21, 14, 18, 75];
let maxWeight = Number.MIN_SAFE_INTEGER;
// let weightMemory = new Array(10).fill({}); 当一个对象被传递给 fill方法的时候, 填充数组的是这个对象的引用。使用 fill 填充的对象指向同一个
let weightMemory: any[] = new Array(20).fill(false).map(() => ({}));

/**
 * @description: 有缓存的0-1背包实现
 * @param {number} k 当前取的物品个数
 * @param {number} n 总的物品个数
 * @param {number} currentW 当前放置的总重量
 * @param {number} weightList 物品列表
 * @param {number} totalWeight 背包可容纳的总重量
 * @return {void}
 */
function fn0_1withMemory(k: number, n: number, currentW: number, weightList: number[], totalWeight: number): void {
  if (k === n || currentW === totalWeight) {
    if (currentW > maxWeight) maxWeight = currentW;
    return;
  }
  if (weightMemory[k][currentW]) return;
  weightMemory[k][currentW] = true;
  fn0_1withMemory(k + 1, n, currentW, weightList, totalWeight);
  if (currentW + weightList[k] <= totalWeight) {
    fn0_1withMemory(k + 1, n, currentW + weightList[k], weightList, totalWeight);
  }
}
```

* 为了直观地看出和原来方式的执行效率的变化，我们稍稍把物品放大一些，并且打印出具体的执行时间。

```typescript
console.time('fn0_1withMemory');
fn0_1withMemory(0, 20, 0, weightList.concat(weightList), 200);
console.timeEnd('fn0_1withMemory');
```

* 罗列一下原先的实现，方便直接执行，并且也打印出具体的执行时间。

```typescript
let maxW = Number.MIN_SAFE_INTEGER;

/**
 * @description: 简易版本的0-1背包实现
 * @param {number} k 当前取的物品个数
 * @param {number} n 总的物品个数
 * @param {number} currentW 当前放置的总重量
 * @param {number} weightList 物品列表
 * @param {number} totalWeight 背包可容纳的总重量
 * @return {void}
 */
function fn0_1withoutMemory(k: number, n: number, currentW: number, weightList: number[], totalWeight: number): void {
  if (k === n || currentW === totalWeight) {
    if (currentW > maxW) maxW = currentW;
    return;
  }
  fn0_1withoutMemory(k + 1, n, currentW, weightList, totalWeight);
  if (currentW + weightList[k] <= totalWeight) {
    fn0_1withoutMemory(k + 1, n, currentW + weightList[k], weightList, totalWeight);
  }
}
console.time('fn0_1withoutMemory');
fn0_1withoutMemory(0, 20, 0, weightList.concat(weightList), 200);
console.timeEnd('fn0_1withoutMemory');
```

* 这里就不打印放置背包重量的结果，仅输出打印出的执行时间。
	* 且为了避免误差，下面罗列两次执行的结果。

```shell
# 执行一次
fn0_1withMemory: 1.136ms
fn0_1withoutMemory: 4.313ms
# 独立地再执行一次
fn0_1withMemory: 0.868ms
fn0_1withoutMemory: 3.064ms
```

* 可以看到在没有缓存的执行时间是有缓存的执行时间的 3-4 倍，差距是显著的。
* 而且，如果想验证缓存后的效率，那么我们可以在上述代码执行后紧接着再次执行并打印出时间。效果也是非常显著的。结果直接通过注释的形式标注如下。

```typescript
console.time('fn0_1withMemory at second time');
fn0_1withMemory(0, 20, 0, weightList.concat(weightList), 200);
console.timeEnd('fn0_1withMemory at second time'); // fn0_1withMemory at second time: 0.007ms
```

### 0-1 背包问题的衍生

* LeetCode 上还是有比较多的相关问题，它们都不是直接基础的 0-1 背包，但可以通过抽象的形式，转化成 0-1 背包问题来求解。下面这道题官方的题解里只有动态规划的解法，在解这道题时，这个章节本人还没有学完，因此用回溯的方式求解，整个过程其实和上面非常相似。
	* [LeetCode-416](https://leetcode-cn.com/problems/partition-equal-subset-sum/) 分割等和子集。

```typescript
/**
 * @description: 回溯的方式执行效率太低，更推荐使用动态规划来解决。
 * @param {number} nums
 * @return {boolean}
 */
function canPartition(nums: number[]): boolean {
  const sum = nums.reduce((prev, cur) => prev + cur, 0);
  // 总和为奇数的肯定不能分割成功
  if (sum % 2 !== 0) return false;
  const half = sum / 2;
  
  // 标识是否能平等划分
  let hasPartitioned = false;
  const memory = new Array(nums.length).fill(null).map(() => ({}));

  /**
   * @description: 回溯的方式，理论上是可以实现的，但是时间复杂度太高会导致运行不通过
   * @param {number} arr
   * @param {number} index
   * @param {number} currentSum
   * @return {void}
   */
  function backtrack(arr: number[], index: number, currentSum: number) {
    if (index === arr.length) {
      // 判断如果能得到总和为一半的数组的情况，那么就是存在相等和的子数组
      if (currentSum === half) hasPartitioned = true;
      return;
    };
    // !通过缓存来降低时间复杂度
    if (memory[index][currentSum]) return;
    memory[index][currentSum] = true;

    backtrack(arr, index + 1, currentSum);
    if (currentSum + arr[index] <= half) {
      backtrack(arr, index + 1, currentSum + arr[index]);
    }
  }

  backtrack(nums, 0, 0);
  return hasPartitioned;
};
```

* 还有其他 0-1 背包的衍生题目。不再一一详细罗列。下面仅再展示一题，目前的解法也是回溯。
	* [LeetCode-494](https://leetcode-cn.com/problems/target-sum/) 目标和。
* 添加缓存后的 0-1 背包回溯解法，执行效率已经和动态规划基本没有差别。下面我们来看看动态规划是如何解决的。

## 0-1 背包的动态规划解法

* 我们把求解过程分成 n 个节点，每个阶段会决策当前物品是否放到背包中。每个物品决策之后（放入背包或者不放入），背包中的物品重量会有多种情况，也就是会有多种状态，对应到二叉树中，就是对应着不同的节点。
* 我们**把每一层重复的状态（也就是节点）合并，只记录不同的状态，然后基于上一层的状态集合来推导下一层的状态集合。通过合并每一层重复的状态，就可以确保每一层不同状态的个数都不会超过 w 个（w表示背包的承载重量）**，也就是上文第一个例子中的 9。这样的方式可以避免每层状态个数的指数级增长，从而降低时间复杂度。
* 我们使用一个二维数组 `dp[n][w+1]` 来记录每层可以达到的不同状态。n 表示放置物品的个数，w 表示背包的对应重量。
	* 第 0 个（下标从 0开始编号）物品的重量是 2，要么放入背包，要么不放入背包。决策之后会有两种不同的背包状态，背包中物品的总重量是 0 或者 2. 我们用 `dp[0][0] = true` 和 `dp[0][2]` 来标识这两种状态。
	* 第 1 个物品的重量也是 2，基于之前的背包状态，在这个物品决策完之后，不同的状态有 3 个，背包中物品的重量分别可能有 `0(0+0)` 和 `2(0+2 or 2+0)`(*这里最后的总重量是 2，先放和后放没有区别所以是同一种状态*) 和 `4(2+2)` . 我们用 `dp[1][0] = true` 和 `dp[1][2] = true` 和 `dp[1][4] = true` 来表示这三种状态。
	* 以此类推直到考察完所有物品，整个 dp 数组就都计算完成。我们用图这种更直观的形式展示出来，**图中 0 表示 false，1 表示 true**。我们只需要在最后一层（物品都放完的时候），找一个值为 true 的最接近 w(这里是 9，也就是从右下侧开始计算)的值，就是背包中可放置物品总重量的最大值。

![[01背包动态规划示意图1.png]]
![[01背包动态规划示意图2.png]]

* **表格横向表示的是背包可容纳的总重量，竖向表示的是放置的物品个数**。最后一张表格展示了两者间的完整关系。
* 下面是 Java 的完整解法。

```java
// weight:物品重量，n:物品个数，w:背包可承载重量
public int knapsack(int[] weight, int n, int w) {
  boolean[][] states = new boolean[n][w+1]; // 默认值false
  states[0][0] = true;  // 第一行的数据要特殊处理，可以利用哨兵优化
  if (weight[0] <= w) {
    states[0][weight[0]] = true;
  }
  for (int i = 1; i < n; ++i) { // 动态规划状态转移
    for (int j = 0; j <= w; ++j) {// 不把第i个物品放入背包
      if (states[i-1][j] == true) states[i][j] = states[i-1][j];
    }
    for (int j = 0; j <= w-weight[i]; ++j) {//把第i个物品放入背包
      if (states[i-1][j]==true) states[i][j+weight[i]] = true;
    }
  }
  for (int i = w; i >= 0; --i) { // 输出结果
    if (states[n-1][i] == true) return i;
  }
  return 0;
}
```

#### 0-1 背包的动态规划 TS 实现

* 使用二维数组的实现。打印输出可以验证结果。

```typescript
/**
 * @description: 0-1 背包的二维数组的动态规划实现
 * @param {number[]} weightList
 * @param {number} totalWeight
 * @return {number}
 */
function fn0_1ByDP(weightList: number[], totalWeight: number): number {
  // 初始化二维数组
  const dp = new Array(weightList.length).fill(null).map(() => new Array(totalWeight + 1));
  // 初始化首项，注意从 0 开始计数
  dp[0][0] = true;
  // 第一个物品可以放下的场景
  if (weightList[0] <= totalWeight) {
    dp[0][weightList[0]] = true;
  }

  // 遍历所有物品，注意起始位置从 1 开始，前面已经初始化 0 的场景。
  for (let i = 1; i < weightList.length; i++) {
    // 当前物品不放入背包的情况，这里是可以等于的
    for (let j = 0; j <= totalWeight; j++) {
      if (dp[i - 1][j]) {
        dp[i][j] = dp[i - 1][j]; // 也就是 true
      }
    }
    // 当前物品可以放入背包的情况
    for (let j = 0; j <= totalWeight - weightList[i]; j++) {
      if (dp[i - 1][j]) {
        dp[i][j + weightList[i]] = true;
      }
    }
  }

  // 取出最接近背包满载重量的值，因此倒序遍历
  for (let i = totalWeight; i >= 0; i--) {
    if (dp[weightList.length - 1][i]) return i;
  }

  return 0;
}
const weightList: number[] = [43, 9, 20, 8, 83, 32, 21, 14, 18, 75];
console.log(`The max weight of bag is ${fn0_1ByDP(weightList, 100)}`); // The max weight of bag is 100
```

* 查看运行时间，可以确认动态规划的执行时间和使用缓存的回溯算法是近似的，甚至稍好一些。

```typescript
console.time('fn0_1ByDP');
fn0_1ByDP(weightList.concat(weightList), 200);
console.timeEnd('fn0_1ByDP');
// fn0_1ByDP: 0.629ms
```

#### 0-1 背包的动态规划的时间复杂度

* 上节提到，回溯算法在不使用缓存的情况下，时间复杂度是 `O(2^n)`，是指数级的。接下来我们来分析动态规划的时间复杂度。
* 动态规划的时间复杂度分析，主要可以聚焦在 for 循环上，这里有两层 for 循环，所以时间复杂度是 `O(n*w)` 。n 表示物品个数，w 表示背包可以承载的总重量。
* 理论上，`O(n*w)`  的时间复杂度明显比 `O(2^n)` 高效。可以更直观地代入数据来感受两者的差距，假设有 10000 件物品，背包能承载的总重量是 30000. 那么两者的时间复杂度分别是：
	* `Math.pow(2, 3000) > Number.MAX_VALUE` 为 true。 即使减少一位，回溯算法的时间复杂度 2 的 3000 次方也已经比 JS 中能表示的最大的数还要大。用兆作为单位都无法衡量。
	* 而 `10000 * 30000 = 300000000` 动态规划解法的时间复杂度相比之下就小了很多，可以说两者不在一个量级上。

#### 0-1 背包的动态规划的一维数组实现

* TS 版本实现。一维数组本质上是对二维数组实现的简化，通过对二维表格的内容进行分析可以发现，竖向的值其实都是相同的，因为求值的时候是==逻辑短路==的，只要有一个状态可以确保背包能够装下当前物品，则当前状态就是成立的。💭
* 打印输出可以验证结果。

```typescript
/**
 * @description: 0-1 背包问题的一维数组的动态规划实现
 * @param {number[]} weightList
 * @param {number} totalWeight
 * @return {number}
 */
function fn0_1ByBetterDP(weightList: number[], totalWeight: number): number {
  const dp = new Array(totalWeight + 1).fill(false);
  dp[0] = true;
  if (weightList[0] <= totalWeight) {
    dp[weightList[0]] = true;
  }
  for (let i = 1; i < weightList.length; i++) {
    for (let j = totalWeight - weightList[i]; j >= 0; j--) {
      if (dp[j]) dp[j + weightList[i]] = true;
    }
  }

  for (let i = totalWeight; i >= 0; i--) {
    if (dp[i]) return i;
  }

  return 0;
}
const weightList: number[] = [43, 9, 20, 8, 83, 32, 21, 14, 18, 75];
console.log(`The max weight of bag is still ${fn0_1ByBetterDP(weightList, 100)}`);
```

* 优化成一维数组的代码，在执行效率上更好一些。下面的运行时间可以验证。

```typescript
console.time('fn0_1ByBetterDP');
fn0_1ByBetterDP(weightList.concat(weightList), 200);
console.timeEnd('fn0_1ByBetterDP');
// fn0_1ByBetterDP: 0.403ms
```

* 下面是原文中的 Java 实现，以供参考。

```java
public static int knapsack2(int[] items, int n, int w) {
  boolean[] states = new boolean[w+1]; // 默认值false
  states[0] = true;  // 第一行的数据要特殊处理，可以利用哨兵优化
  if (items[0] <= w) {
    states[items[0]] = true;
  }
  for (int i = 1; i < n; ++i) { // 动态规划
    for (int j = w-items[i]; j >= 0; --j) {//把第i个物品放入背包
      if (states[j]==true) states[j+items[i]] = true;
    }
  }
  for (int i = w; i >= 0; --i) { // 输出结果
    if (states[i] == true) return i;
  }
  return 0;
}
```

* 我们可以用动态规划把这道题 [LeetCode-416](https://leetcode-cn.com/problems/partition-equal-subset-sum/) 重新实现一遍。查看提交记录了解具体实现。

### 0-1 背包动态规划视频解析

* 这个视频主要讲解的是使用二维数组去解决 0-1 背包问题，主要是理解其中的解决思路和过程。

<iframe src='https://player.bilibili.com/player.html?bvid=BV1cg411g7Y6&cid=366925396&page=1&share_source=copy_web' scrolling='no' border='0' frameborder='no' framespacing='0' allowfullscreen='true' height="520" width="100%"></iframe>

## 0-1 背包的升级版（结合价值）

* 类似上节回溯算法的结尾，我们在扩展里提出如果加入物品的价值，如何来计算求出背包所能装下的物品的最大价值。
	* 这个问题在上节已经使用回溯解法实现。点击[[Tech/技术笔记/《算法与数据结构》学习笔记/算法之回溯算法/index#0-1 背包计算价值的题解|回溯算法计算背包问题的价值]]查看。
* 针对回溯算法的实现代码，我们画出递归树。递归树中每个节点表示一个状态。相比上面的递归树，现在我们需要增加一个变量(`index`, `currentWeight`, `currentValue`)来表示一个状态。
	* `index` 表示决策第 `index` 个物品是否装入背包；
	* `currentWeight` 表示当前背包中物品的总重量；
	* `currentValue` 表示当前背包中物品的总价值。

![[01背包价值二叉树示意图.png]]

* 在上面的递归树中里，有几个节点的 `index` 和 `currentWeight` 是完全相同的，比如 `f(2, 2, 4)` 和 `f(2, 2, 3)`. 在背包中物品总重量一样的情况下，`f(2, 2, 4)`这种状态对应的物品总价值更大，于是我们可以放弃 `f(2, 2, 3)` 这种状态，只需要沿着 `f(2, 2 , 4)` 这条分支(决策)路线继续往下决策就可以。
* 我们还是可以把整个求解过程分成 n 个阶段，每个阶段会决策当前物品是否放到背包中。每个阶段的决策结束之后，背包中的物品的总重量以及总价值，会有多种情况，也就是会有多种不同的状态。
* 我们还是使用一个二维数组 `dp[n][w+1]`, 记录每层可以达到的不同状态。但这里数组存储的就不再是 `boolean` 类型，而是当前状态对应的最大总价值。
	* 我们把每一层中 `(index, currentWeight)` 重复的状态（节点）合并，只记录 `currentValue` 值最大的那个状态，然后基于这个状态来推导下一层的状态。
	* 下面是 Java 的实现。

```java
public static int knapsack3(int[] weight, int[] value, int n, int w) {
  int[][] states = new int[n][w+1];
  for (int i = 0; i < n; ++i) { // 初始化states
    for (int j = 0; j < w+1; ++j) {
      states[i][j] = -1;
    }
  }
  states[0][0] = 0;
  if (weight[0] <= w) {
    states[0][weight[0]] = value[0];
  }
  for (int i = 1; i < n; ++i) { //动态规划，状态转移
    for (int j = 0; j <= w; ++j) { // 不选择第i个物品
      if (states[i-1][j] >= 0) states[i][j] = states[i-1][j];
    }
    for (int j = 0; j <= w-weight[i]; ++j) { // 选择第i个物品
      if (states[i-1][j] >= 0) {
        int v = states[i-1][j] + value[i];
        if (v > states[i][j+weight[i]]) {
          states[i][j+weight[i]] = v;
        }
      }
    }
  }
  // 找出最大值
  int maxvalue = -1;
  for (int j = 0; j <= w; ++j) {
    if (states[n-1][j] > maxvalue) maxvalue = states[n-1][j];
  }
  return maxvalue;
}
```

#### 0-1 背包升级版的 TS 实现

* 详见注释和打印输出。

```typescript
/**
 * @description: 计算背包最大价值的实现
 * 时间复杂度 O(n*w)
 * 空间复杂度 O(n*w)
 * @param {number[]} weightList
 * @param {number[]} valueList
 * @param {number} totalWeight
 * @return {number}
 */
function getMaxValueByDP(weightList: number[], valueList: number[], totalWeight: number): number {
  // 初始化二维数组，默认价值为 -1，方便后续计算比较
  const dp = new Array(weightList.length).fill(null).map(() => (new Array(totalWeight + 1).fill(-1)));
  // 初始化首个节点值
  dp[0][0] = 0;
  // 放入首个商品的价值
  if (weightList[0] <= totalWeight) {
    dp[0][weightList[0]] = valueList[0];
  }

  // 遍历所有物品
  for (let i = 1; i < weightList.length; i++) {
    // 当前商品不放入的情况
    for (let j = 0; j <= totalWeight; j++) {
      // 价值之前的情况一样
      if (dp[i - 1][j] >= 0) dp[i][j] = dp[i - 1][j];
    }

    // 当前商品放入购物车的情况
    for (let j = 0; j <= totalWeight - weightList[i]; j++) {
      if (dp[i - 1][j] >= 0) {
        const value = dp[i-1][j] + valueList[i];
        // 可以放入当前物品的情况下，需要比较放入后的价值和不放的价值，如果超过就更新价值
        if (value > dp[i][j + weightList[i]]) {
          dp[i][j+weightList[i]] = value;
        }
      }
    }
  }

  let maxValue = Number.MIN_SAFE_INTEGER;

  // 遍历一遍取出最大的值，因为价值是不固定的，所以遍历顺序前后没有关系
  for (let j = 0; j <= totalWeight; j++) {
    // 取出最后一个商品放完的组合
    if (dp[weightList.length - 1][j] > maxValue) maxValue = dp[weightList.length - 1][j];
  }

  return maxValue;
}
const weightList = [43, 9, 20, 8, 83, 32, 21, 14, 18, 75];
const valueList = [4, 5, 1, 4, 4, 3, 2, 5, 4, 1]; // weightList.map((num) => num % 5 + 1
console.log(getMaxValueByDP(weightList, valueList, 100)); // max value is 22
```

* 下面是一维数组的实现。

```typescript
/**
 * @description: 计算背包最大价值的一维数组实现
 * @param {number[]} weightList
 * @param {number[]} valueList
 * @param {number} totalWeight
 * @return {number}
 */
function getMaxValue(weightList: number[], valueList: number[], totalWeight: number): number {
  const dp: number[] = new Array(totalWeight + 1).fill(-1);
  dp[0] = 0;
  if (weightList[0] <= totalWeight) {
    dp[weightList[0]] = valueList[0];
  }

  for (let i = 0; i < weightList.length; i++) {
    for (let j = totalWeight - weightList[i]; j >= 0; j--) {
      if (dp[j] >= 0) {
        dp[j + weightList[i]] = dp[j] +valueList[i] > dp[j + weightList[i]] ? dp[j] + valueList[i] : dp[j + weightList[i]]
      };
    }
  }

  // let maxValue = Number.MIN_SAFE_INTEGER;
  // for (let i = 0; i <= totalWeight; i++) {
  //   if (dp[i] > maxValue) maxValue = dp[i];
  // }
  // 直接使用 API 获得最大值
  const maxValue = Math.max(...dp);

  return maxValue;
}

const weightList = [43, 9, 20, 8, 83, 32, 21, 14, 18, 75];
const valueList = [4, 5, 1, 4, 4, 3, 2, 5, 4, 1]; // weightList.map((num) => num % 5 + 1
console.log(getMaxValue(weightList, valueList, 100)); // 22
```

#### 0-1 背包升级版的时间复杂度

* 这里时间复杂度和空间复杂度的分析，和上面的例子几乎是一样的。很快就可以得出结论：
	* 时间复杂度 `O(n*w)`
	* 空间复杂度 `O(n*w)`

### 购物车满减凑单问题

* 购物车凑满减的问题，本质就是 0-1 背包的问题，只不过把重量的概念转变成价格而已。
* 购物车中有 n 个商品，每个商品都要决策是否购买，每次决策之后，会有对应的不同的状态集合。我们还是用二维数组 `dp[n][x]` 来存储所有的状态。唯一不同的是，这里的 x 对应的是什么？
* 0-1 背包的问题中，我们寻找的是小于等于 w 的最大值。x 就是背包最大承载重量 w + 1. 而对于购物车问题，我们要找到的是大于等于 w 的最小值。例如大于等于 200（凑满减的金额）中最小的值，所以不能设置成 200 加 1.
	* 而如果凑满减金额如果太大，也就会失去实际意义。于是我们可以设置一个较为合理的上限，这样就形成一个比较合适的区间。这个上限可以是 1000，也可以是 200 的三倍。
	* 假设 x 为满减金额的三倍，对应到实际场景就是 601.
* 在购物车问题中，我们不仅要找到满足条件的最小金额，还需要找到这个最小金额所对应的要购买的商品。我们还是使用 dp 数组，倒推出选中的商品序列。
	* 结合下面的代码和注释来一起理解这个过程。

```java
// items商品价格，n商品个数, w表示满减条件，比如200
public static void double11advance(int[] items, int n, int w) {
  boolean[][] states = new boolean[n][3*w+1];//超过3倍就没有薅羊毛的价值了
  states[0][0] = true;  // 第一行的数据要特殊处理
  if (items[0] <= 3*w) {
    states[0][items[0]] = true;
  }
  for (int i = 1; i < n; ++i) { // 动态规划
    for (int j = 0; j <= 3*w; ++j) {// 不购买第i个商品
      if (states[i-1][j] == true) states[i][j] = states[i-1][j];
    }
    for (int j = 0; j <= 3*w-items[i]; ++j) {//购买第i个商品
      if (states[i-1][j]==true) states[i][j+items[i]] = true;
    }
  }

  int j;
  for (j = w; j < 3*w+1; ++j) { 
    if (states[n-1][j] == true) break; // 输出结果大于等于w的最小值
  }
  if (j == 3*w+1) return; // 没有可行解
  for (int i = n-1; i >= 1; --i) { // i表示二维数组中的行，j表示列
    if(j-items[i] >= 0 && states[i-1][j-items[i]] == true) {
      System.out.print(items[i] + " "); // 购买这个商品
      j = j - items[i];
    } // else 没有购买这个商品，j不变。
  }
  if (j != 0) System.out.print(items[0]);
}
```

* 和常规 0-1 背包的主要差别在于打印商品序列。
* 状态 `(i, j)` 只可能从 `(i-1, j)` 或者 `(i - 1, j - value[i]) `两个状态推导过来。因此，我们检查这两个状态是否是可达的，也就是 `dp[i-1][j]` 和  `dp[i-1, j - value[i]]` 的值是否是 true.
	* 如果 `dp[i-1][j]` 可达，说明没有购买第 i 个商品，如果  `dp[i-1, j - value[i]]` 可达，说明选择购买第 i 个商品。我们从中选择一个可达的状态（如果两个都可达，就任意选择其中找一个），然后继续迭代地考察其他商品是否有选择购买。这样就能遍历出所有的购买结果。

### 购物车凑满减的 TS 实现

* 二维数组实现。在原文的基础上加以改在，用乘积处理以支持价格是两位小数的场景，可以适配任何价格。

```typescript
/**
 * @description: 购物车凑单满减算法，动态规划实现
 * @param {number[]} prices
 * @param {number} totalPrice
 * @return {number[]}
 */
function getBabyShoppingCart(prices: number[], totalPrice: number): number[]|false {
  // 转化金额，把小数转为整数，兼容金额存在小数的情况
  const priceList: number[] = prices.map((p) => p * 100);
  const length = priceList.length;
  const maxPrice = totalPrice * 100 * 3;
  // 初始化二维数组
  const cartDP = new Array(priceList.length).fill(null).map(() => new Array(maxPrice + 1).fill(false));
  // 初始化首项
  cartDP[0][0] = true;
  // 首个物品可以放下的场景
  if (priceList[0] <= maxPrice) {
    cartDP[0][priceList[0]] = true;
  }

  // 遍历并保存所有状态
  for (let i = 1; i < length; i++) {
    for (let j = 0; j <= maxPrice; j++) {
      if (cartDP[i - 1][j]) cartDP[i][j] = true;
    }
    for (let j = 0; j <= maxPrice - priceList[i]; j++) {
      if (cartDP[i - 1][j]) cartDP[i][j + priceList[i]] = true;
    }
  }

  // 用于标识满足满减的最小金额
  let calculatedPrice: number;
  // 从恰好等于满减金额开始
  for (calculatedPrice = totalPrice * 100; calculatedPrice <= maxPrice; calculatedPrice++) {
    if (cartDP[length - 1][calculatedPrice]) break;
  }

  // 如果凑到三倍满减金额，已经失去满减意义
  if (calculatedPrice === maxPrice + 1) return false;
  console.log('max price is', calculatedPrice / 100);
  
  // 保存最终加入购物车的商品
  const cart: number[] = [];
  // 从后往前，遍历得到保存过对应状态的商品金额
  for (let i = length - 1; i >= 1; i--) {
    if (calculatedPrice - priceList[i] >= 0 && cartDP[i - 1][calculatedPrice - priceList[i]]) {
      cart.push(prices[i]);
      calculatedPrice -= priceList[i];
    }
  }
  if (calculatedPrice !== 0) cart.push(prices[0]);
  return cart;
}
const priceList = [43.1, 9.9, 20.3, 8.6, 83.2, 32.4, 21.6, 14.5, 18.2, 75.8];
console.log(new Date().toLocaleString(), '计算的结果 =>', getBabyShoppingCart(priceList, 100));
// max price is 100.1
// 4/13/2022, 10:47:24 PM 计算的结果 => [ 18.2, 8.6, 20.3, 9.9, 43.1 ]
// 100.1 = 18.2 + 8.6 + 20.3 + 9.9 + 43.1
```

* 一维数组实现，尝试后发现有问题。暂无方案。还需要更深的理解来思考。💭

## 小结

* 我们在这节主要通过两个经典案例来了解动态规划的解题思路。深入理解这两个问题，动态规划就可以说是完全入门的。
* 从这两个例子也可以看出，大多数动态规划可以解决的问题，实际上通过回溯算法也可以解决，只是回溯算法的执行效率比较低，时间复杂度是指数级的。动态规划在效率上是提高很多的，但是其实也是空间换时间的思路，空间复杂度相对会消耗更多。
* 像动态规划这种算法的理论知识，可以理解为是“后验性”的。在解决问题的过程中，通常首先是想到如何使用某个算法思想来解决问题，然后再运用算法理论知识去验证这个算法思想解决问题的正确性。

### 动态规划入门题

* 列举两道简单的示例题。
	* [LeetCode-70](https://leetcode-cn.com/problems/climbing-stairs/) 爬楼梯
	* [LeetCode-509](https://leetcode-cn.com/problems/fibonacci-number/) 斐波那契数列
* 两题的解法基本是一样的，罗列后者的题解如下。

```typescript
function fib(n: number): number {
  if (n < 2) return n;
  const dp: number[] = new Array(n);
  // 定义初始状态
  dp[0] = 0;
  dp[1] = 1;

  for (let i = 2; i <= n; i++) {
    // 应用递推公式
    dp[i] = dp[i - 1] + dp[i - 2];
  }

  return dp[n];
};
```

## 扩展
* 对杨辉三角进行改造，每个位置的数字可以随意填写，经过某个数字只能到达下面一层相邻的两个数字。
	* *假设站在第一层，往下移动，我们把移动到最底层所经过的所有数字之和，定义为路径的长度。求从最高层移动到最底层的最短路径长度。*
	* 对应 [LeetCode-120](https://leetcode-cn.com/problems/triangle/) 三角形最小路径和，解决思路和方案具体在下一节分析。💭

![[杨辉三角改造题.png]]

#ALG 