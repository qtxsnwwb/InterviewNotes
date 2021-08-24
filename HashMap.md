**HashMap 在 JDK8 相较于 JDK7 在底层实现方面的不同**
1. new HashMap()：底层没有创建一个长度为 16 的数组
2. JDK8 底层的数组是 Node[]，而非 Entry[]
3. 首次调用 put() 方法时，底层创建长度为 16 的数组
4. JDK7 底层结构只有：数组 + 链表。JDK8 中底层结构：数组 + 链表 + 红黑树
> 当数组的某一个索引位置上的元素以**链表形式存在的个数 > 8** 且**当前数组长度 > 64** 时，此时此索引位置上的所有数据改为使用红黑树存储



**map.put()过程**

1. map.put("暴躁", "小刘")
2. 获取“暴躁”字符串的 hash 值
3. 经过 hash 值扰动函数，使此 hash 值更散列
4. 构造出 Node 对象
> Hash -->1122
> Key --> "暴躁"
> Value --> "小刘"
> Next --> null
5. 路由算法，找出 Node 应存放在数组的位置
> (table.length - 1) & node.hash



**重要常量**
* DEFAULT_INITIAL_CAPACITY：HashMap 的默认容量：16
* DEFAULT_LOAD_FACTOR：HashMap 的默认加载因子：0.75
* threshold：扩容临界值 = 容量 * 填充因子：16 * 0.75 => 12
* TREEIFY_THRESHOLD：桶中链表长度大于该默认值，转化为红黑树：18
* MIN_TREEIFY_CAPACITY：桶中的 Node 被树化时最小的 hash 表容量：64



**HashMap 构造函数中的参数 initialCapacity**
```java
public HashMap(int initialCapacity, float loadFactor)
```
构造函数中通过 tableSizeFor 方法将 initialCapacity 转化为大于它的 2 的次方数



# HashMap put 方法详解
```java
putVal(hash(key), key, value, false, true)
```
> hash(key)：扰动函数，让 key 的 hashCode 的高 16 位也参与运算

**hash 算法步骤**
1. 取 key 的 hashCode 值
2. 通过扰动函数高位运算
3. 取模运算（路由算法）

**put 方法过程**
1. 判断键值对数组 table[i] 是否为空或为 null，否则执行 resize() 扩容
> 延迟初始化逻辑，第一次调用 putVal 时会初始化 HashMap 中最耗内存的散列表
2. 根据键值 key 计算 hash 值得到插入的数组索引 i ，如果 table[i] == null，直接新建结点添加，转向 6，如果 table[i] 不为空，转向 3
3. 判断 table[i] 的首个元素是否和 key 一样，若相同直接覆盖 value，否则转向 4。此处的相同指 hashCode 以及 equals
4. 判断 table[i] 是否为 treeNode，即 table[i] 是否是红黑树。若是，则直接在树种插入键值对，否则转向 5
5. 遍历 table[i]，插入到链表末尾，若此时链表长度大于 8，则把链表转化为红黑树；遍历过程中若发现 key 已存在，直接覆盖 value 即可
> 3、4、5 对应插入时的三种情况，即桶位首元素与插入元素 key 相同；红黑树；链表
6. 插入成功后，判断实际存在的键值对数量 size 是否超过了最大容量 threshold，如果超过了，进行扩容



# HashMap 扩容机制详解
**为什么需要扩容**
```
	为了解决哈希冲突导致的链化影响查询效率的问题，扩容会缓解该问题
```

**resize 方法过程**
1. 若扩容前 table 长度 > 0，说明 HashMap 中的散列表已经初始化过了，这是一次正常扩容
* 若扩容之前 table 大小已达到最大阈值，则不扩容，且设置扩容条件为 int 最大值
* 扩容前 table 大小翻倍作为扩容后 table 大小，若其小于最大阈值且扩容前 table 大小 >= 16，则下一次扩容阈值设为当前阈值翻倍
2. 若 1 不成立 --> 若扩容前阈值 > 0，则扩容后 table 大小设为扩容前阈值
3. 若 1、2 均不成立，扩容后 table 大小为 16，阈值为 16 * 0.75 = 12
4. 若扩容后阈值 == 0，通过 newCap 和 loadFactor 计算出一个 newThr
5. 若 HashMap 本次扩容之前，table 不为 null，执行以下逻辑
* 遍历全部桶位元素（首元素），若元素不为空，则进入 2、3、4 的判断，分别对应单个数据、红黑树、链表
* 第一种情况：当前桶位只有一个元素，从未发生碰撞，直接计算出当前元素应存放在新数组中的位置，然后扔进去就行了
* 第二种情况：当前结点已树化
* 第三种情况：桶位已经形成链表，进入 6 的逻辑
6. 链表转化到新数组分为两种情况：**低位链表**和**高位链表**
> 解释：原数组大小 16（1111），扩容后 32（11111），扩容后在原基础上多了一位 1
> 原路由寻址：1111 & ~~0000~~ 0101 --> 0 0101
>                           1111 & ~~0001~~ 0101 --> 0 0101
>                           **相同桶**
> 扩容后寻址：1 1111 & 0 0101 --> 0 0101
>			           1 1111 & 1 0101 --> 1 0101
>			           **不同桶**
因此不需要重新计算 hash 值，只需看扩容后多出来的一位是 0 还是 1
* 若为 0，则结果不变，归为低位链表
* 若为 1，则新位置为 （老位置 + 老容量），归为高位链表



# 红黑树
* 与 2-3-4 树一一对应，O(log2n) 时间内完成查找、插入和删除操作
* 每个结点都带有红色或黑色属性的平衡二叉排序树
**满足属性：**
1. 每个结点必须带有红色或黑色
2. 根结点一定是黑色
3. 每个叶子结点都带有两个空的黑色子结点（NIL 结点）
4. 每个红色结点的两个子结点都是黑色结点，即从根结点到叶子结点的所有路径上，不存在两个连续的红色结点
5. 从任一结点到其所能到达的叶子结点的所有路径含有相同数量的黑色结点



**参考：**[HashMap全B站最细致源码分析课程，看完月薪最少涨5k！](https://www.bilibili.com/video/BV1LJ411W7dP?from=search&seid=8760896110575578593)



# HashTable
```
	线程安全，结构、扩容机制等与 HashMap一样，使用 Synchronized 修饰方法的方式来实现多线程同步。HashTable 的同步会锁住整个数组，在高并发的情况下，性能非常差。
```



# ConcurrentHashMap
* 支持高并发、高吞吐量的线程安全 HashMap 实现
**结构：**
```
	包含一个 Segment 数组，每个 Segment 都类似一个 HashMap，包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素。
```
![ConcurrentHashMap结构](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210823/ConcurrentHashMap结构.4g2th6xmc9o0.png)

**加锁原则：**
```
	ConcurrentHashMap 将 Hash 表默认分为 16 个桶，执行 put、remove 等操作时只需要锁住当前线程需要用到的桶（Segment）。只有个别方法，例如 size() 和 containsValue() 可能需要锁定整个 Segment 数组，实现时需按顺序锁定所有桶，操作完毕后再按顺序释放所有桶，此举可防止死锁。
```

**读原则：**
```
	为了实现在读取的时候不加锁而又不会读取到不一致的数据， ConcurrentHashMap 用不变量方式实现。HashEntry 设计成几乎不可变
```
```java
static final class HashEntry<K, V> {
	final K key;
	final int hash;
	volatile V value;
	final HashEntry<K, V> next;
}
```
> put 方法只能在 Hash 链头部增加

**remove 方法：**
```
	把需要删除的结点前面所有结点复制一遍，然后把复制后的 Hash 链的最后一个结点指向待删除结点的后继结点。
```