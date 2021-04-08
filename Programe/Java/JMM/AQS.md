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

  ```Java
  final boolean acquireQueued(final Node node, int arg) {
          boolean failed = true;
          try {
              boolean interrupted = false;
              for (;;) {
                  //开始自旋
                  final Node p = node.predecessor();
                  //若当前线程位于等待队列的第一位，则再尝试获取锁,因为大部分情况下锁很快会被释放,这样设计可提高性能
                  if (p == head && tryAcquire(arg)) {
                      setHead(node);
                      p.next = null; // help GC
                      failed = false;
                      return interrupted;
                  }
                  if (shouldParkAfterFailedAcquire(p, node) &&
                      parkAndCheckInterrupt())
                      interrupted = true;
              }
          } finally {
              if (failed)
                  cancelAcquire(node);
          }
      }
  ```

  

- **队列：**若自旋一定次数之后,抢锁失败,则把当前抢锁的线程存储在队列中，等待锁释放之后,从队列中出队抢锁。

  查看**AbstractQueuedSynchronizer**源码可以看到,其内部定义了一个名为Node的对象作为等待对象,其本质是一个双向链表，其结构如下图:

  ![image-20210329235631649](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210329235631649.png)

  上图中，head指针永远指向头节点，tail指针永远指向队列的尾节点，可以看到头节点的Thread=null，这是初始化队列时的一个初始节点，当head指针与tail指针都指向Thread=null的节点时，意味着等待队列为空。这样设计可以比较优雅的避免java中的空指针异常。

- **LockSupport：**JVM提供的一个API，当线程抢锁失败，进入等待队列中后，使用park()阻塞线程，当锁释放，线程出队抢到锁之后，使用unpark()唤醒阻塞线程。

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
            selfInterrupt();
    }
```

方法的调用链如图:

![image-20210407225731856](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210407225731856.png)

### tryAcquire()

> 尝试获取锁并返回是否获取成功

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
                //若获取到锁的线程又重复获取锁，则状态器state加锁状态增加一个票据单位(acquires);
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

### addWaiter(Node.EXCLUSIVE)

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

## ArrayBlockingQueue

# Semaphore

# CountDownLatch