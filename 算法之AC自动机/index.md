# AC自动机

## 简言

* 基本上所有中文互联网社区网站，都会有敏感词过滤功能，这个功能会自动过滤掉一些淫秽、反动和谩骂等内容。
* 这个功能的核心原理就是字符串匹配算法：通过维护一个敏感词的字典，当用户输入一段文字内容后，通过字符串匹配算法，查找用户输入的这段文字是否包含敏感词。如果有，就用 `'***'` 来替换掉。
* 前面学到的几种字符串算法，其实也可以处理这类问题。但是，对于访问量十分巨大的网站而言，性能是非常重要的，例如淘宝或微博，每天评论的数量有几亿甚至几十亿。在这种场景下，性能显得尤为重要。我们当然不可能让用户在输入文字后，几秒或几十秒后才能够查看输入结果。
* 那么对于这种需要一个高性能的敏感词过滤系统的场景，什么算法可以做到呢？
	* 答案就是**多模式串匹配算法**。

> 学习曲线：★★★★☆


## 基于单模式串和Trie树实现的敏感词过滤

* 前面的章节里面，我们学习了不少字符串匹配算法。如 BF 算法，RK 算法，BM 算法，KMP 算法还有 Trie。前面的四种算法都是单模式串匹配算法，只有 **Trie 是多模式串匹配算法**。
	* **单模式串匹配算法，就是在一个模式串和一个主串之间进行匹配。也就是在一个主串中查找一个模式串。**
	* **多模式串匹配算法，就是在多个模式串和一个主串间做匹配。也就是在一个主串中查找多个模式串。**
* 本质上，单模式串也可以完成多模式串的匹配工作。例如前文提到的敏感词过滤系统，但是如果这样实现的话，每次匹配都需要扫描一遍用户输入的所有内容。整个过程就需要多次扫描用户输入的内容。如果敏感词较多，而且用户输入的内容较长，有上千个字符，那就需要扫描上千遍这样的内容。显然，这种方案是比较低效的。
	* 注意，当我们理解这些匹配过程时，**用户输入的字符串是主串，敏感词字符串才是模式串。**
* 与单模式串匹配相比，多模式串匹配的处理就显得非常高效。**它只需要扫描一遍主串**，就可以在主串中一次性查找多个模式串是否存在，从而极大地提高匹配效率。其实，Trie 树就是一种多模式串匹配算法。
	* 具体的做法是，*对敏感词字典进行预处理，将其构建成 Trie 树结构。这个预处理只需要做一次，后续敏感词字典动态更新，也只需要动态更新 Trie 树即可。*
	* 当用户输入文本内容后，我们把用户输入的内容作为主串，从第一个字符（例如 C）开始在 Trie 树中匹配。当匹配到 Trie 树的叶子节点或者中途遇到不匹配字符的时候，把主串的开始匹配位置后移一位，也就是 C 的下一个字符开始，重新在 Trie 树中匹配。
* 基于 Trie 树的这种处理方式，有些类似于单模式串匹配算法中的 BF 算法。而单模式串匹配算法里，KMP 算法对 BF 算法进行了改进，引入了 next 数组，让字符匹配失败时，模式串可以向后多滑动几位，从而提高匹配效率。我们也可以借鉴 KMP 算法的改进思路，对多模式串 Trie 树进行改造，从而提高 Trie 树的匹配效率。这就需要提到 AC 自动机算法。

## 经典的多模式串匹配算法：AC 自动机

* AC 自动机，全称是 `Aho-Corasick` 算法。
	* Trie 和 AC 自动机之间的关系，类似于单模式串中朴素的串匹配算法和 KMP 算法之间的关系。只是前者针对的是多模式串而已。
	* AC 自动机本质上就是在 Trie 之上，加了 KMP 里的 next 数组，而这个 next 数组，就是构建在树上的。
	* 下面是用 Java 代码表示的 AC 自动机中的节点结构。注意观察里面的各个变量的含义。

```java
public class AcNode {
  public char data; 
  public AcNode[] children = new AcNode[26]; // 字符集只包含a~z这26个字符
  public boolean isEndingChar = false; // 结尾字符为true
  public int length = -1; // 当isEndingChar=true时，记录模式串长度
  public AcNode fail; // 失败指针
  public AcNode(char data) {
    this.data = data;
  }
}
```

### 构建 AC 自动机的过程

* AC 自动机的构建，包含两个主要操作：
	* **把多个模式串构建成 Trie 树。**
	* **在 Trie 树上构建失败指针（相当于 KMP 中的失效函数也就是 next 数组）。**
* 构建 Trie 树就是上一节的内容。
* 这里主要展开的是构建失败指针的过程。

#### 构建失败指针的过程

* 假设有四个模式串: `c`, `bc`, `bcd`, `abcd` 和主串 `abcd`。
* 下图就是基于上述字符串构建好的 Trie 树。

![[基于多模式串构建的Trie树.png]]

* **Trie 树中的每个节点都有一个失败指针，它的作用和使用方式和 KMP 算法中的 next 数组极其相似。** 因此，要理解好这一部分，必须要先理解好 KMP 算法的章节。
* 我们来走一遍整个流程。
	* 假设沿着 Trie 树走到 p 节点，就是下图中的紫色节点。那么 p 的失败指针就是从 root 走到紫色节点而形成的字符串 `abc`，和所有模式串前缀匹配的最长可匹配后缀子串，就是箭头所指的 `bc` 模式串。
		* 这里的最长可匹配后缀子串，就是最长的、可匹配后缀子串。而可匹配后缀子串，就是把字符串 `abc` 的后缀子串 `bc` 和 `c` 这两个子串和其他的模式串进行匹配。**如果某个后缀子串可以匹配某个模式串的前缀，那这个后缀子串就可以称作可匹配后缀子串。**
		* 从中找出最长的一个，就是最长可匹配后缀子串。
		* 个人理解，**其实就是找出最长的好后缀，在此基础上的后一位就对应着 AC 自动机里的失败指针指向的下一位。**
	* 我们把 p 节点的失败指针指向那个最长匹配后缀子串对应的模式串的前缀的最后一个节点，就是下图中箭头所指向的节点。

![[AC自动机首个失败指针示意图.png]]

* 计算每个节点的失败指针的过程看起来有些复杂，但其实如果把树中所有相同深度的节点放到同一层，那某个节点的失败指针只可能出现在它所在层的上一层。
* 可以参照 KMP 算法，求某个节点的失败指针的时候，通过已经求得的，深度更小的那些节点的失败指针来推导。
	* 也就是可以逐层依次来求解每个节点的失败指针。
	* **失败指针的构建过程，实际是一个按层遍历树的过程，也就是层序遍历，对应到代码实现就是广度遍历。**
* 首先，root 的失败指针是 Null，也就是节点本身。那么问题求解的关键，就是*在已经求得某个节点 p 的失败指针之后，如何求得它的子节点的失败指针？*
	* 假设节点 p 的失败指针指向节点 q，我们观察节点 p 的子节点 pc 所对应的字符 `cd` 是否也可以在 q 的子节点中找到，如果找到了节点 q 的一个子节点 qc 也就是 `cd` 字符串正好与其相同，则可以把节点 pc 的失败指针指向节点 qc。

![[AC自动机求解失败指针过程图1.png]]

* 如果节点 q 中没有子节点的字符恰好等于节点 pc 包含的字符，则让 `q = q -> fail`(*fail 表示失败指针*)，继续查找，直到 q 为 root 为止。如果还没有找到相同字符的子节点，就让节点 `pc` 的失败指针指向 root。
	* *这个过程其实就非常像是 KMP 算法里求解 next 的过程。*

![[AC自动机求解失败指针过程图2.png]]

#### 构建失败指针的代码

##### Java 实现

* 下面就是构建失败指针的 Java 代码，不包含构建 Trie 树的代码，因为上一节已经实现。

```java
public void buildFailurePointer() {
  Queue<AcNode> queue = new LinkedList<>();
  root.fail = null;
  queue.add(root);
  while (!queue.isEmpty()) {
    AcNode p = queue.remove();
    for (int i = 0; i < 26; ++i) {
      AcNode pc = p.children[i];
      if (pc == null) continue;
      if (p == root) {
        pc.fail = root;
      } else {
        AcNode q = p.fail;
        while (q != null) {
          AcNode qc = q.children[pc.data - 'a'];
          if (qc != null) {
            pc.fail = qc;
            break;
          }
          q = q.fail;
        }
        if (q == null) {
          pc.fail = root;
        }
      }
      queue.add(pc);
    }
  }
}
```

##### TS 实现

* TS 实现。包括 AC 自动机节点的实现。

```typescript
/**
 * @description: AC 自动机中的节点
 */
/* AcNode is a class that represents a node in a trie */
class AcNode {
  public data: string;
  public children: AcNode = new Array(26).fill(null);
  public isEndingChar = false;
  public pattern = ''; // 当isEndingChar=true时，记录模式串长度
  // public length = -1; // 当isEndingChar=true时，记录模式串长度
  public AcNode: null;
  public fail: AcNode|null;
  constructor(data) {
    this.data = data;
  }
}

export default class AcTree {
  public root: AcNode = new AcNode('/');

  /**
   * @description: 构建失败指针
   * @return {void}
   */
  buildFailurePointer(): void {
    const queue = [];
    this.root.fail = null;
    queue.push(this.root);

    // !广度优先遍历，也是按层遍历
    while (queue.length) {
      const currentStr = queue.shift();
      // 假设只有 26 个字母
      for (let i = 0; i < 26; i++) {
        const pc = currentStr.children[i];
        if (pc === null) continue;
        // 如果父节点就是根节点，那么所有子节点的失败指针都会指向根节点
        if (currentStr === this.root) {
          pc.fail = this.root;
        } else {
          // 如果父节点不是根节点，先取得当前节点的失败指针指向的节点
          let q = currentStr.fail;
          while (q !== null) {
            // 由失败指针指向的节点起始，从上往下找到当前字符对应的子节点
            const index = pc.data.charCodeAt(0) - 'a'.charCodeAt(0);
            const qc = q.children[index];
            if (qc !== null) {
              // 如果该节点存在字符，就设置为当前节点字符的失败指针指向的节点
              pc.fail = qc;
              break;
            }
            // 如果子节点不存在字符，就继续沿着失败指针指向的节点查找下去
            q = q.fail;
          }
          // 如果所有遍历完成后还是没有找到节点，那么就把失败指针指向根节点
          if (q === null) {
            pc.fail = this.root;
          }
        }
        // 在队列中加入下一层节点
        queue.push(pc);
      }
    }
  }
}
```

* 通过按层来计算每个节点的子节点的失效指针。刚才的例子中，最后构建完成的 AC 自动机就如下图。

![[AC自动机求解失败指针过程图3.png]]

* 这就是构建完成的 AC 自动机。接下来我们来一步步分析如何在 AC 自动机上匹配主串。

### 如何在 AC 自动机上匹配主串

* 我们继续使用上文中的例子。在匹配过程中，主串从 `i = 0` 开始，AC 自动机从指针 `p = root` 开始，假设模式串是 b，主串是 a。
	* 如果 p 指向的节点有一个等于 `b[i]` 的子节点是 x，就更新 p 指向 x。这时需要通过失败指针，检测一系列失败指针为结尾的路径是否是模式串。这句话可以结合代码来理解。处理完之后，把 i 加一。继续这两个过程。
	* 如果 p 指向的节点没有等于 `b[i]` 的子节点，那就可以使用失败指针，`p = p -> fail`, 再继续这两个过程。

#### 匹配主串的 Java 实现

* 下面是匹配部分的完整 Java 实现。代码输出的是在主串中每个可以匹配的模式串出现的位置。

```java
public void match(char[] text) { // text是主串
  int n = text.length;
  AcNode p = root;
  for (int i = 0; i < n; ++i) {
    int idx = text[i] - 'a';
    while (p.children[idx] == null && p != root) {
      p = p.fail; // 失败指针发挥作用的地方
    }
    p = p.children[idx];
    if (p == null) p = root; // 如果没有匹配的，从root开始重新匹配
    AcNode tmp = p;
    while (tmp != root) { // 打印出可以匹配的模式串
      if (tmp.isEndingChar == true) {
        int pos = i-tmp.length+1;
        System.out.println("匹配起始下标" + pos + "; 长度" + tmp.length);
      }
      tmp = tmp.fail;
    }
  }
}
```

#### 匹配主串的 TS 实现

```typescript
/**
 * @description: 主串匹配多模式串，并取得模式串的位置和模式串内容
 * @param {string} word
 * @return {string[]}
 */
match(word: string): string[] {
	const length = word.length;
	const uniqueWords = {}; // 用于标识唯一字符
	const matchedWords = []; // 返回所有查找到的模式串
	let p: AcNode = this.root;

	// 获取主串的所有字符进行匹配
	for (let i = 0; i < length; i++) {
		// 获取字符所对应的索引
		const index = word[i].charCodeAt(0) - 'a'.charCodeAt(0);
		// 如果没有查找到，就把当前节点更新为失败指针指向的节点
		while (p.children[index] === null && p !== this.root) {
			p = p.fail; // 更新到失败指针指向的节点
		}
		// 查找到之后继续往下走
		p = p.children[index];
		// 如果下一个字符没有匹配的，就要从根节点开始重新匹配
		if (!p) p = this.root;
		let temp: AcNode = p;
		while (temp !== this.root) {
			// 当匹配到最后一个字符的时候，获取长度计算出字符开始的位置并将其打印出来
			if (temp.isEndingChar) {
				// !这里 temp 的字符和长度是在插入时保存的
				const pos = i - temp.pattern.length + 1;
				console.info(`匹配起始下标: ${pos}; 长度为: ${temp.pattern.length}; 对应字符为: ${temp.pattern}.`);
				if (!uniqueWords[temp.pattern]) {
					uniqueWords[temp.pattern] = temp.pattern;
					matchedWords.push(temp.pattern);
				}
			}
			temp = temp.fail;
		}
	}

	return matchedWords;
}
```

### AC 自动机的完整实现

* 下面是一个使用**数组**实现的 TS 版本完整的 AC 自动机，支持构建失败指针和查找输出。
* 最后的示例直接执行后便可查看日志验证结果。

```typescript
/**
 * @description: AC 自动机中的节点
 */
/* AcNode is a class that represents a node in a trie */
class AcNode {
  public data: string;
  public children: AcNode[] = new Array(26).fill(null);
  public isEndingChar = false;
  public pattern = ''; // 当isEndingChar=true时，记录模式串长度
  // public length = -1; // 当isEndingChar=true时，记录模式串长度
  public AcNode: null;
  public fail: AcNode | null = null;
  constructor(data) {
    this.data = data;
  }
}

/**
 * @description: 使用数组实现的 AC 自动机
 * @return {*}
 */
export default class AcTrieByArray {
  public root: AcNode = new AcNode('/');

  /**
   * @description: 插入一个字符串
   * @param {string} text
   * @return {void}
   */
  public insert(text: string): void {
    let currentTrieNode: AcNode = this.root;
    // 遍历主串
    for (const str of text) {
      const index = str.charCodeAt(0) - 'a'.charCodeAt(0);
      if (!currentTrieNode.children[index]) {
        currentTrieNode.children[index] = new AcNode(str);
      }
      currentTrieNode = currentTrieNode.children[index];
    }
    currentTrieNode.isEndingChar = true;
    // 保存当前完整的字符串到最后一位中
    currentTrieNode.pattern = text;
  }

  /**
   * @description: 在 Trie 树中查找一个字符串
   * @param {string} pattern
   * @return {boolean}
   */
  public search(pattern: string): boolean {
    let currentTrieNode: AcNode = this.root;

    for (const str of pattern) {
      const index = str.charCodeAt(0) - 'a'.charCodeAt(0);
      if (!currentTrieNode.children[index]) return false;
      currentTrieNode = currentTrieNode.children[index];
    }

    return currentTrieNode.isEndingChar;
  }

  /**
   * @description: 创建 AcTrie
   * @param {string[]} patterns
   * @return {void}
   */
  public createAcTrie(patterns: string[]): void {
    for (let i = 0; i < patterns.length; i++) {
      this.insert(patterns[i]);
    }
  }

  /**
   * @description: 构建失败指针
   * @return {void}
   */
  public buildFailurePointer(): void {
    const queue = [];
    this.root.fail = null;
    queue.push(this.root);

    // !广度优先遍历，也是按层遍历
    while (queue.length) {
      const currentAcNode = queue.shift();
      // 假设只有 26 个字母
      for (let i = 0; i < 26; i++) {
        const pc: AcNode = currentAcNode.children[i];
        if (pc === null) continue;
        // 如果父节点就是根节点，那么所有子节点的失败指针都会指向根节点
        if (currentAcNode === this.root) {
          pc.fail = this.root;
        } else {
          // 如果父节点不是根节点，先取得当前节点的失败指针指向的节点
          let q = currentAcNode.fail;
          while (q !== null) {
            // 由失败指针指向的节点起始，从上往下找到当前字符对应的子节点
            const index = pc.data.charCodeAt(0) - 'a'.charCodeAt(0);
            const qc = q.children[index];
            if (qc !== null) {
              // 如果该节点存在字符，就设置为当前节点字符的失败指针指向的节点
              pc.fail = qc;
              break;
            }
            // 如果子节点不存在字符，就继续沿着失败指针指向的节点查找下去
            q = q.fail;
          }
          // 如果所有遍历完成后还是没有找到节点，那么就把失败指针指向根节点
          if (q === null) {
            pc.fail = this.root;
          }
        }
        // 在队列中加入下一层节点
        queue.push(pc);
      }
    }
  }

  /**
   * @description: 主串匹配多模式串，并取得模式串的位置和模式串内容
   * @param {string} word
   * @return {string[]}
   */
  match(word: string): string[] {
    const length = word.length;
    const uniqueWords = {}; // 用于标识唯一字符
    const matchedWords: string[] = []; // 返回所有查找到的模式串
    let p: AcNode = this.root;

    // 获取主串的所有字符进行匹配
    for (let i = 0; i < length; i++) {
      // 获取字符所对应的索引
      const index = word[i].charCodeAt(0) - 'a'.charCodeAt(0);
      // 如果没有查找到，就把当前节点更新为失败指针指向的节点
      while (p?.children[index] === null && p !== this.root) {
        p = p.fail; // 更新到失败指针指向的节点
      }
      // 查找到之后继续往下走
      p = p.children[index];
      // 如果下一个字符没有匹配的，就要从根节点开始重新匹配
      if (!p) p = this.root;

      let temp: AcNode = p;
      // 如果根据当前字符有查找到对应的字符串
      while (temp !== this.root) {
        // 当匹配到最后一个字符的时候，获取长度计算出字符开始的位置并将其打印出来
        if (temp?.isEndingChar) {
          // !这里 temp 的字符和长度是在插入时保存的
          const pos = i - temp.pattern.length + 1;
          console.info(`匹配起始下标: ${pos}; 长度为: ${temp.pattern.length}; 对应字符为: ${temp.pattern}.`);
          if (!uniqueWords[temp.pattern]) {
            uniqueWords[temp.pattern] = temp.pattern;
            matchedWords.push(temp.pattern);
          }
        }
        // 重置指向因为可能存在有一个字符是两个字符串的结尾字符的情况
        temp = temp.fail;
      }
    }

    return matchedWords;
  }
}

const acTrieByArray = new AcTrieByArray();
acTrieByArray.createAcTrie(['she', 'shr', 'say', 'he', 'her', 'has']);
acTrieByArray.buildFailurePointer();
console.log(acTrieByArray.match('one day she says her has eaten many shrimps'));
// 输出内容如下
// 匹配起始下标: 8; 长度为: 3; 对应字符为: she.
// 匹配起始下标: 9; 长度为: 2; 对应字符为: he.
// 匹配起始下标: 12; 长度为: 3; 对应字符为: say.
// 匹配起始下标: 16; 长度为: 2; 对应字符为: he.
// 匹配起始下标: 16; 长度为: 3; 对应字符为: her.
// 匹配起始下标: 20; 长度为: 3; 对应字符为: has.
// 匹配起始下标: 35; 长度为: 3; 对应字符为: shr.
// [ 'she', 'he', 'say', 'her', 'has', 'shr' ]
```

* 更多的代码实现可以查看[AcTrie](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Trie/AcTrie.ts), 里面有**对象**版本的实现，更具普适性。
* 综合上文的代码，其实就是一个敏感词过滤的原型代码，它可以找到所有敏感词出现的位置（在用户输入的文本中的起始下标）。稍加改造，再遍历一遍主串，就可以把文本中所有敏感词替换成 `'***'`.

## 对比 AC 自动机和单模式串匹配

* 下面来聊一聊 AC 自动机实现的敏感词过滤系统，是否比单模式串匹配方法更加高效呢？
* 首先，把敏感词构建成 AC 自动机，包括构建 Trie 树和构建失败指针两个步骤。
	* 上节提到，Trie 树构建的时间复杂度是 `O(m*len)`. len 是敏感词的平均长度，m 表示敏感词的个数。
	* 那么构建失败指针的时间复杂度是多少呢？
	* 下面是时间复杂度的分析，并不十分精准，但可以估算时间复杂度的上限。

#### 构建失败指针的时间复杂度分析

* 假设 Trie 树中总的节点个数是 k，每个节点构建失败指针的时候，最耗时的环节是 `while` 循环中的 `q = q -> fail`，每运行一次这个语句，q 指向节点的深度都会减少 1，而树的高度最高也不会超过 len，因此**每个节点构建失败指针的时间复杂度是 `O(len)`, 整个失败指针的构建就是 `O(k*len)`.**
* 而 AC 自动机的构建过程通常是预先处理好的，构建完成之后，并不会频繁地更新，所以也不会影响敏感词过滤的运行效率。

### AC 自动机匹配的时间复杂度

* 和上文的分析类似，for 循环遍历主串中的每个字符，for 循环内部最耗时的部分也是 while 循环，这一部分的时间复杂度也是 `O(len),` 所以总的**匹配的时间复杂度就是 `O(n*len)`.** 因为敏感词并不会很长，而且这个时间复杂度是上限。实际情况中，**时间复杂度更可能接近于 `O(n)`.** AC 自动机做敏感词过滤，其实性能非常高。
* 从时间复杂度上看，AC 自动机的匹配效率和 Trie 树一样。但实际情况里，因为失效指针可能大部分情况都指向 root 根节点，所以绝大多数情况下，在 AC 自动机上匹配的效率会远高于刚才计算出的时间复杂度。只有在极端情况下，AC 自动机的性能才会退化的和 Trie 树一样。
	* 如下图所示：

![[AC自动机退化成Trie树示意图.png]]

## 小结

### AC 自动机的基本原理

* 这节主要了解多模式串匹配算法：AC 自动机。
* 单模式串匹配算法是为了快速在主串中查找一个字符串，而多模式串匹配算法是为了快速在一个主串中查找多个模式串。
* AC 自动机是基于 Trie 树的一种改进算法。它和 Trie 树的关系，就像单模式串中 KMP 算法和 BF 算法的关系。
	* KMP 算法中有非常关键的 next 数组，类比到 AC 自动机中就是失败指针。
	* 而且 AC 自动机中失败指针的构建过程，非常相似于 KMP 算法中计算 next 的过程。
	* 因此要理解 AC 自动机，就要先掌握 KMP 算法。
* 其实，AC 自动机就是 KMP 算法在多模式串上的改造。

### AC 自动机算法的内容

* AC 自动机算法包括两个部分：
	* 把多个模式串构建成 AC 自动机；这部分又包含两个小步骤：
		* 把模式串构建成 Trie 树；
		* 在 Trie 树上构建失败指针。
	* 在 AC 自动机中匹配主串。

## 扩展

* *试着分析一下各个字符串匹配算法的特点和比较适合的应用场景？*
| 算法       | BM(朴素匹配算法) | RK                   | BM                    | KMP                  | Trie             | AC 自动机            |
| ---------- | ---------------- | -------------------- | --------------------- | -------------------- | ---------------- | -------------------- |
| 核心逻辑   | 暴力匹配         | 求哈希值提高比较效率 | 好后缀和坏字符原则    | 最长公共前缀后缀子串 | 字符索引作为下标 | 基于Trie建立失败指针 |
| 实现难度   | ★★☆            | ★★★               | ★★★★☆                 | ★★★★                 | ★★★☆             | ★★★★☆                |
| 时间复杂度 | `O(m*n)`         | `O(m*n)`            | `O(3n/5n)` 业界未证实 | `O(m+n)`             | `O(n)`           | `O(n)`               |
| 算法类型   | 单模式串匹配算法 | 单模式串匹配算法     | 单模式串匹配算法      | 单模式串匹配算法     | 多模式串匹配算法 | 多模式串匹配算法     |
| 应用场景   | 主串较短字符的判断 | 主串较短字符的判断 | 单字符串查找 | 单字符串查找  | 搜索联想匹配   | 敏感词查找替换 |

#ALG #ADT 