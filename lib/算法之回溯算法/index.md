# 回溯算法

## 简言

* 很多经典的数学问题，最终都是通过回溯算法思想解决的，例如数组、八皇后、0-1 背包、图的着色、旅行商问题、全排列等等。

> 学习曲线：★★★★

## 如何理解回溯算法

* 回溯的处理思想有些类似枚举搜索。我们枚举所有的解，找到满足期望的解。为了有规律地枚举所有可能的解，避免遗漏和重复，我们会把问题求解的过程分成多个阶段。每个阶段，都会面对一个岔路口，我们先随意选择一条路走，当发现这条路走不通的时候（也就是遇到不符合期望的解），就回退到上一个岔路口，另选一种走法继续往下走。
* **回溯算法用更加具体的方式来理解，可以把它理解成一棵树，就像二叉树的深度遍历一样；回溯算法的遍历过程，类似于深度优先遍历一颗 N 叉树，沿着某个分支往下，直到到达叶子节点，于是回到上一个节点也就是父节点继续遍历**。==在某些回溯算法解决的问题里，这颗 N 叉树的所有叶子节点就是问题的解；在另一些问题里，叶子节点的父节点是问题的解。==
	* **为了提高遍历搜索的效率，我们可以在 N 叉树的某个节点通过当前的条件来判断是否要继续搜索下去，当条件在当前节点已经不满足时，不需要再深度优先的搜索下去，也就是提前终止条件。这个过程就是剪枝，减去 N 叉树中一条子树的分叉枝干**。个人认为剪枝的定义通过树来理解非常通俗易懂。
* 用八皇后的经典案例来理解或许更加容易。

### 八皇后

#### 八皇后的条件

* **有一个 `8*8` 的棋盘，往上面放八个棋子（皇后），要求每个棋子所在的行、列、对角线都不能有另一个棋子，也就是对应着国际象棋中的各个皇后不能够互相攻击，可以在一颗棋盘上共存。**
	* 在此背景下，请问有多少种摆放方式，有哪些具体摆法？
* 下面的图里左边是满足要求的，右边则是不满足要求的。

![[八皇后示例.png]]

* 我们把这个问题划分成八个阶段，也就是依次把 8 个棋子放到第一行，第二行、第三行……第八行。在放置的过程中，不停检查当前放置的位置是否满足要求。如果满足就继续往下一行里放置棋子；如果不满足，就找下一个格子尝试放置，直至满足条件。
* 回溯算法也很适合使用递归来实现。下面就是八皇后的 Java 实现，仅供参考，里面有比较多的注释来帮助理解。不过要真正理解八皇后的实现逻辑，还是需要自己动手实现一遍。

```java
int[] result = new int[8];//全局或成员变量,下标表示行,值表示queen存储在哪一列
public void cal8queens(int row) { // 调用方式：cal8queens(0);
  if (row == 8) { // 8个棋子都放置好了，打印结果
    printQueens(result);
    return; // 8行棋子都放好了，已经没法再往下递归了，所以就return
  }
  for (int column = 0; column < 8; ++column) { // 每一行都有8种放法
    if (isOk(row, column)) { // 有些放法不满足要求
      result[row] = column; // 第row行的棋子放到了column列
      cal8queens(row+1); // 考察下一行
    }
  }
}

private boolean isOk(int row, int column) {//判断row行column列放置是否合适
  int leftup = column - 1, rightup = column + 1;
  for (int i = row-1; i >= 0; --i) { // 逐行往上考察每一行
    if (result[i] == column) return false; // 第i行的column列有棋子吗？
    if (leftup >= 0) { // 考察左上对角线：第i行leftup列有棋子吗？
      if (result[i] == leftup) return false;
    }
    if (rightup < 8) { // 考察右上对角线：第i行rightup列有棋子吗？
      if (result[i] == rightup) return false;
    }
    --leftup; ++rightup;
  }
  return true;
}

private void printQueens(int[] result) { // 打印出一个二维矩阵
  for (int row = 0; row < 8; ++row) {
    for (int column = 0; column < 8; ++column) {
      if (result[row] == column) System.out.print("Q ");
      else System.out.print("* ");
    }
    System.out.println();
  }
  System.out.println();
}
```

#### 八皇后的解题思路

* 上文的讲解比较粗糙，这里用自己的理解的方式再细化补充一些。
* 从方便理解的角度，其实我们可以构造两个二维数组，分别存放皇后的具体摆放位置 `queens`，以及皇后的攻击范围 `attack`。随着每一行皇后的摆放，实时更新皇后的攻击范围数组，再在这个攻击范围的条件限定内去查找下一个可以摆放的位置。
* 但更方便的实现是，因为==**已知皇后的行和列不会重合（攻击范围内），所以其实可以用一维数组来表示，下标表示行，下标位置存储的值表示列**==。这样就可以直接避免行列的重复判断，只需要判断对角线，从而简化代码逻辑。

#### TS 实现

* 下面是重新实现的八皇后。

```typescript
// 初始化一个长度为 8 的数组表示棋盘，全局变量方便存取
const res = new Array(8);

/**
 * @description: 判断当前行和列的位置是否可以摆放
 * @param {number} row
 * @param {number} col
 * @return {boolean}
 */
function isQueenOk(row: number, col: number): boolean {
  // 放置列的左右列，分别从左上角和右上角探测，如果值匹配就说明在攻击范围内
  let left = col - 1, right = col + 1;
  // 判断是否是已有的皇后的攻击范围内，从当前行的上一行倒推判断
  for (let i = row - 1; i >= 0; i--) {
    if (res[i] === col) return false; // 竖向的列已经摆放过棋子，也就是当前位置的正上方已经有棋子
    if (left >= 0) {
      if (res[i] === left) return false; // 前面所有行的左上角是否有棋子
    }

    if (right < 8) {
      if (res[i] === right) return false; // 前面所有行的右上角是否有棋子
    }
    
    // 扩展左右对角线的上下限，往更前一行的左右上方的位置是否已经存在棋子
    left--;
    right++;
  }
  return true;
}

/**
 * @description: 计算八皇后摆法
 * @param {number} row
 * @return {void}
 */
function calc8Queens(row: number): void {
  // 如果到第八行，表示摆放结束，直接打印出结果
  if (row === 8) {
    printQueens(res);
    return;
  }
  
  // 从左往右遍历所有列
  for (let col = 0; col < 8; col++) {
    // 判断是否在前面皇后的攻击范围内，是否是已摆放棋子的辐射范围内
    if (isQueenOk(row, col)) {
      res[row] = col; // 更新摆放的位置
      // 进入下一行的判断
      calc8Queens(row + 1);
    }
  }
}

/**
 * @description: 打印出所有棋盘的摆列
 * @param {number[]} res
 * @return {void}
 */
function printQueens(res = []): void {
  console.log('current Queen is ', res);
  for (let row = 0; row < 8; row++) {
    let rowStr = '|'; // 打印每行内容
    for (let col = 0; col < 8; col++) {
      if (res[row] === col) rowStr += 'Q|';
      else rowStr += '*|';
    }
    console.log(rowStr);
  }
}

calc8Queens(0); // 执行八皇后的计算
```

* 下面展示其中一种解的打印输出。
	* **`current Queen is  [7, 3, 0, 2, 5, 1, 6, 4]`**.

```shell
|*|*|*|*|*|*|*|Q|
|*|*|*|Q|*|*|*|*|
|Q|*|*|*|*|*|*|*|
|*|*|Q|*|*|*|*|*|
|*|*|*|*|*|Q|*|*|
|*|Q|*|*|*|*|*|*|
|*|*|*|*|*|*|Q|*|
|*|*|*|*|Q|*|*|*|
```

#### N皇后

* N 皇后是更通用的方式。
* 下面罗列一种写法。也可以查看 LeetCode 对应的题解，题目已完成。

```typescript
/**
 * @description: N 皇后
 * @param {number} n
 * @return {void}
 */
function buildNQueens(n: number): void {
  this.queens = new Array(n); // 存储摆放结果

  /**
   * @description: 确认摆放位置是否可以
   * @param {number} row
   * @param {number} col
   * @return {boolean}
   */
  function isOk(row: number, col: number): boolean {
    let leftDiag: number = col - 1;
    let rightDiag: number = col + 1;
    for (let i = row - 1; i >= 0; i--) {
      if (this.queens[i] === col) return false;
      // 分别判断左上和右上的对角线
      if (leftDiag >= 0) {
        if (this.queens[i] === leftDiag) return false;
      }

      if (rightDiag < n) {
        if (this.queens[i] === rightDiag) return false;
      }
      leftDiag--;
      rightDiag++;
    }
    return true;
  }

  /**
   * @description: 构建 N 皇后
   * @param {number} row
   * @return {void}
   */
  function buildQueens(row: number) {
    if (row === n) {
      print(this.queens);
      return;
    }

    for (let col = 0; col < n; col++) {
      if (isOk(row, col)) {
        this.queens[row] = col;
        buildQueens(row + 1);
      }
    }
  }

  /**
   * @description: 用一个较形象美观的方式打印出 N 皇后的摆列
   * @param {number[]} queens
   * @return {void}
   */
  function print(queens: number[]) {
    console.log(`--[${queens}]--`);
    for (let row = 0; row < n; row++) {
      let rowPosition = '|';
      for (let col = 0; col < n; col++) {
        rowPosition += queens[row] === col ? 'Q|' : '·|'
      }
      console.log(rowPosition);
    }
  }  

  return buildQueens(0);
}

// 分别打印出四皇后五皇后和六皇后
buildNQueens(4);
buildNQueens(5);
buildNQueens(6);
```

* 下面展示四五六皇后各一种摆放示意解。

```shell
# [2,4,1,3,0]
|·|·|Q|·|·|
|·|·|·|·|Q|
|·|Q|·|·|·|
|·|·|·|Q|·|
|Q|·|·|·|·|

# [2,4,1,3,0]
|·|·|Q|·|·|
|·|·|·|·|Q|
|·|Q|·|·|·|
|·|·|·|Q|·|
|Q|·|·|·|·|

# [4,2,0,5,3,1]
|·|·|·|·|Q|·|
|·|·|Q|·|·|·|
|Q|·|·|·|·|·|
|·|·|·|·|·|Q|
|·|·|·|Q|·|·|
|·|Q|·|·|·|·|
```

* 对应 [LeetCode-51](https://leetcode-cn.com/problems/n-queens/) 题: N 皇后，是对八皇后的扩展。题解的结果更加通用。
	* 以及 [LeetCode](https://leetcode-cn.com/problems/eight-queens-lcci/)，题目类似，解法相同。
* [LeetCode-79](https://leetcode-cn.com/problems/word-search/)  这道题有点像是数独，可以当作扩展实现。💭

## 回溯算法应用举例分析

* 回溯算法的理论比较好理解，但是实现起来相对困难一些。尤其是使用递归来实现的思路并不太容易想到。下面我们通过两个经典案例来回顾回溯算法的应用和实现。

### 0-1 背包

* 0-1 背包是非常经典的算法问题，因为很多场景都可以抽象成这个问题模型。而且这个问题的经典解法是动态规划，但也有一种相对不那么高效却简单的解法，就是回溯算法。这节主要关注回溯算法，后续章节重点介绍动态规划。

#### 0-1 背包的题干

* 0-1 背包的变体很多，这里先介绍最基础的。*我们有一个背包，背包总的承载重量是 Wkg。假设有 n 个物品，每个物品的重量不等，并且不可分割。现在期望选择几件物品装载到背包中，在不超过背包所能装载重量的前提下，如何让背包中物品的总重量最大？*
* 在前面贪心的章节里，也提到过一个背包问题，不过当时那个问题装载的物品是可以分割的，可以装一部分到背包。而这个问题的物品是不可分割的，只存在装与不装两种情况，所以称之为 0-1 背包。这样就不能使用贪心算法来解。

#### 0-1 背包的解题思路

* 使用回溯算法的思路，对每个物品来说，都有两种选择，装进背包或者不装进背包。对于 n 个物品，总的装法就有 2 的 n 次方种，去掉总重量超过 Wkg 的，从剩下的装法中选择最接近 Wkg 的。现在的问题就是，如何来不重复地穷举出这 `2^n` 种装法？
* 我们把物品依次排列，整个问题就会分解成 n 个阶段，每个阶段对应一个物品如何选择。对第一个物品进行处理，选择装进去或者不装进去，再递归地处理剩下的物品。
* 这里代码实现的时候也使用了剪枝的技巧，就是在发现已选择的物品重量超过 Wkg 之后，停止探测剩下的物品。下面是具体的代码。

```java
public int maxW = Integer.MIN_VALUE; //存储背包中物品总重量的最大值
// cw表示当前已经装进去的物品的重量和；i表示考察到哪个物品了；
// w背包重量；items表示每个物品的重量；n表示物品个数
// 假设背包可承受重量100，物品个数10，物品重量存储在数组a中，那可以这样调用函数：
// f(0, 0, a, 10, 100)
public void f(int i, int cw, int[] items, int n, int w) {
  if (cw == w || i == n) { // cw==w表示装满了;i==n表示已经考察完所有的物品
    if (cw > maxW) maxW = cw;
    return;
  }
  f(i+1, cw, items, n, w);
  if (cw + items[i] <= w) {// 已经超过可以背包承受的重量的时候，就不要再装了
    f(i+1,cw + items[i], items, n, w);
  }
}
```

#### 0-1 背包 TS 实现

* 下面是具体的 TS 实现。我们设定一个具体的条件来验证：假设有 10 个物品，每个重量各不相同，书包的总承重是 100，求能存放物品的最大的大小。

```typescript
let maxW = Number.MIN_SAFE_INTEGER;

/**
 * @description: 0-1 背包求解实现
 * @param {number} i
 * @param {number} currentW
 * @param {number[]} items
 * @param {number} n
 * @param {number} w
 * @return {void}
 */
function fn0_1(i: number, currentW: number, items: number[], n: number, w: number): void {
  // 已经达到最大重量或者物品已经遍历结束
  if (maxW === w || i === n) {
    if (currentW > maxW) maxW = currentW;
    return;
  }
	  // !这里执行一次的作用是用于判断当前背包是否已经满足条件，其实也就是对应不装的场景
  fn0_1(i + 1, currentW, items, n, w);
  // 在满足条件下继续添加
  if (currentW + items[i] <= w) {
    fn0_1(i + 1, currentW + items[i], items, n, w);
  }
}

fn0_1(0, 0, [43, 9, 20, 8, 83, 32, 21, 14, 18, 75], 10, 100);
console.log('max weight is', maxW); // max weight is 100
```

### 正则表达式

* 正则表达式中最重要的算法思想之一就是回溯思想。

#### 设定正则表达式的条件

* 正则表达式里有很重要的通配符操作。通过把不同的通配符结合在一起，可以实现出非常多的语义。这里为了方便理解，我们先假设只有 `*` 和  `?` 两种通配符，并且简化这两种通配符的语义。
* *设  `*` 为匹配任意多个（大于等于 0 个）任意字符，`?` 匹配 0个或者 1 个任意字符。*

#### 正则表达式的实现思路

* 基于上述定义，我们来做一个简易的正则表达式匹配。
* 我们依次考察正则表达式中的每个字符，当遇到不是通配字符的时候，就直接和文本的字符进行匹配，如果相同，继续往下处理，如果不同，就回溯处理。
	* 如果遇到特殊字符，就会有多种处理方式，也就是会有岔路口。比如 `*` 有多种匹配方法，可以匹配任意个文本串中的字符，我们先随意的选择一种匹配方案，继续考察剩下的字符，如果中途发现无法匹配下去，就回到上个岔路口，重新选择一种别的匹配方案，再继续匹配剩下的字符。
* 下面是相关的代码实现。

```java
public class Pattern {
  private boolean matched = false;
  private char[] pattern; // 正则表达式
  private int plen; // 正则表达式长度

  public Pattern(char[] pattern, int plen) {
    this.pattern = pattern;
    this.plen = plen;
  }

  public boolean match(char[] text, int tlen) { // 文本串及长度
    matched = false;
    rmatch(0, 0, text, tlen);
    return matched;
  }

  private void rmatch(int ti, int pj, char[] text, int tlen) {
    if (matched) return; // 如果已经匹配了，就不要继续递归了
    if (pj == plen) { // 正则表达式到结尾了
      if (ti == tlen) matched = true; // 文本串也到结尾了
      return;
    }
    if (pattern[pj] == '*') { // *匹配任意个字符
      for (int k = 0; k <= tlen-ti; ++k) {
        rmatch(ti+k, pj+1, text, tlen);
      }
    } else if (pattern[pj] == '?') { // ?匹配0个或者1个字符
      rmatch(ti, pj+1, text, tlen);
      rmatch(ti+1, pj+1, text, tlen);
    } else if (ti < tlen && pattern[pj] == text[ti]) { // 纯字符匹配才行
      rmatch(ti+1, pj+1, text, tlen);
    }
  }
}
```

#### 正则表达式 TS 简易实现

* 下面是 TS 的实现以及简易的使用，含有相关日志输出。

```typescript
/**
 * @description: 简易的正则构造类
 */
class Pattern {
  private matched = false;
  private pattern = '';
  private pLen = 0;

  constructor(pattern: string) {
    this.pattern = pattern;
    this.pLen = pattern.length;
  }

  /**
   * @description: 内部的匹配逻辑
   * @param {number} textIdx
   * @param {number} patternIdx
   * @param {string} text
   * @param {number} tLen
   * @return {void}
   */
  private reMatch(textIdx: number, patternIdx: number, text: string, tLen: number): void {
    if (this.matched) return;
    // 如果正则已经匹配完成
    if (patternIdx === this.pLen) {
      // 字符串也匹配完成
      if (textIdx === tLen) this.matched = true;
      return;
    }

    if (this.pattern[patternIdx] === '*') { // '*' 匹配任意个字符
      for (let i = textIdx; i < tLen; i++) {
        this.reMatch(i, patternIdx + 1, text, tLen);
      }
    } else if (this.pattern[patternIdx] === '?') { // '?' 表示匹配任意字符 0 个或 1 个
      this.reMatch(textIdx, patternIdx + 1, text, tLen);
      this.reMatch(textIdx + 1, patternIdx + 1, text, tLen);
    } else if (textIdx < tLen && text[textIdx] === this.pattern[patternIdx]) { // 纯字符匹配，需要完全相等
      this.reMatch(textIdx + 1, patternIdx + 1, text, tLen);
    }
  }

  /**
   * @description: 对外的匹配 API
   * @param {string} text
   * @return {boolean}
   */
  public match(text: string): boolean {
    this.matched = false;
    this.reMatch(0, 0, text, text.length);
    return this.matched;
  }
}

const regexp = new Pattern('*str');
console.log(regexp.match('a str')); // true
console.log(regexp.match('a s1tr')); // false

const regexp2 = new Pattern('str?end');
console.log(regexp2.match('str end')); // true
console.log(regexp2.match('an end')); // false
console.log(regexp2.match('an end str')); // false
console.log(regexp2.match('a str1end')); // false
console.log(regexp2.match('strend')); // true
```

* 正则的题对应 [LeetCode-10](https://leetcode-cn.com/problems/regular-expression-matching/). 这道题比本文中的示例稍难一些，且它的官方解法使用的是动态规划，当前暂未解决。💭

## 回溯算法视频解析

* 理论解析。

<iframe src='https://player.bilibili.com/player.html?bvid=BV1cy4y167mM&cid=562531932&page=1&share_source=copy_web' scrolling='no' border='0' frameborder='no' framespacing='0' allowfullscreen='true' width="100%" height="520px"></iframe>

## 小结

* 回溯算法的思想本身比较抽象，大多数情况下用来解决广义的搜索问题。从一组可能的解中，选择出一个满足要求的解。回溯算法非常适合用递归来实现，在实现的过程中，剪枝操作是提高回溯效率的一种技巧。
	* **使用剪枝，可以不用穷举搜索所有的情况，从而提高搜索效率。**
* 回溯算法的原理比较简单，但可以解决很多类问题：组合类问题，子集类问题，排列问题，切割问题，对应到具体的经典问题，就比如深度优先搜索、N 皇后问题、0-1 背包问题、图的着色、旅行商问题、数独问题，全排列以及正则表达式等等。
* ==在回溯算法里，对于选择类的问题，通常情况下递归函数的调用顺序打乱并不会影响执行结果。==
	* **例如 0-1 背包中的选择跳过当前物品，或者选择放入当前物品，这两者对应的执行逻辑的先后顺序如果发生改变，不会影响执行结果。**

### 回溯经典题目

* 可以试着自己实现这些经典问题，如果能够熟练写出它们的实现，也就能够基本掌握回溯算法。
	* [LeetCode-39](https://leetcode-cn.com/problems/combination-sum/): 组合总和（0-1背包的衍生™题）
	* [LeetCode-46](https://leetcode-cn.com/problems/permutations/): 全排列
	* [LeetCode-47](https://leetcode-cn.com/problems/permutations-ii/): 全排列2
	* [LeetCode-77](https://leetcode-cn.com/problems/combinations/): 组合
	* [LeetCode-78](https://leetcode-cn.com/problems/subsets/): 子集
* 下面是[**全排列**](https://leetcode-cn.com/problems/permutations/)的题解，供参考。

```typescript
/**
 * @description: 全排列题解
 * 数组内元素唯一，不存在重复元素
 * @param {number[]} nums
 * @return {numbe[][]}
 */
function permute(nums: number[]): number[][] {
  const res: number[][] = []; // 保存所有排列的结果的数组
  const cur: number[] = []; // 保存当前这一轮排列的结果
  const unset: boolean[] = []; // !保存当前这一轮的元素的使用状态
  
  /**
   * @description: 递归遍历
   * @param {number[]} cur
   * @param {boolean[]} unset
   * @return {void}
   */
  function getSubArray(cur: number[], unset: boolean[]): void {
    // 当这一轮数组达到对应长度的时候，排列完成
    if (cur.length === nums.length) {
      res.push(cur.slice());
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (unset[i]) continue;
      // 对当前这轮遍历进行标识，跳过已遍历的元素
      unset[i] = true;
      cur.push(nums[i]);
      // !把 unset 通过参数传递，保证每一轮的状态都是对应的，这个数组是保证能够得到全排列的核心
      getSubArray(cur, unset);
      // 重置状态
      unset[i] = false;
      // 更新数组内容
      cur.pop();
    }
  }

  getSubArray(cur, unset);

  return res;
}

permute([1, 4]); // [ [ 1, 4 ], [ 4, 1 ] ] 
permute([1, 3, 4]); // [[ 1, 3, 4 ],[ 1, 4, 3 ],[ 3, 1, 4 ],[ 3, 4, 1 ],[ 4, 1, 3 ],[ 4, 3, 1 ]]
permute([1, 3, 4, 2]); // 结果较多，不再一一展示
```

* 下面是[**全排列II**](https://leetcode-cn.com/problems/permutations-ii/)的题解，使用带剪枝的回溯实现。

```typescript
/**
 * @description: 回溯算法，剪枝处理
 * @param {number[]} nums
 * @return {number[][]}
 */
function permuteUnique(nums: number[]): number[][] {
  const res: number[][] = [];

  /**
   * @description: 含剪枝的递归操作
   * @param {number[]} cur
   * @param {boolean[]} unset
   * @return {void}
   */
  function getPermuteByCuttingBran(cur: number[], unset: boolean[]): void {
    if (cur.length === nums.length) {
      res.push(cur.slice());
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      // !剪枝操作，排序过后，相等数字所得到的排列组合一定是一样的，利用这一点来剪枝，提高执行效率
      if (unset[i] || (i > 0 && nums[i] === nums[i - 1] && unset[i - 1])) continue;
      unset[i] = true;
      cur.push(nums[i]);
      getPermuteByCuttingBran(cur, unset);
      cur.pop();
      unset[i] = false;
    }
  }

  // 一定要有排序操作
  nums.sort((a, b) => a - b);
  getPermuteByCuttingBran([], []);

  return res;
}

console.log(permuteUnique([1, 1, 2])); // [ [ 1, 1, 2 ], [ 1, 2, 1 ], [ 2, 1, 1 ] ]
```

## 扩展

* *对上文中的 0-1 背包做一个简单改动，如果每个物品不仅重量不同，价值也不同，如何在不超过背包重量的情况下，让背包中的总价值最大？*

### 0-1 背包计算价值的实现

* 对上面的实现稍加改动即可。
* 为了方便演示，这里直接把物品的价值设为物品的重量对 5 取模。

```typescript
let maxW = Number.MIN_SAFE_INTEGER;
let maxV = Number.MIN_SAFE_INTEGER;

/**
 * @description: 0-1 背包求解实现
 * @param {number} i
 * @param {number} currentW
 * @param {number} currentV
 * @param {number} items
 * @param {number} n
 * @param {number} w
 * @return {void}
 */
function fn0_1Plus(i: number, currentW: number, currentV, items: number[],  n: number, w: number): void {
  // 已经达到最大重量或者物品已经遍历结束
  if (maxW === w || i === n) {
	  // !主要判断逻辑在这里，要保证在相同重量的情况下，总价值最大；总价值和总重量不可分开判断
    if (currentV > maxV) {
      maxV = currentV;
			if (currentW > maxW) maxW = currentW;
    }
    return;
  }
  // !这里执行一次的作用是用于判断当前背包是否已经满足条件
  fn0_1Plus(i + 1, currentW, currentV, items, n, w);
  // 在满足条件下继续添加
  if (currentW + items[i] <= w) {
    // !为了方便演示，这里直接把物品的价值设为物品的重量对 5 取模
    fn0_1Plus(i + 1, currentW + items[i], currentV + items[i] * (items[i] % 5), items, n, w);
  }
}

fn0_1Plus(0, 0, 0, [43, 9, 20, 8, 83, 32, 21, 14, 18, 75], 10, 100);
console.log('max weight is', maxW); // max weight is 100
console.log('max value is', maxV); // max value is 309
// 物品是 100 = 83 + 8 + 9 = 100
// 价值是 309 = 9 * 4 + 83 * 3 + 8 * 3
```

#ALG 