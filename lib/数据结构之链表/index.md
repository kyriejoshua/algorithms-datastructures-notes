# 链表

## 简言

* 学习链表，首先可以了解链表的应用场景。它的一个经典应用场景就是 LRU 缓存淘汰算法。
* 缓存是一种提高数据读取性能的技术，在硬件设计、软件开发中都有着极为广泛的应用，例如常见的 *CPU 缓存*、*数据库缓存*、*浏览器缓存*等等。
* 但实际中，缓存的大小是有限的，当缓存接近用满时，需要决定哪些数据应该被清除，然后用这些被清除后所剩的空间来存放新的数据。
* 常见的缓存淘汰策略有以下三种：
	* 先进先出策略 FIFO
	* 最少使用策略 LFU
	* 最近最少使用策略 LRU
* 本节的重点，就是如何使用链表实现缓存淘汰策略。

> 学习曲线：★★★☆

## 链表的结构

* 链表(`Linked List`) 是比数组稍微复杂一些的数据结构。
* 我们首先从底层的存储结构来了解链表，并且对比数组以便更好地理解他们的定义和区别。

![[链表内存.png]]

* *数组是连续的内存空间，而链表是不连续的。*
	* **链表通过指针将不连续的内存块串起来使用。**
* 链表的结构有很多种，这里先学习常见的三类：**单链表，循环链表和双向链表。**

### 单链表

![[单链表.png]]

* 单链表中，第一个结点和最后一个结点比较特殊，分别称之为**头结点**和**尾结点**。
	* **头结点用于记录链表的基地址，有了头结点，就可以遍历得到整条链表。**
	* **尾结点的指针不是指向下一个结点，而是指向一个空地址 null.**
* 单链表结点的 TS 实现:

```typescript
/**
 * 链表结点
 */
class ListNode<T> {
  constructor(value: T) {
    this.value = value;
    this.next = null;
  }
}
```

#### 链表的插入，删除和查找

* 数组的插入和删除操作，时间复杂度是 `O(n)`.
* 链表的插入和删除操作，时间复杂度是 `O(1)`. 因为它只需要考虑相邻结点的指针改变。

![[链表的插入和删除.png]]

* 但是链表查找的时间复杂度是 `O(n)`, 需要进行遍历。
* 上述操作的时间复杂度，是针对单链表而言。

#### 单链表的实现

* 单链表的 TS 实现，含相关验证日志。
	* 以及在 [LeetCode](https://leetcode-cn.com/problems/design-linked-list/submissions/) 上的实现，对于部分异常场景的处理略有不同。
	* 下文中的实现是早期版本，最新版本可直接查看对应仓库的内容。[Github: javascript-datastructure](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/LinkedList/index.ts).

```typescript
/**
 * @description: 链表的节点
 * @param {*}
 * @return {*}
 */
class LinkedNode<T> {
  public element: T;
  public next: TLinkedNode<T>;

  constructor(element: T, next: TLinkedNode<T> = null) {
    this.element = element;
    this.next = next;
  }
}

type TLinkedNode<T> = LinkedNode<T> | null;

class LinkedList<T> {
  protected count = 0;
  protected head: TLinkedNode<T> = null;

  /**
   * @description: 根据元素内容获取对应的索引
   * @param {T} element
   * @return {number}
   */
  indexOf(element: T): number {
    let current: TLinkedNode<T> = this.head;
    let i = 0;
    while (i < this.count) {
      if (element === current?.element) return i;
      current = (current as LinkedNode<T>).next;
      i++;
    }
    return -1;
  }

  /**
   * @description: 索引是否溢出或不合法
   * @param {number} index
   * @return {boolean}
   */
  isOverflow(index: number): boolean {
    return index < 0 || index > this.count;
  }

  /**
   * @description: 根据索引获取结点
   * @param {number} index
   * @return {TLinkedNode<T>}
   */
  getLinkedNodeByIndex(index: number): TLinkedNode<T> {
    if (this.isOverflow(index)) return null;

    if (index === 0) {
      return this.head;
    }
    let i = 1;
    let current: TLinkedNode<T> = (this.head as LinkedNode<T>).next;
    let prev: TLinkedNode<T> = this.head;

    while (i < index) {
      prev = (prev as LinkedNode<T>).next;
      current = (current as LinkedNode<T>).next;
      i++;
    }

    return current;
  }

  /**
   * @description: 根据结点索引获取结点值
   * @param {number} index
   * @return {T|undefined}
   */
  getElementByIndex(index: number): T | undefined {
    return this.getLinkedNodeByIndex(index)?.element;
  }

  /**
   * @description: 链表末尾推入元素
   * @param {T} element
   * @return {number}
   */
  add(element: T): number {
    const node: TLinkedNode<T> = new LinkedNode(element);

    // 如果头结点为空则直接设置成头结点
    if (this.head === null) {
      this.head = node;
    } else {
      // 找到末尾的结点
      const current: TLinkedNode<T> = this.getEnd();
      console.info('---------', current);
      (current as LinkedNode<T>).next = node;
    }
    this.count++;

    return this.count;
  }

  /**
   * @description: 插入结点
   * @param {T} element
   * @param {number} index
   * @return {boolean}
   */
  insertByIndex(element: T, index: number): boolean {
    // 如果溢出或者索引为负，直接添加不成功
    if (this.isOverflow(index)) return false;

    const node = new LinkedNode(element);
    let i = 1;

    // 设置头结点或者正常插入
    if (index === 0) {
      this.head = node;
    } else {
      let current: TLinkedNode<T> = (this.head as LinkedNode<T>).next;
      let prev: TLinkedNode<T> = this.head;

      while (i < index) {
        prev = (prev as LinkedNode<T>).next;
        current = (current as LinkedNode<T>).next;
        i++;
      }
      node.next = current;
      (prev as LinkedNode<T>).next = node;
    }
    this.count++;

    return true;
  }

  /**
   * @description: 在指定索引处删除结点
   * @param {number} index
   * @return {T|null}
   */
  removeByIndex(index: number): T | null {
    // 如果溢出或者索引为负，删除失败
    if (this.isOverflow(index)) return null;
    let current: TLinkedNode<T>;

    if (index === 0) {
      current = this.head;
      this.head = (this.head as LinkedNode<T>).next;
    } else {
      const prev: TLinkedNode<T> = this.getLinkedNodeByIndex(index - 1);
      current = (prev as LinkedNode<T>).next;
      (prev as LinkedNode<T>).next = (current as LinkedNode<T>).next;
    }

    this.count--;

    return (current as LinkedNode<T>).element;
  }

  /**
   * @description: 根据结点值移除元素
   * @param {T} element
   * @return {T|null}
   */
  remove(element: T): T | null {
    const index: number = this.indexOf(element);
    return this.removeByIndex(index);
  }

  /**
   * @description: 链表的结点数量
   * @return {number}
   */
  size(): number {
    return this.count;
  }

  /**
   * @description: 链表是否为空
   * @return {boolean}
   */
  isEmpty(): boolean {
    return this.size() === 0;
  }

  /**
   * @description: 获取头结点
   * @return {TLinkedNode<T>}
   */
  getHead(): TLinkedNode<T> {
    return this.head;
  }

  /**
   * @description: 获取尾结点
   * @return {TLinkedNode}
   */
  getEnd(): TLinkedNode<T> {
    let current = this.head;

    while (current && current.next !== null) {
      current = current.next;
    }

    return current;
  }

  /**
   * @description: 清空当前链表
   * @return {void}
   */
  clear(): void {
    this.count = 0;
    this.head = null;
  }

  /**
   * @description: 把所有结点转成字符串
   * @return {string}
   */
  toString(): string {
    if (!this.getHead()) return '';

    let current: TLinkedNode<T> = this.head;
    let str = '';
    while (current !== null) {
      str += `${current.element}`;
      current = current.next;
      if (current) str += ' -> ';
    }

    return str;
  }
}
```

* 我们创建一个单链表并执行操作来验证链表实现的正确性。

```typescript
const linked = new LinkedList();
linked.add(3);
linked.add(7);
linked.add(11);
linked.add(13);
linked.add(4);
linked.add(6);
console.log('Before inserting', linked, linked.size()); // count: 6
linked.insertByIndex(5, 1);
console.log('After inserting', linked, linked.size()); // count: 7
console.log('Head node is', linked.getHead());
console.log('Removing node at', linked.removeByIndex(5)); // Removing node at 4
console.log('Now index 5 is', linked.getLinkedNodeByIndex(5)); // Now index 5 is LinkedNode { element: 6, next: null }
console.log('End node is', linked.getEnd()); // End node is LinkedNode { element: 6, next: null }
console.log('Stringified linkedlist', linked.toString()); // Stringified linkedlist 3 -> 5 -> 7 -> 11 -> 13 -> 6
console.log('IndexOf 4', linked.indexOf(4)); // IndexOf 4 -1
console.log('IndexOf 3', linked.indexOf(11)); // IndexOf 3 3
linked.clear();
console.log('After clearing', linked, linked.size()); // After clearing LinkedList { count: 0, head: null } 0
```

### 循环链表

![[循环链表.png]]

* 循环链表其实是特殊的单链表。它的不同之处在于，循环链表的尾结点的指针指向头结点。
* 循环链表可以比单链表更方便地解决[约瑟夫问题](https://www.wikiwand.com/zh-hans/%E7%BA%A6%E7%91%9F%E5%A4%AB%E6%96%AF%E9%97%AE%E9%A2%98)。

### 双向链表

![[双向链表.png]]

* **双向链表比单链表多了一个额外的空间 `prev`, 用来指向前驱结点的内存地址。**
* 理论上，双向链表比单链表的删除操作的时间复杂度要好，但是实际场景需要实际分析。

#### 常见的删除操作

1. 删除结点中值等于某个给定值的节点。
2. 删除给定指针指向的节点。

* 第一种情况，为了查找到给定值的结点，还是需要从头结点开始一个个遍历，直到找到对应结点。这个查找的时间复杂度是 `O(n)`. 根据加法法则，尽管删除的时间复杂度较低，但整体的时间复杂度还是为 `O(n)`。
* 第二种情况，已经找到要删除的节点，但是链表删除的方式是把前驱结点的指针指向删除结点的后一个结点，因此还是需要通过遍历找到删除节点的前驱结点，这时时间复杂度就仍为 `O(n)`。
	* 遍历过程：p 是 q 的前驱结点。

	```javascript
	while (p.next) {
	  if (p.next === q) {
	    return p;
	  }
	  p = p.next;
	}
	```

* 在这第二种情况下，双向链表是有着显著的优势的。因为双向链表已经保存了前驱结点，因此可以在 `O(1)` 的时间复杂度内搞定。
* 同理，在某个指定节点进行插入操作也是 `O(1)` 的时间复杂度。
* 对于一个有序链表，双向链表的按值查询的效率也比单链表高一些。

#### 双向链表的实际应用与缓存

* 实际应用中，虽然双向链表更加耗费内存，但是应用却更广泛。这就涉及到**空间换时间的设计思想**。当内存空间足够，如果要追求代码的执行速度，就可以选择空间复杂度较高，但时间复杂度较低的算法或者数据结构。相反如果内存比较紧缺，比如代码跑在手机或者单片机上，这个时候就要反过来用**时间换空间的设计思路**。

* *缓存实际上就是利用了空间换时间的设计思想。如果我们把数据存储在硬盘上，会比较节省内存，但每次查找数据都要询问一次硬盘，会比较慢。但如果我们通过缓存技术，事先将数据加载在内存中，虽然会比较耗费内存空间，但是每次数据查询的速度就会大大提高。

* 对于执行较慢的程序，可以通过**消耗更多的内存（空间换时间）** 来进行优化；而消耗过多内存的程序，可以通过 **消耗更多的时间（时间换空间）** 来降低内存的消耗。

#### 双向链表的实现

* 基于单链表的实现： [查看源码](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/DuLinkedList/index.ts)

### 双向循环链表

![[双向循环链表.png]]

* 结合上文，可以更好的理解双向循环链表的概念。

## 数组和链表的性能对比

| 数据结构 | 数组 | 链表 |
| -------- | ---- | ---- |
| 插入删除 | O(n)    | O(1)    |
| 随机访问 | O(1)    | O(n)    |

## 链表在实际开发中的应用

* 数组简单易用，底层使用的是连续的内存空间，可以借助 CPU 的缓存机制，预读数组中的数据，所以访问效率更高。而链表在内存中并不是连续存储，所以对 CPU 缓存不友好，没办法有效预读。
* 数组的缺点是大小固定，一经声明就要占用整块连续内存空间。如果声明的数组过大，系统可能没有足够的连续内存空间分配给它，导致“内存不足（out of memory）”。如果声明的数组过小，则可能出现不够用的情况。这时只能再申请一个更大的内存空间，把原数组拷贝进去，非常费时。链表本身没有大小的限制，天然地支持动态扩容，这也是与数组最大的区别之一。
* **概括的说，当插入数据超过了数组的连续空间大小的时候，数组会重新申请内存空间，并且将原来的数据拷贝到新的内存空间里。可能出现的状况是，一是这个操作比较耗时耗性能，二是可能存在内存不足的问题。**
* **对内存的使用条件比较苛刻，就适合使用数组。链表需要额外的空间来存放指向下个结点的指针，内存消耗会翻倍。对链表的频繁插入，删除会导致频繁的内存申请和释放，容易造成内存碎片。例如在 Java 中就会触发频繁的 GC 垃圾回收。**

## 常用的缓存淘汰策略

* 我们着重关注的是 LRU 缓存策略。

### 先进先出策略 FIFO

### 最少使用策略 LFU

### 最近最少使用策略 LRU

#### LeetCode 题目

* LRU 缓存策略是经常会在面试中问到的，例如手写一个满足 LRU 约束的数据结构。
	* 也有对应的算法题目。[LeetCode-172](https://leetcode.cn/problems/lru-cache/)

#### 基于链表实现 LRU 缓存淘汰算法

* 使用有序单链表数据结构，越靠近尾部的结点就是越早访问的结点。
* 当一个新的数据访问时，从头开始遍历链表：
	* 如果数据之前已经缓存在链表中，则删除该结点，并将数据插入链表的头部。
	* 如果数据之前不在缓存的链表中，判断：
		* 如果此时缓存的链表未满，则直接把该结点插入链表的头部。
		* 如果此时缓存的链表满了，则先删除尾结点，再插入链表的头部。
* 这种实现思路的时间复杂度是 `O(n)`.
* 链表的实现较为复杂且更完善，因此代码量也较多，可以直接点击 **[查看源码](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Cache/LRUCacheByLinkedNode.ts)** 。

##### LRU 的 TS 类实现(基于数组)

* 而使用数组实现的思路是类似的，就是使用数组实现类似队列的效果。（先进先出 FIFO）
* 因为 js 中链表结构本身需要开发，这里用数组实现 LRU 缓存淘汰算法，其思路是一样的。

```typescript
export interface ICacheItem {
  key: number;
  value: number;
}

/**
 * LRU 缓存淘汰算法-数组实现
 */
class LRUCacheByArray {
  private cacheList: ICacheItem[]; // 缓存数组
  private maxCacheLength: number; // 缓存容量

  constructor(capacity: number) {
    this.cacheList = [];
    this.maxCacheLength = capacity;
  }

  /**
   * @description: 缓存的读取
   * @param {number} key
   * @return {number}
   */
  get(key: number): number {
    // 查找索引是否存在
    const cachedIndex = this.cacheList.findIndex((item: ICacheItem): boolean => item.key === key);
    if (cachedIndex === -1) {
      return cachedIndex;
    }
    // 修改原缓存数组
    const [cached] = this.cacheList.splice(cachedIndex, 1);
    // 把最近读取的放在第一位
    this.cacheList.unshift(cached);
    return cached.value;
  }

  /**
   * @description: 缓存的修改
   * @param {number} key
   * @param {number} value
   * @return {void}
   */
  put(key: number, value: number): void {
    // 首先查找是否原先已缓存
    const cachedIndex = this.cacheList.findIndex((item: ICacheItem): boolean => item.key === key);
    // 如果已缓存，就更新值
    if (cachedIndex !== -1) {
      // 删除原先的缓存内容
      this.cacheList.splice(cachedIndex, 1);
      // 进行更新，放到最近的位置
      this.cacheList.unshift({ key, value });
      return;
    } else {
      this.cacheList.unshift({ key, value });
      // 如果超出了，就移除最早缓存的
      this.cacheList.length > this.maxCacheLength && this.cacheList.pop();
      return;
    }
  }
}
```

* 应用 LRU 并打印出结果来验证。

```typescript
const lru = new LRUCacheByArray(5);
console.log(lru.put(1, 1)); // undefined
console.log(lru.get(2)); // -1
console.log(lru.get(1)); // 1
console.log(lru.put(3, 3)); // undefined
console.log(lru.put(4, 4)); // undefined
console.log(lru.get(3)); // 3
```

## 链表和数组的区别

* 链表适合插入和删除，时间复杂度 `O(1)`.
* 数组支持随机访问，根据下标随机访问的时间复杂度为 `O(1)`.

### 访问的区别

* **随机访问**：数组在内存中是按顺序存放的，可以通过下标直接定位到某一个元素存放的位置。
* 顺序访问：链表在内存中不是按顺序存放的，而是通过指针连在一起，为了访问某一元素，必须从链头开始顺着指针才能找到某一个元素。

## 链表实践的技巧

### 技巧一：理解指针或引用的含义

* 指针或引用就是存储所指对象的内存地址。
* 将某个变量赋值给指针，实际上就是将这个变量的地址赋值给指针，或者反过来说，指针中存储了这个变量的内存地址，指向这个变量，通过指针就能找到这个变量。

### 技巧二：警惕指针丢失和内存泄漏

* 错误示范：

  ```javascript
	p.next = x;
	x.next = p.next;
	```

* 在给指针赋值的时候，需要格外注意赋值的顺序。稍有不慎可能会将指针丢失，进而引发问题。

### 技巧三：利用哨兵简化实现难度

* 链表中，插入第一个结点和删除最后一个结点都需要特殊处理。通过这样的方式实现会很繁琐，也会比较容易出错。
* 可以通过定义前哨结点或叫哑结点的方式来解决。
* 如果我们引入哨兵结点，在任何时候，不管链表是不是空，head指针都会一直指向这个哨兵结点。我们也把这种有哨兵结点的链表叫**带头链表**。相反，没有哨兵结点的链表就叫作**不带头链表**。
* 哨兵结点是不存储数据的。因为哨兵结点一直存在，所以插入第一个结点和插入其他结点，删除最后一个结点和删除其他结点，都可以统一为相同的代码实现逻辑了。

![[带头链表.png]]

### 技巧四：重点留意链表的边界问题

1. 链表为空时
2. 链表只包含一个结点时
3. 链表只包含两个结点时
4. 代码逻辑在处理头结点和尾结点时是否正常工作

### 技巧五：举例画图，辅助思考

![[链表实践的技巧之举例画图.png]]

### 技巧六：多写多练

* 下面几个案例是链表最基础也是最常用的算法实践，要求必须倒背如流，考虑到各种边界场景，不允许有任何出错。

#### 单链表翻转

* 时间复杂度 `O(n)`, 空间复杂度 `O(1)`.

```typescript
/**
 * 翻转单链表
 * @param head 
 * @returns 
 */
function reverseLinkedList(head: ListNode | null): ListNode | null {
  if (head === null || head.next === null) {
    return head;
  }

  let node = head;
  let prev = null;

  while (node !== null) {
    // 保存链表的指针引用
    const temp = node.next;
    node.next = prev;
    // 前指针更新到当前节点
    prev = node;
    // 遍历更新到下一节点
    node = temp;
  }

  // 此时的 node 是 head 遍历到的尾结点，但因为设置了指针，所以 node 是最后结点，实际 prev 是反转链表后的头结点
  return prev;
}
```

#### 获取链表的中间节点

* 时间复杂度 `O(n)`, 空间复杂度 `O(1)`.

```typescript
/**
 * 获取链表的中间节点
 * 快慢指针
 * @param head 
 * @returns 
 */
function getMiddleNode(head: ListNode | null): ListNode | null {
  if (head === null || head.next === null) {
    return head;
  }

  let slow = head;
  let fast = head;

  while (fast && fast.next !== null) {
    slow = slow.next;
    fast = fast.next.next;
  }

  return slow;
}
```

#### 检测链表中是否有环

* 时间复杂度 `O(n)`, 空间复杂度 `O(1)`.

```typescript
/**
 * 检测链表中是否有环
 * 快慢指针
 * @param head 
 * @returns 
 */
function hasLoop(head: ListNode | null): ListNode | null {
  if (head === null || head.next === null) {
    return false;
  }
  
  let slow = head;
  let fast = head.next;

  while (slow !== fast) {
    if (fast === null || fast.next === null) return false;
    slow = slow.next;
    fast = fast.next.next;
  }

  return true;
}
```


#### 删除链表倒数第 n 个结点

* 时间复杂度 `O(n)`, 空间复杂度 `O(1)`.

```typescript
/**
 * 删除倒数第 n 个结点
 * 先循环得到快指针的位置
 * 双指针
 * @param head 
 * @param n 
 * @returns 
 */
function deleteN(head: ListNode | null, n: number): ListNode | null {
  if (head === null) {
    return head;
  }

  let node = new ListNode(-1);
  // 注意这里需要使用哑结点，否则可能会导致快指针溢出
  let slow = node;
  let fast = head;
  node.next = head;

  let i = 0;

  while (i < n) {
    fast = fast.next;
    i++;
  }

  while (fast !== null) {
    slow = slow.next;
    fast = fast.next;
  }

  slow.next = slow.next.next;

  return node.next;
}
```

#### 合并两个有序链表

* 时间复杂度 `O(n)`, 空间复杂度 `O(1)`.

```typescript
/**
 * 合并两个有序链表
 * @param l1 
 * @param l2 
 * @returns 
 */
function mergeTwoLinkedList(l1: ListNode | null, l2: ListNode | null): ListNode | null {
  if (l1 === null) {
    return l2;
  }
  if (l2 === null) {
    return l1;
  }

  let head;
  // 设置哑结点
  let node = new ListNode(-1);
  // 设置起始位置
  if (l1.val <= l2.val) {
    head = l1;
    l1 = l1.next;
  } else {
    head = l2;
    l2 = l2.next;
  }
  // 哑节点指针指向合并的链表
  node.next = head;

  // 当有一个链表遍历结束就循环结束，后面可以直接把指针指向剩余的结点，因为剩余的链表也是有序且大于当前链表的
  while (l1 && l2) {
    if (l1.val <= l2.val) {
      head.next = l1;
      l1 = l1.next;
    } else {
      head.next = l2;
      l2 = l2.next;
    }
    head = head.next;
  }

  head.next = l1 ? l1 : l2;

  // 返回哑节点的指针
  return node.next;
}
```

#Tech #ALG #ADT