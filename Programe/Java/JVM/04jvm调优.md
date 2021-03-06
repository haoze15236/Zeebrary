# jinfo

```shell
#查看jvm配置
jinfo -flags [pid]
#查看java系统参数
jinfo -sysprops [pid]
```

# jps

查看jvm进程pid

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/EC84D5FEA98348298810AA1F885943EE/96322](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/EC84D5FEA98348298810AA1F885943EE/96322)

# jmap

此命令可以用来查看内存信息，实例个数以及占用内存大小

**应用场景：**内存长期不释放,导致内存溢出(OOM)

## -histo

```shell
jmap -histo 14660  #查看当前时刻生成的实例
jmap -histo:live 14660  #查看当前时刻存活的实例，执行过程中可能会触发一次full gc
jmap -histo [PID] > [文件名] #打印内存信息到指定文件中
```

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/B6CBAEA18BA34FF3BC25D03D3933A7D2/96302](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/B6CBAEA18BA34FF3BC25D03D3933A7D2/96302)

| 列名       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| num        | 序号排名                                                     |
| instances  | 实例数量                                                     |
| bytes      | 占用空间大小,单位是字节，1024字节=1KB                        |
| class name | 类名称，[C is a char[]，[S is a short[]，[I is a int[]，[B is a byte[]，[[I is a int[][] |

## -heap

此参数用于查看堆信息,比如堆的配置信息(Heap Configuration),堆的使用情况(Heap Usage)

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/7273899836004DFD8921CB96ADA63635/96304](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/7273899836004DFD8921CB96ADA63635/96304)

## -dump

到处jvm的堆信息dump文件，用于可视化工具(如jvisualvm)做数据分析。

```shell
#将PID=14660的进程堆信息到处为dump文件eureka.hprof
jmap -dump:format=b,file=eureka.hprof 14660
#也可以设置内存溢出自动导出dump文件(内存很大的时候，可能会导不出来)
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./   #（路径）
```

# jstack

用于查看jvm的线程信息,一般用于分析**CPU占用过高**的情况。

```shell
#查看PID=14660的jvm进程的线程信息
jstack 14660
```

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/4FB8482FD6954EB58256FFDE72FAF417/96306](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/4FB8482FD6954EB58256FFDE72FAF417/96306)

| 参数                            | 说明                      |
| ------------------------------- | ------------------------- |
| "Thread-1"                      | 线程名                    |
| prio=5                          | 优先级=5                  |
| tid=0x000000001fa9e000          | 线程id                    |
| nid=0x2d64                      | 线程对应的本地线程标识nid |
| java.lang.Thread.State: BLOCKED | 线程状态                  |

# 快速定位CPU异常代码

1.使用命令top -p <pid> ，显示你的java进程的内存情况，pid是你的java进程号，比如19663

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/95F189D34BB34C09816B6D95CAB62005/96310](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/95F189D34BB34C09816B6D95CAB62005/96310)

2.按H，获取每个线程的内存情况

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/449A8C6C90F647818D9B15E279F8DACE/96324](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/449A8C6C90F647818D9B15E279F8DACE/96324)

3.找到内存和cpu占用最高的线程tid，比如19664

4.使用**printf "%x\n" 19664** 打印对应PID十六进制 0x4cd0，此为线程id的十六进制表示

5.执行 jstack 19663|grep -A 10 4cd0，得到线程堆栈信息中 4cd0 这个线程所在行的后面10行，从堆栈中可以发现导致cpu飙高的调
用方法

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/90A784EAB9894E209E9161B7C17096F5/96326](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/90A784EAB9894E209E9161B7C17096F5/96326)

# jstat

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：

jstat [-命令选项] [vmid] [间隔时间(毫秒)] [查询次数]

注意：使用的jdk版本是jdk8

## -gc：垃圾回收统计

```shell
# 查看PID=13998的gc情况(每隔1000ms执行1次命令，共执行10次,不加则只打印当前时刻)
jstat -gc 13998 1000 10
```

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/A1CCAEEFC56D4EE387D7EC6348C6F8BB/96314](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/A1CCAEEFC56D4EE387D7EC6348C6F8BB/96314)

| 字段 | 含义                          |
| ---- | ----------------------------- |
| S0C  | 第一个幸存区的大小，单位KB    |
| S1C  | 第二个幸存区的大小            |
| S0U  | 第一个幸存区的使用大小        |
| S1U  | 第二个幸存区的使用大小        |
| EC   | 伊甸园区的大小                |
| EU   | 伊甸园区的使用大小            |
| OC   | 老年代大小                    |
| OU   | 老年代使用大小                |
| MC   | 方法区大小(元空间)            |
| MU   | 方法区使用大小                |
| CCSC | 压缩类空间大小                |
| CCSU | 压缩类空间使用大小            |
| YGC  | 年轻代垃圾回收次数            |
| YGCT | 年轻代垃圾回收消耗时间，单位s |
| FGC  | 老年代垃圾回收次数            |
| FGCT | 老年代垃圾回收消耗时间，单位s |
| GCT  | 垃圾回收消耗总时间，单位s     |

### 新生代垃圾回收统计

​    ![0](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/8FECFAF1EE994EBBAEB93E1BA7E87AC9/96317)

- S0C：第一个幸存区的大小
- S1C：第二个幸存区的大小
- S0U：第一个幸存区的使用大小
- S1U：第二个幸存区的使用大小
- TT:对象在新生代存活的次数
- MTT:对象在新生代存活的最大次数
- DSS:期望的幸存区大小
- EC：伊甸园区的大小
- EU：伊甸园区的使用大小
- YGC：年轻代垃圾回收次数
- YGCT：年轻代垃圾回收消耗时间

### 新生代内存统计

​    ![0](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/8E0C8BC666064276AA8CEA80BA60D219/96315)

- NGCMN：新生代最小容量
- NGCMX：新生代最大容量
- NGC：当前新生代容量
- S0CMX：最大幸存1区大小
- S0C：当前幸存1区大小
- S1CMX：最大幸存2区大小
- S1C：当前幸存2区大小
- ECMX：最大伊甸园区大小
- EC：当前伊甸园区大小
- YGC：年轻代垃圾回收次数
- FGC：老年代回收次数

### 老年代垃圾回收统计

​    ![0](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/390D87E5743D48BF9CA052A6E8652D46/96319)

- MC：方法区大小
- MU：方法区使用大小
- CCSC:压缩类空间大小
- CCSU:压缩类空间使用大小
- OC：老年代大小
- OU：老年代使用大小
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

### 老年代内存统计

​    ![0](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/B529A9B7E2404516942E67285253771F/96318)

- OGCMN：老年代最小容量
- OGCMX：老年代最大容量
- OGC：当前老年代大小
- OC：老年代大小
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

### 元数据空间统计

 ![0](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/E25CDA3658324BD6B9E13985209D566D/96316)

- MCMN:最小元数据容量
- MCMX：最大元数据容量
- MC：当前元数据空间大小 
- CCSMN：最小压缩类空间大小
- CCSMX：最大压缩类空间大小
- CCSC：当前压缩类空间大小
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

​    ![0](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/A4241648D94C49E994C970E5C2FA9EED/96321)

- S0：幸存1区当前使用比例
- S1：幸存2区当前使用比例
- E：伊甸园区使用比例
- O：老年代使用比例
- M：元数据区使用比例
- CCS：压缩使用比例
- YGC：年轻代垃圾回收次数
- FGC：老年代垃圾回收次数
- FGCT：老年代垃圾回收消耗时间
- GCT：垃圾回收消耗总时间

# JVM优化

## **优化原则**

其实简单来说就是尽量让每次<span style = "color:red">Young GC后的存活对象小于Survivor区域的50%</span>,避免由于触发**对象动态年龄判断机制**导致年龄大的对象直接进入老年代,同时要兼顾使<span style = "color:red">老年代剩余区域大小大于每次young GC之后存活对象大小</span>(**老年代空间担保机制**),尽量减少Full GC的频率，避免频繁Full GC对JVM性能的影响。

## 优化思路

首先命令构建如图所示的线上系统运行模型。

![https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/4AD3574884C54C7DBF8836737492178E/96307](https://note.youdao.com/yws/public/resource/5cc182642eb02bc64197788c7722baae/xmlnote/4AD3574884C54C7DBF8836737492178E/96307)

### **年轻代对象增长的速率**

#### 平均算法

<span style = "color:green">eden区大小(MB)</span>/<span style = "color:blue">平均发生Young GC的时间(S)</span>= <span style = "color:red">线程每秒产生的对象数量(MB/s)</span>

<span style = "color:green">eden区大小(MB):</span> 

1. 可通过`jinfo -flags`查看jvm参数设置查看

2. 可通过`jmap -heap` 查看堆配置信息

![image-20210303003838512](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210303003838512.png)

<span style = "color:blue">平均发生Young GC的时间(S)</span>：系统运行时间/Young GC 次数 = （12小时*3600）/306 = 141S(s)

1. 通过`jstat -gc `查看Young GC 次数YGC

![image-20210303003917474](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210303003917474.png)

通过计算,每秒大概生成27M的对象,

#### 实时监控

可以执行命令 jstat -gc pid 1000 10 (每隔1秒执行1次命令，共执行10次)，通过观察EU(eden区的使用)来估算每秒eden大概新增多少对
象，如果系统负载不高，可以把频率1秒换成1分钟，甚至10分钟来观察整体情况。注意，一般系统可能有高峰期和日常期，所以需要在不同的时间分别估算不同情况下对象增长速率。

### 老年代对象增长速率

每次young GC后进入老年代的对象大小。

#### 实时监控

这个因为之前已经大概知道Young GC的频率，假设是每5分钟一次，那么可以执行命令 jstat -gc pid 300000 10 ，观察每次结果eden，
survivor和老年代使用的变化情况，在每次gc后eden区使用一般会大幅减少，survivor和老年代都有可能增长，这些增长的对象就是每次
Young GC后存活的对象，同时还可以看出每次Young GC后进去老年代大概多少对象，从而可以推算出老年代对象增长速率。

