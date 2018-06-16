---
title: 基于Java 7/8 的 HashMap 源码解析
date: 2018-04-22 11:38:33
tags: [Java,HashMap,数据结构与算法]
---

本篇我们来看看数据结构中一个非常基础的应用，哈希表。<!-- more --> 

##  Java 7 的实现

[HashMap.java](http://androidxref.com/7.1.2_r36/xref/libcore/ojluni/src/main/java/java/util/HashMap.java)

基本原理讲解  

**什么是哈希表(hash table)**

我们看下维基百科对哈希（hash) 的解释，

> **散列**（英语：Hashing）是电脑科学中一种对数据的处理方法，通过某种特定的函数/算法（称为**散列函数**/算法）将要检索的项与用来检索的索引（称为**散列**，或者**散列值**）关联起来，生成一种便于搜索的数据结构（称为**散列表**）

 也就是说，哈希表就是用来快速查找用的，哈希函数（也称为散列函数）就是用来生成检索的索引（index)的。



内部数据结构架构









基本流程概述



默认table的初始大小为 16,

```
132     static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
144     static final float DEFAULT_LOAD_FACTOR = 0.75f;
```



源码剖析流程

[文件路径](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/HashMap.java)

构造函数，默认初始化容量为 16，加载因子为0.75

```java
280     public HashMap() {
281         this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR); 
282     }
```

接着调用带参数的构造函数，初始化加载因子和 threshold

```java
250     public HashMap(int initialCapacity, float loadFactor) {
251         if (initialCapacity < 0)
252             throw new IllegalArgumentException("Illegal initial capacity: " +
253                                                initialCapacity);
254         if (initialCapacity > MAXIMUM_CAPACITY)
255             initialCapacity = MAXIMUM_CAPACITY;
256         if (loadFactor <= 0 || Float.isNaN(loadFactor))
257             throw new IllegalArgumentException("Illegal load factor: " +
258                                                loadFactor);
259 
260         this.loadFactor = loadFactor;
261         threshold = initialCapacity;
262         init();//为空
263     }
```



put 函数

```java
490     public V put(K key, V value) {
491         if (table == EMPTY_TABLE) {
            //初始化默认大小的table，表的大小为16，同时更新下次resize的阈值threhold=12,(大小*加载因子）
492             inflateTable(threshold); 
493         }
494         if (key == null)//处理 Key=null case, HashMap 允许 key为null
495             return putForNullKey(value);
496         int hash = hash(key);//获取该key的 hash值
497         int i = indexFor(hash, table.length);//获取该key映射对应的数组索引
           //若key相同，则做value的更新操作，然后返回，否则就插入新的节点
498         for (Entry<K,V> e = table[i]; e != null; e = e.next) {
499             Object k;
500             if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
501                 V oldValue = e.value;
502                 e.value = value;
503                 e.recordAccess(this);
504                 return oldValue;
505             }
506         }
507 
508         modCount++;
509         addEntry(hash, key, value, i);//采用头部插入法，插入新的链表节点
510         return null;
511     }
```



```java
315     private void inflateTable(int toSize) {
316         // Find a power of 2 >= toSize,确保 capacity为最小的2的整数次幂
317         int capacity = roundUpToPowerOf2(toSize);
318 	    //增长阈值为容量*加载因子，首次的阈值为 16*0.75=12
319         threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
320         table = new Entry[capacity];//初始化table，开始时大小为 16
321         initHashSeedAsNeeded(capacity); //初始化 hash mask 值
322     }
```



```java
881     void addEntry(int hash, K key, V value, int bucketIndex) {
     //若此时table的size 不小于阈值了且刚插入的这个值又冲突了，重新调整大小
882         if ((size >= threshold) && (null != table[bucketIndex])) {
     // 将table数组大小扩容为2倍，数组大小发生变化，所以 hash,index都需重新计算一次
883             resize(2 * table.length);
884             hash = (null != key) ? hash(key) : 0;
885             bucketIndex = indexFor(hash, table.length);
886         }
887 
    
899     void createEntry(int hash, K key, V value, int bucketIndex) {
    //查找到位于table数组中位置的节点
900         Entry<K,V> e = table[bucketIndex];
    //采用头部插入法插入节点,也就是若每次发生冲突，则都将新的节点插入到链表首位
901         table[bucketIndex] = new Entry<>(hash, key, value, e);
902         size++;
903     }
    
576     void resize(int newCapacity) {
    //获取旧table，以及数据容量大小
577         Entry[] oldTable = table;
578         int oldCapacity = oldTable.length;
579         if (oldCapacity == MAXIMUM_CAPACITY) {
580             threshold = Integer.MAX_VALUE;
581             return;
582         }
583 
    //重新new了一个数组
584         Entry[] newTable = new Entry[newCapacity];
    //旧table到新table做数据迁移
585         transfer(newTable, initHashSeedAsNeeded(newCapacity));
586         table = newTable;
    //更新 threhold，比如初始化表的大小16，扩容2倍为32，,此时resize的阈值为 32*0.74=24.
587         threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
588     }
```

```java
593     void transfer(Entry[] newTable, boolean rehash) {
594         int newCapacity = newTable.length;
595         for (Entry<K,V> e : table) {
596             while(null != e) {
597                 Entry<K,V> next = e.next;
598                 if (rehash) {
599                     e.hash = null == e.key ? 0 : hash(e.key);
600                 }
601                 int i = indexFor(e.hash, newCapacity);
602                 e.next = newTable[i];
603                 newTable[i] = e;
604                 e = next;
605             }
606         }
607     }
```



**get**方法

```java
418     public V get(Object key) {
    // key为null情况，单独处理
419         if (key == null)
420             return getForNullKey();
421         Entry<K,V> entry = getEntry(key);
422 
423         return null == entry ? null : entry.getValue();
424     }
```

获取Entry，查找到并且返回。

```java
461     final Entry<K,V> getEntry(Object key) {
462         if (size == 0) {
463             return null;
464         }
465 
466         int hash = (key == null) ? 0 : hash(key);
467         for (Entry<K,V> e = table[indexFor(hash, table.length)];
468              e != null;
469              e = e.next) {
470             Object k;
471             if (e.hash == hash &&
472                 ((k = e.key) == key || (key != null && key.equals(k))))
473                 return e;
474         }
475         return null;
476     }
```



## Overview



### 继承关系

HashMap 继承于 AbstractMap<K,V> 

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```

## 实例源码分析

```java
import java.util.HashMap;

public class Test {
    public static void main(String[] args) {
        HashMap<String, Integer> hm = new HashMap<>();
        hm.put("one", 1);
        System.out.println("hm: " + hm);
    }
}
```

[HashMap源代码](http://androidxref.com/8.0.0_r4/xref/libcore/ojluni/src/main/java/java/util/HashMap.java)

### 构造函数

初始化 加载因子 0.75

```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

### put方法 

`key` 为 "one", `value` 为 

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

会用到内置方法 `putVal` ，首先会将 key 做 hash 计算，我们接着看下 `hash()`方法

```java

```



调用 Key 对象的 hashCode()方法生成 hash值，对于整数，哈希值就是它自己，对于字符串，它的值如下，可以查看相应的 `hashCode()` 方法，结果就是得到 32位的 hash值。

$hash = s[0]*31^{n-1} + s[1]*31^{n-2}+...+s[n-1]$



```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```



## Android的优化

