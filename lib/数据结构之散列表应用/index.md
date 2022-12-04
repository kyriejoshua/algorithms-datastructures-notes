# 如何打造工业级水平的散列表（哈希表）

## 简言

* 在极端情况下，恶意攻击者可以通过构造的数据，使所有的数据经过散列函数之后，都散列到同一个槽内。如果使用基于链表的冲突解决方法，那么这个时候散列表就会退化为链表，查询的时间复杂度从 `O(1)` 退化到 `O(n)`。
* 在这种情况下，查询操作就会消耗大量 CPU 或者线程资源，导致系统无法响应其他请求，从而达到拒绝服务攻击（Dos）的目的。
	* *这也是散列表碰撞攻击的基本原理。*
* 下面我们尝试着设计一个可以应对各种异常情况的工业级散列表，来避免在散列冲突情况下散列表性能急剧下降的问题，并且能有效抵抗散列碰撞攻击。

> 学习曲线：★★★☆

## 如何设计散列函数

* 散列函数的设计直接影响散列表冲突的概率和散列表的性能。

### 散列函数的设计要点

* **散列函数不能太复杂。**
	* 过于复杂的散列函数，会消耗很多计算时间，也会间接影响散列表的性能。
* **散列函数生成的值要尽可能随机并且均匀分布。**
	* 这样可以避免或最小化散列冲突，即便出现冲突，散列到每个槽内的数据也会比较平均，不会出现某个槽内数据特别多的情况。
* **实际应用中，还需要考虑其他因素例如关键字的长度，特点，分布，还有散列表的大小等等。**

#### 散列函数的简单案例

* 下面我们来讨论几个常用也简单的散列函数的设计方法来帮助理解。
* 上一节运动会编号的案例中，把编号的后两位作为散列值。还可以用类似的方案来处理手机号码，因为手机号前几位重复的可能性较大，但末尾几位就会比较随机。可以取手机号的后四位作为散列值。这种散列函数的设计方法，一般叫做 **“数据分析法”**。
* 另一个例子是，实现 Word 应用里的拼写检查功能。可以这样设计：把单词中每个字母的 ASCII 码值“进位”相加，再跟散列表的大小求余，取模，作为散列值。
	* 比如英文单词 nice，转化出来的散列值就是下面这样

```shell
hash("nice")=(("n" - "a") * 26*26*26 + ("i" - "a")*26*26 + ("c" - "a")*26+ ("e"-"a")) / 78978
```

* 其他散列函数的设计方法还有很多，简单了解即可。
	* 这些方法包括：**直接寻址法，平方取中法，折叠法，随机数法**等。

## 装载因子的处理

### 装载因子过大的场景

* 装载因子过大，说明散列表中的元素越多，空闲位置越少，散列冲突的概率就越大。
* 插入数据的过程中要多次寻址或者建立很长的链表，查找的过程也会因此变得很慢。
* 对于没有频繁插入和删除的静态数据集合来说，我们很容易根据数据的特点、分布等，设计出完美的、极少冲突的散列函数，因为毕竟之前的数据都是已知的。
* 对于动态散列表来说，数据集合是频繁变动的，我们无法预估将要加入的数据个数，所以我们也无法事先申请一个足够大的散列表。随着数据慢慢加入，装载因子就会慢慢变大。当装载因子大到一定程度之后，散列冲突就会变得不可接受。这个时候，我们该如何处理呢？
	* 这里其实可以借鉴数组、栈或队列的动态扩容，来实现散列表的动态扩容。
	* **针对散列表，当装载因子过大时，我们也可以进行动态扩容，重新申请一个更大的散列表，将数据搬移到这个新散列表中**。假设每次扩容我们都申请一个原来散列表大小两倍的空间。如果原来散列表的装载因子是 0.8，那经过扩容之后，新散列表的装载因子就下降为原来的一半，变成了 0.4。
* 针对数组的扩容，数据搬移操作比较简单。而针对散列表的扩容，数据搬移操作要复杂很多。因为散列表的大小变了，数据的存储位置也随之改变，**需要通过散列函数重新计算每个数据的存储位置，这就是散列表扩容的复杂部分**。
* 可以观察下图里的例子。在原来的散列表中，21 这个元素原来存储在下标为 0 的位置，搬移到新的散列表中，存储在下标为 7 的位置。

![[散列表的扩容示例.png]]

* 插入一个数据，最好情况下是不需要扩容，因此最好时间复杂度是 `O(1)`。最坏情况下，散列表装载因子过高，启动扩容，我们需要重新申请内存空间，重新计算哈希位置，并且搬移数据，所以时间复杂度是 `O(n)`。
	* **用摊还分析法，均摊情况下，时间复杂度接近最好情况，因此插入数据的时间复杂度就是 `O(1)`。**
* 对于动态散列表，随着数据的删除，散列表中的数据会越来越少，空闲空间会越来越多。如果我们对空间消耗非常敏感，我们可以在装载因子小于某个值之后，启动动态缩容。当然，如果我们更加在意执行效率，能够容忍多消耗一点内存空间，那就可以使用较大的内存空间。
	* **当散列表的装载因子超过某个阈值时，就需要进行扩容。**
		* 装载因子阈值需要选择得当。如果太大，会导致冲突过多；如果太小，会导致内存浪费严重。
	* **装载因子阈值的设置要权衡时间、空间复杂度。**
		* 如果内存空间不紧张，对执行效率要求很高，可以降低负载因子的阈值；相反，如果内存空间紧张，对执行效率要求又不高，可以增加负载因子的值，甚至可以大于 1。

## 如何避免低效扩容

* 大部分情况下，动态扩容的散列表插入一个数据都很快，只是在特殊情况下，当装载因子达到阈值时，才需要先进行扩容，再插入数据。这时插入数据就会变得很慢，甚至会无法接受。
	* 极端的例子，如果散列表大小当前是 1GB，扩容为原先的两倍大小，就需要对 1GB 的数据重新结算哈希值，并且从原来的散列表搬移到新的散列表。这是非常耗时的。
* 如果业务代码直接服务于用户，尽管大部分情况下，插入数据的操作都很快，但是极个别非常慢的插入操作，会极大影响用户的使用体验，从而让人崩溃。这时的“一次性”扩容机制是不合适的。

### 一次性扩容的解决方案

* 为了解决一次性扩容耗时过多的问题，可以将扩容操作穿插在插入操作的过程中，分批完成。
	* 当装载因子达到阈值后，申请新的空间，但暂时并不将老的数据搬移到新的散列表中。
	* **当有新数据插入时，直接将该数据插入新散列表中，并且从老的散列表中拿出一个数据放入到新散列表。** 每次插入数据，都重复上面的过程。经过多次的插入操作之后，老的散列表数据就渐渐全部搬移到新散列表中了。这样没有一次性的搬移操作，每次插入就能够变得很快。
	* 搬移过程如下所示：

![[散列表分批搬移示意.png]]

#### 扩容的查询操作

* 上述主要分析的是插入操作，**查询的操作可以先从新的散列表中查找，如果没有找到，再去老的散列表中查找。**
* 通过均摊的方式，将一次性扩容的代价，均摊到多次插入操作中，避免了一次性扩容耗时过多的情况。
	* 这种实现，保证了任何时候插入一个数据的时间复杂度都是 `O(1)`.

## 如何选择冲突解决方法

* 上一小节主要介绍了两种主要的散列冲突解决方法，**开放寻址法和链表法**。
* 下面来分析两者的主要优劣势和各自的适用场景。

### 1. 开放寻址法

#### 开放寻址法的优势

* 使用这种方法，散列表中的数据都存在数组中，可以有效地利用 CPU 缓存加快查询速度。
* 这种散列表序列化起来比较简单。链表法中包含指针，序列化起来不方便。而序列化也是一种常见的场景。

#### 开放寻址法的劣势

* 开放寻址法解决冲突的散列表，解决冲突比较麻烦，需要特殊标记已经删除的数据。
* 而且，开放寻址法中所有的数据都存储在数组中，比起链表法，冲突的代价更高。因此，使用开发寻址法的散列表的装载因子的上限不能太大，这样会导致这种方法会更浪费内存空间。

#### 开放寻址法小结

* **当数据量比较小，装载因子比较小的时候，适合使用开放寻址法**。这也是 Java 中 ThreadLocalMap 使用开放寻址法解决散列冲突的原因。

### 2. 链表法

#### 链表法的优势

* **链表法对内存的利用率比开放寻址法要高。** 因为链表结点可以在需要的时候再创建，并不需要像开放寻址法那样需要事先申请好。
* **链表法比起开放寻址法，对大装载因子的容忍度更高。 开放寻址法只能适用装载因子小于 1 的情况**。装载因子接近 1 时，就可能会有大量的散列冲突，导致大量的探测、再散列等，性能会下降很多。但是对于链表法来说，只要散列函数的值随机均匀，即使装载因子变成 10，也就是链表的长度变长了而已，虽然查找效率有所下降，但是比起顺序查找还是快很多。

#### 链表法应用

* 链表章节有提到，链表因为要存储指针，所以对于比较小的对象的存储，是比较消耗内存的，还有可能会让内存的消耗翻倍。而且，因为链表中的结点是零散分布在内存中的，不是连续的，所以对 CPU 缓存是不友好的，这方面对于执行效率也有一定的影响。
	* 而如果我们存储的是大对象，也就是说要存储的对象的大小远远大于一个指针的大小（4个字节或者 8 个字节），那链表中指针的内存消耗在大对象面前就可以忽略不计。
* 实际上，我们对链表法稍加改造，就可以实现一个更加高效的散列表。那就是，将链表法中的链表改造为其他高效的动态数据结构，比如**跳表**或**红黑树**。这样，即便出现散列冲突，极端情况下，所有的数据都散列到同一个桶内，那最终退化成的散列表的查找时间也只不过是 `O(logn)`。这样也就有效避免了前面讲到的散列碰撞攻击。

![[链表法解决冲突示意.png]]
* **基于链表的散列冲突处理方法比较适合存储大对象、大数据的散列表。而且，比起开放寻址法，它更加灵活，支持更多的优化策略，比如用红黑树来替代链表。**

## 工业级散列表的举例分析

* 以 Java 中的 HashMap 这样一个工业级的散列表为例。看看这些技术是如何应用的。

### 1. 初始大小

* **HashMap 的初始大小是 16，这个默认值可以修改**。如果事先知道数据量有多少体量，可以通过修改默认初始大小来减少动态扩容的次数，这样会大大提高 HashMap 的性能。

### 2. 装载因子和动态扩容

* **最大装载因子默认是 0.75**。当 HashMap 中的元素个数超过 `0.75 * capacity` (capacity 表示散列表的容量)的时候，就会启动扩容，每次扩容都会变成原来的两倍大小。

### 3. 散列冲突解决方法

* **HashMap 底层采用链表法来解决冲突**。即使负载因子和散列函数设计得再合适，也避免不了拉链过长的情况，一旦出现拉链过长，就会严重影响 HashMap 的性能。
* 在 JDK 1.8 版本中，为了进一步优化，引入了红黑树。当链表长度太长（默认超过 8）时，链表就转为红黑树。
	* 可以利用红黑树能够快速增删改查的特点，来提高 HashMap 的性能。
	* 当红黑树结点个数少于 8 个的时候，又会将红黑树转化为链表。因为在数据量较小的情况下，红黑树要维护平衡，比起链表来，性能上的优势并不明显。

### 4. 散列函数

* 散列函数的设计并不复杂，追求的是简单高效、分布均匀。
* 下面是 Java 中的应用分析：

```java
int hash(Object key) {
    int h = key.hashCode()；
    return (h ^ (h >>> 16)) & (capicity -1); //capicity表示散列表的大小
}
```

* `hashCode()` 返回的是 Java 对象的 `hash code`。比如 String 类型的对象的 `hashCode()` 就是下面这样：
```java
public int hashCode() {
  int var1 = this.hash;
  if (var1 == 0 && this.value.length > 0) {
    char[] var2 = this.value;
    for(int var3 = 0; var3 < this.value.length; ++var3) {
      var1 = 31 * var1 + var2[var3];
    }
    this.hash = var1;
  }
  return var1;
}
```

## 工业级散列函数的设计

### 考虑工业级的散列表的特性

* 支持快速地查询、插入、删除操作；
* 内存占用合理，不能浪费过多的内存空间；
* 性能稳定，极端情况下，散列表的性能也不会退化到无法接受的地步；
* 实现这样的散列表，就要考虑设计合适的散列函数：
	* 定义装载因子阈值，并且设计动态扩容策略；
	* 选择合适的散列冲突解决方法。

## 小结

### 本节内容

* 上一节偏理论，这一节侧重实战。主要分成三部分：
	* 如何设计散列函数；
	* 如何根据装载因子动态扩容；
	* 以及如何选择散列冲突解决方法。

### 简要概括

* 关于散列函数的设计，要尽可能让散列后的值随机且均匀分布，这样也可以减少散列冲突，即便冲突之后，分配到每个槽内的数据也比较均匀。除此之外，散列函数的设计也不能太复杂，太复杂就会耗费更多时间，从而影响散列表的性能。
* 关于散列冲突解决方法的选择，分别对比了开放寻址法和链表法两种方法的优劣和各自的适应场景。
	* 大部分情况下，链表法更加普适。而且，我们还可以通过将链表法中的链表改造成其他动态查找的数据结构，比如红黑树，来避免散列表时间复杂度退化成 `O(n)`，从而能够抵御散列碰撞攻击。
	* 而对于小规模数据、装载因子不高的散列表，比较适合用开放寻址法。
* 对于动态散列表来说，不管我们如何设计散列函数，选择什么样的散列冲突解决方法。随着数据的不断增加，散列表总会出现装载因子过高的情况。这时，我们就需要启动动态扩容。

## 扩展

* 在 JS 中，有哪些数据类型是基于散列表实现的，散列函数是如何设计的？散列冲突是通过何种方法解决的？是否支持动态扩容？
	* **实际上，JS 中的对象就是基于散列表实现的。**
* 使用 TS 实现一个简单的散列表类。
	* 支持 `put`, `delete`, `hash`, `size` 等方法和属性.
	* 对应的 [Github](https://github.com/kyriejoshua/javascript-datastructure/blob/main/src/HashTable) 仓库链接。

### 使用线性探测解决冲突的散列表 TS 实现

* 下面的方案是基于线性探测的开放寻址法。

```typescript
export interface IHashObject {
  key: string | number;
  value: unknown;
  deleted?: boolean; // 保存被删除索引或数组的属性的统计
}

export interface IHashFunc {
  (key: string): string | number;
}

/**
 * @description: 最基础的散列表，不包含装载因子
 */
export default class BasicHashTable<T extends IHashObject> {
  private basicHashTable: T[] = [];
  private size = 9533; // 10000 以内的最大质数
  private hash: IHashFunc = this.defaultHash;
  // private deletedKeys: (number | string)[];

  /**
   * @description: 构造函数
   * @param {number} size 设置散列表大小，默认是一万
   * @param {function} hash 散列函数
   */
  constructor(size?: number, hash?) {
    if (!this.isSizeLegal(size)) return;
    this.size = size || this.size;
    this.hash = hash || this.defaultHash;
  }

  /**
   * @description: 校验传入的散列表大小是否合法
   * size 大小判断
   * 整数判断
   * @param {number} size
   * @return {boolean}
   */
  private isSizeLegal(size?: number): boolean {
    const errorMessage = 'Please set a logical size for hashTable.';
    if (size === 0) {
      throw new Error(errorMessage);
    }
    if (size) {
      if (size > Number.MAX_SAFE_INTEGER || size < 0) throw new Error(errorMessage);
      if (Math.floor(size) !== size) throw new Error(errorMessage);
    }
    return true;
  }

  /**
   * @description: 默认散列函数的冲突解决方式：线性探测
   * @param {number} numberedHashKey
   * @return {number}
   */
  private getDefaultNextHashKey(numberedHashKey: number): number {
    console.warn('The Hash key is used! Starting finding the next key.');
    // 解决散列冲突
    let findedNextUndefinedHashKey = numberedHashKey;
    while (this.basicHashTable[findedNextUndefinedHashKey] !== undefined) {
      findedNextUndefinedHashKey++;
      findedNextUndefinedHashKey %= this.size;
    }
    return findedNextUndefinedHashKey;
  }

  /**
   * @description: 默认的哈希函数取所有的字符的 ascii 编码值
   * @param {string} key
   * @return {number}
   */
  private defaultHash(key: string): number {
    let hashKey = key;
    if (typeof hashKey !== 'string') {
      hashKey = String(hashKey);
    }

    // let numberedHashKey = Number(hashKey.slice(hashKey.length - 2));
    let numberedHashKey = 0;
    for (const str of hashKey) {
      // chartCodeAt 支持传入索引作为参数，如果不传，那么默认取第一个，也就是它的值为 0
      numberedHashKey += str.charCodeAt(0);
    }
    // 这里取模的方式在 size 较小的时候就会比较容易重复，容易引发散列冲突；
    // 而散列函数的核心就是不同的值不能输出相同的，但通常不会使用 size 特别小的散列表
    numberedHashKey %= this.size;

    return numberedHashKey;
  }

  /**
   * @description: 获取当前散列表的有效长度
   * @param {T[]} basicHashTable
   * @return {number}
   */
  private getHashMapLength(basicHashTable: T[]): number {
    return basicHashTable.filter(Boolean).length;
  }

  /**
   * @description: 获取对应的散列表的存储的内容
   * @param {string} key
   * @return {T}
   */
  private getHashByKey(key: string): T {
    const hashKey = this.hash(key);
    if (this.basicHashTable[hashKey]?.key === hashKey) {
      if (this.basicHashTable[hashKey]?.deleted) throw new Error('This value has been deleted!');
      return this.basicHashTable[hashKey];
    }
    const numberedHashKey = Number(hashKey);
    let currentHashKey = numberedHashKey;
    while (this.basicHashTable[currentHashKey]?.key !== key) {
      currentHashKey++;
      currentHashKey %= this.size;
    }
    if (this.basicHashTable[hashKey]?.deleted) throw new Error('This value has been deleted!');
    return this.basicHashTable[currentHashKey];
  }

  /**
   * @description: 支持按照索引插入，也支持属性插入，暂不支持混合插入，校验会有问题
   * 如果不传参数，默认按照 undefined 字符串的解析
   * @param {string} key
   * @param {any} val
   * @return {T[]}
   */
  public put = (key: string, val: unknown): T[] => {
    // 如果当前散列表已满，可以是数组内元素撑满，也可以是属性个数撑满
    const currentHashMapLength = this.getHashMapLength(this.basicHashTable);
    if (currentHashMapLength >= this.size || Object.keys(this.basicHashTable).length >= this.size) {
      throw new Error('This HashMap is totally full!');
    }
    // 如果当前索引没有值，直接插入，允许当作属性的情况
    const hashKey = this.hash(key);
    console.log('current hash key ===>', hashKey);
    if (this.basicHashTable[hashKey] === undefined) {
      // 注意这里存储的 key 是原始值，否则取值时无法一一对应
      this.basicHashTable[hashKey] = {
        key,
        value: val,
      };
      return this.basicHashTable;
    }
    const numberedHashKey = Number(hashKey);
    // 散列冲突的场景，开放寻址法的线性查找
    if (Number.isNaN(numberedHashKey)) {
      throw new Error('The Hash key must be converted to numbers');
    }

    const findedNextUndefinedHashKey = this.getDefaultNextHashKey(numberedHashKey);
    this.basicHashTable[findedNextUndefinedHashKey] = {
      key,
      value: val,
    } as T;

    return this.basicHashTable;
  };

  /**
   * @description: 取值
   * 默认按照 hashKey 取，取不到时向后依次遍历
   * @param {string} key
   * @return {unknown}
   */
  public get = (key: string): unknown => {
    return this.getHashByKey(key)?.value;
  };

  /**
   * @description: 删除，给对应的位置加一个标来标识
   * @param {string} key
   * @return {boolean}
   */
  public delete = (key: string): boolean => {
    const currentHash: T = this.getHashByKey(key);
    if (currentHash) {
      currentHash.deleted = true;
      return true;
    }
    return false;
  };

  /**
   * @description: 清空所有值
   * @return {void}
   */
  public clear = (): void => {
    this.basicHashTable = [];
  };
}

const basicHashTable = new BasicHashTable(3);
console.log(basicHashTable);
// 正常插入
basicHashTable.put('001', 1);
console.log(basicHashTable);
// 继续插入
basicHashTable.put('04', 41);
console.log(basicHashTable);
// 有冲突的插入
basicHashTable.put('10', 10);
console.log(basicHashTable);
// 读取
console.log('current 10 val', basicHashTable.get('10')); // 10
console.log('current 04 val', basicHashTable.get('04')); // 41
console.log(basicHashTable);
// 散列表满的状态下插入，会抛出异常
basicHashTable.put('03', 30); // Error: This HashMap is totally full!
console.log(basicHashTable);
```

### 使用链表法解决冲突的 TS 实现

* 链接同上，具体内容如下。

```typescript
import LinkedList, { TLinkedNode } from '@/LinkedList';

export interface IHashTableObject<T> {
  key: string;
  value: T;
}

export interface IHashFunc {
  (key: string): string | number;
}

// T 表示传入值的泛型， THashTable 表示哈希对象内容的泛型，注意这两者的区别
export default class HashTableByLinkedList<T, THashTable extends IHashTableObject<T>> {
  private buckets: LinkedList<THashTable>[] = []; // 桶（槽）
  private size = 9533;
  private hash: IHashFunc = this.defaultHash;
  private keys: Record<string, string | number> = {}; // 存储所有的键，方便取用

  constructor(size?: number, hash?: IHashFunc) {
    if (!this.isSizeLegal(size)) return;
    this.size = size ?? this.size;
    this.hash = hash ?? this.defaultHash;
    this.buckets = new Array(this.size).fill(null).map(() => new LinkedList());
  }

  /**
   * @description: 校验传入的散列表大小是否合法
   * size 大小判断
   * 整数判断
   * @param {number} size
   * @return {boolean}
   */
  private isSizeLegal(size?: number): boolean {
    const errorMessage = 'Please set a logical size for hashTable.';
    if (size === 0) {
      throw new Error(errorMessage);
    }
    if (size) {
      if (size > Number.MAX_SAFE_INTEGER || size < 0) throw new Error(errorMessage);
      if (Math.floor(size) !== size) throw new Error(errorMessage);
    }
    return true;
  }

  /**
   * @description: 默认的哈希函数取所有的字符的 ascii 编码值
   * @param {string} key
   * @return {number}
   */
  private defaultHash(key: string): number {
    let hashKey = key;
    if (typeof hashKey !== 'string') {
      hashKey = String(hashKey);
    }

    let numberedHashKey = 0;
    for (const str of hashKey) {
      numberedHashKey += str.charCodeAt(0);
    }
    numberedHashKey %= this.size;

    return numberedHashKey;
  }

  /**
   * @description: 根据键值查询链表中对应的节点
   * @param {string} key
   * @return {TLinkedNode<THashTable>}
   */
  private getLinkedNodeByKey = (key: string): TLinkedNode<THashTable> => {
    const currentHashKey = this.hash(key);
    const currentBucket = this.buckets[currentHashKey];

    return currentBucket.find(key, (node) => {
      return node.element.key === key;
    });
  };

  /**
   * @description: 哈希表存入值
   * @param {string} key
   * @param {T} value
   * @return {boolean}
   */
  public put = (key: string, value: T): boolean => {
    const currentHashKey = this.hash(key);
    const currentBucket: LinkedList<THashTable> = this.buckets[currentHashKey];
    const currentNode: TLinkedNode<THashTable> = this.getLinkedNodeByKey(key);
    this.keys[key] = currentHashKey;
    currentBucket.size() && console.log(`>>> 出现哈希冲突，此时 key 为 ${key}`);

    if (currentNode) {
      currentNode.element.value = value;
    } else {
      currentBucket.add({ key, value } as THashTable);
    }

    return true;
  };

  /**
   * @description: 删除哈希表中的值
   * @param {string} key
   * @return {boolean}
   */
  public delete = (key: string): boolean => {
    const currentHashKey = this.hash(key);
    const currentBucket = this.buckets[currentHashKey];
    const deleted: T = currentBucket.remove(key, (node) => node.element.key === key);
    delete this.keys[key];

    return !!deleted;
  };

  /**
   * @description: 根据键获取值
   * @param {string} key
   * @return {T}
   */
  public get = (key: string): T => {
    const currentNode: TLinkedNode<THashTable> = this.getLinkedNodeByKey(key);

    return currentNode?.element?.value;
  };

  /**
   * @description: 判断哈希表中是否有这个键
   * @param {string} key
   * @return {boolean}
   */
  public has(key: string): boolean {
    return Object.hasOwnProperty.call(this.keys, key);
  }

  /**
   * @description: 获取哈希表中的所有的键
   * @return {string[]}
   */
  public getKeys(): string[] {
    return Object.keys(this.keys);
  }

  /**
   * @description: 获取哈希表中所有的值
   * @return {T[]}
   */
  public getValues(): T[] {
    return this.buckets.reduce((prev: T[], bucket: LinkedList<THashTable>) => {
      const values = bucket.toArray().map((element: THashTable) => element.value);
      return prev.concat(values);
    }, []);
  }

  /**
   * @description: 获取当前哈希表中的数据长度
   * @return {number}
   */
  public getLength(): number {
    return this.getValues().length;
  }

  /**
   * @description: 获取装载因子
   * @return {number}
   */
  public getLoadFactor(): number {
    return parseFloat((this.getLength() / this.size).toFixed(2));
  }
}

const hashTableByLinkedList = new HashTableByLinkedList(5);

console.log('Inited hash table is', hashTableByLinkedList);
// 正常插入
hashTableByLinkedList.put('001', 1);
console.log('After inserting 001', hashTableByLinkedList);
// 继续插入
hashTableByLinkedList.put('04', 41);
console.log('After inserting 04', hashTableByLinkedList);
// 继续插入
hashTableByLinkedList.put('abc', 3);
console.log('After inserting abc', hashTableByLinkedList);
// 覆盖原值
hashTableByLinkedList.put('001', 2);
console.log('After conflicting insert 001', hashTableByLinkedList, hashTableByLinkedList.getValues()); // [2, 41, 3]

// 读取
console.log('current 10 val is', hashTableByLinkedList.get('10')); // current 10 val is undefined
console.log('current 04 val is', hashTableByLinkedList.get('04')); // current 04 val is 41
console.log('current abc val is', hashTableByLinkedList.get('abc')); // current abc val is 3

// 删除
hashTableByLinkedList.delete('04'); // true
console.log('After deleting, current 04 val is', hashTableByLinkedList.get('04')); // After deleting, current 04 val is undefined

// 判断键是否存在
console.log('Has key 04?', hashTableByLinkedList.has('04')); // Has key 04? false
console.log('Has key abc?', hashTableByLinkedList.has('abc')); // Has key abc? true
console.log('Get all keys', hashTableByLinkedList.getKeys()); // Get all keys [ '001', 'abc' ]
console.log('The loadfactor is', hashTableByLinkedList.getLoadFactor()); // The loadfactor is 0.4

// 在使用链表实现的散列表里，不太会存在因为溢出而抛出异常的问题，除非我们限定了链表的长度。
```

#ADT #ALG
