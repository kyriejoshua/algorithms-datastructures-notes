# BM 算法

## 简言

* 假如我们想实现文本编辑器中的查找替换功能，实际上使用的就是字符串匹配算法。
* 上一节提到的 BF 算法和 RK 算法虽然简单实用，可以实现这种场景，但它们也有着各自的局限性。
	* BF 算法在某些极端情况下，性能会退化严重。
	* RK 算法需要用到哈希算法，而设计一个可以应对各种类型字符的哈希算法是不简单的。
* 对于文本查找这种工业级应用，我们希望算法尽可能简单高效，而且能够经得起极端情况下的考验，性能退化不至于十分严重。

> 那么，文本编辑器中的查找功能通常使用哪种算法实现呢？
> 学习曲线：★★★★☆

## BM 算法的核心思想

* BM 算法全称 (`Boyer-Moore`) 算法，是非常高效的字符串匹配算法。
* 我们可以把模式串和主串匹配的过程，理解成是**模式串在主串中不停移动**，当匹配到不同字符串时，BF 和 RK 算法的做法是模式串向后移动一位，然后从模式串的第一个字符开始重新匹配。
	* 如下图所示：

![[BF&RK匹配过程示意图.png]]

* 在这个例子中，主串里的 c 在模式串中是不存在的，所以实际上我们可以把模式串向后移动更多位数，而不仅仅是 1 位。
	* 例如这里，可以直接把模式串移动到 c 的后面。

![[BM模拟示意图.png]]

* BM 算法的核心思想，就是找到一种规律，可以让模式串一次性往后多滑动几位，从而提高匹配的效率。

## BM 算法原理分析

* BM 算法包含两部分，**坏字符原则和好后缀原则**。下面来分析这两个原则是如何相辅相成工作的。

### 坏字符原则

* 前面章节里，匹配过程都是按模式串的下标从小到大的顺序，依次与主串中的字符进行的。这种匹配比较符合思维习惯，而 BM 算法的匹配是相反的，它从后往前进行匹配，也就是下标从大到小进行匹配。

![[字符串算法通常的匹配顺序.png]]
![[BM算法匹配顺序.png]]

* 从后往前匹配时，当发现有某个字符没有匹配上，我们就把这个不匹配的字符称之为坏字符。
	* **注意：坏字符是主串中的字符。**
	* 个人理解，坏字符是从后往前不匹配的第一个字符。

![[坏字符示意图.png]]

* 我们使用坏字符在模式串中查找，发现模式串中完全不存在坏字符，也就是字符 c 不会与模式串中的任何字符匹配。对照上图，我们可以直接把模式串向后滑动三位，把模式串滑到 c 后面的位置，再从模式串的末尾字符重新开始比较。

![[坏字符移动示意图.png]]

* 移动后，主串中的坏字符 a，和模式串的末位不相等，但是在模式串中也是存在的。这时就不能暴力的移动模式串的长度也就是不能直接移动三位了。
* 因为模式串中的下标为 0 的位置的字符也是 a，我们就需要把模式串向后移动两位，目的是让模式串中的 a 和主串中的 a 上下对齐，然后再从模式串中的末位开始匹配。

![[坏字符移动示意图2.png]]

* 上面两个案例，我们根据坏字符的位置来移动的位数是不同的。可以发现，显然存在某种规律，可以参照这种规律来移动模式串。
* 当模式串和主串中的子串比较时发生不匹配，我们把**坏字符对应的模式串中的字符的下标记作 `si`**.
* 如果坏字符在模式串中存在，那把这个坏字符在模式串中的下标位置记作 `xi`. 如果 `xi` 不存在，就把它当成是 -1.
* 那么**模式串移动的位数就是 `si - xi`.**

![[坏字符移动规律示意图.png]]

* 而**如果出现多个 xi，我们就选择其中最靠后的。** 也就是移动位数最少的。这样就可以避免因为滑动过多而导致错过可能匹配的情况。
* 应用坏字符规则，BM 算法在最好情况下的时间复杂度是非常低的 `O(n/m)`.
	* 例如，主串是 `aaabaaabaaabaaabaaab`, 模式串是 `baaa`, 每次匹配模式串都可以直接向后移动四位。
* 但如果仅仅使用坏字符规则也不够，**在某些情况下， `si-xi` 计算出来的可能会是负数**，根据负数是无法移动的。
	* 例如，主串是 `aaaaaaaaaaaaaa`, 模式串是 `baaa`. 这时计算出的移动位数就是负数。`xi = 1`, `si = 0`, 也就是 `0 - 1 = -1`
	* 在这种场景下，就需要好后缀原则来辅助判断。

### 好后缀原则

* 好后缀原则和坏字符串原则是相似的。
* 在下图中，模式串滑动到图中位置的时候，模式串和主串末尾的 2 个字符是匹配的，直到倒数第 3 个字符发生了不匹配的情况。

![[好后缀示意图.png]]

* 上图中，也可以使用坏字符原则，但这里使用好后缀原则更加合适。至于两者的取舍和优先级，后文会讲到。这里先继续了解好后缀原则的工作原理。
* **我们把已经匹配的字符叫做好后缀，也就是上图中的 `bc`，记作 `{u}`，拿到模式串中查找，如果找到了另一个和 `{u}` 匹配的子串 `{u*}`, 我们就可以把模式串滑动到子串 `{u*}` 与主串中 `{u}` 对齐的位置。**

![[好后缀移动示意图.png]]

* 如果在模式串中找不到另一个等于 `{u}` 的子串，我们就可以直接把模式串滑动到主串中 `{u}` 的后面。
* 因为前面的任意子串都没有和这个后缀 `{u}` 匹配的场景，可以直接略过。

![[好后缀跳过移动示意图.png]]

* 那么，真的是这样吗？在上面的场景下，我们直接滑动模式串到主串 `{u}` 的后面，会不会有一些特殊场景？
* 下面这个例子展示了意外的情况。
	* 这里 `bc` 是好后缀，而且在模式串的子串中不存在另一个相匹配的子串 `{u*}`, 但如果直接移动到跳过后缀的位置，会发现其实错过了一种可能匹配的场景。

![[好后缀移动过头示意图.png]]

* 当模式串滑动到前缀与主串中 `{u}` 的后缀有部分重合的时候，并且重合部分相等的时候，就有可能存在完全匹配的情况。

![[主串后缀与模式串前缀匹配示意图.png]]

* 针对这种特殊场景，我们不仅要确认好后缀在模式串中的匹配情况，还需要**观察好后缀的后缀子串是否和模式串的前缀子串相匹配的场景。**
* **后缀子串：最后一个字符和当前字符对齐的子串。**
	* *abc 的后缀子串就是 bc 和 c。*
* **前缀子串：起始字符和当前字符对齐的子串。**
	* *abc 的前缀子串就是 a 和 ab.*
* 我们需要从好后缀的后缀子串中，找一个最长的且能够和模式串的前缀子串匹配的。假设它是 `{v}`, 然后可以如下图滑动。

![[主串后缀与模式串前缀移动示意图‘.png]]

### 两大原则的取舍

* 上面就是坏字符原则和好后缀原则的基本原理。现在我们来回顾，基于这两种原则，如何计算模式串向后移动的位数？
* **首先需要分别计算这两大原则所需要向后移动的位数，然后取两个数中更大的作为向后移动的位数。这种处理方式可以避免仅靠坏字符原则计算得到的移动位数是负数的情况。**

## BM 算法代码实现

### 坏字符原则的实现思路

* 坏字符实现的核心并不难理解，就是计算往后移动的位数，也就是 `si - xi`. 最主要的就是求出 `xi` 的值。也就是坏字符在模式串中出现的位置。
* 这里的核心就是使用散列表。我们把模式串中的每个字符和下标都存储到散列表中。通过这种方式来快速找到坏字符在模式串中的位置下标。
* 我们这里只讨论最简单的场景。假设字符串的字符集并不是很大，我们只使用大小为 256 的数组，存储每个假设长度是 1 字节的字符，记录他们在模式串中出现的位置。**数组的下标对应着字符的 ASCII 码值，数组中存储的是这个字符在模式串中最后出现的位置。**

![[坏字符通过散列表存储示意图.png]]

* 把上面的内容转化为代码，下面是 Java 的实现。**其中变量 b 是模式串，m 是模式串的长度，bc 是散列表。**

```java
private static final int SIZE = 256; // 全局变量或成员变量
private void generateBC(char[] b, int m, int[] bc) {
  for (int i = 0; i < SIZE; ++i) {
    bc[i] = -1; // 初始化bc
  }
  for (int i = 0; i < m; ++i) {
    int ascii = (int)b[i]; // 计算b[i]的ASCII值
    bc[ascii] = i;
  }
}
```

* 上面的代码实现是坏字符规则的内容。我们把这些逻辑先应用到 BM 算法中，暂且不考虑好后缀规则的实现和坏字符规则的边界条件（移动值为负数的情况）。

```java
public int bm(char[] a, int n, char[] b, int m) {
  int[] bc = new int[SIZE]; // 记录模式串中每个字符最后出现的位置
  generateBC(b, m, bc); // 构建坏字符哈希表
  int i = 0; // i表示主串与模式串对齐的第一个字符
  while (i <= n - m) {
    int j;
    for (j = m - 1; j >= 0; --j) { // 模式串从后往前匹配
      if (a[i+j] != b[j]) break; // 坏字符对应模式串中的下标是j
    }
    if (j < 0) {
      return i; // 匹配成功，返回主串与模式串第一个匹配的字符的位置
    }
    // 这里等同于将模式串往后滑动j-bc[(int)a[i+j]]位
    i = i + (j - bc[(int)a[i+j]]); 
  }
  return -1;
}
```

![[坏字符实现示意图.png]]

* 以上就是坏字符实现的框架代码了。我们可以试着再用 TS 来实现一遍坏字符规则，散列表实际上对应的就是 TS 中的对象。
* 首先是用对象存储每个字符的位置下标，也可以理解为坏字符规则表。

```typescript
/**
 * @description: 坏字符匹配规则
 * @param {string} search
 * @return {Record<string, number>}
 */
function createBadCharMatch(search: string): Record<string, number> {
  const map = {};
  const hash = {};

  for (let i = 0; i < search.length; i++) {
    const str = search[i];
    if (hash[str]) {
      map[i] = hash[str];
    } else {
      map[i] = -1;
    }
    hash[str] = i;
  }
  return map;
}
```

* 其次是应用坏字符的计算逻辑来实现初版的 bm 算法。暂不考虑一些极端场景。

```typescript
/**
 * @description: 仅应用坏字符原则的 bm 算法实现
 * @param {string} str
 * @param {string} pattern
 * @return {number}
 */
function bm(str: string, pattern: string): number {
  const sLen = str.length;
  const pLen = pattern.length;
  const badCharMap: Record<string, number> = createBadCharMatch(str);

  let i = 0;

  while (i <= sLen - pLen) {
    for (let j = pLen - 1; j >= 0; j--) {
      // 如果最后一个字符不相等
      if (str[i + j] !== pattern[j]) break;
    }
    // 说明已经匹配完成
    if (j < 0) {
      return i;
    }

    // !相当于把模式串向后滑动 `j - badCharMap[str[i + j]]` 位
    i += j - badCharMap[str[i + j]];
  }

  return -1;
}
```

### 好后缀原则的实现思路

* 相比于坏字符原则，好后缀的原则会复杂一些。我们先回顾好后缀原则的核心内容：
	* **在模式串中，查找和好后缀匹配的另一个子串；**
	* **在好后缀的后缀子串中，查找最长的、可以和模式串前缀子串匹配的后缀子串。**
* 这里完全可以通过暴力匹配的方式来实现上述核心两点。但是这样做就没有意义了，BM 算法本身是高效的，我们也需要设计出高效的查找方式来提高它的效率。
* 因为**好后缀本身也是模式串的后缀子串，我们可以在模式串和主串匹配之前，通过预处理模式串，提前计算好模式串的每个后缀子串，对应的另一个可能匹配子串的位置。**
	* 我们逐步来分析这个预处理的过程，这部分稍复杂一些。
* 首先的问题是，如何表示模式串中不同的后缀子串？
	* 后缀子串的最后一个字符的位置是固定的，下标为 `m - 1`，通过记录长度，就可以确认一个唯一的后缀子串。

![[后缀子串最后一位示意图.png]]

* 这里需要引入一个变量 `suffix` 数组。
	* **`suffix` 数组的下标 k 表示后缀子串的长度，下标对应的数组值存储的是在模式串中和好后缀 `{u}` 相匹配的子串 `{u*}` 的起始下标值。**
	* 观察下图来帮助理解。**值为 -1 表示不存在和后缀子串匹配的子串，意味着无法根据好后缀来移动。**

![[好后缀suffix数组示意图.png]]

* 但是我们知道，**可能会出现有多个子串和后缀子串 `{u}` 匹配，而 `suffix` 数组中存储的就是模式串中最靠后的子串的起始位置。这样可以避免模式串向后滑动时移动过头。**
	* **`suffix` 承载的功能就是存储和好后缀匹配的另一个子串的起始位置。**
* 而好后缀规则中还包括一条，**在好后缀的后缀子串中，查找最长的能和模式串前缀子串匹配的后缀子串。这里就需要另一个数组来存储。**
* 我们**使用 `prefix` 数组来记录模式串的后缀子串中是否有能匹配模式串的前缀子串。
	* **`prefix` 存储的是 `boolean` 类型的值，下标为匹配的前缀子串的长度。**

![[好后缀prefix数组示意图.png]]

#### suffix 和 prefix 的实现

* 接下来的内容就是如何来实现这两个数组。
* 我们取下标从 0 到 i 的子串（i 的取值可以是 0 至 `m - 2`）与整个模式串，求公共后缀子串。
	* **如果公共后缀子串的长度是 k，我们就记录 `suffix[k] = j` (*j 表示公共后缀子串的起始下标*)。**
	* **如果 j 等于 0，也就是公共后缀子串同时是模式串的前缀子串，我们就记录 `prefix[k] = true`.**

![[实现suffix与prefix示意图.png]]

* 下面是 `suffix` 数组和 `prefix` 数组的 Java 实现。

```java
// b表示模式串，m表示长度，suffix，prefix数组事先申请好了
private void generateGS(char[] b, int m, int[] suffix, boolean[] prefix) {
  for (int i = 0; i < m; ++i) { // 初始化
    suffix[i] = -1;
    prefix[i] = false;
  }
  for (int i = 0; i < m - 1; ++i) { // b[0, i]
    int j = i;
    int k = 0; // 公共后缀子串长度
    while (j >= 0 && b[j] == b[m-1-k]) { // 与b[0, m-1]求公共后缀子串
      --j;
      ++k;
      suffix[k] = j+1; //j+1表示公共后缀子串在b[0, i]中的起始下标
    }
    if (j == -1) prefix[k] = true; //如果公共后缀子串也是模式串的前缀子串
  }
}
```

* 类似地，我用 TS 实现了一遍，并附上实践的案例和日志。

```typescript
interface IGoodCharMatch {
  suffix: number[];
  prefix: boolean[];
}

/**
 * @description: 创建好后缀规则的关键数组
 * @param {string} b 模式串
 * @return {IGoodCharMatch}
 */
function createGoodCharMatch(b: string):IGoodCharMatch {
  const len = b.length; // 模式串的长度
  const suffix: number[] = new Array(len).fill(-1); // 初始化
  const prefix: boolean[] = new Array(len).fill(false);

  // 遍历主串的字符
  for (let i = 0; i < len - 1; i++) {
    let j = i;
    let k = 0; // 公共后缀子串的长度

    // !双指针比较字符串当前位置和末尾的字符
    while (j >= 0 && b[j] === b[len - 1 - k]) {
      --j; // 前缀子串从当前位置往前
      ++k;
      suffix[k] = j + 1;
    }
    // j 计算到 -1 说明已经找到公共后缀子串，长度为 k
    if (j === -1) prefix[k] = true;   
  }

  return { prefix, suffix };
}

const str = 'cabcabca';
console.log(createGoodCharMatch(str));
// {
//   prefix: [ false, false, true, false, false, true, false, false],
//   suffix: [-1, 4, 3, 2, 1, 0, -1, -1]
// }
```

* 接着我们来计算模式串和主串匹配的过程中，遇到不能匹配的字符时，如何根据好后缀规则得到模式串往后滑动的位数？
* 假设好后缀的长度是 k。先取出好后缀，在 `suffix` 数组中查找匹配的子串。
	* 如果 `suffix[k] !== -1` (*-1 表示不存在匹配的子串*), 那就把模式串移动 `j - suffix[k] + 1` 位(*j 表示坏字符对应的模式串中的字符的下标)*;
	* 如果 `suffix[k] === -1`, 表示模式串中不存在另一个好后缀匹配的子串片段，就根据下面的规则来处理。

![[计算过程.png]]

* 好后缀的后缀子串 `b[r, m - 1]`, (*r 取值从 J+2 至 m - 1*) 的长度 `k = m - r`.
	* **如果 `prefix[k] === true`，表示长度 k 的后缀子串，有可匹配的前缀子串，这样我们可以直接把模式串后移 r 位。**
	* 详细解释就是，后缀子串的起始位置 r 减去前缀子串的起始位置 0，也就是后移 r 。

![[好后缀中存在后缀子串与前缀子串匹配的移动过程.png]]

* 如果好后缀的两条规则都没有找到可以匹配好后缀以及后缀子串的子串，就可以把整个模式串向后移动 m 位。
	* 详细解释就是，好后缀的规则不适用，因此需要移动 m 位，这比坏字符应用的移动位数要多；当两者移动的位数不同时，以更大的移动位数为准，所以这里应用好后缀原则的移动位数，也就是 m 位。

![[好后缀规则未适用的移动.png]]

### BM 算法的完整实现

* 上述便是好后缀的实现原理和过程。这里先通过 Java 的方式把完整过程用代码实现一遍。

```java
// a,b表示主串和模式串；n，m表示主串和模式串的长度。
public int bm(char[] a, int n, char[] b, int m) {
  int[] bc = new int[SIZE]; // 记录模式串中每个字符最后出现的位置
  generateBC(b, m, bc); // 构建坏字符哈希表
  int[] suffix = new int[m];
  boolean[] prefix = new boolean[m];
  generateGS(b, m, suffix, prefix);
  int i = 0; // j表示主串与模式串匹配的第一个字符
  while (i <= n - m) {
    int j;
    for (j = m - 1; j >= 0; --j) { // 模式串从后往前匹配
      if (a[i+j] != b[j]) break; // 坏字符对应模式串中的下标是j
    }
    if (j < 0) {
      return i; // 匹配成功，返回主串与模式串第一个匹配的字符的位置
    }
    int x = j - bc[(int)a[i+j]];
    int y = 0;
    if (j < m-1) { // 如果有好后缀的话
      y = moveByGS(j, m, suffix, prefix);
    }
    i = i + Math.max(x, y);
  }
  return -1;
}

// j表示坏字符对应的模式串中的字符下标; m表示模式串长度
private int moveByGS(int j, int m, int[] suffix, boolean[] prefix) {
  int k = m - 1 - j; // 好后缀长度
  if (suffix[k] != -1) return j - suffix[k] +1;
  for (int r = j+2; r <= m-1; ++r) {
    if (prefix[m-r] == true) {
      return r;
    }
  }
  return m;
}
```

* TS 版本的完整实现。这是参考网上一篇文章的内容略加修改而成。

```typescript
interface ISuffixAndPrefix {
  suffix: number[];
  prefix: boolean[];
}

/**
 * @description: bm 算法实现
 * @param {string} str 主串
 * @param {string} pattern 模式串
 * @return {number}
 */
function bm(str: string, pattern: string): number {
  const badChar: Record<string, number> = generateBadChar(pattern);
  const { suffix, prefix }: ISuffixAndPrefix = generateGoodSuffix(pattern);

  const sLen = str.length, pLen = pattern.length;
  let i = 0;
	// 循环遍历
  while (i <= sLen - pLen) {
    // 从子串末尾向前遍历查找坏字符
    let j = pLen - 1;
    while (j >= 0) {
      // 找到坏字符，退出循环
      if (str[j + i] !== pattern[j]) break;
      j--;
    }
    // !如果j < 0,说明没有坏字符，字符串匹配完成，直接返回下标
    if (j < 0) return i;

    // 计算坏字符移动位数
    let bc = j;
    // 如果坏字符在前面有对应的字符，就移动对应的距离，否则就是移动坏字符的距离
    if (badChar[str[j + i]] !== undefined) {
      bc = j - badChar[str[j + i]];
    }

    // 计算好后缀移动位数
    let gs = 0;
    // j 比 pLen - 1小才会有好后缀
    if (j < pLen - 1) {
      gs = moveByGs(j, pLen, suffix, prefix);
    }

    // 从坏字符和好后缀中取较大者
    // 当较大值为0的时候，移动1位
    i += Math.max(bc, gs) || 1;
  }

  return -1;
}

/**
 * @description: 创建badChar的映射，方面快速定位p中字符的位置
 * @param {string} p
 * @return {Record<string, number>}
 */
function generateBadChar(p: string ): Record<string, number> {
  const badChar: Record<string, number> = {}
  for (let i = 0; i < p.length; i++) {
    // 相同字符中，我们只关注靠后的字符
    // 所以重复的字符只记录最后一次出现的位置即可
    badChar[p[i]] = i;
  }

  return badChar;
}

/**
 * @description: 生成好后缀哈希
 * @param {string} pattern
 * @return {ISuffixAndPrefix}
 */
function generateGoodSuffix(pattern: string): ISuffixAndPrefix {
  const pLen = pattern.length;
  const suffix: number[] = [];
  const prefix: boolean[] = [];

  // b 表示模式串，m 表示长度，suffix，prefix 数组事先申请好了
  for (let i = 0; i < pLen; ++i) {
    // 初始化
    suffix[i] = -1;
    prefix[i] = false;
  }

  for (let i = 0; i < pLen - 1; ++i) {
    let j = i;
    let k = 1; // k用来标记模式串后缀长度

    // 从后往前对比子串和模式串后缀
    // p[j] 用来标记子串后缀的首字母
    // p[pLen - k] 用来标记模式串后缀的首字母
    while (j >= 0 && pattern[j] === pattern[pLen - k]) {
      suffix[k] = j;
      j--;
      k++;
    }

    // 0 ~ i 子串同时也是模式串的前缀
    // j == -1，表示该前缀和模式串的后缀完全相等
    if (j == -1) prefix[k - 1] = true;
  }
  // console.log(suffix, prefix);

  return { suffix, prefix };
}

/**
 * @description: 利用好后缀移动
 * @param {number} index 坏字符对应的字符下标
 * @param {number} pLen 模式串长度
 * @param {number[]} suffix
 * @param {boolean[]} prefix
 * @return {number[]}
 */
function moveByGs(index: number, pLen: number, suffix: number[], prefix: boolean[]): number {
  const len = pLen - 1 - index; // 好后缀长度
  // 如果suffix有值，直接计算出移动的距离
  if (suffix[len] !== -1) return index - suffix[len] + 1;
  for (let i = len; i > 0; i--) {
    if (prefix[i]) {
      // 把模式串头部对齐好后缀的后缀
      return pLen - i;
    }
  }

  return pLen;
}
```

* 实践一下上述代码。输入字符串并打印结果。

```typescript
const str = 'a dog jump over a fox';
const pattern1 = 'god';
const pattern2 = 'dog';
const pattern3 = 'fox';

console.log(bm(str, pattern1)); // -1
console.log(bm(str, pattern2)); // 2
console.log(bm(str, pattern3)); // 18
```

* 结果符合预期。

## BM 算法的性能分析及优化

### 空间消耗

* 分析 BM 的内存消耗，算法过程总共使用到 3 个额外的数组，bc 数组的大小和字符集大小有关，`suffix` 数组和 `prefix` 数字的大小和模式串的长度 m 有关。
* 如果处理字符集很大的字符串匹配问题，bc 数组的内存消耗就会比较多。而因为好后缀规则和坏字符规则是独立的，在运行环境对内存要求苛刻的情况下，可以选择只使用好后缀原则，不适用坏字符原则。这样就可以避免 bc 数组过多的内存消耗。当然，仅仅使用好后缀的 BM 算法效率也会相应下降。

### 执行效率

* BM 算法的时间复杂度分析是非常复杂的。不同论文对在最坏情况下，BM 算法的比较次数上限的结论甚至有所不同，一说是 5n，另一说是 3n。
* 针对执行效率这一点，还有比较巧妙的方式来提升。因为好后缀原则和坏字符原则的移动位数主要与搜索词有关，与原字符串无关。因此可以预先计算生成《坏字符规则表》和《好后缀规则表》。使用时，只需要查表比较一下就可以，而无需反复的比较和计算。
	* 但是在这节所讲述的初级版本 BM 算法里，预处理 `suffix` 和 `prefix` 在极端情况下的性能会比较差。

## 小结

* BM 算法的核心思想是，利用模式串本身的特点，在模式串中某个字符与主串中的字符不匹配的时候，让模式串多向后多滑动几位，以此来减少不必要的字符比较，从而提高匹配的效率。
* BM 算法构建的规则有两类，坏字符规则和好后缀规则。好后缀规则可以独立于坏字符规则使用。因为坏字符规则的实现会比较耗内存，为了节省内存可以只用好后缀规则来实现 BM 算法。

### 其他参考

> [原理和实现 segmentfault](https://segmentfault.com/a/1190000021632197)
> [原理和实现 zhihu](https://zhuanlan.zhihu.com/p/63596339)

#ALG 