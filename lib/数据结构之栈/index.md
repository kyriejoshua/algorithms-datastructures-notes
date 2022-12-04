# 栈

## 简言

> 学习曲线：★★★

## 什么是栈

* 从操作特性来看，栈是操作受限的**线性表**，**只允许在一端插入和删除数据**。
* 在 JS 中，可以通过数组或链表来实现栈这种数据结构。但是栈也有着自身**存在的意义**，*因为特定的数据结构是对特定场景的抽象*。而且链表和数组已经暴露了太多的操作接口，操作上比较灵活，也意味着使用时会比较不可控，自然也会比较容易出错。

### 栈的应用场景

* **当某个数据集合只涉及在一端插入和删除数据，并且满足*后进先出，先进后出*的特性，这时就适合使用栈。**

## 如何实现栈

* 使用数组或者链表都可以实现栈。
	* 用数组实现的栈称为**顺序栈**。
	* 用链表实现的栈称为**链式栈**。

### 顺序栈的 TS 实现

* 基于数组的顺序栈，下面是 TS 实现：

```typescript
/**
 * 基于数组实现的顺序栈
 */
class ArrayStack<T> {
  private items: T[];
  private length: number;
  private count: number;

  public constructor(n: number) {
    this.items = [];
    this.length = n;
    this.count = 0;
  }

  /**
   * 栈的推入
   * @param value 
   * @returns 
   */
  public push(value: T):boolean {
    // 因为是插入后 count++, 所以可以直接比较长度，而不用减一
    if (this.count === this.length) {
      return false;
    }
    this.items[this.count] = value;
    // this.items.push(value);
    this.count++;
    return true;
  }

  /**
   * 栈的弹出
   * @returns 
   */
  public pop():T | null  | undefined {
    if (this.count === 0) {
      return null;
    }
    this.count--;
    const value: T = this.items[this.count]; // 不用数组 API 的方式，需要将 count 先减去多的
		this.items.length = this.count; // 注意一定要清除
    // return this.items.pop();
		return value;
  }
}

const stack = new ArrayStack(3);
console.info(stack);
stack.push(2);
stack.push(11);
console.info(stack);
stack.pop();
console.info(stack);
```

* 更新的栈的实现可查看 [Github](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Stack/index.ts)，包括使用两个队列模拟实现栈。

#### 支持动态扩容的顺序栈

* 当数组空间不足够时，就会重新申请一块更大的内存，将原来数组中的数据统统拷贝过去。这就实现了支持动态扩容的数组。
* 所以要实现动态扩容的顺序栈，只需要底层依赖一个支持动态扩容的数组就可以。

![[动态扩容.png]]

#### 动态扩容栈的实际应用-实现最小栈

* 使用数组实现最小栈。这里不考虑数组的动态扩容的具体实现。
* 来自 [LeetCode-155](https://leetcode-cn.com/problems/min-stack/)

```typescript
/**
 * 用纯数组的方式解决
 * 时间复杂度 O(1)
 * 空间复杂度 O(n)
 */
class MinStack {
  private stack: number[]; // 数据栈
  private minStack: number[]; // 辅助栈：用于保存最小栈，需要把每个值都保存，最小值在栈顶

  public constructor() {
    this.stack = [];
    this.minStack = [Number.MAX_SAFE_INTEGER];
  }

  /**
   * 栈的推入
   * @param value
   * @returns
   */
  public push(value: number): void {
    this.stack.push(value);
    // 注意这里推入的并不一定是传入的值，如果这个值比当前值大，那么推入会是原先的较小值
    // 这么做的目的是为了保存辅助栈和数据栈的同步
    this.minStack.push(Math.min(this.minStack[this.minStack.length - 1], value));
  }

  /**
   * 栈的弹出（在非空栈上调用）
   * @returns
   */
  public pop(): void {
    this.stack.pop();
    this.minStack.pop();
  }

  /**
   * 获取栈顶元素
   * @returns
   */
  public top(): number {
    return this.stack[this.stack.length - 1];
  }

  /**
   * 获取栈中的最小值
   * 时间复杂度 O(1)
   * @returns
   */
  getMin(): number {
    return this.minStack[this.minStack.length - 1];
  }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * var obj = new MinStack()
 * obj.push(val)
 * obj.pop()
 * var param_3 = obj.top()
 * var param_4 = obj.getMin()
 */
```

#### 分析动态扩容栈的复杂度

* 出栈操作的时间复杂度 `O(1)`. 因为不涉及数据的搬移和内存的重新申请。
* 当栈中有空闲空间时，入栈操作的时间复杂度是 `O(1)`.
* 当栈中空间不够时，入栈操作的时间复杂度变成了 `O(n)`.
* 也就是入栈操作的最好情况时间复杂度是 `O(1)`，最坏情况时间复杂度 `O(n)`.

##### 摊还分析法分析

* 定义不涉及内存搬移的入栈操作为 `simple-push` 操作，时间复杂度为 `O(1)`。
* 如果当前栈大小为 K，并且已满，当再有新的数据要入栈时，就需要重新申请 2 倍大小的内存，并且做K个数据的搬移操作，然后再入栈。但是，接下来的 K-1 次入栈操作，我们都不需要再重新申请内存和搬移数据，所以这 K-1 次入栈操作都只需要一个 `simple-push` 操作就可以完成。

![[入栈时间复杂度.png]]

* 这 K 次入栈操作，总共涉及了 K 个数据的搬移，以及 K 次 `simple-push` 操作。将 K 个数据搬移均摊到 K 次入栈操作，那每个入栈操作只需要一个数据搬移和一个 `simple-push` 操作。以此类推，入栈操作的均摊时间复杂度就为 `O(1)`。
* **均摊时间复杂度一般都等于最好情况时间复杂度。**
* 因为在大部分情况下，入栈操作的时间复杂度O都是 `O(1)`，只有在个别时刻才会退化为 `O(n)`，所以把耗时多的入栈操作的时间均摊到其他入栈操作上，平均情况下的耗时就接近 `O(1)`。

## 栈的实际应用

### 栈在函数调用中的应用

* **操作系统给每个线程分配了一块独立的内存空间，这块内存被组织成“栈”这种结构,用来存储函数调用时的临时变量。** 每进入一个函数，就会将临时变量作为一个栈帧入栈，当被调用函数执行完成，返回之后，将这个函数对应的栈帧出栈。

```java
int main() {
   int a = 1; 
   int ret = 0;
   int res = 0;
   ret = add(3, 5);
   res = a + ret;
   printf("%d", res);
   reuturn 0;
}

int add(int x, int y) {
   int sum = 0;
   sum = x + y;
   return sum;
}
```

* 从代码中我们可以看出，`main()` 函数调用了 `add()` 函数，获取计算结果，并且与临时变量 a 相加，最后打印 `res` 的值。为了让你清晰地看到这个过程对应的函数栈里出栈、入栈的操作，我画了一张图。图中显示的是，在执行到 `add()` 函数时，函数调用栈的情况。

![[函数中调用.png]]

### 栈在表达式求值中的应用

* 这里我们将算术表达式简化为只包含加减乘除四则运算，来方便理解。
* **编译器通过两个栈来实现基本的四则运算。**
	* 其中一个是保存操作数的栈，另一个是保存运算符的栈。
* 从左向右遍历表达式，遇到数字就直接压入操作数栈，遇到运算符，就与运算符栈顶的元素进行比较
	* 如果比运算符栈顶元素的优先级高，就把当前运算符压入栈，如果比运算符栈顶元素优先级低或者相同，就从运算符栈中取得栈顶运算符，从操作数栈顶取两个操作数，然后把它们进行计算，把计算结果压入操作数栈，继续进行比较（从刚才终止的运算符开始）。
* 下面是 **3+5*8-6** 的计算过程：

![[表达式计算过程.png]]

* 依次加入后，乘号的优先级比加号高，所以继续推入栈，直到遇到减号，因为优先级比栈内的元素小，所以先放一边，先计算栈内的运算结果。计算之后，再执行减号相关的操作。


### 栈在括号匹配中的应用

* 可以借助栈来检查表达式中的括号是否匹配。也就是检查括号的合法性。
* 用栈来保存未匹配的左括号，从左到右一次扫描字符串。当扫描到左括号时，将其压入栈，当扫描到右括号时，从栈顶取出一个左括号。如果可以匹配，比如 **`“(”`** 和 **`“)”`** 匹配，**`“[”`** 和 `“]”` 匹配，**`“{”`** 和 **`“}”`** 匹配，则继续扫描剩下的字符串。如果扫描的过程中遇到不能匹配的右括号，或者栈中没有数据，则说明为非法格式。
* 当所有的括号都扫描完成之后，如果栈为空，则说明字符串为合法格式；否则，说明有未匹配的左括号，为非法格式。

#### LeetCode 有效的括号

* 对应有 [LeetCode-20](https://leetcode.cn/problems/valid-parentheses/)：有效的括号.
* 下面是实现，核心是模拟栈后入先出的行为：

```typescript
/*
 * @lc app=leetcode id=20 lang=typescript
 * [20] Valid Parentheses
 */
/**
 * @description: 栈的思路
 * @param {string} s
 * @return {boolean}
 */
function isValid(s: string): boolean {
  // 定义对象方便存储
  const parentMap = {
    '}': '{',
    ']': '[',
    ')': '(',
  };
  const stack = [];

  for (const str of s) {
    // 如果遇到开括号就推入栈
    if (str === '{' || str === '[' || str === '(') {
      stack.push(str);
      continue;
    }
    // 如果遇到闭括号就判断栈尾是不是对应的括号，不对应时可直接返回结果，对应则弹出栈尾的括号
    if (stack[stack.length - 1] !== parentMap[str]) return false;
    stack.pop();
  }

  return !stack.length;
}
```

#### LeetCode 有效的括号字符串

* 另一道比较有代表性的题目，使用双栈的思路解决。
* [LeetCode-678](https://leetcode.cn/problems/valid-parenthesis-string/)

```typescript
/*
 * @lc app=leetcode id=678 lang=typescript
 * [678] Valid Parenthesis String
 */
/**
 * @description: 双栈的思路
 * 时间复杂度 O(n)
 * 空间复杂度 O(n)
 * @param {string} s
 * @return {boolean}
 */
function checkValidString(s: string): boolean {
  // 分别使用两个栈来保存左括号和星号
  const leftStack: number[] = [];
  const asteriskStack: number[] = [];

  for (let i = 0; i < s.length; i++) {
    const str = s[i];

    // 存入对应的栈中
    // !注意存入的是索引而不是字符，方便后续的比较
    if (str === '(') {
      leftStack.push(i);
    } else if (str === '*') {
      asteriskStack.push(i);
    } else {
      // 遇到右括号，就优先查找匹配的左括号，如果没有再匹配星号，如果还是没有，说明不是合法的括号字符串
      if (leftStack.length) {
        leftStack.pop();
      } else if (asteriskStack.length) {
        asteriskStack.pop();
      } else {
        return false;
      }
    }
  }

  // 确认保存的两个栈中的索引位置，如果左括号的位置在星号之后，说明无法形成有效括号
  while (leftStack.length && asteriskStack.length) {
    const leftIndex = leftStack.pop();
    const asteriskIndex = asteriskStack.pop();
    if (leftIndex > asteriskIndex) return false;
  }

  // 说明还有未匹配的括号
  return leftStack.length === 0;
}
// 这道题还有贪心的实现，更加巧妙，空间复杂度更低
// 也有动态规划的实现，但效率并不高
```

### 如何实现浏览器的前进后退

* 我们使用两个栈，X 和 Y，我们把首次浏览的页面依次压入栈 X，当点击后退按钮时，再依次从栈 X 中出栈，并将出栈的数据依次放入栈 Y。当我们点击前进按钮时，我们依次从栈 Y 中取出数据，放入栈 X 中。当栈 X 中没有数据时，那就说明没有页面可以继续后退浏览了。当栈 Y 中没有数据，那就说明没有页面可以点击前进按钮浏览了。
* 比如你顺序查看了 a，b，c 三个页面，我们就依次把 a，b，c 压入栈，这个时候，两个栈的数据就是这个样子：

![[浏览器中的栈1.png]]

* 当你通过浏览器的后退按钮，从页面 c 后退到页面 a 之后，我们就依次把 c 和 b 从栈 X 中弹出，并且依次放入到栈 Y。这个时候，两个栈的数据就是这样：

![[浏览器中的栈2.png]]

* 这个时候你又想看页面 b，于是你又点击前进按钮回到 b 页面，我们就把 b 再从栈 Y 中出栈，放入栈 X 中。此时两个栈的数据是这个样子：

![[浏览器中的栈3.png]]

* 这个时候，如果你通过页面 b 又跳转到新的页面 d 了，页面 c 就无法再通过前进、后退按钮重复查看了，所以需要清空栈 Y。此时两个栈的数据这个样子：

![[浏览器中的栈4.png]]

#### 浏览器的操作

* 打开一个新页面，将页面 urlA 推入栈 x 中。
	* `[urlA], []`
* 当前页面前往新的地址，新地址的 urlB 推入栈 x 中。
	* `[urlA, urlB,], []`
* 当前页面前往新的地址，新地址的 urlC 推入栈 x 中。
	* `[urlA, urlB, urlC], []`
* 点击返回，x 栈顶弹出元素 urlC，放入栈 y 中。
	* `[urlA, urlB, ], [urlC]`
* 点击返回，x 栈顶弹出元素 urlB，放入栈 y 中。
	* `[urlA,  ], [urlC, urlB,]`
* 点击前进，y 栈顶弹出元素 urlB，放入栈 x 中。
	* `[urlA, urlB,], [urlC, ]`
* 再次打开新页面地址，新地址的 urlD 推入栈 x 中。栈 y 清空内容，也就是清除了 urlC.
	* `[urlA, urlB, urlD], []`

## 扩展

* *本文中有提到，使用函数调用栈来保存临时变量，这里为何使用栈来保存而不是其他数据呢？*
	* 因为函数中的临时变量在内存中的存活时间满足 **“后入先出”** 的顺序，所以是最适合用栈这种数据结构来实现的。其他数据结构也能做到，但不如栈的实现这样方便。

#ALG 