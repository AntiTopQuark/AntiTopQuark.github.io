---
layout: post
cid: 93
title: [转]HashMap 扩容 原文：https://blog.csdn.net/lkforce/article/details/89521318
slug: 93
date: 2021/04/16 16:57:23
updated: 2021/04/16 16:57:23
status: publish
author: AntiTopQuark
categories: 
  - 编程语言
tags: 
  - 接口
  - int
  - hashmap
  - static
  - cloneable
  - default
  - initial
customSummary: 
noThumbInfoStyle: default
outdatedNotice: no
reprint: standard
thumb: 
thumbChoice: default
thumbDesc: 
thumbSmall: 
thumbStyle: default
---


## HashMap基础

HashMap继承了AbstractMap类，实现了Map，Cloneable，Serializable接口

HashMap的容量，默认是16

```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

HashMap的加载因子，默认是0.75

```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

当HashMap中元素数超过容量*加载因子时，HashMap会进行扩容。

 

## HashMap实现原理

#### Node和Node链

首先来了解一下HashMap中的元素类型

HashMap类中的元素是Node类，翻译过来就是节点，是定义在HashMap中的一个内部类，实现了Map.Entry接口。

Node类的定义如下：

```java
/**
 * Basic hash bin node, used for most entries.  (See below for
 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
 
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
 
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
 
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
 
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
 
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

可以看到，Node类的基本属性有：

>  hash：key的哈希值
>
> key：节点的key，类型和定义HashMap时的key相同
>
> value：节点的value，类型和定义HashMap时的value相同
>
> next：该节点的下一节点

值得注意的是其中的next属性，记录的是下一个节点本身，也是一个Node节点，这个Node节点也有next属性，记录了下一个节点，于是，只要不断的调用Node.next.next.next……，就可以得到：

 Node-->下个Node-->下下个Node……-->null

这样的一个链表结构，而对于一个HashMap来说，只要明确记录每个链表的第一个节点，就能顺序遍历链表上的所有节点。

 

#### 拉链法

HashMap使用拉链法管理其中的每个节点。

由Node节点组成链表之后，HashMap定义了一个Node数组：

```java 
transient Node<K,V>[] table;
```

这个数组记录了每个链表的第一个节点，于是最终形成了HashMap下面这样的数据结构：

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/2019042518014336.jpg)

这种数组+链表的数据结构，使得HashMap可以较为高效的管理每一个节点。

 

#### 关于Node数组 table

对于table的理解，对后面关于扩容的理解很有帮助。

table在第一次往HashMap中put元素的时候初始化。

如果HashMap初始化的时候没有指定容量，那么初始化table的时候会使用默认的DEFAULT_INITIAL_CAPACITY参数，也就是16，作为table初始化时的长度。

如果HashMap初始化的时候指定了容量，HashMap会把这个容量修改为2的倍数，然后创建对应长度的table。

table在HashMap扩容的时候，长度会翻倍。

所以table的长度肯定是2的倍数。

修改容量的方法是这样的：

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

所以要注意，如果要往HashMap中放1000个元素，又不想让HashMap不停的扩容，最好一开始就把容量设为2048，设为1024不行，因为元素添加到七百多的时候还是会扩容。

 

#### 散列算法

当调用HashMap.put()方法时，经历了以下步骤：

1，对key进行hash值计算

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

2，hash值和table.length取模

取模的方法是(table.length - 1) & hash，算法直接舍弃了二进制hash值在table.length以上的位，因为那些位都代表table.length的2的n次方倍数。

取模的结果就是Node将要放入table的下标。

比如，一个Node的hash值是5，table长度是4，那么取余的结果是1，也就是说，这个Node将被放入table[1]所代表的链表（table[1]本身指向的是链表的第一个节点）。

3，添加元素

如果此时table的对应位置没有任何元素，也就是table[i]=null，那么就直接把Node放入table[i]的位置，并且这个Node的next==null。

如果此时table对应位置是一个Node，说明对应的位置已经保存了一个Node链表，则需要遍历链表，如果发现相同hash值则替换Node节点，如果没有相同hash值，则把新的Node插入链表的末端，作为之前末端Node的next，同时新Node的next==null。

如果此时table对应位置是一个TreeNode，说明链表被转换成了红黑树，则根据hash值向红黑树中添加或替换TreeNode。（JDK1.8）

4，如果添加元素之后，Node链表的节点数超过了8个，则该链表会考虑转为红黑树。（JDK1.8）

5，如果添加元素之后，HashMap总节点数超过了阈值，则HashMap会进行扩容。

相关代码是这样的：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
 
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)                       //注释1
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))   //注释2
            e = p;
        else if (p instanceof TreeNode)                        //注释3
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);               //注释4
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
    if (++size > threshold)                 //注释5
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

代码解析：

1，注释1，table对应位置无节点，则创建新的Node节点放入对应位置。

2，注释2，table对应位置有节点，如果hash值匹配，则替换。

3，注释3，table对应位置有节点，如果table对应位置已经是一个TreeNode，不再是Node，也就说，table对应位置是TreeNode，表示已经从链表转换成了红黑树，则执行插入红黑树节点的逻辑。

4，注释4，table对应位置有节点，且节点是Node（链表状态，不是红黑树），链表中节点数量大于TREEIFY_THRESHOLD，则考虑变为红黑树。实际上不一定真的立刻就变，table短的时候扩容一下也能解决问题，后面的代码会提到。

5，注释5，HashMap中节点个数大于threshold，会进行扩容。

 

#### HashMap和红黑树

从JDK1.8开始，在HashMap里面定义了一个常量TREEIFY_THRESHOLD，默认为8。当链表中的节点数量大于TREEIFY_THRESHOLD时，链表将会考虑改为红黑树，代码是在上面putVal()方法的这一部分：

```java
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
```

其中的treeifyBin()方法就是链表转红黑树的方法，这个方法的代码是这样的：

```java
/**
 * Replaces all linked nodes in bin at index for given hash unless
 * table is too small, in which case resizes instead.
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

可以看到，**如果table长度小于常量MIN_TREEIFY_CAPACITY时，不会变为红黑树，而是调用resize()方法进行扩容。MIN_TREEIFY_CAPACITY的默认值是64。显然HashMap认为，虽然链表长度超过了8，但是table长度太短，只需要扩容然后重新散列一下就可以。**

后面的代码中可以看到，如果table长度已经达到了64，就会开始变为红黑树，else if中的代码把原来的Node节点变成了TreeNode节点，并且进行了红黑树的转换。

 

#### 关于TreeNode

当HashMap把链表转为红黑树的时候，原来的Node节点就会被转为TreeNode节点，TreeNode也是HashMap中定义的内部类，定义大概是这样的：

```java
/**
 * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
 * extends Node) so can be used as extension of either regular or
 * linked node.
 */
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
 
    /**
     * Returns root of tree containing this node.
     */
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
 
    /**
     * Ensures that the given root is the first node of its bin.
     */
    static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
        int n;
        if (root != null && tab != null && (n = tab.length) > 0) {
            int index = (n - 1) & root.hash;
            TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
            if (root != first) {
                Node<K,V> rn;
                tab[index] = root;
                TreeNode<K,V> rp = root.prev;
                if ((rn = root.next) != null)
                    ((TreeNode<K,V>)rn).prev = rp;
                if (rp != null)
                    rp.next = rn;
                if (first != null)
                    first.prev = root;
                root.next = first;
                root.prev = null;
            }
            assert checkInvariants(root);
        }
    }
 
    /**
     * Finds the node starting at root p with the given hash and key.
     * The kc argument caches comparableClassFor(key) upon first use
     * comparing keys.
     */
    final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
        TreeNode<K,V> p = this;
        do {
            int ph, dir; K pk;
            TreeNode<K,V> pl = p.left, pr = p.right, q;
            if ((ph = p.hash) > h)
                p = pl;
            else if (ph < h)
                p = pr;
            else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                return p;
            else if (pl == null)
                p = pr;
            else if (pr == null)
                p = pl;
            else if ((kc != null ||
                      (kc = comparableClassFor(k)) != null) &&
                     (dir = compareComparables(kc, k, pk)) != 0)
                p = (dir < 0) ? pl : pr;
            else if ((q = pr.find(h, k, kc)) != null)
                return q;
            else
                p = pl;
        } while (p != null);
        return null;
    }
```

可以看到，TreeNode继承了LinkedHashMap的Entry，TreeNode节点在构造时，也指定了hash值，key，value，下一节点next，这些都是和Node相同的结构。

同时，TreeNode还保存了父节点parent， 左孩子left，右孩子right，上一节点prev，另外还有红黑树用到的red属性。

 

#### 红黑树基础

红黑树是一种近似平衡的二叉查找树，他并非绝对平衡，但是可以保证任何一个节点的左右子树的高度差不会超过二者中较低的那个的一倍。

红黑树有这样的特点：

1，每个节点要么是红色，要么是黑色。

2，根节点必须是黑色。叶子节点必须是黑色NULL节点。

3，红色节点不能连续。

4，对于每个节点，从该点至叶子节点的任何路径，都含有相同个数的黑色节点。

5，能够以O(log2(N))的时间复杂度进行搜索、插入、删除操作。此外,任何不平衡都会在3次旋转之内解决。

 

## HashMap扩容机制

当HashMap决定扩容时，会调用HashMap类中的resize(int newCapacity)方法，参数是新的table长度。在JDK1.7和JDK1.8的扩容机制有很大不同。

 

### JDK1.7下的扩容机制

JDK1.7下的resize()方法是这样的：

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
 
    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

代码中可以看到，如果原有table长度已经达到了上限，就不再扩容了。

如果还未达到上限，则创建一个新的table，并调用transfer方法：

```java
/**
 * Transfers all entries from current table to newTable.
 */
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;              //注释1
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity); //注释2
            e.next = newTable[i];                  //注释3
            newTable[i] = e;                       //注释4
            e = next;                              //注释5
        }
    }
}
```

transfer方法的作用是把原table的Node放到新的table中，使用的是头插法，也就是说，新table中链表的顺序和旧列表中是相反的，在HashMap线程不安全的情况下，这种头插法可能会导致环状节点。

其中的while循环描述了头插法的过程，这个逻辑有点绕，下面举个例子来解析一下这段代码。

假设原有table记录的某个链表，比如table[1]=3，链表为3-->5-->7，那么处理流程为：

1，注释1：记录e.next的值。开始时e是table[1]，所以e\=\=3，e.next\=\=5，那么此时next==5。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182351160.jpg)

2，注释2，计算e在newTable中的节点。为了展示头插法的倒序结果，这里假设e再次散列到了newTable[1]的链表中。

3，注释3，把newTable [1]赋值给e.next。因为newTable是新建的，所以newTable[1]=\=null，所以此时3.next==null。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182444135.jpg)

4，注释4，e赋值给newTable[1]。此时newTable[1]=3。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182459234.jpg)

5，注释5，next赋值给e。此时e==5。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182526345.jpg)

此时newTable[1]中添加了第一个Node节点3，下面进入第二次循环，第二次循环开始时e==5。

1，注释1：记录e.next的值。5.next是7，所以next==7。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/2019042518255152.jpg)

2，注释2，计算e在newTable中的节点。为了展示头插法的倒序结果，这里假设e再次散列到了newTable[1]的链表中。

3，注释3，把newTable [1]赋值给e.next。因为newTable[1]是3（参见上一次循环的注释4），e是5，所以5.next==3。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/2019042518273242.jpg)

4，注释4，e赋值给newTable[1]。此时newTable[1]==5。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182747304.jpg)

5，注释5，next赋值给e。此时e==7。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182802465.jpg)

此时newTable[1]是5，链表顺序是5-->3。

下面进入第三次循环，第二次循环开始时e==7。

1，注释1：记录e.next的值。7.next是NULL，所以next==NULL。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182818968.jpg)

2，注释2，计算e在newTable中的节点。为了展示头插法的倒序结果，这里假设e再次散列到了newTable[1]的链表中。

3，注释3，把newTable [1]赋值给e.next。因为newTable[1]是5（参见上一次循环的注释4），e是7，所以7.next==5。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182835578.jpg)

4，注释4，e赋值给newTable[1]。此时newTable[1]==7。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182912253.jpg)

5，注释5，next赋值给e。此时e==NULL。

![img](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20190425182930307.jpg)

此时newTable[1]是7，循环结束，链表顺序是7-->5-->3，和原链表顺序相反。

注意：**这种逆序的扩容方式在多线程时有可能出现环形链表，出现环形链表的原因大概是这样的：线程1准备处理节点，线程二把HashMap扩容成功，链表已经逆向排序，那么线程1在处理节点时就可能出现环形链表。**

 

另外单独说一下indexFor(e.hash, newCapacity);这个方法，这个方法是计算节点在新table中的下标用的，这个方法的代码如下：

```java
/**
 * Returns index for hash code h.
 */
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

计算下标的算法很简单，hash值 和 (length-1)按位与，使用length-1的意义在于，length是2的倍数，所以length-1在二进制来说每位都是1，这样可以保证最大的程度的散列hash值，否则，当有一位是0时，不管hash值对应位是1还是0，按位与后的结果都是0，会造成散列结果的重复。

 

### JDK1.8下的扩容机制

JDK1.8对resize()方法进行很大的调整，JDK1.8的resize()方法如下：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)                      //注释1
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {                                 //注释2
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)                                        //注释3
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {                      //注释4
                            if (loTail == null)                            //注释5
                                loHead = e;
                            else
                                loTail.next = e;                           //注释6
                            loTail = e;                                    //注释7
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {                                  /注释8
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

代码解析：

1，在resize()方法中，定义了oldCap参数，记录了原table的长度，定义了newCap参数，记录新table长度，newCap是oldCap长度的2倍（注释1），同时扩展点也乘2。

2，注释2是循环原table，把原table中的每个链表中的每个元素放入新table。

3，注释3，e.next==null，指的是链表中只有一个元素，所以直接把e放入新table，其中的e.hash & (newCap - 1)就是计算e在新table中的位置，和JDK1.7中的indexFor()方法是一回事。

4，注释// preserve order，这个注释是源码自带的，这里定义了4个变量：loHead，loTail，hiHead，hiTail，看起来可能有点眼晕，其实这里体现了JDK1.8对于计算节点在table中下标的新思路：

正常情况下，计算节点在table中的下标的方法是：hash&(oldTable.length-1)，扩容之后，table长度翻倍，计算table下标的方法是hash&(newTable.length-1)，也就是hash&(oldTable.length*2-1)，于是我们有了这样的结论：这新旧两次计算下标的结果，要不然就相同，要不然就是新下标等于旧下标加上旧数组的长度。

举个例子，假设table原长度是16，扩容后长度32，那么一个hash值在扩容前后的table下标是这么计算的：



hash值的每个二进制位用abcde来表示，那么，hash和新旧table按位与的结果，最后4位显然是相同的，唯一可能出现的区别就在第5位，也就是hash值的b所在的那一位，如果b所在的那一位是0，那么新table按位与的结果和旧table的结果就相同，反之如果b所在的那一位是1，则新table按位与的结果就比旧table的结果多了10000（二进制），而这个二进制10000就是旧table的长度16。

换言之，hash值的新散列下标是不是需要加上旧table长度，只需要看看hash值第5位是不是1就行了，位运算的方法就是hash值和10000（也就是旧table长度）来按位与，其结果只可能是10000或者00000。

 

所以，注释4处的e.hash & oldCap，就是用于计算位置b到底是0还是1用的，只要其结果是0，则新散列下标就等于原散列下标，否则新散列坐标要在原散列坐标的基础上加上原table长度。

 

理解了上面的原理，这里的代码就好理解了，代码中定义的四个变量：

loHead，下标不变情况下的链表头

loTail，下标不变情况下的链表尾

hiHead，下标改变情况下的链表头

hiTail，下标改变情况下的链表尾

而注释4处的(e.hash & oldCap) == 0，就是代表散列下标不变的情况，这种情况下代码只使用了loHead和loTail两个参数，由他们组成了一个链表，否则将使用hiHead和hiTail参数。

其实e.hash & oldCap等于0和不等于0后的逻辑完全相同，只是用的变量不一样。

以等于0的情况为例，处理一个3-->5-->7的链表，过程如下：

首先处理节点3，e\==3，e.next==5

1，注释5，一开始loTail是null，所以把3赋值给loHead。

2，注释7，把3赋值给loTail。

然后处理节点5，e\==5，e.next==7

1，注释6，loTail有值，把e赋值给loTail.next，也就是3.next==5。

2，注释7，把5赋值给loTail。

现在新链表是3-->5，然后处理节点7，处理完之后，链表的顺序是3-->5-->7，loHead是3，loTail是7。可以看到，链表中节点顺序和原链表相同，不再是JDK1.7的倒序了。

代码到注释8这里就好理解了，

只要loTail不是null，说明链表中的元素在新table中的下标没变，所以新table的对应下标中放的是loHead，另外把loTail的next设为null

反之，hiTail不是null，说明链表中的元素在新table中的下标，应该是原下标加原table长度，新table对应下标处放的是hiHead，另外把hiTail的next设为null。