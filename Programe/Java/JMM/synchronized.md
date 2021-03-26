# synchronized

synchronized内置锁是一种对象锁(锁的是对象而非引用)，作用粒度是对象，可以用来实现对临界资源的同步互斥访问。

## 发展

### jdk1.6之前

jvm实现synchronized锁机制是通过给每个对象维护一个Monitor管程,而管程依赖于操作系统的Mutex互斥量，jvm要调用Mutex互斥量需要在内核态下调用Pthread库,由于jvm是运行在用户态下，因此为了调用操作系统底层库就涉及到从**用户态**到**内核态**的转换,这种转换十分的消耗性能。

### jdk1.6之后

由于1.6以前的机制性能十分低下,doug li教授一己之力实现了AQS框架(JUC包下的大部分内容),这让收购了sun公司的oracle对synchronized进行了大幅度优化:引入了偏向锁，轻量级锁,重量级锁等锁的概念,膨胀升级,提升了synchronized的性能。

## 使用

synchronized使用很简单，但是我们必须先要理解synchronized锁的粒度，在此之前要先清楚一个概念——**java中每个类(class)可以创建任意多个实例对象(instance),同时类(class)本身也是一个对象,通过`类名.class`可以获取类对象**。结合前面说的synchronized是一种对象锁,那么其锁的粒度就很好理解了。

### 锁类(class)对象

<span style="color:red">当使用sychronized锁住class对象时，该类所有new出来的实例对象(instance)也会被锁住。即无论new了多少个对象,同一时间所有对象只会有一个线程能获取到锁。</span>

#### 加锁方式

##### 锁static修饰的方法

很好理解，因为static修饰的方法或者变量，可以不用创建实例对象直接访问,这种机制的底层原理就是由于每个类都有一个类(class)对象，代码示例如下:

```java
public class JucSync {
    public static synchronized void descStock(){
		//业务代码
    }
}
```

##### 同步代码块中声明锁的类对象

```java
public class JucSync {
    public void descStock(){
        synchronized(xxx.class){
            //业务代码
        }
    }
}
```

### 锁实例对象(instance)

<span style="color:red">new出来的对象之间不互斥,同一个实例对象同一时间只会有一个线程获取到锁</span>

#### 加锁方式

##### 锁非静态方法

```java
public class JucSync {
    public synchronized void descStock(){
		//业务代码
    }
}
```

##### 同步代码块中声明锁的实例对象

```java
public class JucSync {
    public void descStock(){
        //this代表锁当前类的当前实例对象
        synchronized(this){
			//业务代码
        }
        //或者任意一个实例对象,比如锁住前面创建的test对象
        LockObject test = new LockObject();
        synchronized(test){
			//业务代码
        }
    }
}
```

## 原理



