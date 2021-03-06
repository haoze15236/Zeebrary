[并发编程思维导图](https://www.processon.com/view/link/5d81dec7e4b04c14c4e7aac8#map)

# JMM介绍

Java内存模型(java memory model)是一个**抽象**概念,与计算机底层硬件并无直接对应。JMM抽象出来两个概念:

**主内存:** 	所有java实例对象(常量，静态变量，类信息，成员变量，方法中的局部变量)，存在的区域统一称为主内存。

**工作内存:** 每个线程只访问自己的工作内存获取变量,对于8大基础数据类型变量，直接在**栈帧**上创建，而非基础数据类型变量，都只持有一个对象引用，指向对象在主内存的内存地址,需要的时候从主内存区域拷贝一个副本到工作内存，方法执行完毕，将副本修改刷新到主内存。

模型图如下:

![image-20210324001505317](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210324001505317.png)

在JMM中,主内存与工作内存的交互，定义了8大**原子**操作来完成。

（1）lock(锁定)：作用于主内存的变量，把一个变量标记为一条线程独占状态
（2）unlock(解锁)：作用于主内存的变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
（3）read(读取)：作用于主内存的变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
（4）load(载入)：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中
（5）use(使用)：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎
（6）assign(赋值)：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量
（7）store(存储)：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作
（8）write(写入)：作用于工作内存的变量，它把store操作从工作内存中的一个变量的值传送到主内存的变量中

![image-20210324002314576](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210324002314576.png)

由于JMM只要求上述操作必须按顺序执行，而没有保证必须是连续执行。

# 并发三特性

**原子性：** 指一个操作是不可中断的，要么全部完成，要么执行失败。

​	jvm实现了基础数据类型的原子性操作,另外提供了synchronized和Lock可以保证操作的原子性。

**可见性：** 当某个线程修改了某个共享变量,会使用原子操作刷新到主内存中,同时其他线程会立即从主内存中重新读取该变量最新的值。

​	java提供volatile关键字保证变量的可见性

**有序性：**在单个线程中代码顺序执行。

​	由于JMM定义了工作内存的概念，每个线程都有自己的工作内存，因此java天然具有有序性。另外synchronized和Lock保证了同一时间只有一个线程执行代码,也保证了代码的有序性。

# volatile

volatile是Java虚拟机提供的轻量级的同步机制。volatile关键字有如下两个作用：

1. 保证被volatile修饰的共享变量对所有线程总数可见的，也就是当一个线程修改了一个被volatile修饰共享变量的值，新值总是可以被其他线程立即得知。
2. 禁止指令重排序优化