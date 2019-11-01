title: HashMap
author: Sunkz
tags:
  - hashmap
  - java
photos:
  - 'https://s2.ax1x.com/2019/11/02/Kbc4s0.png'
categories: []
date: 2018-07-08 01:14:00
---
## Key的Hash值计算过程

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

//模拟 hash(key) 计算过程
HashMap<String,Object> map = new HashMap<>();
map.put("name","skz");
/**
 * 0000 0000 0011 0011 0111 1010 1000 1011  //"name".hashCode()
 * 0000 0000 0000 0000 0000 0000 0011 0011  //"name".hashCode() >>> 16
 * 0000 0000 ‭0011 0011 0111 1010 1011 1000  //"name".hashCode() ^ h
 * ----高16位不变------ -低16位是高低异或结果-
 */
int h = "name".hashCode() >>> 16;
System.out.println("name".hashCode()); //3373707
System.out.println(h); //51
System.out.println("name".hashCode() ^ h); //3373752
```

## 懒加载

```java
//HashMap数据结构
transient Node<K,V>[] table; 
//在第一次put时候才会初始化table
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
//如果new HashMap()使用无参构造器 则阈值threshold = 0.75 * 16
table = (Node<K,V>[])new Node[DEFAULT_INITIAL_CAPACITY];
threshold = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
```

## 添加putVal(key,value)元素

```java
/**
 * 懒加载初始化后开始添加第一个元素
 *
 * n是table的length = 16 
 * length-1 & hash计算出的结果总是在[0,length)之间
 * tab[i] =tab[15 & 3373752] = tab[8]
 * 0000 0000 0000 0000 0000 0000 0000 1111  //length -1
 * 0000 0000 ‭0011 0011 0111 1010 1011 1000  //hash("name")
 * 0000 0000 0000 0000 0000 0000 0000 1000  //i = length-1 & hash("name")
 *
 * 如果tab[8]上没有则将newNode(hash, "name", "skz", null)放入hashTable
 */
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
/**
 * 如果hashTable的tab[8]位置不为空 , 且tab[8]的key和"name" equals
 * tab[8]位置的 oldValue is replaced By the new value
 */
if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
    V oldValue = p.value;
	p.value = value;
```

## 判断Key相等

先根据hash值 ,  再根据equals

```java
hash(key)
key.equals(k)
```

如果key相等则 : 新的value将覆盖旧的value

```java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 */
```

## 扩容的两种情形

- 情形一 : tab.size 达到  threadhold


```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认容量 16
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//默认情况下阈值 threadhold = 0.75*16 = 12
threadhold = DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY;
//当 ++size > threshold 时候 开始resize 
if (++size > threshold)
	resize();
//新的容量增大一倍 32 , 新的阈值增大一倍 24
newCap = oldCap << 1;
newThr = oldThr << 1;
```

- 情形二 : binCount 达到 TREEIFY_THRESHOLD 且 tab.length < MIN_TREEIFY_CAPACITY

```java
//当某个bin中的数量大于等于8时候 且tab.length<64时候 , 会扩容
static final int TREEIFY_THRESHOLD = 8;
static final int MIN_TREEIFY_CAPACITY = 64;
if (binCount >= TREEIFY_THRESHOLD - 1)
	treeifyBin(tab, hash);
if (tab.length < MIN_TREEIFY_CAPACITY)
    resize();
```

## Node -> TreeNode

```java
/**
 * 当bin中Node节点数量大于8 且 tab.length>=64 
 * 若该bin中继续添加元素 , 则该bin中的所有 Node 将会替换为 TreeNode
 */
TreeNode<K,V> tree = replacementTreeNode(node, null);
//TreeNode节点的数据结构
TreeNode<K,V> parent;  // red-black tree links
TreeNode<K,V> left;
TreeNode<K,V> right;
TreeNode<K,V> prev;    // needed to unlink next upon deletion
boolean red;
TreeNode(int hash, K key, V val, Node<K,V> next) {
    super(hash, key, val, next);
}
```

# Other Map

```java
for(Entry<Integer, String> entry : tmap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

HashMap //遍历entrySet无序,因为是计算key的hash插入table
LinkedHashMap // 遍历输出顺序同插入顺序
TreeMap // 红黑树,插入时就排序(类比二叉查找树)
```

```
HashMap   
  |hash|key|value|next|
  table是单向链表,next用于维护单向链表的下一个节点
LinkedHashMap
  |before|hash|key|value|next|after|
  before after 用于维护entry插入的是顺序.
```


