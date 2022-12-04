# 队列

## 简言

> 学习曲线：★★★

## 什么是队列

* **队列和栈一样，也是一种操作受限的线性表数据结构。**
* 队列的主要操作是入队和出队。

![[队列和栈.png]]

## 顺序队列和链式队列

### 顺序队列

* 和栈一样，队列可以用数组来实现，也可以用链表来实现，也可以用链表来实现。用数组实现的栈叫做顺序栈，用链表实现的栈叫链式栈。同样，**用数组实现的队列叫做顺序队列，用链表实现的队列叫做链式队列。**
* 用数组实现一个最简单的队列（直接使用 js 数组的 api 而不考虑头尾，实际上是应当考虑的）：

```typescript
/**
 * 使用数组实现队列
 * 可以设置队列的大小
 */
class ArrayQueue<T> {
  private queue: T[];
  private length: number;

  constructor(n: number) {
    this.queue = [];
    this.length = n;
  }

  /**
   * 入队
   * @param item 
   */
  enqueue(item: T) {
    this.queue.push(item);
    if (this.queue.length === this.length) {
      this.dequeue();
    }
  }

  /**
   * 出队
   * @returns 
   */
  dequeue(): T | undefined {
    return this.queue.shift();
  }
}
```

* 上面是使用数组 API 实现的队列。如果不使用数组的 API 而是直接来实现，就是下面这样。

```typescript
/**
 * 数组实现队列，不使用数组 API
 */
class ArrayQueue<T> {
  private length: number; // 队列长度
  private queue: T[]; // 队列
  private head: number; // 队首指针
  private tail: number; // 队尾指针
  
  constructor(n: number) {
    this.length = n;
    this.queue = [];
    this.head = 0;
    this.tail = 0;
  }
  
  /**
   * 入队
   * 返回是否入队成功
   * @param item 
   * @returns 
   */
  enqueue(item: T): boolean {
    if (this.tail === this.length) {
      return false;
    }
    this.queue[this.tail] = item;
    this.tail++;
    return true;
  }
  
  /**
   * 出队操作
   * 返回队首值的同时，将指针往后移
   * @returns 
   */
  dequeue(): T | undefined | null {
    if (this.head === this.tail) {
      return null;
    }
    const item = this.queue[this.head];
    this.head++;
    return item;
  }
}
```

* **这是队列更完整的实现[链接🔗](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Queue/index.ts)。同时包含了使用双栈实现队列的代码。**
* 对于栈来说，我们只需要一个栈顶指针就可以了，但是队列需要两个指针，一个是 `head` 指针，指向队头，一个是 `tail` 指针，指向队尾。因此上面的实现定义了这两个变量。所以实际上第一种实现是不够完善的。
* 可以观察下面示意图的变化来理解两个指针的变化。
* 下面是初始化队列后，不断进行入队操作。

![[队列的指针示意1.png]]

* 下图是进行了两次出队操作。

![[队列的指针示意2.png]]

* 随着不断地进行入队和出队的操作，`head` 和 `tail` 都会持续往后移动。当 `tail` 移动到最右边的时候，即使左边数据还有空闲空间，也无法继续往队列中添加数据了。
* 为了解决这个问题，只需要再集中触发一次数据搬移操作。类似之前数组的处理。因此出队函数 `dequeue` 保持不变，入队函数进行一些优化。

```typescript
/**
 * 优化后的入队
 * 在队列的两个指针间数据填满，但数组未填满时，可以将这些数据搬移到数组中 0 到 tail-head 的位置
 * 也就是 tail === length，但 head 不为 0 的场景
 * 返回是否入队成功
 * @param item 
 * @returns 
 */
enqueue(item: T): boolean {
  if (this.tail === this.length) {
    // 说明从头到尾都空间都占满了
    if (this.head === 0) {
      return false;
    }
    for (let i = this.head; i < this.tail; i++) {
      this.queue[i - this.head] = this.queue[i];
    }
    // this.tail = this.queue.length;
    this.tail -= this.head;
    this.head = 0;
  }
  this.queue[this.tail] = item;
  this.tail++;
  return true;
}
```

* 图片演示更加直观一些。

![[队列的数据搬移.png]]

### 链式队列

* 基于链表的实现，我们同样需要两个指针：`head` 指针和 `tail` 指针。它们分别指向链表的第一个结点和最后一个结点。如图所示，入队时，`tail->next = new_node`, `tail = tail->next` 出队时，`head = head->next`。后续实现再贴入代码。

![[链表实现队列.png]]

## 循环队列

* 刚才我们用数组实现队列的时候，在 `tail === n` 时，会有数据搬移操作。
* 而循环队列就是为了避免数据搬移操作的实现。

![[循环队列.png]]

![[循环队列2.png]]

* **循环队列的实现重点在于队列队空和队满的条件判断。**
* **队空的判定方式和常规队列是一样的，`tail === head`.**
* 队满的判定方式则有所区别。观察下图，试着总结队满时的规律。

![[循环队列队满.png]]

* 就像图中画的队满的情况，`tail = 3`，`head = 4`，`n = 8`，所以总结一下规律就是：`(3 + 1) % 8 = 4`。多画几张队满的图，就可以发现，当队满时，**`(tail + 1) % n = head`**。
* 当循环队列满时，图中的 `tail` 指向的位置实际上是没有存储数据的。所以，循环队列实际上会浪费一个数组的存储空间。

### 循环队列的实现

* 对前面的队列实现稍加改动，就可以实现循环队列：

```typescript
/**
 * 数组实现循环队列
 */
class CircleQueue<T> {
  private length: number; // 队列长度
  private queue: T[]; // 队列
  private head: number; // 队首指针
  private tail: number; // 队尾指针

  constructor(n: number) {
    this.length = n;
    this.queue = [];
    this.head = 0;
    this.tail = 0;
  }

  /**
   * 入队
   * 返回是否入队成功
   * @param item 
   * @returns 
   */
   enqueue(item: T):boolean {
    if ((this.tail + 1) % this.length === this.head) {
      return false;
    }
    this.queue[this.tail] = item;
    this.tail = (this.tail + 1) % this.length;
    return true;
  }

  /**
   * 出队操作
   * 返回队首值的同时，将指针往后移
   * @returns 
   */
  dequeue(): T | undefined | null {
    if (this.head === this.tail) {
      return null;
    }

    const item = this.queue[this.head];
    this.head = (this.head + 1) % this.length;
    return item;
  }
}

const circleQueue = new CircleQueue(9);
circleQueue.enqueue(13);
console.info(circleQueue);
circleQueue.enqueue(7);
console.info(circleQueue);
circleQueue.enqueue(1);
console.info(circleQueue);
circleQueue.dequeue();
console.info(circleQueue);
```

* 更多实现在 [Github](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Queue/CircleQueue.ts).
* [ts 在线演示](https://www.typescriptlang.org/zh/play?#code/MYGwhgzhAEDCCWAnUBTAigVxVgPAFQD5oBvAKGmgAdF4A3MAFxWhBQDsBzBgCwC5o2GALYAjFIgDc0APTTogfDTA6EqB-VMBleuSo16TaAEcsWfngDaAXSmyFijdTqNm3FGAAm-QaPEW58wGmZgcGNACJSbLXtoBjB4EDdhMUkZb0A+HUDSDWAAezYIBkQMYAY0xAAKNmiPRABKEg0KHngIADpWTh5oAF4BCWqw7jr6-WxmdrNOihqehscXNugABhHR2obwyOm5jQBfFIppACod6p3oQFNFeQPoQBX4wD21QHozQDIVE8AEI0B8pTOAAUowRDAhaHgmH7eiBQDAwiEy0AO0mq7H6WEKfxQQiM5V4IjSaVYYDYVVGvwAZtBCoVFvVliBoABqaAARkqAFJur0mlxuG1Wu0SZNnJUyLiKECQWDoHiwCAICh5hRNriSbCUMYSWTTNMEUJJYylhFye1ieNSVrKTT6RrGuwWeqBaDsdksPNNhpdvtRodAF+K8kAsyaAHXkzldfIAeBUAIW6AGBVAG+mgBh-wBgOoFAAH6gDgVQDfnoDgVaYJCNM4UHLCijoHhoAAfaAYNgZvHwNgoZwFgQYEAgXmjeAE3W9Llsjl6sk8rr85NCwR19X23HpTIMX7-aaygzyzlOZzmLpzqY65dVqm06AMknMngWvvY1V20j20dZaDAJCoTADaYVgDucCvrBvcIAnOVOmeMSh6uW8WkhSXsgL4zp+pDAdeM71DCM6FNSADM4Hfqwf5sABQHPugYFflhr6-rBAyFAA7MhGQQD+aEYZBoEDMheHQYRcK0l+5GUf+gE0dhdG4SB3FYPUGZZmRmTsehnEMTxKRAA)

#### LeetCode

* 当然循环队列这样的经典应用也有对应的题目。[LeetCode-622](https://leetcode.cn/problems/design-circular-queue/)

## 并发队列和阻塞队列

* 有一些具有特殊特性的队列应用比较广泛，例如阻塞队列和并发队列。

### 阻塞队列

* 阻塞队列其实就是在队列基础上增加了阻塞操作。
	* 在队列为空的时候，从队头获取（读取）数据的操作会阻塞住，因为此时还没有数据可取，直到队列中有数据才能返回；
	* 如果队列已经满了，那么插入数据的操作就会被阻塞，直到队列中有空闲位置后再插入数据，然后再返回。
* 个人理解，这些过程是自动的，由阻塞队列内部实现。

![[阻塞队列.png]]

* 这里的定义很像“生产者-消费者模型”，所以可以使用阻塞队列来实现 **“生产者-消费者模型”**。
* 这种基于阻塞队列实现的“生产者-消费者模型”，可以有效地协调生产和消费的速度。当“生产者”生产数据的速度过快，“消费者”来不及消费时，存储数据的队列很快就会变满。这个时候，生产者就阻塞等待，直到“消费者”消费了数据，“生产者”才会被唤醒继续“生产”。
* 而且基于阻塞队列，我们还可以通过协调“生产者”和“消费者”的个数，来提高数据的处理效率。比如前面的例子，我们可以多配置几个“消费者”，来应对一个“生产者”。

![[阻塞队列2.png]]

### 并发队列

* 阻塞队列可以有多个消费者。也就是说，在多线程情况下，会有多个线程同时操作队列，这个时候就会存在线程安全问题，那如何实现一个线程安全的队列呢？
* **线程安全的队列我们叫作并发队列。**
* 最简单直接的实现方式是直接在 `enqueue()`、`dequeue()` 方法上加锁，但是锁粒度大并发度会比较低，同一时刻仅允许一个存或者取操作。实际上，基于数组的循环队列，利用 CAS 原子操作，可以实现非常高效的并发队列。这也是循环队列比链式队列应用更加广泛的原因。在后续实战学习 Disruptor 的时候，再深入探讨并发队列的应用。

### 基于队列和基于链表实现方式对于排队请求的区别

* 而线程池没有空闲线程时，新的任务请求线程资源时，线程池该如何处理？各种处理策略又是如何实现的呢？
* 我们一般有两种处理策略。
	* 第一种是非阻塞的处理方式，直接拒绝任务请求；
	* 另一种是阻塞的处理方式，将请求排队，等到有空闲线程时，取出排队的请求继续处理。
		* 那么如何存储排队的请求呢？
		* 基于链表的实现方式，可以实现一个支持无限排队的**无界队列（unbounded queue）**，但是可能会导致过多的请求排队等待，请求处理的响应时间过长。所以，针对响应时间比较敏感的系统，基于链表实现的无限排队的线程池是不合适的。
		* 而基于数组实现的**有界队列（bounded queue）**，队列的大小有限，所以线程池中排队的请求超过队列大小时，接下来的请求就会被拒绝，这种方式对响应时间敏感的系统来说，就相对更加合理。不过，设置一个合理的队列大小，也是非常有讲究的。队列太大导致等待的请求太多，队列太小会导致无法充分利用系统资源、发挥最大性能。
* 除了前面讲到队列应用在线程池请求排队的场景之外，**队列可以应用在任何有限资源池中**，用于排队请求，比如数据库连接池等。实际上，对于大部分资源有限的场景，当没有空闲资源时，基本上都可以通过“队列”这种数据结构来实现请求排队。
	* 对应到前端中，React 内部的任务调度也是应用到了队列这种数据结构。

## 小结

* **阻塞队列就是入队和出队的操作可以阻塞，并发队列就是队列的操作多线程安全。**
* 队列的应用比较广泛，比如一些有额外特性的队列，循环队列，并发队列，阻塞队列。它们在很多偏底层操作系统、架构和中间件的开发中有着关键的作用。

#ALG 