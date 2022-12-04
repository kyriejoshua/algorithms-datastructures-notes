# 深度优先和广度优先的搜索

## 简言

* 社交网络中，有一个六度分割理论。即你与世界上的另一个人间隔的关系不会超过六度，也就是平均只需要六步就可以联系到任何两个互不相识的人。
* 一个用户的一度好友就是他的好友，二度用户就是好友的好友，三度用户就是他的好友的好友的好友。在社交网络中，往往通过用户之间的连接关系，来实现推荐 *“可能认识的人”* 这一功能。

> 那么如何找到一个用户的所有三度好友关系（包含一度、二度好友）？
> 学习曲线：★★★☆

## 什么是搜索算法

* 算法是基于具体的数据结构之上的。深度优先算法和广度优先算法都是**基于非线性表数据结构例如图或树**上。
	* **而深度优先搜索算法和广度优先搜索算法都是基于图这种数据结构的。因为图这类数据结构的表达能力强，大部分涉及搜索的场景都可以抽象成图。**
* **不论是深度优先还是广度优先，都是可以应用在有向图和无向图上的。**
* 上一节提到，图的存储方式有两种，**邻接矩阵和邻接表**。下面我们用邻接表来存储图。
* 邻接表存储图的代码实现。下面是 TS 的实现：

```typescript
export default class Graph {
  private v: number; // 顶点的个数
  private adj: LinkedList<number>[]; // 邻接表链表

  constructor(v: number) {
    this.v = v;
    this.adj = new Array(v);
    for (let i = 0; i < v; i++) {
      this.adj[i] = new LinkedList();
    }
  }

  /**
   * @description: 添加边
   * 无向图需要两个方向都添加
   * @param {number} s
   * @param {number} t
   * @return {void}
   */
  public addEdge(s: number, t: number): void {
    this.adj[s].add(t);
    this.adj[t].add(s);
  }
}

const graph = new Graph(5);
graph.addEdge(1, 3);
console.log(graph);
```

* 其中，上面代码所使用的链表来自于前面的章节中的 [[Tech/技术笔记/《算法与数据结构》学习笔记/数据结构之数组与链表/index#单链表的实现|链表的实现]]。
* 下面是原文中的 Java 实现。

```java
public class Graph { // 无向图
  private int v; // 顶点的个数
  private LinkedList<Integer> adj[]; // 邻接表

  public Graph(int v) {
    this.v = v;
    adj = new LinkedList[v];
    for (int i=0; i<v; ++i) {
      adj[i] = new LinkedList<>();
    }
  }

  public void addEdge(int s, int t) { // 无向图一条边存两次
    adj[s].add(t);
    adj[t].add(s);
  }
}
```

## 广度优先搜索（BFS)

* **广度优先搜索（breadth-First-Search）**，简称 BFS。它类似一种地毯式层层推进的搜索策略，先查找离当前顶点最近的，然后是次近的，依次往外搜索。

![[BFS示意图.png]]

### 广度优先搜索的实现

* 原文中的 Java 实现，供参考。

```java
public void bfs(int s, int t) {
  if (s == t) return;
  boolean[] visited = new boolean[v];
  visited[s]=true;
  Queue<Integer> queue = new LinkedList<>();
  queue.add(s);
  int[] prev = new int[v];
  for (int i = 0; i < v; ++i) {
    prev[i] = -1;
  }
  while (queue.size() != 0) {
    int w = queue.poll();
   for (int i = 0; i < adj[w].size(); ++i) {
      int q = adj[w].get(i);
      if (!visited[q]) {
        prev[q] = w;
        if (q == t) {
          print(prev, s, t);
          return;
        }
        visited[q] = true;
        queue.add(q);
      }
    }
  }
}

private void print(int[] prev, int s, int t) { // 递归打印s->t的路径
  if (prev[t] != -1 && t != s) {
    print(prev, s, prev[t]);
  }
  System.out.print(t + " ");
}
```

* TS 版本的实现。
	* 下面是基础实现，最新的版本可查看 [Github](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/Graph/index.ts).

```typescript
import LinkedList from '@/LinkedList';

/**
 * @description: 无向图图
 * @return {*}
 */
export default class Graph {
  private v: number; // 顶点的个数
  private adj: LinkedList<number>[]; // 邻接表链表

  constructor(v: number) {
    this.v = v;
    this.adj = new Array(v);
    for (let i = 0; i < v; i++) {
      this.adj[i] = new LinkedList();
    }
  }

  /**
   * @description: 添加边
   * 无向图需要两个方向都添加
   * @param {number} s
   * @param {number} t
   * @return {void}
   */
  public addEdge(s: number, t: number): void {
    this.adj[s].add(t);
    this.adj[t].add(s);
  }

  /**
   * @description: 广度优先搜索
   * @param {number} s
   * @param {numebr} t
   * @return {void}
   */
  public bfs(s: number, t: number) {
    if (s === t) return;
    const visited: boolean[] = new Array(this.v);
    const queue = new LinkedList();
    const prev: number[] = new Array(this.v).fill(-1);
    visited[s] = true;
    queue.add(s);
    while (queue.size()) {
      const w = queue.removeByIndex(0) as number;
      for (let i = 0; i < this.adj[w].size(); i++) {
        const q = this.adj[w].getElementByIndex(i) as number;
        if (q && q !== 0 && !visited[q]) {
          prev[q] = w;
          if (q === t) {
            return this.print(prev, s, t);
          }
          visited[q] = true;
          queue.add(q);
        }
      }
    }
  }

  /**
   * @description: 打印路线
   * @param {T[]} prev
   * @param {number} s
   * @param {number} t
   * @return {void}
   */
  private print(prev: number[], s: number, t: number) {
    // 递归打印s->t的路径
    if (prev[t] !== -1 && t !== s) {
      this.print(prev, s, prev[t]);
    }
    console.log(t + '->');
  }
}
```

* 这里面有三个变量需要着重关注。`visited`、`queue`、`prev`.
	* **`visited` 用于记录已经被访问的顶点，用来避免顶点重复访问**。如果顶点 q 已经被访问，那么 `visited[q]` 的值会设置为 true。
	* **`queue` 就是队列，用来存储已经被访问，但相连的顶点还没有被访问的顶点**。 因为广度优先是逐层遍历的，只有先把第 k 层的所有顶点都访问完成，才会继续遍历到第 k + 1 层。在访问第 k 层的时候，需要把这一层的顶点记录下来，然后才能根据 k 层的顶点来继续查找 k + 1 层的顶点。这个队列用于动态记录当前遍历层内的顶点。
	* **`prev` 用来记录搜索路径。** 当从顶点 s 开始，广度优先搜索到顶点 t 时，prev 数组中存储的就是搜索的路径。只不过这个路径是反向存储的。`prev[w]` 存储的是顶点 w 是从哪个前驱顶点遍历过来的。
		* 例如，通过顶点 2 的邻接表访问到顶点 3，`prev[3]` 就等于 2.
		* 为了正向打印出路径，就需要递归来打印，可以参见上文代码中 `print` 的实现。
	* 下面是广度优先搜索的过程分解图。

![[广度优先搜索分解图1.png]]
![[广度优先搜索分解图2.png]]
![[广度优先搜索分解图3.png]]

#### 广度优先搜索遍历出最短路径

* 参照上面的图片中的示例我们创建一个有着 **8 个顶点的无向图**。
	* 通过 `addEdge` 方法添加边，这个方法**在无向图的内部会从两个顶点分别记录边。**
* 通过广度优先搜索**打印出从顶点 0 到顶点 7** 的**最短路径**。

```typescript
const graph = new Graph(8);
graph.addEdge(0, 1);
graph.addEdge(0, 3);
graph.addEdge(1, 4);
graph.addEdge(1, 2);
graph.addEdge(3, 4);
graph.addEdge(2, 5);
graph.addEdge(4, 5);
graph.addEdge(4, 6);
graph.addEdge(6, 7);
graph.addEdge(5, 7);
console.log(graph);
console.log(graph.bfs(0, 7)); // 0 -> 1 -> 4 -> 5 -> 7
```

### 广度优先搜索的复杂度

* 我们来确认广度优先搜索的时间复杂度和空间复杂度。
* 最坏情况下，终止顶点 t 距离起始顶点 s 很远，需要遍历整个图才能找到。这样每个顶点都需要进出一遍队列，每个边也都会访问一次。这时**广度优先的时间复杂度是 `O(V+E)`, V 是顶点的个数，E 是边的个数。**
* 而对一个**连通图**来说，一个图中所有的顶点都是相连的，此时 E 肯定大于等于 V - 1. 因此**广度优先搜索的时间复杂度可以简写为 `O(E)`.**
	* **无向图里，图中任意两点都是连通的，就称之为[连通图](https://www.wikiwand.com/zh-hans/%E8%BF%9E%E9%80%9A%E5%9B%BE)。**
* 广度优先搜索的空间消耗主要在 `visited` 和 `queue` 队列以及 `prev` 数组上，而这三个存储空间的大小都不会超过顶点的个数，所以**空间复杂度就是 `O(V)`.**

## 深度优先搜索（DFS）

* **深度优先搜索（Depth-First-Search）** 最直观的例子就是走迷宫。
	* 在迷宫的某个岔口，需要找到出口。那么就需要把每条路都走到底来验证是否能走通。如果走不通，就需要回退到上一个岔口，重新走另一条路继续走到底。这种走法就是深度优先搜索策略。
* 下面的图里，起始点是 s，终点是 t。我们会在图中查找一条从顶点 s 到顶点 t 的路径。
	* 下面把深度递归算法把整个搜索的路径标记出来，实线箭头表示遍历，虚线箭头表示回退。
	* 从图中可以发现，深度优先找出的路径，不是顶点 s 到顶点 t 的最短路径。

![[深度优先搜索示意图.png]]

* **深度优先搜索其实是应用了回溯思想**。这种思想解决问题的过程，很适合用递归来实现。回溯算法的内容我们后续会深入了解。

### 深度优先搜索的实现

* 把思路理出来并实现，就是下面这样。

```java
boolean found = false; // 全局变量或者类成员变量

public void dfs(int s, int t) {
  found = false;
  boolean[] visited = new boolean[v];
  int[] prev = new int[v];
  for (int i = 0; i < v; ++i) {
    prev[i] = -1;
  }
  recurDfs(s, t, visited, prev);
  print(prev, s, t);
}

private void recurDfs(int w, int t, boolean[] visited, int[] prev) {
  if (found == true) return;
  visited[w] = true;
  if (w == t) {
    found = true;
    return;
  }
  for (int i = 0; i < adj[w].size(); ++i) {
    int q = adj[w].get(i);
    if (!visited[q]) {
      prev[q] = w;
      recurDfs(q, t, visited, prev);
    }
  }
}
```

* 下面是 TS 的实现。
	* 直接打印出遍历的结果。

```typescript
  /**
   * @description: 深度优先搜索
   * @param {number} s
   * @param {number} t
   * @return {void}
   */
  public dfs(s: number, t: number): void {
    const found = false;
    const visited = new Array(this.v);
    const prev: number[] = new Array(this.v).fill(-1);
    this.recuriveDfs(s, t, visited, prev, found);
    this.print(prev, s, t);
  }

  /**
   * @description: 打印路线
   * @param {T[]} prev
   * @param {number} s
   * @param {number} t
   * @return {void}
   */
  private print(prev: number[], s: number, t: number): void {
    // 递归打印s->t的路径
    if (prev[t] !== -1 && t !== s) {
      this.print(prev, s, prev[t]);
    }
    console.log(t + '->');
  }

  /**
   * @description: 递归遍历逻辑
   * @param {number} w
   * @param {number} t
   * @param {boolean[]} visited
   * @param {number[]} prev
   * @param {boolean} found
   * @return {void}
   */
  private recuriveDfs(w: number, t: number, visited: boolean[], prev: number[], found = false): void {
    if (found) return;
    visited[w] = true;
    if (w === t) {
      found = true;
      return;
    }
    for (let i = 0; i < this.adj[w].size(); i++) {
      const q = this.adj[w].getElementByIndex(i) as number;
      if (!visited[q]) {
        prev[q] = w;
        this.recuriveDfs(q, t, visited, prev, found);
      }
    }
  }


// 复用上文的逻辑，直接调用 dfs 方法
console.log(graph.dfs(0, 7)); // 0 -> 1 -> 4 -> 5 -> 7
```

* 深度优先代码也使用了 `prev`，`visited` 变量和 `print` 函数。它们和广度优先搜索代码实现里的作用是一样的。
* 深度优先代码实现里，有个变量 `found`，它的作用是当代码已经找到终止顶点 t 之后，就不再继续递归查找。

### 深度优先搜索的复杂度

* 从上面的图可以看到，每条边最多会访问两次，一次是遍历，一次是回退。所以图上的深度优先搜索算法的时间复杂度是 `O(E)`, E 是边的个数。
*  深度优先搜索的算法消耗内存主要是 `visited`、`prev` 数组和递归调用栈。`visited` 和 `prev` 数组的大小和顶点的个数 V 成正比，递归调用栈的最大深度不会超过顶点的个数，所以空间复杂度就是 `O(V)`.

## 图和搜索的视频讲解

* 这里的视频讲解内容包括了上一章节，图、图的实现，然后也包括本节搜索相关的内容。
* 视频的内容比较浅显易懂但也不够深入，仅作为补充，实际还需要自己多尝试理解深度优先搜索和广度优先搜索的具体实现。

<iframe src="https://player.bilibili.com/player.html?bvid=BV1254y1976m&p=4&cid=284638929&page=1&share_source=copy_web" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="520px"></iframe>

## 如何查找用户的三度好友关系

* 使用图的广度优先算法，计算距离当前顶点的边的距离分别是 1 和 2 和 3 的好友，也就是记录前三层的好友数据。记录用户的好友，也可理解为一度好友；再记录用户好友的好友，二度好友，依次往后到三度。
	* 个人理解，这个也可以算是层序遍历，记录和起始顶点关联的前三层的每一层的顶点数据。
	* 招聘网站领英似乎就是使用此种标识来拉近好友关系的。

## 小结

* 深度优先搜索和广度优先搜索是图上的两种最基础也是最常用的两种搜索算法。它们比其他高级搜索算法例如 `A*` 和 `IDA*` 等要更加简单粗暴，没有什么优化空间，所以又被称为暴力搜索算法。它们只适用于图的数据不大的搜索场景。
* 广度优先搜索就是地毯式层层推进的搜索，从起始顶点开始依次往外遍历。广度优先遍历需要借助队列来实现，遍历得到的路径就是起始顶点到终止顶点的最短路径。
* 深度优先搜索使用的是回溯思想，适合使用递归实现。深度优先搜索是借助栈来实现的。
* 在执行效率方面，深度优先和广度优先搜索的时间复杂度都是 `O(E)`, 空间复杂度是 `O(V)`.

## 扩展

* 用深度优先来解决查找用户的三度好友关系？
	* 在遍历的时候记录层数，也就是边的数量，在相应的层数内记录好友数据。其他时刻不做处理，继续遍历。
* 如何把迷宫抽象成一个图？或者说如何在计算机中存储一个迷宫？
	* 邻接矩阵的方式来存储迷宫，在可达的地方设置数值为 1 ，不可达的地方设置数值 0.

#ALG 