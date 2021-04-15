# AQS(AbstractQueuedSynchronizer)

Java并发编程核心在于java.concurrent.util包而juc当中的大多数同步器实现都是围绕着共同的基础行为，比如等待队列、条件队列、独占获取、共享获取等，而这个行为的抽象就是基于AbstractQueuedSynchronizer简称AQS，AQS定义了一套多线程访问共享资源的同步器框架，是一个依赖状态(state)的同步器。

## AQS的设计原理

- **CAS（compare and swap）：**比较与交换,在一个**原子性操作**中比较是否可以占有锁,如果可以则占有，不能则操作失败，保证同一时间只能有一个线程能操作成功。

  使用Unsafe类提供的CAS操作实现,示例代码如下:

  ```java
  public class TestAQS {
  	public volatile int casValue=0;
  	private static Unsafe unsafe =null;
  	private static Field getUnsafe = null;
  	private static final long stateOffset;
  	static {
  		try {
              //直接使用Unsafe.getUnsafe()会报错,此处使用反射获取Unsafe对象
  			getUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
  			getUnsafe.setAccessible(true);
  			unsafe = (Unsafe) getUnsafe.get(null);
  			stateOffset = unsafe.objectFieldOffset
  					(TestAQS.class.getDeclaredField("casValue"));
  		} catch (Exception ex) { throw new Error(ex); }
  	}
  	@Test
  	public void testCas(){
          //当前值为0,准备更新为1
  		unsafe.compareAndSwapInt(this, stateOffset, 0, 1);
  	}
  }
  ```

- **自旋：**多个线程在竞争锁时,若CAS操作失败,则进行循环,重复去使用CAS比较是否可以持有锁。

- **队列：**若自旋一定次数之后,抢锁失败,则把当前抢锁的线程存储在队列中，等待锁释放之后,从队列中出队抢锁。

  查看**AbstractQueuedSynchronizer**源码可以看到,其内部定义了一个名为Node的对象作为等待对象,其本质是一个双向链表，其结构如下图:

  ![image-20210408223450020](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210408223450020.png)

  上图中，head指针永远指向头节点，tail指针永远指向队列的尾节点，可以看到头节点的Thread=null，<span style="color:red">这是初始化队列时的一个初始节点，当head指针与tail指针都指向Thread=null的节点时，意味着等待队列为空。</span>

- **LockSupport：**JVM提供的一个API，当线程抢锁失败，进入等待队列中后，使用park()阻塞线程，当锁释放，线程出队抢到锁之后，使用unpark()唤醒阻塞线程。

  LockSupport.park():

## 源码解析

- **state**:状态器

- **exclusiveOwnerThread**:当前获取到锁的线程

- **Node**：CLH队列的基础组成部分,包含以下几个内容
  - **thread**：指向等待锁的线程
  - **prev**:指向前一个Node节点
  - **next**:指向后一个Node节点
  - **nextWaiter**：指向后一个条件等待队列的Node节点
  - **waitStatus**：线程的等待状态
- **head**：指向当前锁的CLH队列头Node节点
- **tail**：指向当前锁的CLH队列尾Node节点

# ReentrantLock

ReentrantLock是一种基于AQS框架的应用实现，是JDK中的一种线程并发访问的同步手段，它的功能类似于synchronized是一种互斥锁，可以保证线程安全。而且它具有比synchronized更多的特性，比如它支持手动**加锁与解锁，支持加锁的公平性**。

- **锁的公平性**

  线程在竞争锁的时候,除了判断当前锁是否可以抢占之外，还要判断等待队列中是否有其他线程,公平锁的含义就是需要在等待队列中排队。

- **锁的可重入性**

  持有锁的线程，第二次又尝试获取该锁,能获取则表示为可重入锁。

- **允许中断**

  ReentrantLock.lock()方法若等待线程被中断(Thread.interupt()给线程打上中断标记),则在线程获取到锁之后,可通过Thread.interrupted()判断线程在阻塞过程中是否被中断过。即这个锁不能被强制中断，而是由程序员判断是否在获取锁之后需要中断线程。

  ReentrantLock.lockInterruptibly()方法在线程未获取到锁被阻塞的过程中，若被中断则抛出InterruptedException异常()。即此锁可以被强制中断，并以异常的方式进行后续处理。

## 源码解析

从`ReentrantLock.lock()`跟进源码中看,通过内部类`FairSync`继承`sync`,调用了`AbstractQueuedSynchronizer.acquire()`方法

```java
/*
以独占模式获取，忽略中断。通过调用至少一次tryAcquire来实现，并在成功时返回。否则线程将排队，可能会反复阻塞和取消阻塞，调用tryAcquire直到成功
*/
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            //若线程在竞争锁的过程中被中断过,则给该线程打中断标记
            selfInterrupt();
    }
```

方法的调用链如图:

![image-20210407225731856](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210407225731856.png)

### tryAcquire

> 尝试获取锁,并返回是否获取成功

```java

protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //hasQueuedPredecessors():等待队列是否为空,或者当前等待线程是否处于等待队列的第一位,即判断当前线程能否直接获取到锁
        //compareAndSetState(0, acquires):设置状态器为加锁状态
        //setExclusiveOwnerThread:设置获取到锁的线程为当前线程
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        //若获取到锁的线程又重复获取锁，则状态器state加锁状态增加一个票据单位(acquires);——————可重入特性
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

### addWaiter

> 等待锁的线程进入CLH队列

```java
private Node addWaiter(Node mode) {
    //Node.EXCLUSIVE 可以看到是null,所以这个Node节点的nextWaiter指针为空，即创建了一个独占模式的节点
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
    	//CLH队列不为空时,把当前线程的Node节点尝试加入队列最后
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            //入队成功之后,将之前队列中最后一个节点的next指针指向当前线程node节点
            pred.next = node;
            return node;
        }
    }
    //入队失败或者队列未创建，则CAS自旋入队
    enq(node);
    return node;
}
```

#### enq

> CAS自旋进入等待队列

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) {
            //CLH队列为空则初始化队列
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            //继续尝试入队
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

### acquireQueued

> 阻塞当前线程并返回线程是否被中断过状态

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                //或者当前线程Node节点在队列中的前置节点
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                //若当前线程位于CLH队列第一位,则继续尝试获取锁,成功则出队
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    //parkAndCheckInterrupt:阻塞当前线程并返回线程是否被中断过
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### shouldParkAfterFailedAcquire

> 判断当前线程是否应该阻塞

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    	//判断当前线程在队列中的前一个等待线程的状态
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            //前置节点状态为Node.SIGNAL则表示当前节点可以安全的阻塞
            return true;
        if (ws > 0) {
            //前置节点已经被取消,将前置Node节点移出CLH队列,直至前置节点为正常节点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //前置节点的waitStatus为0(初始化节点还未设置状态)，或者为 Node.PROPAGATE(广播状态),则当前线程不应该阻塞，并将前置节点状态设置为Node.SIGNAL(已就绪状态)
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

#### cancelAcquire

> 当前线程取消竞争锁

```java
 private void cancelAcquire(Node node) {
        // 当前节点为空直接返回
        if (node == null)
            return;
        node.thread = null;
        //跳过当前节点前面已失效节点,失效节点在此处被真正清理
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
        //获取前置节点的后一个节点,即当前节点
        Node predNext = pred.next;
        //失效当前节点
        node.waitStatus = Node.CANCELLED;
        // 若当前节点为CLH队列的队尾,则移除当前节点
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            int ws;
            //当前节点的前置节点不为CLH队列的队首,即当前节点不是CLH队列中第一个等待获取锁的节点
            if (pred != head &&
                //前置节点为可Node.SIGNAL(就绪状态)或者将前置节点状态设置为Node.SIGNAL(就绪状态)成功
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                //当前节点的下一个节点是正常节点时
                if (next != null && next.waitStatus <= 0)
                    //将前置节点next指针指向当前节点的下一个节点
                    compareAndSetNext(pred, predNext, next);
            } else {
                //当前节点是CLH队列中第一个等待获取锁的节点,或者前置节点并发下也已失效
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
```

##### unparkSuccessor

> 唤醒当前节点的下一个有效节点

```java
private void unparkSuccessor(Node node) {
    	//设置当前线程为0状态(初始化状态)
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
    	//下一个节点并发下已不存在或者已失效
        if (s == null || s.waitStatus > 0) {
            s = null;
            //从CLH队尾遍历,获取当前节点有效的下一个节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
    	//唤醒当前节点的下一个节点
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

## 拓展

### 线程中断

上述公平锁源码中有个地方需要拓展讲解一下,当线程在竞争锁的过程中被中断了，根据上述源码，并不会抛出`InterruptedException`,而是在方法中调用如下代码:

```java
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

**Thread.interrupt()**: 并不是字面上的中断线程的意思，而是给线程打上一个中断标记,线程依然会继续执行后续的代码,当我们使用ReentrantLock.lokc()时，可以编写代码通过**Thread.interrupted()**获取到此标记，判断线程是否被中断过，来做其他处理。

**Thread.interrupted()**: 获取线程是否中断过的状态,返回true或者false,并将线程中断标记置空，所以重复调用，第二次会是false;

**Thread.isInterrupted()**：获取线程是否中断过的状态，不会置空线程中断标记。

ReentrantLock也提供了**lockInterruptibly()**方法，在线程竞争锁时出现中断，直接抛出`InterruptedException`。

# BlockingQueue

BlockingQueue，是java.util.concurrent 包提供的用于解决并发生产者 - 消费者问题的最有用的类，它的特性是在任意时刻只有一个线程可以进行take或者put操作(**线程安全**)，并且BlockingQueue提供了超时return null的机制，在许多生产场景里都可以看到这个工具的身影。

- **队列类型**

  - 无限队列 （unbounded queue ） - 几乎可以无限增长

    ArrayBlockingQueue 由数组支持的有界队列
    LinkedBlockingQueue 由链接节点支持的可选有界队列

  - 有限队列 （ bounded queue ） - 定义了最大容量
    PriorityBlockingQueue 由优先级堆支持的无界优先级队列
    DelayQueue 由优先级堆支持的、基于时间的调度队列

- **队列数据结构**

  - 通常用链表或者数组实现
  - 一般而言队列具备FIFO先进先出的特性，当然也有双端队列（Deque）优先级队列
  - 主要操作：入队（EnQueue）与出队（Dequeue）

- **BlockingQueue API**

  BlockingQueue 接口的所有方法可以分为两大类：负责向队列添加元素的方法和检索这些元素的方法。在队列满/空的情况下，来自这两个组的每个方法的行为都不同。

| 方法                                    | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| add()                                   | 如果插入成功则返回 true，否则抛出 IllegalStateException 异常 |
| put()                                   | 将指定的元素插入队列，如果队列满了，那么会阻塞直到有空间插入 |
| offer()                                 | 如果插入成功则返回 true，否则返回 false                      |
| offer(E e, long timeout, TimeUnit unit) | 尝试将元素插入队列，如果队列已满，那么会阻塞直到有空间插入   |
| take()                                  | 获取队列的头部元素并将其删除，如果队列为空，则阻塞并等待元素变为可用 |
| poll(long timeout, TimeUnit unit)       | 检索并删除队列的头部，如有必要，等待指定的等待时间以使元素可用，如果超时，则返回 null |

## ArrayBlockingQueue

是一个用数组实现的有界阻塞 队列，按照先进先出(`FIFO`)的原则对元素进行排序。定义了notFull,notEmpty两个ConditionObject,其中各自维护了一个条件等待队列，根据条件唤醒对应队列里面的线程。

## LinkedBlockingQueue

用链表实现的有界阻塞队列。此队列的默认和最大长度为 `Integer.MAX_VALUE`.此队列按照先进先出的原则对元素进行排序。

## PriorityBlockingQueue

是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序升序排列。也可以自定义类实现`compareTo()`方法来指定元素排序规则，或者初始化`PriorityBlockingQueue`时，指定构造参数`Comparator`来对元素进行排序。需要注意的是不能保证同优先级元素的顺序。

## DelayQueue

`DelayQueue`是一个支持延时获取元素的无界阻塞队列。队列使用`PriorityQueue`来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素

# Semaphore

Semaphore 字面意思是信号量的意思，它的作用是控制访问特定资源的线程数目，底层依赖AQS的状态State，是在生产当中比较常用的一个工具类。一般用于资源访问，服务限流(Hystrix里限流就有基于信号量方式)。

**acquire()** ：表示阻塞并获取许可
**release()** ：表示释放许可

# CountDownLatch

CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

CountDownLatch.countDown()：
CountDownLatch.await()：

# CyclicBarrier

栅栏屏障，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。CyclicBarrier默认的构造方法是CyclicBarrier（int parties），其参数表示屏障拦截的线程数量，每个线程调用await方法告CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

cyclicBarrier.await()：

跟CountDownLatch反向使用类似效果，但是CyclicBarrier可重复使用