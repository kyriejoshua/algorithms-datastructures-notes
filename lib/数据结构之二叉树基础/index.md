# 二叉树基础

## 简言

* 树是非线性表结构，它比线性表结构复杂，内容也会比较多，分成下面几个章节来分析。

| 章节 |          内容          |
| ---- |:---------------------:|
| 01(19)   |       树、二叉树       |
| 02(20)   |       二叉查找树       |
| 03(21)   | 平衡二叉查找树、红黑树   |
| 04(22)   |         红黑树        |
| 05(23)   |         递归树        |

* 带着下面两个疑问，我们来进一步了解二叉树。

>  二叉树有哪些存储方式？哪种二叉树适合用数组存储？
> 学习曲线：★★☆

## 树(Tree)

* 观察下面的树的结构：

![[树的示意.png]]

* 树里面的每个元素称之为节点，而用来**连接相邻节点之间的关系**，叫做**父子关系**。
* 比如下面这幅图，A 节点就是 B 节点的父节点，B 节点是 A 节点的子节点。B、C、D 这三个节点的父节点是同一个节点，所以它们之间互称为兄弟节点。我们把没有父节点的节点叫做根节点，也就是图中的节点 E。我们把**没有子节点的节点叫做叶子节点或者叶节点**，比如图中的 G、H、I、J、K、L 都是叶子节点。

![[树的示意2.png]]

* 关于“树”，还有三个比较相似的概念：**高度（Height）、深度（Depth）、层（Level）**。它们的定义是这样的：

![[树相关的概念.png]]

* **节点的高度：节点到叶子节点的最长路径（边数）；**
* **节点的深度：根节点到这个结点的所经历的边的个数；**
* **节点的层数：节点的深度 + 1；**
* **树的高度：根节点的高度；**
	* 下面这张图能帮助我们理解：

![[树的概念的示意图.png]]

* 类比到生活中的概念来理解这些概念。
	* **“高度”的概念，其实就是从下往上度量**，比如我们要度量第 10 层楼的高度、第 13 层楼的高度，起点都是地面。所以，树这种数据结构的高度也是一样，从最底层开始计数，并且计数的起点是 0。
	* **“深度”这个概念在生活中是从上往下度量的*，比如水中的鱼的深度，是从水平面开始度量的。所以，树这种数据结构的深度也是类似的，从根结点开始度量，并且计数起点也是 0。
	* **“层数”跟深度的计算类似**，不过，计数起点是 1，也就是说根节点位于第 1 层。

## 二叉树(Binary Tree)

* 树的结构很多，但最常用的还是二叉树。
* **二叉树是每个节点最多有两个子节点的树，两个子节点分别是左子节点和右子节点。** *但二叉树并不要求每个节点都一定要有两个子节点，每个节点可以只有左子节点或右子节点。*
	* 由此也可以衍生出，四叉树和八叉树的效果。
* 下面是几个不同的二叉树：

![[二叉树示意图.png]]

* 上图中编号 2 和编号 3 的二叉树是比较特殊的二叉树。
* 其中，观察编号 2 的二叉树可以发现：除子节点外，其他节点都有两个子节点，这类二叉树就叫**满二叉树**。
* 而编号 3  这种二叉树：叶子节点都在最底下两层，而且最后一层的叶子节点都靠左排列，并且其它层的节点个数达到最大，这就是**完全二叉树**。
* 从定义上可以比较好地理解满二叉树，但可能不太好直观地理解完全二叉树，观察下面的完全二叉树示意图来帮助理解。
	* 下图中第一个是完全二叉树，其他都不是完全二叉树。

![[完全二叉树示意图.png]]

#### 二叉树的存储

* 要理解完全二叉树，首先要了解如何表示（或存储）二叉树。
* 存储二叉树有两种方式，**一种是基于指针或引用的二叉链式存储法，一种是基于数组的顺序存储法。**
* 第一种链式存储法比较简单直观，下图中有三个字段，其中一个字段存储数据，另外两个是指向左右子节点的指针。只要知道根节点，就可以通过左右子节点把整棵树串起来。这种存储方式比较常用。

![[链式存储二叉树.png]]

* 而基于数组的顺序存储法，把根节点存储在下标 `i = 1` 的位置，左子节点存储在 `i * 2 = 2` 的位置，右子节点存储在 `i * 2 + 1 = 3` 的位置，依次类推，B 节点的左子节点位置在 `i * 2 = 2 * 2 = 4` 的位置，右子节点存储在 `2 * i + 1 = 2 * 2 + 1 = 5` 的位置。

![[顺序存储法表示完全二叉树.png]]
* 概括一下，如果节点 X 存储在数组中下标为 i 的位置，那么下标 `2 * i` 的位置就存储着它的左子节点，下标 `2 * i + 1` 的位置就是它的右子节点。反过来，下标为 `Math.floor(i / 2)` 的位置就是当前节点的父节点的位置。通过这种方式，只要知道根节点的位置，就可以通过下标计算，把整棵树都串起来。
	* 一般情况下，为了方便计算子节点，根节点会存储在下标为 1 的位置。
	* 刚才以完全二叉树为例子，所以数组存储仅仅浪费了一个下标为 0 的存储位置，如果是非完全二叉树，那么就会浪费比较多的存储空间。譬如下图：

![[顺序存储法表示非完全二叉树.png]]

* 所以**完全二叉树适合数组存储，因为浪费的空间只有一个**。其他二叉树会浪费更多的空间。
* 而**数组存储的方式是最节省内存的方式，因为数组存储的方式不需要像链式存储那样要存储额外的左右子节点的指针**。
	* 其实，**满二叉树就是完全二叉树的其中一种。**
* 后面的章节解析堆和堆排序的时候可以发现，堆其实就是一种完全二叉树，最常用数组来存储。
 
### 二叉树的结构

* 我们使用 TS 描述基本结构。

```typescript
export interface ITreeNode<T> {
  value: T;
  left: ITreeNode<T> | null;
  right: ITreeNode<T> | null;
}

export class TreeNode<T> {
  private left: ITreeNode<T> | null;
  private right: ITreeNode<T> | null;
  private value: T;

  constructor(value?: T, left?: ITreeNode<T> | null, right?: ITreeNode<T> | null) {
    this.value = (value === undefined ? 0 : value) as T;
    this.left = left === undefined ? null : left;
    this.right = right === undefined ? null : right;
  }
}

console.log(new TreeNode());
console.log(new TreeNode(123));
```

### 二叉树的遍历

* 如何遍历二叉树的所有节点，有三种经典的方法：前序遍历，中序遍历，后序遍历。
	* **前序遍历：父->左->右，对于树中的任意节点来说，先打印这个节点，然后再打印它的左子树，最后打印它的右子树。**
	* **中序遍历：左->父->右，对于树中的任意节点来说，先打印它的左子树，然后再打印它本身，最后打印它的右子树。**
	* **后序遍历：左->右->父，对于树中的任意节点来说，先打印它的左子树，然后再打印它的右子树，最后打印这个节点本身。**

![[二叉树遍历.png]]

* 二叉树的遍历实际上就是递归的过程。前序遍历就是先打印根节点，再递归打印左子树，再递归打印右子树。

```java
// 前序遍历的递推公式：
preOrder(r) = print r->preOrder(r->left)->preOrder(r->right)

// 中序遍历的递推公式：
inOrder(r) = inOrder(r->left)->print r->inOrder(r->right)

// 后序遍历的递推公式：
postOrder(r) = postOrder(r->left)->postOrder(r->right)->print r
```

#### 二叉树的递归遍历

* 下面是细化之后递归版本的具体实现：

```typescript
/**
 * @description: 前序遍历
 * @param {INode} node
 * @return {void}
 */
function preOrderTraversal(node: INode): void {
  if (node === null) return;
  console.info(node);
  node.left && preOrderTraversal(node.left);
  node.right && preOrderTraversal(node.right);
}

/**
 * @description: 中序遍历
 * @param {INode} node
 * @return {void}
 */
function inOrderTraversal(node: INode): void {
  if (node === null) return;
  node.left && inOrderTraversal(node.left);
  console.info(node);
  node.right && inOrderTraversal(node.right);
}

/**
 * @description: 后序遍历
 * @param {INode} node
 * @return {void}
 */
function postTraversal(node: INode): void {
  if (node === null) return;
  node.left && postTraversal(node.left);
  node.right && postTraversal(node.right);
  console.info(node);
}
```

#### 二叉树的迭代遍历

* 下面是各自的迭代实现以及 LeetCode 对应题。
* 前序遍历: [LeetCode-144](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

```typescript
/**
 * @description: 前序遍历，迭代实现
 * @param {TreeNode} root
 * @return {number[]}
 */
function preorderTraversal(root: TreeNode | null): number[] {
  if (root === null) return [];
  let stack: TreeNode[] = [root];
  let res: number[] = [];

  while (stack.length) {
    const node = stack.pop();
    res.push(node.val);
    // 先进后出，右子树是后出的，所以要现金
    node.right && stack.push(node.right);
    node.left && stack.push(node.left);
  }

  return res;
}
```

* 中序遍历：[LeetCode-94](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

```typescript
/**
 * @description: 中序遍历，迭代实现
 * @param {TreeNode} root
 * @return {number[]}
 */
function inorderTraversal(root: TreeNode | null): number[] {
  if (root === null) return [];
  let stack: TreeNode[] = [];
  let node: TreeNode = root;
  let res: number[] = [];

  while (stack.length || node !== null) {
    // 把左子树先推入栈内
    while (node !== null) {
      stack.push(node);
      node = node.left;
    }
    const cur = stack.pop();
    res.push(cur.val);
    // 再推入右子树
		node = cur.right;
  }
  return res;
}
```

* 后序遍历：[LeetCode-145](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

```typescript
/**
 * @description: 后序遍历，迭代实现
 * @param {TreeNode} root
 * @return {number[]}
 */
function postorderTraversal(root: TreeNode | null): number[] {
  if (root === null) return [];
  let stack: TreeNode[] = [root];
  let res: number[] = [];

  while (stack.length) {
    const node = stack.pop();
    // 从前推入值,也就是每个当前节点的值都会相对在较后的位置
    res.unshift(node.val);
    node.left && stack.push(node.left);
    node.right && stack.push(node.right);
  }

  return res;
}
```

#### 二叉树遍历的时间复杂度

* 二叉树的遍历，每个节点最多会遍历两遍(*读取当前值以及子节点*），也就是和节点的数量成正比，所以遍历二叉树的时间复杂度就是 `O(n)`。

### 二叉树的实现

* 上面实现基本的二叉树结构和遍历方式。更具体的实现及应用内容可查看[仓库](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Tree/BinaryTree.ts);

## 小结

* 这节主要关注一种非线性表数据结构——树。
* 关于树，有这些比较常用的概念：根节点，叶子节点，父节点，子节点，兄弟节点，以及节点的高度，深度，层数和树的高度。
* 实际开发中使用最多的就是二叉树。二叉树的每个节点最多有两个子节点，分别是左子节点和右子节点。
* 二叉树中有两种比较特殊的树，满二叉树和完全二叉树。其中，满二叉树又是完全二叉树的一种特殊情况。
* 二叉树可以使用链式存储，也可以使用数组顺序存储。数组顺序存储的方式适合完全二叉树，其他类型的二叉树使用数组存储会比较浪费存储空间。
* 二叉树的遍历有几种经典的方式，前序遍历，中序遍历和后序遍历。它们的时间复杂度都是 `O(n)`。关于遍历要非常熟练地写出这三种方式具体实现，当然还有循环的实现。

## 扩展

### 判断是否是堂兄弟节点

* [LeetCode-993](https://leetcode.cn/problems/cousins-in-binary-tree/)

### 构建不同的二叉树

* 给定一组数据，可以构建出多少种不同的二叉树？
	* 计算这个问题的结果，实际上就是理解组合数学中的[卡特兰数](https://www.wikiwand.com/zh-hans/%E5%8D%A1%E5%A1%94%E5%85%B0%E6%95%B0)。
	* 由推算演变得到卡特兰数。目前还只是非常简单的了解，死记硬背套用公式。
	* 对应 [LeetCode-96](https://leetcode-cn.com/problems/unique-binary-search-trees/)

```typescript
/**
 * @description: 数学公式的方式，卡特兰数
 * @param {number} n
 * @return {number}
 */
function numTrees(n: number): number {
  let C: number = 1;

  for (let i = 0; i < n; i++) {
    C *= 2 * (2 * i + 1) / (i + 2);
  }

  return C;
};
```

### 如何实现按层遍历

* 实现按层遍历，也至少有两种方式。
	* 可以通过递归的方式，传入参数保存层数；
	* 也可以通过迭代的方式，创建两层循环来实现。
* 对应 [LeetCode-102](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
	* 下面贴一个循环的实现：

```typescript
/**
 * 循环的方式实现层序遍历
 * @param {TreeNode} root
 * @return {number[][]}
 */
function levelOrder(root: TreeNode | null): number[][] {
  if (!root) {
    return [];
  }
  let queues: TreeNode[] = [root];
  let res: number[][] = [];

  while (queues.length) {
    // 当前层级的长度
    let len = queues.length;
    res.push([]);
    // 遍历同一个层级
    while (len--) {
      const node: TreeNode | null = queues.shift();
      // 不存在就直接跳过
      if (!node) {
        continue;
      }
      if (node.val !== null) {
        res[res.length - 1].push(node.val);
      }
      node.left && queues.push(node.left);
      node.right && queues.push(node.right);
    }
  }

  return res;
};
```

#ALG #ADT