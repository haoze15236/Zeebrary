# Map

## HashMap

采用位运算取模，让index分布更加均匀

- 加载因子为什么是0.75？

  过早扩容，会造成内存空间的浪费，比如0.5，那么只能用一半，就开始扩容。

  过完扩容，会增加hash碰撞的几率，导致链表过长，降低查询效率

>  jdk1.7之前

数组+链表

扩容时:	采用头插，链表元素倒序,<span style="color:red">多线程下扩容有可能形成环形链表</span>，导致之后put元素若碰到此环形链表导致死循环

> jdk1.8之后

数组+链表+红黑树

链表转红黑树阈值默认为8，链表长度>=8,数组容量>=64时会转为红黑树，否则优先扩容。使用红黑树提升查询速度，时间复杂度为O(logn)

扩容时: 	使用元素的hash值与数组长度进行与运算,计算扩容后被用于计算index的高位值,若不为0,则将元素放置到高位链表，置于扩容后的数组位置。

<span style="color:red">虽然解决了多线程会现成环形链表导致死循环的问题，但是，在put元素时，多线程情况下仍然会出现丢失数据的情况。</span>

```java
Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
Node<K,V> next;
do {
    next = e.next;
    //判断元素hash值扩容后被新增进来计算index的高位，若为0,则通过tab[i = (n - 1) & hash])还是在数组原下标
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }
    else {
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

默认是16,为了跟简洁讲清楚，我们假设比如数组从8扩容到16

```java
//**********************hash值1*****************************
	1111 0001	
&运算
    0000 0111   8-1
=   0000 0001
扩容后&运算
    0000 1111   16-1
=	0000 0001
//我们使用hash值1取&扩容前的数组大小8可以得到
	1111 0001	hash值2
&运算
    0000 1000   8
=   0000 0000   //不为0,扩容后的index=原index(1)
//***********************hash值2****************************      
	1111 1001	
&运算
    0000 0111   8-1
=   0000 0001
扩容后&运算
    0000 1111   16-1
=	0000 1001	9
//我们使用hash值2取&扩容前的数组大小8可以得到
	1111 1001	hash值2
&运算
    0000 1000   8
=   0000 1000   //不为0,扩容的index=原index(1)+原数组长度(8) = 9
```

## ConcurrentHashMap

> 1.7之前

[什么是ConcurrentHashMap](https://mp.weixin.qq.com/s/1yWSfdz0j-PprGkDgOomhQ)

通过ReentrantLock实现分段锁segment，保证不同segment之间可并发写入。

> 1.8

通过synchronized+CAS算法保证线程安全

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                //使用CAS初始化Map
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //cas添加第一个hash桶中第一个元素
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                //synchronized插入hash碰撞的元素到链表中
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```



## ConcurrentSkipListMap(跳表)

[有序的线程安全hashmap](https://blog.csdn.net/sunxianghuang/article/details/52221913)

# List

## ArrayList

线程不安全，查询快，插入，删除慢

## CopyOnWriteArrayList

空间换时间,每次写入元素时，复制一个数组副本，加锁操作写入元素,存在数据的脏读，因此它只保证数据的最终一致性，适用于读多，写少的情景。

## Vector

使用synchronized修饰了方法,线程安全,但是调用的时候由于底层时数组，可能存在数组越界的情况;

# Set

## CopyOnWriteArraySet

线程安全，内部使用的是CopyOnWriteArrayList存储元素

