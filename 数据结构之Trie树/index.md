# Trie

## 简言

* 搜索引擎的搜索关键词功能，底层使用的是什么数据结构呢？
	* 答案就是 Trie。

> 学习曲线：★★★☆

## 什么是 Trie

* **Trie，也叫字典树或者前缀树**。**它是树形结构，一种专门处理字符串匹配的数据结构，用于解决在一组字符串集合中快速查找某个字符串的问题。**
* 解决快速查找字符串的问题，其实很多数据结构都能做到，但是 Trie 有着自己独特的优势，后文会提到。我们一步步来分析。
* 要了解 Trie 的结构，我们可以先举个例子，有这样六个字符串：`how`, `hi`, `her`, `hello`, `so`, `see`. 我们需要在里面多次查找某个字符串是否存在。如果每次查找都是对所有字符串进行匹配，那么效率必然会比较低，显然可以有更好的方式。
	* 更好的方式就是对这 6 个字符串先做一次预处理，组织成 Trie 树的结构，之后每次查找，都是在树中进行匹配查找。
* **Trie 的本质，就是利用字符串之间的公共前缀，把重复的前缀合并在一起。**
	* 如下所示：

![[Trie树示意图.png]]

* 其中，**根节点不包含任何信息，每个节点表示一个字符串中的字符，从根节点到红色节点的一条路径表示一个字符串**。（*红色节点并不都是叶子节点*）
* 下面是 Trie 构造的分解过程，构造过程的每一步，其实都是往 Trie 中插入一个字符，当所有字符都插入之后，Trie 就构造完成。

![[构造Trie树示意图1.png]]
![[构造Trie树示意图2.png]]

* 假如在 Trie 树中查找一个字符串 `her`，我们就把这个字符串拆成 `h`、`e`、`r` 三个字符，再从 Trie 树的根节点开始匹配。
	* 如图所示，绿色路径就是在 Trie 树中匹配的路径。

![[Trie匹配示意图.png]]

## 如何实现 Trie

### Trie 的实现原理

* 从 Trie 树的介绍中可以发现，它主要有两个操作，**一是把字符串集合构造成 Trie 树，也就是把字符串一个个插入 Trie 树的过程；另一则是在 Trie 树中查询一个字符串。**
* 了解它的操作之后，我们来看下 Trie 树如何存储。
* 由上面的几张图可以发现，Trie 树是一个多叉树。我们知道二叉树的节点是有左右指针指向两个子节点来存储的。
	* 二叉树的结构可以查看过往的笔记记录。[[数据结构之二叉树基础#二叉树的结构]]
	* 而多叉树的存储则有所不同。
* 它的存储方式有多种，这里介绍一个比较经典的存储方式，**就是应用散列表的思想，通过下标与字符一一映射的数组，来存储子节点的指针。**

![[存储Trie树.png]]

* 假设这里的字符串只有 a 到 z 这 26 个小写字母，我们在数组中下标为 0 的位置，存储指向子节点 a 的指针，下标为 1 的位置存储指向子节点 b 的指针，依次类推，下标 25 的位置，存储的值指向子节点 z 的指针。如果某个字符的子节点不存在，那么对应的下标位置存储的是 `null`。
* 下面是 Java 示意：

```java
class TrieNode {
  char data;
  TrieNode children[26];
}
```

* 在 Trie 中查找字符串的时候，就可以通过字符的 ASCII 码减去 a 的 ASCII 码，迅速找到匹配的子节点的指针。例如，d 的 ASCII 码减去 a 的 ASCII 码就是 3，那子节点 d 的指针就存储在数组中下标为 3 的位置中。
* 在了解基本原理后，就可以实现一个简易的 Trie。

### Trie 的实现代码

#### JAVA 实现

* 下面是 Java 的完整实现。

```java
public class Trie {
  private TrieNode root = new TrieNode('/'); // 存储无意义字符

  // 往Trie树中插入一个字符串
  public void insert(char[] text) {
    TrieNode p = root;
    for (int i = 0; i < text.length; ++i) {
      int index = text[i] - 'a';
      if (p.children[index] == null) {
        TrieNode newNode = new TrieNode(text[i]);
        p.children[index] = newNode;
      }
      p = p.children[index];
    }
    p.isEndingChar = true;
  }

  // 在Trie树中查找一个字符串
  public boolean find(char[] pattern) {
    TrieNode p = root;
    for (int i = 0; i < pattern.length; ++i) {
      int index = pattern[i] - 'a';
      if (p.children[index] == null) {
        return false; // 不存在pattern
      }
      p = p.children[index];
    }
    if (p.isEndingChar == false) return false; // 不能完全匹配，只是前缀
    else return true; // 找到pattern
  }

  public class TrieNode {
    public char data;
    public TrieNode[] children = new TrieNode[26];
    public boolean isEndingChar = false;
    public TrieNode(char data) {
      this.data = data;
    }
  }
}
```

#### TS 实现

* 下面是 TS 的完整实现。
	* 这里使用数组存储字典，是比较耗费内存空间的实现方式，只是和上文的实现比较相似。因此放在这里。
	* 在 JS 中，其实更适合使用对象来存储，也就是使用对象(*本质是散列表*)来当做字典实现前缀树。具体实现可以查看[这里](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Trie/index.ts)。

```typescript
class ITrieNode<T> {
  data: T;
  children: ITrieNode<T>[];
  isEndingChar: boolean;
}

class TrieNodeByArray<T> implements ITrieNode<T> {
  public data: T;
  public children: ITrieNode<T>[] = new Array(26).fill(null);
  public isEndingChar = false;

  constructor(data: T) {
    this.data = data;
    this.children = [];
  }
}

/**
 * @description: 使用数组存储字典的前缀树
 * 比较耗费内存空间
 * @return {*}
 */
export default class TrieByArray {
  private root: ITrieNode<string> = new TrieNodeByArray('/');
  private charAtA: number = 'a'.charCodeAt(0);

  /**
   * @description: 插入一个字符
   * @param {string} text
   * @return {void}
   */
  public insert(text: string): void {
    let currentTrieNode: ITrieNode<string> = this.root;
    for (const str of text) {
      const index = str.charCodeAt(0) - this.charAtA;
      if (!currentTrieNode.children[index]) {
        currentTrieNode.children[index] = new TrieNodeByArray(str);
      }
      currentTrieNode = currentTrieNode.children[index];
    }
    currentTrieNode.isEndingChar = true;
  }

  /**
   * @description: 在 Trie 树中查找一个字符串
   * @param {string} pattern
   * @return {boolean}
   */
  public search(pattern: string): boolean {
    let currentTrieNode: ITrieNode<string> = this.root;

    for (const str of pattern) {
      const index = str.charCodeAt(0) - this.charAtA;
      if (!currentTrieNode.children[index]) return false;
      currentTrieNode = currentTrieNode.children[index];
    }
    if (currentTrieNode.isEndingChar === false) return false;
    return true;
  }

  /**
   * @description: 在 Trie 树中查找一个前缀字符串
   * @param {string} prefix
   * @return {boolean}
   */
  public startsWith(prefix: string): boolean {
    let currentTrieNode: ITrieNode<string> = this.root;

    for (const str of prefix) {
      const index = str.charCodeAt(0) - this.charAtA;
      if (!currentTrieNode.children[index]) return false;
      currentTrieNode = currentTrieNode.children[index];
    }
    return true;
  }
}

const trieByArray = new TrieByArray();
console.log(trieByArray.insert('apple')); // void
console.log(trieByArray.search('apple')); // 返回 true
console.log(trieByArray.search('app')); // 返回 false
console.log(trieByArray.startsWith('app')); // 返回 true
console.log(trieByArray.insert('app')); // void
console.log(trieByArray.search('app')); // 返回 true
```

* 在 [LeetCode-208](https://leetcode-cn.com/problems/implement-trie-prefix-tree/) 上也有对应题目。

## Trie 查找字符串的时间复杂度

* Trie 查询的高效体现在频繁地查找某些字符串。
* 构建 Trie 的过程需要扫描所有字符串，时间复杂度是 `O(n)`, **n 是所有字符串的长度和。**
* 构建成功后，如果查询的字符串长度是 k，那么只需要对比 k 个节点就可以完成查询操作。和 Trie 树的大小没有关系。因此构建好 Trie 后，查找字符串的时间复杂度是 `O(k)`, **k 表示要查找的字符串的长度。**

## Trie 的内存消耗

* 上文提到，比较经典的存储 Trie 树的方式是用数组来存储一个节点的子节点的指针。如果字符串包含 a 到 z 这 26 个字母，那么每个节点都要存储一个长度为 26 的数组，并且每个数组元素要存储一个 8 字节的指针（也可能是 4 字节，和操作系统等条件有关）。因此，即使这个节点并没有用到所有的 26 个字母，但也需要维护一个能够存储 26 个元素的，长度为 26 的数组。这部分的空间消耗是不小的。
	* 换算一下，每个元素是 8 字节，数组长度 26，每个节点就会额外需要 `26 * 8 = 208` 个字节。
	* 这是 26 个字符的情况，而实际的场景中，显然会有更多字符的情况出现，例如大写字母，各种标点符号以及中文等等。这种情况下，所耗费的内存也就会更多。这时的 Trie 树就并不太能起到节省内存的作用。
* 为了解决耗费内存的问题，我们可以稍微牺牲部分查询的效率，把每个节点中的数组换成其他数据结构，来存储当前节点的子节点指针。这里的选择也就很多了，包括有序数组、跳表、散列表、红黑树等等都可以。
* 假设我们使用有序数组，数组中的指针按照所指向的子节点中的字符的大小顺序排列。
	* 查询时，我们就使用二分查找的方式，快速查找到某个字符应该匹配的子节点的指针。
	* 当然，在插入一个字符的时候，也会因为要维持数组中数据的有序性，会稍微慢一些。
* 为了解决内存问题，Trie 也发展出很多变体。例如有一种方式叫作**缩点优化**，对只有一个子节点的节点，这个节点不是一个串的结束节点，那么就可以把当前节点和子节点进行合并。
	* 这样可以节省空间，不过也会增加编码难度。这就需要做出取舍。
	* 效果如下图所示：

![[缩点优化示意图.png]]

## Trie 与散列表、红黑树的比较

* 字符串的匹配问题，其实就是数据的查找问题，对于支持动态数据高效操作的数据结构，前面我们也已经学过一些，例如散列表，红黑树，跳表等等。这些数据结构其实都可以实现字符串查找的功能。
* 我们选取散列表和红黑树，与 Trie 树对比一下，列出它们各自的优缺点和应用场景。
* Trie 树对于要处理的字符串有着非常严格的要求：
	1. 字符串中包含的字符集不能太大。
		* 如果太大，存储空间就可能会浪费很多。即使可以优化，也需要付出牺牲查询和插入的效率的代价。
	2. 要求**字符串的前缀重合比较多**，否则空间消耗会很大。
	3. 如果要用 Trie 树来解决问题，需要从 0 开始实现，它并没有现成的数据结构。
		* 在工程上，这是把简单问题复杂化，并不太可取。
	4. Trie 树中使用到了指针。而通过指针串起来的数据块是不连续的，对缓存并不友好，性能相对也会差一些。

### Trie 的使用场景

* 综合上述四点，其实在实际开发中，我们更倾向于使用散列表或者红黑树来实现字符串中查找字符的问题。
* 因此，我们有必要确认这些数据结构各自的应用场景。
	* **精确匹配查找，适合使用散列表或者红黑树来解决。**
	* **Trie 树适合查找前缀匹配的字符串，但不适合精确匹配查找。**

## Trie 的应用

### 如何使用 Trie，实现搜索关键词提示的功能？

* 假设关键词库由用户的热门搜索关键词组成。我们把这个词库构建成一个 Trie。当用户输入某个单词的时候，把这个单词作为一个前缀子串在 Trie 树中进行匹配。
* 当用户输入 h 的时候，我们就把 `hello`, `her`, `hi`, `how` 展示在搜索提示框内。当用户继续键入 e 的时候，我们就把 he 为前缀的 `hello` 和 `her` 展示在搜索提示框内。
* 这就是搜索关键词提示的最基本的算法原理。

![[搜索关键词示意图.png]]

* 这是最基本的原理。搜索引擎的关键词提示功能会更加复杂，会基于上面的逻辑进行更多的改造。
* 在现有的逻辑下，我们显然不能处理以下的这些问题：
	* *针对含有中文的词库，数据该如何构建成 Trie 呢？*（个人猜测是 Unicode 符或者拼音或者 ASCII 码）
	* *如果词库中有很多关键词，搜索提示的时候，用户输入关键词，作为前缀在 Trie 树中可以匹配的关键词也有很多，如何来决定展示的顺序和内容呢？*
	* *Google 搜索引擎可以在单词拼写错误的情况下，仍然显示正确的拼写来做关键词提示，这是如何实现的？*
	* 这些问题可以先思考，解决方案留待日后更新。我们会在后续的章节中学习到。
* Trie 的实际应用可以扩展到更加广泛的场景中，例如自动输入补全，包括输入法自动补全功能，IDE 代码编辑器的自动补全功能，浏览器网址输入的自动补全功能等等。

## 小结

* Trie 是一种解决字符串快速匹配问题的数据结构。
	* 如果用来构建 Trie 的字符串中，前缀重复的情况不是很多，那么这种数据结构总体上是比较耗费内存的，其实就是空间换时间的解决思路。
* 在对内存不敏感或内存消耗在可控范围内的情况下，Trie 中做字符串匹配是非常高效的。时间复杂度是 `O(k)`, k 表示要匹配的字符串的长度。
* Trie 的优势就是查找前缀匹配的字符串，例如搜索引擎中的关键词提示功能，就非常适合用它来解决。**而精确匹配查找等动态集合数据的查找，则更适合使用散列表和红黑树来实现。**

## 扩展

* *给到一个很大的字符串集合，例如包含一万条记录，如何通过编程量化分析这组字符串集合是否适合使用 Trie 解决呢？也就是如何统计字符串的字符集大小以及前缀重合的程度呢？*
	* 💭

#ALG #ADT 