# Map

```java
//jdk8,computeIfAbsent:若key对应的value为空，会将第二个参数的返回值存入并返回
Map<String, set<String>> map = new HashMap<>();
Set<String> set = map.computeIfAbsent(key, k -> new HashSet<>());
```

## HashMap

底层结构首先是一个初始化为16的数组（也叫hash桶）,当数组元素超过hash桶总数的加载因子时(默认是0.75)，会将hash桶扩容至原来的两倍。

- 加载因子为什么是0.75？

  过早扩容，会造成内存空间的浪费，比如0.5，那么只能用一半，就开始扩容。

  过完扩容，会增加hash碰撞的几率，导致链表过长，降低查询效率

- 为什么是2倍扩容?

  保证扩容后元素的key不变。

>  jdk1.7之前

底层结构是：数组+单链表

扩容时:	采用头插，链表元素倒序,<span style="color:red">多线程下扩容有可能形成环形链表</span>，导致之后put元素若碰到此环形链表导致死循环。

> jdk1.8之后

数组+链表/红黑树

链表转红黑树阈值默认为8，链表长度>=8并且数组容量>=64时会转为红黑树，否则优先扩容。而当红黑树节点<=6时,重新转回单链表

使用红黑树提升查询速度，时间复杂度为O(logn)

扩容时: 	使用元素的hash值与数组长度进行与运算,计算扩容后被用于计算index的高位值,若不为0,则将元素放置到高位链表，置于扩容后的数组位置。

<span style="color:red">虽然解决了多线程会现成环形链表导致死循环的问题，但是，在put元素时，多线程情况下仍然会出现丢失数据的情况。</span>



采用位运算取模，让index分布更加均匀

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

默认是16,为了更简洁的讲清楚，我们假设比如数组从8扩容到16

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

底层还是数组+链表，但是引入了继承ReentrantLock的segment这个概念,hash桶中存放的时segment,segment相当于一个带锁的hashMap,通过这种方式保证不同segment之间可并发写入。

> 1.8

通过synchronized+CAS算法保证线程安全,1.7时 synchronized有一波大的优化。

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

底层是数组,通过数组下标直接查询,**查询速度快,增删的效率底**,因为涉及到数组元素拷贝。
**线程不安全**:新增,删除元素都是直接通过下标操作,多线程下肯定会出现IndexOutOfBoundsException。
初始化时,数据量为0,add时，默认为10,每次扩容成之前的1.5倍。`int newCapacity = oldCapacity + (oldCapacity >> 1);`

## LinkedList

带有头指针和尾指针的双向链表,支持头插`linkFirst(E)`，尾插`linkLast(E)`,插入删除的效率高,内部维护了一个size属性记录链表长度。

## CopyOnWriteArrayList

空间换时间,每次写入元素时，复制一个数组副本，加锁操作写入元素,存在数据的脏读，因此它只保证数据的最终一致性，适用于读多，写少的情景。

## Vector

底层是数组,扩容是扩容成之前的2倍，由于大部分方式使用了synchronized修饰,所以这些方法是线程安全的,但是多个方法之间并不保证，例如：

```java
public void put(String element){
    if (!vector.contains(element)) 
    	//线程1通过判断,还没执行add操作时,线程2也会通过判断,此时就会存在数组越界的情况
        vector.add(element); 
    }
}
```



# Set

```java
//set转数组
Set<Class<?>> interfaces = new HashSet<>();
Class<?>[] classes = interfaces.toArray(new Class<?>[0]);
```



## CopyOnWriteArraySet

线程安全，内部使用的是CopyOnWriteArrayList存储元素

