# 二叉搜索树

## 简言

* 上一节我们主要关注树，二叉树和二叉树的遍历。这节将会主要关注一种特殊的二叉树，也就是二叉查找树。
* **二叉查找树的最大特点就是支持动态数据集合的快速插入、删除和查找操作。**

> 散列表也支持动态数据的插入删除和查找等操作，并且散列表的这些操作比二叉查找树更加高效，时间复杂度是 `O(1)`. 那么为什么还要使用二叉树呢？有没有一些使用场景中，是更适合或者说必须使用二叉树来实现呢？
> 学习曲线：★★★☆

## 二叉查找树(Binary Search Tree)

* 二叉查找树是二叉树中最常用的一种类型，也叫**二叉搜索树**。
	* *二叉查找树是为了快速查找而诞生的。*
* 当然，**二叉查找树也支持快速地插入和删除操作。**
* **二叉查找树的特点：在树中的任意一个节点，它的左子树的每个节点的值都小于这个节点的值，而右子树的每个节点的值大于这个节点的值。**
* 下面是二叉查找树的示意图：

![[二叉查找树示意图.png]]

* 我们来分别理解每个操作的具体实现。

## 二叉查找树的操作

### 二叉查找树的查找

* 首先，我们关注如何在二叉查找树中查找一个节点。
* 先取得根节点，如果它等于我们要查找的数据，就返回该数据。
	* 如果要查找的数据比根节点的值小，就在它的左子树中递归查找；
	* 如果要查找的数据比根节点的值大， 就在右子树中递归查找。

![[二叉查找树的查找.png]]

#### 二叉查找树的查找实现
* 二叉搜索树查找的 TS 的实现。
	* 下面是用循环实现的。

```typescript
/**
 * @description: 二叉查找树查找相等值
 * 时间复杂度 O(logn)
 * 空间复杂度 O(1)
 * @param {TreeNode|null} root
 * @param {number} val
 * @return {TreeNode|null}
 */
function findValueOfBinarySearchTree(root: TreeNode | null, val: number): TreeNode | null {
  if (root === null) return null;
  let node: TreeNode | null = root;

  while (node !== null && val !== node.val) {
    if (node.left === null && node.right === null) return null;
    if (val > node.val) {
      node = node.right;
    } else {
      node = node.left;
    }
  }

  return node;
}
```

* 为什么不用递归来实现呢？因为递归过程中是无法中断的，而我们找到结果就应该终止搜索。
* 下面是 java 的实现，仅供参考。

```java
public class BinarySearchTree {
  private Node tree;

  public Node find(int data) {
    Node p = tree;
    while (p != null) {
      if (data < p.data) p = p.left;
      else if (data > p.data) p = p.right;
      else return p;
    }
    return null;
  }

  public static class Node {
    private int data;
    private Node left;
    private Node right;

    public Node(int data) {
      this.data = data;
    }
  }
}
```

### 二叉查找树的插入

* 二叉查找树的插入其实也类似查找操作。
	* 如果插入的数据比节点的数据大，并且节点的右子树为空，就把新数据直接插到右子节点的位置；
	* 如果不为空，就继续递归遍历右子树，查找插入位置；同理，应用到比当前节点的值更小的场景。

![[二叉查找树的插入.png]]

#### 二叉查找树的插入实现

* TS 的实现，来自 [LeetCode-701](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)
	* 同样是使用循环实现的。

```typescript
/**
 * @description: 循环的方式，不是递归的方式
 * 时间复杂度 O(n)
 * 空间复杂度 O(1)
 * @param {TreeNode} root
 * @param {number} val
 * @return {TreeNode}
 */
function insertIntoBST(root: TreeNode | null, val: number): TreeNode | null {
  if (root === null) return new TreeNode(val);
  let node = root;

  // 循环到尽头位置
  while (node !== null) {
    // 如果节点值比传入值小，就从右边开始找
    if (node.val < val) {
      if (node.right === null) {
        node.right = new TreeNode(val);
        break;
      }
      node = node.right;
    } else {
      if (node.left === null) {
        node.left = new TreeNode(val);
        break;
      }
      node = node.left;
    }
  }

  return root;
};
```

* 下面是 java 的实现，仅供参考。

```java
public void insert(int data) {
  if (tree == null) {
    tree = new Node(data);
    return;
  }

  Node p = tree;
  while (p != null) {
    if (data > p.data) {
      if (p.right == null) {
        p.right = new Node(data);
        return;
      }
      p = p.right;
    } else { // data < p.data
      if (p.left == null) {
        p.left = new Node(data);
        return;
      }
      p = p.left;
    }
  }
}
```

### 二叉查找树的删除

* 删除操作会比前面两种操作稍微复杂一些。区分三种情况来处理。
	* 被删除的节点没有子节点，那么**只要把这个节点的父节点指向它的指针直接设置为 null **即可。比如删除上图中的节点 **55**，直接删除即可。
	* 被删除的节点只有一个子节点，左子节点或者右子节点，那就需要 **更新这个删除节点的父节点指向它的指针，更新为它的唯一子节点** 即可。例如删除上图中的 **13**.
	* 被删除的节点有多个子节点，**就需要先找到这个节点的右子树中的最小节点，把这个节点替换到要删除的节点上，再删除这个最小节点**。因为最小节点肯定没有左子节点，不然就不是最小节点了。利用右边子节点肯定比左边大的特性，替换到当前节点之后，左子节点肯定也比当前这个右子树中的最小节点更小。例如删除图中的 **18**.
		* 注意，从右子树中最小的值找出来放到被删除的节点的位置后，还需要把当前左子树的指针指向原先左子树的位置。
* 除此之外，如果被删除的节点是根节点，同样也要特殊处理。
	* 删除根节点的时候，从左子树中找到最大的节点或者右子树中最小的节点，替换到当前节点，其他关系不变即可。

![[二叉查找树的删除.png]]

#### 二叉查找树的删除实现

* 二叉查找树的删除的 ts 实现。对应 [LeetCode-450](https://leetcode-cn.com/problems/delete-node-in-a-bst/)
	* 下面是基于递归的方式。
	* 循环的方式暂未实现，实际上使用循环来实现会相对复杂，具体可以参照本题的题解。

```typescript
/**
 * @description: 递归的实现，比较取巧
 * @param {TreeNode|null} root
 * @param {number} key
 * @return {TreeNode|null}
 */
function deleteNode(root: TreeNode | null, key: number): TreeNode | null {
  if (root === null) return null;
  // 如果小于树的当前节点，从左子树中找
  if (root.val > key) {
    root.left = deleteNode(root.left, key);
    // 反之从右子树中找
  } else if (root.val < key) {
    root.right = deleteNode(root.right, key);
  } else {
    // 如果只有一个子节点或者没有子节点，直接赋值到当前节点即可
    if (root.left === null || root.right === null) {
      root = root.left === null ? root.right : root.left;
    } else {
      let node = root.right;
      // 从右子树中找到最小的值作为当前被删除节点的替代值
      while (node.left !== null) {
        node = node.left;
      }
      // 覆盖当前的值
      root.val = node.val;
      // 再把右子树中最小的值所对应的节点删除（因为已经移到当前位置）
      root.right = deleteNode(root.right, node.val);
    }
  }

  return root;
};
```

* 删除操作的 java 实现，仅供参考。

```java
public void delete(int data) {
  Node p = tree; // p指向要删除的节点，初始化指向根节点
  Node pp = null; // pp记录的是p的父节点
  while (p != null && p.data != data) {
    pp = p;
    if (data > p.data) p = p.right;
    else p = p.left;
  }
  if (p == null) return; // 没有找到

  // 要删除的节点有两个子节点
  if (p.left != null && p.right != null) { // 查找右子树中最小节点
    Node minP = p.right;
    Node minPP = p; // minPP表示minP的父节点
    while (minP.left != null) {
      minPP = minP;
      minP = minP.left;
    }
    p.data = minP.data; // 将minP的数据替换到p中
    p = minP; // 下面就变成了删除minP了
    pp = minPP;
  }

  // 删除节点是叶子节点或者仅有一个子节点
  Node child; // p的子节点
  if (p.left != null) child = p.left;
  else if (p.right != null) child = p.right;
  else child = null;

  if (pp == null) tree = child; // 删除的是根节点
  else if (pp.left == p) pp.left = child;
  else pp.right = child;
}
```

* **关于删除操作，比较取巧的方式是直接标记要删除的节点，使用一个标志位，标记为“已删除”，但是并不从树中真正地去掉。**
* 这样原来的节点还存储着，会浪费内存空间，但是删除操作就变得简单很多。可以理解为是空间换时间的思路。
* 而且这个方法也没有增加插入、查找操作代码的实现难度。

## 二叉查找树的其他操作

* 二叉查找树支持快速地查找最大节点和最小节点，还有前驱节点和后继节点。这些可以尝试着自己实现。
* 二叉查找树的特性中，还包括中序遍历二叉查找树。中序遍历可以输出有序的数据序列，时间复杂度是 `O(n)`, 非常高效。因此*二叉查找树也叫二叉排序树。*

### 支持重复数据的二叉查找树

* 前面熟悉二叉树的过程中，所有的例子默认把节点值都当成是数字。而在实际应用中，节点存储的更多是对象，对象会包含很多字段内容。我们使用对象的某个字段作为**键值(key)** 来构建二叉查找树。对象中的其他字段叫做**卫星数据**。
* 前面的分析中没有考虑过存储值相同的场景，如果值相同该如何处理呢？

#### 重复数据的二叉查找树的处理方式

* 这里有两种处理方式。
* 第一种比较容易。在每一个节点存储不止一个数据，使用链表或者支持动态扩容的数组等数据结构，把值相同的数据都存储在同一个节点上。
* 第二种复杂一些，不过更为优雅。每个节点仍然只存储一个节点值，在查找插入的过程中，如果遇到相同的值，就把插入数据放在这个值相同的节点的右子树，也就是把新插入的值当成是大于当前值的情况来处理。

![[二叉查找树的相同值插入场景.png]]

* 当查找数据时，遇到值相同的节点，也继续在右子树中查找，直到遇到叶子节点才停止。这样可以把所有和查找值相同的节点都找出来。

![[二叉查找树的相同值的查找场景.png]]

* 删除操作也是类似的，查找每个相同值的节点，依次进行删除。

![[二叉查找树的相同值的删除场景.png]]

## 二叉查找树的时间复杂度

* 二叉查找树的形态也是各式各样的，不同形态下的时间复杂度会相差很多。
* 下面是同一组数据的三种不同形态。第一种二叉查找树实际上已经退化成了链表，时间复杂度度就是 `O(n)`. 这是最糟糕的情况。

![[二叉查找树的不同形态.png]]

* 理想情况下，**二叉树查找树是完全二叉树（或满二叉树）。**
* 在这种情况下(完全二叉树)，它的时间复杂度是多少呢？

### 完全二叉树的时间复杂度

* 结合前面的所有分析可以发现，任何操作的时间复杂度其实都和树的高度成正比，也就是 `O(height)`. 这样就把问题转换为如何获取一颗节点数为 n 的完全二叉树的高度？
* 树的高度等于最大层数减一，为了方便计算，我们把高度转换成层来表示。从前面的图中可以发现，n 个节点的完全二叉树中，第一层是 1 个节点，第二层是 2 个节点，第三层是 4 个节点，第四层是 `2^(4 - 1)`, 第 k 层就是 `2^(k - 1)`. 除了最后一层， 每一层的个数都是前面一层的 2 倍。
	* 最后一层的个数在 1 到 `2^(k - 1)` 之间。
	* 换算成公式就是下面这样。

```typescript
n >= 1 + 2 + 4 + 8 + ...2^(k - 2) + 1
n <= 1 + 2 + 4 + 8 + ...2^(k - 2) + 2^(k - 1)
```

* 借助等比数列的求和公式，计算得到层数 L 的范围是 `[log2(n+1), log2n+1]`. 完全二叉树的层数小于等于 `log2n+1`，也就是完全二叉树的高度小于等于 `log2n`.
	* **时间复杂度可以理解为 `O(logn)`.**
* 极度不平衡的二叉树的性能是糟糕的，不能满足实际需求。我们需要的是平衡的二叉树，能够保证不管怎么删除和插入数据，都尽可能保持任意节点左右子树的平衡。这是下一节要分析的一种特殊的二叉查找树，平衡二叉查找树。
	* 平衡二叉查找树的高度接近 `logn`, 所以插入和查找和删除操作的时间复杂度也比较稳定，是 `O(logn)`.

## 平衡二叉查找树和散列表的对比

* 从时间复杂度上来看，二叉查找树相对散列表的优势并不明显，那为何还会在实际应用中使用二叉查找树呢？
* 主要有以下几种原因（*下面分析的二叉查找树主要是指较为平衡的二叉查找树*）：
	* 散列表中的数据是无序存储的，如果要输出有序数据，需要先进行排序。而二叉查找树只要进行**中序遍历**，就能在 `O(logn)` 复杂度内输出有序的数据序列。
	* 散列表扩容比较耗时，而且遇到散列冲突时，性能不稳定。普通的二叉查找树不稳定，但是实际中最常使用的平衡二叉查找树的性能是稳定的，时间复杂度稳定在 `O(logn)`.
	* 虽然从时间复杂度上看，一般情况下 `O(1)` 的常量级会比 `O(logn)` 小，但前面也分析过，这是不一定的。尤其当散列表出现散列冲突的时候，外加上散列函数执行的耗时，整体执行效率不一定比平衡二叉查找树的效率更高。
	* 散列表的构造比二叉查找树复杂，需要考虑多方面的内容。例如散列函数的设计，冲突解决办法，扩容和缩容等。平衡二叉查找树只需要考虑平衡性这一个问题。而这个问题的解决方案已经比较成熟。
	* 为了避免过多的散列冲突，散列表的装载因子不能太大，特别是基于开放寻址法解决冲突的散列表，也就说明它的内存空间利用率相对不是那么高。
* 综上，平衡二叉查找树在某些方面的效果会比散列表更好。至于在实际开发中如何使用，需要结合看具体的场景。

## 小结

* 这节主要关注一种特殊的二叉树，二叉查找树。它能支持快速地查找、插入和删除操作。
* 二叉查找树中，每个节点的值都大于左子树节点的值，小于右子树节点的值。这是针对没有重复数据的情况。
* 对于存在重复数据的二叉查找树，有两种构建方法：
	* 一种是让每个节点存储多个值相同的数据；
	* 另一种是，每个节点中存储一个数据。针对这种情况，我们只需要稍加改造原来的插入、删除、查找操作即可。
* 在二叉查找树中，查找、插入、删除等很多操作的时间复杂度都跟树的高度成正比。两个极端情况的时间复杂度分别是 `O(n)` 和 `O(logn)`，分别对应二叉树退化成链表的情况和完全二叉树。
* 为了避免时间复杂度的退化，针对二叉查找树，历史的发展中又设计了一种更加复杂的树，平衡二叉查找树，时间复杂度可以做到稳定的 `O(logn)`，这种树下一节具体来讲。

## 扩展

### 获取二叉树的高度/深度

* 上面我们基于理论分析二叉树的高度，下面可以试着自己实现求出一颗给定二叉树的高度的方法？
	* 与本题 [LeetCode-102](https://leetcode.cn/problems/maximum-depth-of-binary-tree/) 计算二叉树的深度的思路是一样的。
	* 下面的题解使用递归实现，使用循环实现也是类似。

```typescript
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     val: number
 *     left: TreeNode | null
 *     right: TreeNode | null
 *     constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.left = (left===undefined ? null : left)
 *         this.right = (right===undefined ? null : right)
 *     }
 * }
 */

/**
 * @description: 递归获取二叉树的最大深度（也即高度）
 * @param {TreeNode} root
 * @return {number}
 */
function maxDepth(root: TreeNode | null): number {
  if (root === null) return 0;

  let maxDepth = 0;
  
  /**
   * @description: 递归方法
   * @param {TreeNode} node
   * @param {number} depth
   * @return {void}
   */
  function dfs(node: TreeNode, depth: number): void {
    // 如果已经遍历到叶子节点，就判断并保存当前的深度
    if (!node.left && !node.right) {
      maxDepth = depth > maxDepth ? depth : maxDepth;
      return;
    }
    node.left && dfs(node.left, depth + 1);
    node.right && dfs(node.right, depth + 1);
  }

  // 递归遍历
  dfs(root, 1);
  return maxDepth;
};
```

#ALG #ADT 
