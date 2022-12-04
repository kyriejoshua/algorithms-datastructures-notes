# 递归

> 学习曲线：★★★

## 什么是递归

* **在函数内调用函数本身。**

### 递归的三个条件

#### 1. 一个问题的解可以分解为几个子问题的解

#### 2. 当前问题与分解的子问题，除了数据规模不同，求解思路完全一样

#### 3. 存在递归终止条件

## 如何实现递归

* 实现递归代码的两个关键：
	* **写出递归公式**
	* **找到终止条件**

### 举个递归的例子

* 下面我们以爬楼梯作为例子，计算爬固定的楼梯，有多少种爬楼梯的解法。

#### 爬楼梯

* 假如这里有 n 个台阶，每次你可以跨 1 个台阶或者 2 个台阶，请问走这 n 个台阶有多少种走法？
* 如果有 7 个台阶，你可以 2，2，2，1 这样上去，也可以 1，2，1，1，2 这样上去，总之走法有很多，那如何用编程来解总共有多少种走法呢？
* **可以根据第一步的走法把所有走法分为两类。** 第一类是第一步走了 1 个台阶，另一类是第一步走了 2 个台阶。所以 n 个台阶的走法就等于先走 1 阶后，n - 1 个台阶的走法加上先走 2 阶后，n - 2 个台阶的走法。用公式表示就是：

```typescript
f(n) = f(n - 1) + f(n - 2)
```

* 有了递推公式，递归代码基本上就已经完成一半。再来看下终止条件。当有一个台阶时，我们不需要再继续递归，就只有一种走法。所以 `f(1) = 1` 这是一个基本条件。但这点作为递归的终止条件足够吗？我们可以用 n = 2，n = 3 这样比较小的数来实践验证。
* 当 n = 2 时，`f(2) = f(1) + f(0)`。如果递归终止条件只有一个 `f(1) = 1`，那 f(2) 就无法求解。所以除 `f(1) = 1` 这一个递归终止条件外，还要有 `f(0) = 1`，表示走 0 个台阶有一种走法，不过这样就不太符合正常的逻辑思维。所以，可以把 `f(2) = 2` 作为一种终止条件，表示走 2 个台阶，有两种走法，一步走完或者分两步来走。
* 所以，递归终止条件就是 **`f(1) = 1，f(2) = 2`**。
* 小结一下就是：

```typescript
f(1) = 1;
f(2) = 2;
f(n) = f(n - 1) + f(n - 2)
```

* 将其转化为具体的 ts 代码并执行验证：

```typescript
function f(n: number): number {
  if (n === 1) {
    return 1;
  }
  if (n === 2) {
    return 2;
  }

  return f(n - 1) + f(n - 2); 
}

f(4); // 5
```

* 编写递归代码的关键在于：
	* **充分理解问题，找到把大问题转化为小问题的规律，再推敲出终止条件。**
	* **据此试着写出伪代码，包含递归公式和终止条件，再实现成对应语言的代码。**
* 如果一个问题 A 可以分解为若干子问题 B、C、D，你可以假设子问题 B、C、D 已经解决，在此基础上思考如何解决问题 A。而且，你只需要思考问题 A 与子问题 B、C、D 两层之间的关系即可，不需要一层一层往下思考子问题与子子问题，子子问题与子子子问题之间的关系。屏蔽掉递归细节，这样理解起来就会简单许多。
* **递归其实需要抽象出上层的核心问题的解决方式，而不要关注细节的实现。**

#### LeetCode 爬楼梯

* 这道经典题在 [LeetCode-70](https://leetcode.cn/problems/climbing-stairs/) 中也有收录。
* 下面的注意事项小节里会分享带备忘录的递归实现，当然这题更适合使用动态规划来解，在后面的章节会学习到。

## 递归的注意事项

### 1. 递归代码需警惕堆栈溢出

* **函数调用会使用栈来保存临时变量。** 每调用一个函数，都会将临时变量封装为栈帧压入内存栈，等函数执行完成返回时，才会出栈。系统栈或者虚拟机的栈空间一般都不大。如果递归求解的数据规模很大，调用层次很深，一直压入栈而来不及弹出，就会有堆栈溢出的风险。

#### 如何避免堆栈溢出

* 可以通过在代码中限制递归调用的最大深度这种方式来避免这个问题。
* 我们可以借助一个全局变量来实现最大深度的限制，当然也可以使用闭包的方式来实现。
* 这是使用全局变量的方式：

```typescript
/**
 * @description: 限制最大深度，使用全局变量的方式
 * @param {number} n
 * @return {*}
 */
window.__depth__= 0;
function fn(n: number):number {
  window.__depth__++;
  if (window.__depth__ >= 100000000) {
    throw new Error("Max number is 100000000");
  }
  if (n === 1) {
    return 1;
  }
  if (n === 2) {
    return 2;
  }

  return fn(n - 1) + fn(n - 2); 
}

fn(38); // 正常输出 78176337
fn(39); // 抛出异常
```

* 这是使用闭包的方式：

```typescript
/**
 * @description: 限制最大深度，使用闭包的方式
 * @param {number} n
 * @return {number}
 */
const fnByClosure = (() => {
  let _depth = 0;
  
  return function (n: number): number {
    _depth++;
    if (_depth >= 100000000) {
      throw new Error("Max number is 100000000");
    }
    if (n === 1) {
      return 1;
    }
    if (n === 2) {
      return 2;
    }
    return fnByClosure(n - 1) + fnByClosure(n - 2); 
  };
})();

fnByClosure(38); // 正常展示 78176337
fnByClosure(39); // 抛出异常
```

* 这个方案的难点在于获取上限值，上面的例子比较简单，可以轻易获取限制的值。而具体场景里的上限往往是动态的。
* 因为最大允许的递归深度跟当前线程剩余的栈空间大小有关，事先无法计算。如果实时计算，代码过于复杂，就会影响代码的可读性。所以，如果最大深度比较小，比如 10、50，就可以用这种方法，否则这种方法并不是很实用。

### 2. 递归代码要避免重复计算

![[递归的树.png]]

* 从图中可以直观地看到，想要计算 `f(5)`，需要先计算 `f(4)` 和 `f(3)`，而计算 `f(4)` 还需要计算 `f(3)`，因此，`f(3)` 就被计算了多次，这就是重复计算问题。
* 为了避免重复计算，我们可以通过一个数据结构（比如散列表）来保存已经求解过的 `f(k)`。当递归调用到 `f(k)` 时，先确认是否已经求解过了。如果是，则直接从散列表中取值返回，不需要重复计算，这样就能避免重复计算的问题。
* 对应到 js 中，就可以用对象或者 Map 来做缓存处理。
* 下面是使用 Map 做缓存的实现，也就是对上文的实现进行了优化：

```typescript
/**
 * @description: 爬楼梯解法，使用 Map 数据结构 
 * @param {number} stair
 * @param {any} stairMap
 * @return {number}
 */
function climbStairsByMap(stair: number, stairMap?: any ): number {
  if (stair <= 2) {
    return stair;
  }
  stairMap = stairMap || new Map();

  if (stairMap.get(stair)) {
    return stairMap.get(stair);
  }

  // 使用 Map 缓存处理
  const stair1: number = climbStairsByMap(stair - 1, stairMap);
  const stair2: number = climbStairsByMap(stair - 2, stairMap);
  stairMap.set(stair - 1, stair1);
  stairMap.set(stair - 2, stair2);

  return stair1 + stair2;
}

// 用时间标记来观察使用缓存和不使用缓存的差距
console.time();
climbStairsByMap(43);
console.timeEnd(); // default: 0.1689453125 ms

// 附赠使用对象缓存的闭包的实现
/**
 * @description: 使用闭包的方式避免产生全局变量等，更为优雅但不易理解
 * @param {*} function
 * @return {number}
 */
const climbStairsByClosure = (function () {
  let memory = new Object();
  return function fn(n: number): number {
    if (memory[n] !== undefined) {
      return memory[n];
    }
    return memory[n] = n === 1 || n === 2 ? n : climbStairsByClosure(n - 1) + climbStairsByClosure(n - 2);
  }
})();
```

* 下面我们和不使用缓存方式的实现作一次对比，查看日志。
* 可以看到在传入参数 **43** 的时候，执行时间的差距已经十分明显。

```typescript
/**
 * @description: 不使用缓存的爬楼梯最简版
 * @param {number} stair
 * @return {number}
 */
function climbStairs(stair: number): number {
  if (stair <= 2) {
    return stair;
  }

  return climbStairs(stair - 1) + climbStairs(stair - 2);
}

console.time();
climbStairs(43);
console.timeEnd(); // default: 3372.154052734375 ms
```

### 3. 其他注意事项

* 除了上述两者，递归在分析算法的空间复杂度上也有所影响。
* 在时间效率上，由于递归代码里多了很多函数调用，当这些函数调用的数量较大时，就会积累成一个比较可观的时间成本。在空间复杂度上，因为递归调用一次就会在内存栈中保存一次现场数据，所以在分析递归代码空间复杂度时，需要额外考虑这部分的开销，比如前面讲到的电影院递归代码，空间复杂度并不是 `O(1)`，而是 `O(n)`。

## 递归代码改写成非递归代码

* 这里顺带可以想想什么场景是适合用递归的。
* 下面是使用循环改写爬楼梯的实现，核心是动态规划的思想，具体分析在动态规划那一节我们再回顾。

```typescript
/**
 * @description: 使用迭代的方式替代递归实现爬楼梯
 * 实际就是使用了动态规划
 * 这个实现所花费的时间是最短的，比先前缓存的方案也会短一些
 * @param {number} n
 * @return {number}
 */
function climbStairsByIterator(n: number): number {
  if (n <= 2) return n;
  
  let ret = 0; // 当前的方案数量
  let pre = 2; // 上一个楼梯数的方案
  let cur = 1; // 当前楼梯数新增的方案

  for (let i = 3; i <= n; ++i) {
    ret = pre + cur; // 当前的方案等于上个楼梯的方案加上新增的方案
    cur = pre; // 更新新增的方案
    pre = ret; // 更新楼梯的方案，把当前的当成是下个楼梯的前置
  }

  return ret;
}

console.time();
climbStairsByIterator(43);
console.timeEnd(); // default: 0.0810546875 ms
```

* 查看执行时间的日志可以发现，循环的实现比使用缓存的递归实现还要快一些。
* 因为递归本身就是借助栈来实现的，只不过我们使用的栈是系统或者虚拟机本身提供的，所以我们没有感知。如果我们自己在内存堆上实现栈，手动模拟入栈、出栈过程，这样任何递归代码都可以改写成循环的方式。
* 事实上，也可以通过检测环的方式来避免堆栈溢出或者无限递归的问题。这点具体在后面会讲到。

## 小结

* 递归代码虽然简洁高效，但也有很多弊端。比如，堆栈溢出、重复计算、函数调用耗时多、空间复杂度高等。所以，在编写递归代码的时候，一定要注意控制好这些副作用，否则本来实用的递归很容易反噬我们。

#ALG 