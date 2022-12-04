# 动态规划实践

## 简言

* 在 Trie 树的章节，我们可以使用这种数据结构实现搜索引擎的关键字提示功能，节省用户输入搜索关键词的时间。但还有一种场景，在优化搜索体验时也经常遇到，就是*拼写纠错功能*。
* 拼写纠错功能主要在体现在搜索框或代码编辑器中，智能地检测出拼写错误，然后用正确的单词来进行搜索。
* 那么这个功能是如何实现的呢？下面来一探究竟。

> 学习曲线：★★★★

## 如何量化两个字符串的相似度

### 编辑距离

* 有一个非常有名的**量化方法用来衡量两个字符串间的相似程度，叫作编辑距离（`Edit Distance`).**
* **编辑距离指的是，把一个字符串转化成另一个字符串，需要的最少编辑操作次数（比如增加一个字符、删除一个字符、替换一个字符）。**
	* **编辑距离越大，说明两个字符串的相似程度越小；**
	* **编辑距离越小，说明两个字符串的相似程度越大；**
	* **两个完全相同的字符串，编辑距离就是 0.**
* 根据编辑操作种类的不同，编辑距离也有多种不同的计算方式，比较著名的有**莱文斯坦距离（`Levenshtein distance`）** 和**最长公共子串长度（`Longest common substring length`）**。
	* **莱文斯坦距离允许增加、删除、替换字符这三个编辑操作；**
	* **最长公共子串值允许增加、删除字符这两个编辑操作。**
* 莱文斯坦距离和最长公共子串长度是从两个完全相反的角度来分析字符串的相似程度的。
	* **莱文斯坦距离的大小，表示两个字符串差异的大小；**
	* **最长公共子串的大小，表示两个字符串相似程度的大小。**
* 这两个子串的计算方法，可以观察下面的图来帮助理解。其中，两个字符串 `mitcmu` 和 `mtacnu` 的莱文斯坦距离是 3， 最长公共子串的长度是 4. 下面我们来详细计算和验证。

![[莱文斯坦距离和最长公共子串.png]]

### 如何计算得到莱文斯坦距离

* 这个问题的本质是把一个字符串变成另一个字符串所需要的最少编辑次数。整个求解过程，涉及多个决策阶段，需要依次检查一个字符串中的每个字符，和另一个字符串中的每个字符是否匹配，包括匹配如何处理以及不匹配又如何处理。因此，这个问题是符合**多阶段决策最优解模型**的。
* 在尝试使用动态规划来解决之前，我们可以先试着用回溯算法来解决。

#### 回溯算法计算莱文斯坦距离

* 回溯是递归处理的过程， 我们先梳理所有的可能：
	* 如果 `a[i]` 和 `b[j]` 匹配，就继续递归考察 `a[i + 1]` 和 `b[j + 1]`;
	* 如果 `a[i]` 和 `b[j]` 不匹配，就会有多种处理方式可选：
		* 可删除 `a[i]`, 然后递归检查 `a[i + 1]` 和 `b[j]`;
		* 可删除 `b[j]`, 然后递归检查 `a[i]` 和 `b[j + 1]`;
		* 可在 `a[i]` 前面添加一个和 `b[j]` 相同的字符，然后递归检查 `a[i]` 和 `b[j + 1]`;
		* 可在 `b[j]` 前面添加一个和 `a[i]` 相同的字符，然后递归检查 `b[j]` 和 `a[i + 1]`;
		* 可以把 `a[i]` 替换成 `b[j]`,或者把 `b[j]` 替换成 `a[i]` , 然后递归检查 `a[i + 1]` 和 `b[j + 1]`.
* 把这段处理思路翻译成代码就是下面这样。

```java
private char[] a = "mitcmu".toCharArray();
private char[] b = "mtacnu".toCharArray();
private int n = 6;
private int m = 6;
private int minDist = Integer.MAX_VALUE; // 存储结果
// 调用方式 lwstBT(0, 0, 0);
public lwstBT(int i, int j, int edist) {
  if (i == n || j == m) {
    if (i < n) edist += (n - i);
    if (j < m) edist += (m - j);
    if (edist < minDist) minDist = edist;
    return;
  }
  if (a[i] == b[j]) { // 两个字符匹配
    lwstBT(i+1, j+1, edist);
  } else { // 两个字符不匹配
    lwstBT(i + 1, j, edist + 1); // 删除a[i]或者b[j]前添加一个字符
    lwstBT(i, j + 1, edist + 1); // 删除b[j]或者a[i]前添加一个字符
    lwstBT(i + 1, j + 1, edist + 1); // 将a[i]和b[j]替换为相同字符
  }
}
```

#### 动态规划计算莱文斯坦距离

* 根据回溯算法的实现，可以画出对应的递归树，观察是否存在重复子问题。
	* 如果存在重复子问题，就可以考虑使用动态规划来解决；
	* 如果不存在重复子问题，回溯就是最好的实现方案。

![[莱文斯坦计算过程递归树.png]]

* 在递归树中，**每个节点代表一个状态，状态包含三个变量 `(i, j, edist)`. 其中，`edist` 表示处理到 `a[i]` 和 `b[j]` 时，已经执行的编辑操作的次数。**
* 在递归树里，`(i, j)` 两个变量重复的节点很多，比如 `(3, 2)` 和 `(2, 3)`. 对于 `(i, j)` 相同的节点，只需要保留 `edist` 最小的，接着继续处理就可以，其他节点可以舍弃（我们只关心实际的最少编辑树）。
	* 状态 `(i, j, edist)` 就可以变成 `(i, j, min_edist)`, **其中 `min_edist` 表示处理到 `a[i]` 和 `b[j]` 时已经执行的最少编辑次数。**
* 这个问题其实和前两节的问题是非常相似的，只是处理上会多一点难度，状态转移方式相对更复杂一些。
* 上一节的最短矩阵路径问题，到达状态 `(i, j)` 只可能会从 `(i, j - 1)` 或 `(i - 1, j)` 这两者中的一个转移过来。而这个问题，状态 `(i, j)` 可能会从 `(i, j - 1)` 或 `(i - 1, j)` 或 `(i - 1, j - 1)` 这三者中的任意一个转移过来。

![[莱文斯坦距离的状态转移过程.png]]

* 基于前面的分析和上图，我们可以用公式表示出状态转移的过程，也就是**状态转移方程**。
* 用伪代码表示。

```shell
如果：a[i]!=b[j]，那么：min_edist(i, j)就等于：
min(min_edist(i-1,j)+1, min_edist(i,j-1)+1, min_edist(i-1,j-1)+1)

如果：a[i]==b[j]，那么：min_edist(i, j)就等于：
min(min_edist(i-1,j)+1, min_edist(i,j-1)+1，min_edist(i-1,j-1))

其中，min表示求三数中的最小值。     
```

* 了解状态间的递推关系后，可以画出相应的二维状态表，按行依次来填充状态表中的每个值。

![[莱文斯坦距离计算的填表过程.png]]

* 根据填表过程和状态转移方程，就可以写出实现代码。

```java
public int lwstDP(char[] a, int n, char[] b, int m) {
  int[][] minDist = new int[n][m];
  for (int j = 0; j < m; ++j) { // 初始化第0行:a[0..0]与b[0..j]的编辑距离
    if (a[0] == b[j]) minDist[0][j] = j;
    else if (j != 0) minDist[0][j] = minDist[0][j-1]+1;
    else minDist[0][j] = 1;
  }
  for (int i = 0; i < n; ++i) { // 初始化第0列:a[0..i]与b[0..0]的编辑距离
    if (a[i] == b[0]) minDist[i][0] = i;
    else if (i != 0) minDist[i][0] = minDist[i-1][0]+1;
    else minDist[i][0] = 1;
  }
  for (int i = 1; i < n; ++i) { // 按行填表
    for (int j = 1; j < m; ++j) {
      if (a[i] == b[j]) minDist[i][j] = min(
          minDist[i-1][j]+1, minDist[i][j-1]+1, minDist[i-1][j-1]);
      else minDist[i][j] = min(
          minDist[i-1][j]+1, minDist[i][j-1]+1, minDist[i-1][j-1]+1);
    }
  }
  return minDist[n-1][m-1];
}

private int min(int x, int y, int z) {
  int minv = Integer.MAX_VALUE;
  if (x < minv) minv = x;
  if (y < minv) minv = y;
  if (z < minv) minv = z;
  return minv;
}
```

#### 莱文斯坦距离的 TS 实现

* 我们用前端熟悉的语言再来自己实现一遍。

##### 回溯实现

* 首先是 TS 的回溯实现，包含验证内容也就是打印日志。
* 连续调用后，终端里的执行时间在 3s 左右。

```typescript
/**
 * @description: 回溯的方式计算莱文斯坦距离
 * 时间复杂度指数级别，仅可计算两个长度很小的字符串的距离
 * @param {string} word1
 * @param {string} word2
 * @return {number}
 */
 function lwstBT(word1: string, word2: string): number {
  //  保存各自字符串的长度用于后续比较
  const aLen = word1.length;
  const bLen = word2.length;
  let minEdist = Number.MAX_SAFE_INTEGER;

  /**
   * @description: 回溯比较两个字符串
   * @param {number} i 第一个字符串的比较位置
   * @param {number} j 第二个字符串的比较位置
   * @param {number} edist 最小编辑距离
   * @return {void}
   */
  function getMinDist(i: number, j: number, edist: number): void {
    // 递归到两个字符串其中一个字符串的结尾,就保存最终的结果并返回
    if (i === aLen || j === bLen) {
      // 保存两个中剩下的字符串的差异，并更新保存编辑距离
      if (i < aLen) edist += (aLen - i);
      if (j < bLen) edist += (bLen - j);
      if (edist < minEdist) minEdist = edist;
      return;
    }

    // 如果当前的两个字符相等，就继续考察下一个
    if (word1[i] === word2[j]) {
      getMinDist(i + 1, j + 1, edist);
    } else { // 回溯的方式，考察三种情况，注意这里并不修改 edist 自身，而是每次传递的时候加 1 处理
      // 删除 word1[i] 或者 word2[j] 前添加一个字符
      getMinDist(i + 1, j, edist + 1);
      // 删除 word2[j] 或者 word1[i] 前添加一个字符
      getMinDist(i, j + 1, edist + 1);
      // 把 word1[i] 和 word2[j] 替换为相同字符
      getMinDist(i + 1, j + 1, edist + 1);
    }
  }

  getMinDist(0, 0, 0);
  return minEdist;
}

console.log(lwstBT('abcde', 'abcdd')); // 1
console.log(lwstBT('intention', 'execution')); // 5
console.log(lwstBT('zhuzhengyuan', 'qiuxiao')); // 10
console.log(lwstBT('kyriejoshua', 'zhuzhengyuan')); // 10
```

##### 动态规划实现

* 下面是动态规划的实现，包含验证内容也就是打印日志。
* 执行时间约在 1s，效率提升明显。

```typescript
/**
 * @description: 动态规划的方式计算莱文斯坦距离
 * @param {string} a
 * @param {string} b
 * @return {number}
 */
function lwstByDP(a: string, b: string): number {
  const aLen = a.length;
  const bLen = b.length;
  // 如果存在其中一个字符串长度为 0 的情况，直接返回长度
  if (aLen * bLen === 0) {
    return aLen + bLen;
  } 

  // 生成 dp 数组
  const dp: number[][] = new Array(aLen).fill(null).map((): number[] => new Array(bLen));

  // !初始化状态，这里直接进行比较求出当前索引对应的状态值
  for (let j = 0; j < bLen; j++) {
    // 如果当前字符和第一个字符串的首字符相等，那么状态值就是当前字符串的索引值
    if (a[0] === b[j]) dp[0][j] = j;
    // 如果索引不为 0，而且字符也不相等，就从前一个状态推导得到当前值
    else if (j !== 0) dp[0][j] = dp[0][j - 1] + 1;
    // 第一个字符不相等，就为 1；能走到这个判断条件的，实际上此时 j 为 0
    else dp[0][j] = 1;
  }

  for (let i = 0; i < aLen; i++) {
    if (b[0] === a[i]) dp[i][0] = i;
    else if (i !== 0) dp[i][0] = dp[i - 1][0] + 1;
    else dp[i][0] = 1;
  }

  for (let i = 1; i < aLen; i++) {
    for (let j = 1; j < bLen; j++) {
      // !根据状态推导，如果当前字符相等，则可以从上一个状态推导出来;如果不等，就从推导的状态里继续加 1
      // 其实就是 dp[i][j] = dp[i - 1][j - 1]
      if (a[i] === b[j]) {
        dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1]);
      } else {
        dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1] + 1);
      }
    }
  }

  /**
   * @description: 获取三个值中最小的值
   * @param {number} x
   * @param {number} y
   * @param {number} z
   * @return {number}
   */
  function min(x: number, y: number, z: number): number {
    let minDist = Number.MAX_SAFE_INTEGER;
    if (x < minDist) minDist = x;
    if (y < minDist) minDist = y;
    if (z < minDist) minDist = z;
    return minDist;
  }
  printMatrix(a, b, dp);
  // !dp[aLen - 1][bLen - 1] 表示 a[0...aLen - 1] 字符和 b[0...bLen - 1] 字符的莱文斯坦距离
  return dp[aLen - 1][bLen - 1];
}

console.log(lwstByDP('mitcmu', 'mtacnu')); // 3
console.log(lwstByDP('intention', 'execution')); // 5
console.log(lwstByDP('zhuzhengyuan', 'qiuxiao')); // 10
```

* 为了直观展示表的内容，也把 dp 数组的内容打印出来，方便理解。
	* 下面是打印方法和打印出的内容。

```typescript
/**
 * @description: 打印出矩阵
 * 直观展示两个字符串比较并填表后的所有状态
 * @param {string} text1
 * @param {string} text2
 * @param {number[][]} dp
 * @return {void}
 */
function printMatrix(text1: string, text2: string, dp: number[][]): void {
  console.log(' ',...text2);
  let i = 0;
  for (const arr of dp) {
    console.log(text1[i++], arr.toString());
  }
}
```

* 打印出的结果，以文中的两个字符串为例。

```shell
  m t a c n u
m 0,1,2,3,4,5
i 1,1,2,3,4,5
t 2,1,2,3,4,5
c 3,2,2,2,3,4
m 4,3,3,3,3,4
u 5,4,4,4,4,3
```

* 可以看到，它和上文中的填表内容是完全一样的。
* 下面是另一种动态规划的实现，推导状态的过程略有不同，整体思路大同小异。

```typescript
/**
 * @description: 动态规划计算莱文斯坦距离
 * 这个方式的推导和上面的本质是一样的，只是初始化方便一些，全部从前面状态推导求得
 * @param {string} word1
 * @param {string} word2
 * @return {number}
 */
function getLwstDistance(word1: string, word2: string): number {
  const aLen = word1.length;
  const bLen = word2.length;

  if (aLen * bLen === 0) return aLen + bLen;

  function min(x: number, y: number, z: number): number {
    let m = Number.MAX_SAFE_INTEGER;
    if (x < m) m = x;
    if (y < m) m = y;
    if (z < m) m = z;
    return m;
  }

  const distanceMatrix = new Array(aLen + 1).fill(null).map(() => new Array(bLen + 1));
  // 各自初始化行列
  for (let i = 0; i <= aLen; i++) {
    distanceMatrix[i][0] = i;
  }
  for (let j = 0; j <= bLen; j++) {
    distanceMatrix[0][j] = j;
  }

  for (let i = 1; i <= aLen; i++) {
    for (let j = 1; j <= bLen; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        distanceMatrix[i][j] = min(distanceMatrix[i][j - 1] + 1, distanceMatrix[i - 1][j] + 1, distanceMatrix[i - 1][j - 1]);
      } else {
        distanceMatrix[i][j] = min(distanceMatrix[i][j - 1] + 1, distanceMatrix[i - 1][j] + 1, distanceMatrix[i - 1][j - 1] + 1);
      }
    }
  }
  printMatrix(word1, word2, distanceMatrix);

  return distanceMatrix[aLen][bLen];
}

console.log(getLwstDistance('mitcmu', 'mtacnu')); // 3
console.log(getLwstDistance('zhuzhengyuan', 'qiuxiao')); // 10
console.log(getLwstDistance('zhuzhengyuan', 'kyriejoshua')); // 10
```

* 同样的，打印出 dp 数组内容。

```typescript
function printMatrix(text1: string, text2: string, dp: number[][]): void {
  console.log('  /', ...text2);
  let i = -1;
  for (const arr of dp) {
    console.log(text1[i++] || '/',arr.toString());
  }
}
```

* 可以发现和上一种实现的区别，主要在于初始化的部分。

```shell
  / m t a c n u
/ 0,1,2,3,4,5,6
m 1,0,1,2,3,4,5
i 2,1,1,2,3,4,5
t 3,2,1,2,3,4,5
c 4,3,2,2,2,3,4
m 5,4,3,3,3,3,4
u 6,5,4,4,4,4,3
```

#### 莱文斯坦距离 LeetCode

* 这道题也有对应的算法题目。
* [LeetCode-72](https://leetcode.cn/problems/edit-distance/): 编辑距离
	* 这里使用回溯实现是无法通过的，执行时间会超时，而使用动态规划的实现可以通过。

#### 解题思路方法论

* 在求解问题的时候，可以先不思考计算机会如何实现该问题。而是从人的角度触发去思考解决这个问题，人脑的思考通常是具象化的，因此可以实例化几个测试数据，通过分析具体实例的解，然后总结规律，再尝试套用算法，看看是否能够解决。

### 如何计算最长公共子串长度

* 前面提到，*最长公共子串作为编辑距离的一种，只允许增加和删除字符两种编辑操作*。从名字上可能看起来和编辑距离关系不大，但实际上它也表示了两个字符串之间的相似程度。
* 计算最长公共子串长度的过程和莱文子串的计算过程非常相似，也可以使用都动态规划来解决。这里我们直接定义状态，再定义状态转移方程。
* 每个状态还是包括三个变量 `(i, j, max_lcs)`, `max_lcs` 表示 `a[0...i]` 和 `b[0...j]` 的最长公共子串长度。接下来我们关注 `(i, j)` 的这个状态是从哪些状态转移的呢？
* 还是先看回溯的处理方式，从 `a[0]` 和 `b[0]` 开始，依次考察两个字符串中的字符是否匹配。
	* 如果 `a[i]` 和 `b[j]` 互相匹配，我们把最大公共子串长度加一，并且继续考察 `a[i + 1]` 和 `b[j + 1]`。
	* 如果 `a[i]` 和 `b[j]` 不匹配，最长公共子串的长度不变，这时有两种决策选择：
		* 删除 `a[i]`, 或者在 `b[j]` 前面加上字符 `a[i]`, 然后继续考察 `a[i + 1]` 和 `b[j]`.
		* 删除 `b[j]`, 或者在 `a[i]` 前面加上字符 `b[j]`, 然后继续考察 `a[i]` 和 `b[j + 1]`.
* 根据这个过程，可以推断出 `a[0...i]` 和 `b[0...j]` 的最长公共长度 `max_lcs(i, j)`，只可能通过下面三个状态转移过来：
	* `(i - 1, j - 1, max_lcs)`，其中 `max_lcs` 表示 `a[0...i-1]` 和 `b[0...j-1]` 的最长公共子串长度。
	* `(i - 1, j, max_lcs)`，其中 `max_lcs` 表示 `a[0...i-1]` 和 `b[0...j]` 的最长公共子串长度。
	* `(i, j - 1, max_lcs)`其中 `max_lcs` 表示 `a[0...i-1]` 和 `b[0...j]` 的最长公共子串长度。
* 把状态转移过程用状态转移方程写出来，就是下面这样：

```shell
如果：a[i]==b[j]，那么：max_lcs(i, j)就等于：
max(max_lcs(i-1,j-1)+1, max_lcs(i-1, j), max_lcs(i, j-1))；

如果：a[i]!=b[j]，那么：max_lcs(i, j)就等于：
max(max_lcs(i-1,j-1), max_lcs(i-1, j), max_lcs(i, j-1))；

其中max表示求三数中的最大值。
```

* 根据状态转移方程，就可以写出代码实现。

```java
public int lcs(char[] a, int n, char[] b, int m) {
  int[][] maxlcs = new int[n][m];
  for (int j = 0; j < m; ++j) {//初始化第0行：a[0..0]与b[0..j]的maxlcs
    if (a[0] == b[j]) maxlcs[0][j] = 1;
    else if (j != 0) maxlcs[0][j] = maxlcs[0][j-1];
    else maxlcs[0][j] = 0;
  }
  for (int i = 0; i < n; ++i) {//初始化第0列：a[0..i]与b[0..0]的maxlcs
    if (a[i] == b[0]) maxlcs[i][0] = 1;
    else if (i != 0) maxlcs[i][0] = maxlcs[i-1][0];
    else maxlcs[i][0] = 0;
  }
  for (int i = 1; i < n; ++i) { // 填表
    for (int j = 1; j < m; ++j) {
      if (a[i] == b[j]) maxlcs[i][j] = max(
          maxlcs[i-1][j], maxlcs[i][j-1], maxlcs[i-1][j-1]+1);
      else maxlcs[i][j] = max(
          maxlcs[i-1][j], maxlcs[i][j-1], maxlcs[i-1][j-1]);
    }
  }
  return maxlcs[n-1][m-1];
}

private int max(int x, int y, int z) {
  int maxv = Integer.MIN_VALUE;
  if (x > maxv) maxv = x;
  if (y > maxv) maxv = y;
  if (z > maxv) maxv = z;
  return maxv;
}
```

#### 最长公共子串的 TS 实现

* 继续用熟悉的 TS 来实现一遍。
##### 动态规划实现

* 这里列举一种动态规划实现，和上面的第二种实现类似。

```typescript
/**
 * @description: 动态规划计算最长公共子串
 * @param {string} text1
 * @param {string} text2
 * @return {number}
 */
function getLcsByDP(text1: string, text2: string): number {
  const aLen = text1.length;
  const bLen = text2.length;
  if (aLen * bLen === 0) return 0;

  // 建立 dp 数组
  const dp = new Array(aLen + 1).fill(null).map(() => new Array(bLen + 1));

  // 初始化第一个字符串相关的状态，第一个字符不作比较，长度就是 0
  for (let i = 0; i <= aLen; i++) {
    dp[i][0] = 0;
  }
  // 初始化第二个字符串相关的状态
  for (let j = 0; j <= bLen; j++) {
    dp[0][j] = 0;
  }

  for (let i = 1; i <= aLen; i++) {
    for (let j = 1; j <= bLen; j++) {
      // 更新状态的过程，如果上一个值相等，就把值加 1
      const len = text1[i - 1] === text2[j - 1] ? 1 : 0;
      dp[i][j] = max(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1] + len);
    }
  }

  /**
   * @description: 求出三个数中的最大值
   * @param {number} x
   * @param {number} y
   * @param {number} z
   * @return {number}
   */
  function max(x: number, y: number, z: number): number {
    let maxLcs = Number.MIN_SAFE_INTEGER;
    if (x > maxLcs) maxLcs = x;
    if (y > maxLcs) maxLcs = y;
    if (z > maxLcs) maxLcs = z;
    return maxLcs;
  }
  // 打印 dp 数组，更形象展示填表内容
  printMatrix(text1, text2, dp);
  return dp[aLen][bLen];
}
// 展示打印状态的结果，会和本节内容有所不同
console.log(getLcsByDP('mitcmu', 'mtacnu')); // 4
console.log(getLcsByDP('abc', 'def')); // 0
console.log(getLcsByDP('zhuzhengyuan', 'qiuxiao')); // 2
```

* 打印 dp 数组内容，打印方法不再赘述。

```shell
  / m t a c n u
/ 0,0,0,0,0,0,0
m 0,1,1,1,1,1,1
i 0,1,1,1,1,1,1
t 0,1,2,2,2,2,2
c 0,1,2,2,3,3,3
m 0,1,2,2,3,3,3
u 0,1,2,2,3,3,4
```

##### 回溯实现

* 实现思路和求解莱文斯坦距离类似。

```typescript
/**
 * @description: 回溯的解决
 * @param {string} a
 * @param {string} b
 * @return {number}
 */
function getLcs(a: string, b: string): number {
  const aLen = a.length;
  const bLen = b.length;
  let maxLcs = Number.MIN_SAFE_INTEGER;

  function recursive(i: number, j: number, lcs: number): void {
    if (i === aLen || j === bLen) {
      if (maxLcs < lcs) maxLcs = lcs;
      return;
    }
    if (a[i] === b[j]) {
      recursive(i + 1, j + 1, lcs + 1);
    } else {
      recursive(i + 1, j + 1, lcs);
      recursive(i + 1, j, lcs);
      recursive(i, j + 1, lcs);
    }
  }

  recursive(0, 0, 0);
  return maxLcs;  
}
console.log(getLcs('mitcmu', 'mtachu')); // 4
console.log(getLcs('zhuzhengyuan', 'qiuxiao')); // 2
```

#### 最长公共子串 LeetCode

* [LeetCode-1143](https://leetcode.cn/problems/longest-common-subsequence/): 最大公共子序列.
	* 这道题和该小节内容本质是一样的，上面的动态规划实现完全可以直接当成题解。

## 拼写纠错是如何实现的

* 当用户在搜索框内，输入一个拼写错误的单词时，我们拿这个单词和词库中的单词一一进行比较，计算编辑距离，把编辑距离最小的单词作为纠正之后的单词，提示给用户。
* 这个过程就是拼写纠错的基本原理。但是工业级别的搜索引擎的原理，拼写纠错不止这么简单。
	* 一方面，单纯使用编辑距离来纠错，效果并不一定好；
	* 另一方面，词库中的数据量可能非常大，搜索引擎要支持海量的搜索，因此对纠错的性能要求很高。

### 纠错效果的优化

* 针对纠错的效果的优化，通常有以下几种思路：
	* 在计算出编辑距离的时候，不是取出编辑距离最小的单词，而是**取出编辑距离最小的 Top 10 单词**，再根据其他参数来决定使用哪个单词作为提示。例如，**使用热门搜索程度来决定使用 Top 10 里的哪个单词作为拼写纠错单词。**
	* **可以同时使用多种编辑距离计算的方法**，例如本文中的两种，分别计算得出编辑距离最小的 Top 10 单词，**再取两者的交集，用交集的结果继续优化处理。**
	* **还可以统计用户的搜索日志，得到最经常拼错的单词列表，以及对应的拼写正确单词。** 搜索引擎在拼写纠错的时候，首先在这个单词列表内查找，如果找到就直接返回拼写纠错单词，这种方式的纠错效果提升非常显著。
	* **基于上一种方式，我们可以引入个性化因素。 针对每一个用户，维护这个用户特有的搜索喜好，也就是常用的搜索关键词。** 当用户拼写错误的时候，我们现在这个搜索关键词内计算编辑距离，查找编辑距离最小的单词。

### 纠错性能的优化

* 针对纠错性能也有相应的优化方式，这里介绍两种**分治思路**的优化方式：
	* 如果纠错功能的 **TPS**(`Transactions Per Second`) *服务器每秒传输的事物处理个数* 不高，我们可以部署多台机器，每台机器运行一个独立的纠错功能。**每当有一个纠错请求的时候，通过负载均衡，分配到其中一台机器，来计算编辑距离，得到纠错单词。**
	* 如果纠错的响应时间很长，也就是每个纠错的请求处理时间过长。那么可以把纠错的词库，分割到很多台机器里去，**当有一个纠错请求的时候，就把这个纠错单词同时发送到多台机器上去，让多台机器并行处理，分别得到编辑距离最小的单词，然后再对比合并，最终得出一个最优的拼写纠错单词。**
* 当然，实践中的拼写纠错，远远会比这里介绍的过程要复杂得多，但是其核心是类似的。

## 小结

* 动态规划至此三个章节就结束了。它的理论总结起来就是“一个模型三个特征”。
* 要熟练地掌握动态规划，还是需要不断地练习，不断地做题。
* 三个章节里的案例一共有 8 道经典题。实际问题中的很多问题，都是由这 8 个问题衍生出来的，所以掌握这 8 道题，也就能理解动态规划，已经能够应付常规的动态规划问题。

## 扩展

* 下面是动态规划的一些实践题目。主要围绕着递增子序列展开。

### 最长递增子序列

* 这里有对应的 [LeetCode-300](https://leetcode-cn.com/problems/longest-increasing-subsequence/)
* 下面是其中一种动态规划方式的题解。

```typescript
/**
 * @description: 求最长递增子序列长度
 * 动态规划的实现
 * 时间复杂度 O(n2)
 * 空间复杂度 O(n)
 * @param {number[]} nums
 * @return {number}
 */
function lengthOfLIS(nums: number[]): number {
  const dp: number[] = new Array(nums.length).fill(1); // 初始化 dp 数组

  // 从末位向前遍历，比较容易理解
  for (let i = nums.length - 2; i >= 0; i--) {
    // 取当前这一位数字前面的所有数字与当前数字比较，得到当前这一位的最长递增子序列长度
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[j] > nums[i]) {
        // !注意是 dp[j] + 1 和 dp[i] 比较，而不是和 dp[i] + 1 比较
        dp[i] = dp[i] < dp[j] + 1 ? dp[j] + 1 : dp[i]; // dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }

  return Math.max(...dp);
}
```

### 衍生题

#### 最长连续递增子序列

* 简单题，当前题目的简化版 [LeetCode-674](https://leetcode-cn.com/problems/longest-continuous-increasing-subsequence/): 最长连续递增子序列。

#### 最长递增子序列的个数

* 类似中等题，[LeetCode-673](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/) 返回最长递增子序列的个数。

```typescript
function findNumberOfLIS(nums: number[]): number {
  const dp: number[] = new Array(nums.length).fill(1); // 保存状态，当前索引所对应的递增子序列的长度
  const counts: number[] = new Array(nums.length).fill(1); // 保存当前索引对应的最长递增子序列的个数

  // 从后向前，得到每一位的最长递增子序列长度
  for (let i = nums.length - 2; i >= 0; i--) {
    // 从当前位向后比较，更加容易理解
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[j] > nums[i]) {
        // 如果获得的递增子序列长度一致，则叠加保存有效递增子序列的数组
        if (dp[i] === dp[j] + 1) {
          // !注意，这里叠加的不是 1，而是当前比较项所保存的值
          counts[i] += counts[j] ;
          // 如果递增子序列长度比当前保存的更长，则重置
        } else if (dp[j] + 1 > dp[i]) {
          // !注意，这里重置的核心是：重置为当前比较项所保存的值，而不是 1
          counts[i] = counts[j];
          // 保存更长的子序列长度
          dp[i] = dp[j] + 1;
        }
      }
    }
  }

  // 找到最大值，并且叠加每个最大值所对应的有效数组数量
  const max = Math.max(...dp);
  return dp.reduce((prev: number, curr: number, index) => {
    if (curr === max) prev += counts[index];
    return prev;
  }, 0)
};
```

#### 列举所有递增子序列

* 类似中等题，[LeetCode-491](https://leetcode-cn.com/problems/increasing-subsequences/) 列出所有递增子序列。
	* 这道题用动态规划其实比较难做，因为它的重复子问题并不那么直观。目前自己的理解是没有重复子问题，不适合使用动态规划。
	* 因此实际上更适合使用回溯算法，下面使用 Set 数据结构来去重提高效率，但并没有降低执行的时间复杂度。而更好的方式是使用数组来实现剪枝的效果，这点可以留待日后回顾的时候进行思考和优化。

```typescript
/**
 * @description: 回溯的方式
 * @param {number[]} nums
 * @return {number[][]}
 */
function findSubsequences(nums: number[]): number[][] {
  const res: number[][] = []; // 存储结果的数组
  const cur: number[] = []; // 存放当前增长序列的数组
  const set = new Set(); // 避免重复的数组存储计算

  /**
   * @description: 递归遍历得到所有递增子序列
   * @param {number} startIndex
   * @return {void}
   */
  function getSubsequences(startIndex: number): void {
    // 如果数组内有值且之前没有保存过
    if (cur.length > 1) {
      if (!set.has(cur.join(','))) {
        res.push(cur.slice()); // 保存副本，避免后续变化污染
        set.add(cur.join(','));
      }
    }
    for (let i = startIndex; i < nums.length; i++) {
      // !比较大小，仅在大于当前数组中末尾数的时候才推入当前数
      // 如果数组中没有值，也需要推入
      if (cur.length && nums[i] < cur[cur.length - 1]) continue;
      // if (nums[i] >= cur[cur.length - 1] || !cur.length) {}
      cur.push(nums[i]);
      getSubsequences(i + 1);
      cur.pop(); // 弹出末尾值让数组能够持续更新
    }
  }

  getSubsequences(0);
  return res;
}
```

#ALG 