# 1. Arthas（阿尔萨斯）概述

> Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。

![_images/arthas.png](http://arthas.gitee.io/_images/arthas.png)

当你遇到以下类似问题而束手无策时，arthas可以帮助你解决：

1. 这个类是从哪个jar包加载的？为什么会报各种类相关的Exception？
2. 我改的代码为什么没有执行到？难道是我没Commit？分支搞错了？
3. 遇到问题无法在线上debug，难道只能通过加日志再重新发布？
4. 线上遇到某个用户的数据处理有问题，单线上同样无法debug，线下无法再现。
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？
7. 怎么快速定位应用的热点，生成火焰图？

# 2. 安装Arthas

## 2.1 运行环境要求

`Arthas` 支持 JDK6+, 支持Linux/Mac/Windows，采用命令行交互模式，同事提供丰富的Tab自动补全功能，进一步方便进行问题的定位和诊断。

## 2.2 快速安装

### 2.2.1 使用arthas-boot.jar

1. 下载`arthas-boot.jar`，然后用`java -jar`的方式启动：

   ```sh
   curl -O https://arthas.aliyun.com/arthas-boot.jar
   java -jar arthas-boot.jar
   
   # 启动arthas-boot.jar时需要后台有正在运行的java进程，否则会无法启动，错误信息如下
   [ligen@center_server ~]$ java -jar arthas-boot.jar 
   [INFO] arthas-boot version: 3.4.6
   [INFO] Can not find java process. Try to pass <pid> in command line.
   Please select an available pid.
   ```

2. 启动并选中java进程，成功安装并启动arthas

   ```sh
   [ligen@center_server ~]$ java -jar arthas-boot.jar
   [INFO] arthas-boot version: 3.4.6
   [INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
   * [1]: 32416 docker-demo.jar
   1
   [INFO] Start download arthas from remote server: https://arthas.aliyun.com/download/3.4.6?mirror=aliyun
   [INFO] File size: 11.99 MB, downloaded size: 3.38 MB, downloading ...
   [INFO] File size: 11.99 MB, downloaded size: 6.47 MB, downloading ...
   [INFO] File size: 11.99 MB, downloaded size: 10.08 MB, downloading ...
   [INFO] Download arthas success.
   [INFO] arthas home: /home/ligen/.arthas/lib/3.4.6/arthas
   [INFO] Try to attach process 32416
   [INFO] Attach process 32416 success.
   [INFO] arthas-client connect 127.0.0.1 3658
     ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.                           
    /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'                          
   |  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.                          
   |  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |                         
   `--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'                          
                                                                                   
   
   wiki      https://arthas.aliyun.com/doc                                         
   tutorials https://arthas.aliyun.com/doc/arthas-tutorials.html                   
   version   3.4.6                                                                 
   pid       32416                                                                 
   time      2021-01-30 15:14:01     
   ```

3. 启动时打印帮助信息

   ```sh
   java -jar arthas-boot.jar -h
   ```

4. 下载较慢使用阿里云镜像：

   ```sh
   java -jar arthas-boot.jar --repo-mirror aliyun --use-http
   ```

### 2.2.2 使用as.sh

Arthas 支持在 Linux/Unix/Mac 等平台上一键安装，请复制以下内容，并粘贴到命令行中，敲 `回车` 执行即可：

```sh
curl -L https://arthas.aliyun.com/install.sh | sh
```

上述命令会下载启动脚本文件 `as.sh` 到当前目录，你可以放在任何地方或将其加入到 `$PATH` 中。

直接在shell下面执行`./as.sh`，就会进入交互界面。

也可以执行`./as.sh -h`来获取更多参数信息。

## 2.3 全量安装

最新版本，点击下载：[![https://img.shields.io/maven-central/v/com.taobao.arthas/arthas-packaging.svg?style=flat-square](img/arthas-packaging.svg)](https://arthas.aliyun.com/download/latest_version?mirror=aliyun)

解压后，在文件夹里有`arthas-boot.jar`，直接用`java -jar`的方式启动：

```sh
java -jar arthas-boot.jar
```

打印帮助信息：

```sh
java -jar arthas-boot.jar -h
```

# 3. 快速入门

1. 启动`arthas-demo.jar`文件

   `arthas-demo`是一个简单的程序，每隔一秒生成一个随机数，再执行质因数分解，并打印出分解结果。

   `arthas-demo`源代码：[查看](https://github.com/alibaba/arthas/blob/master/demo/src/main/java/demo/MathGame.java)

   ```sh
   [ligen@center_server ~]$ cd ./.arthas/lib/3.4.6/arthas/
   [ligen@center_server arthas]$ ls -alrt
   总用量 13216
   drwxr-xr-x. 3 ligen wheel       20 1月  30 15:13 ..
   -rw-r--r--. 1 ligen wheel     8889 1月  30 15:13 arthas-spy.jar
   -rw-r--r--. 1 ligen wheel 12866578 1月  30 15:14 arthas-core.jar
   -rw-r--r--. 1 ligen wheel     2020 1月  30 15:14 logback.xml
   -rw-r--r--. 1 ligen wheel      311 1月  30 15:14 arthas.properties
   -rw-r--r--. 1 ligen wheel     8438 1月  30 15:14 arthas-agent.jar
   -rw-r--r--. 1 ligen wheel   428578 1月  30 15:14 arthas-client.jar
   -rw-r--r--. 1 ligen wheel      635 1月  30 15:14 install-local.sh
   -rw-r--r--. 1 ligen wheel    32131 1月  30 15:14 as.sh
   -rw-r--r--. 1 ligen wheel     3091 1月  30 15:14 as.bat
   -rw-r--r--. 1 ligen wheel     4484 1月  30 15:14 arthas-demo.jar
   -rw-r--r--. 1 ligen wheel   138993 1月  30 15:14 arthas-boot.jar
   -rw-r--r--. 1 ligen wheel     7744 1月  30 15:14 as-service.bat
   drwxr-xr-x. 2 ligen wheel      156 1月  30 15:14 async-profiler
   drwxr-xr-x. 2 ligen wheel        6 1月  30 15:27 arthas-output
   drwxr-xr-x. 4 ligen wheel     4096 1月  30 15:27 .
   
   
   # 启动arthas-demo.jar
   java -jar arthas-demo.jar
   ```

2. 启动`arthas`并attach上对应的`arthas-demo.jar`文件

   ```sh
   [ligen@center_server arthas]$ java -jar arthas-boot.jar 
   [INFO] arthas-boot version: 3.4.6
   [INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
   * [1]: 32416 docker-demo.jar
     [2]: 15888 arthas-demo.jar
   2
   [INFO] arthas home: /home/ligen/.arthas/lib/3.4.6/arthas
   [INFO] Try to attach process 15888
   [INFO] Attach process 15888 success.
   [INFO] arthas-client connect 127.0.0.1 3658
     ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.                           
    /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'                          
   |  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.                          
   |  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |                         
   `--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'                          
                                                                                   
   
   wiki      https://arthas.aliyun.com/doc                                         
   tutorials https://arthas.aliyun.com/doc/arthas-tutorials.html                   
   version   3.4.6                                                                 
   pid       15888                                                                 
   time      2021-01-30 15:27:31                                                   
   
   [arthas@15888]$ 
   ```

3. 注意事项

   启动`arthas`时可能会如果已经`attach`了进程可能会报错，提示`arthas`的默认端口3658已经被其他java进程占用，信息如下：

   ```sh
   [ligen@center_server arthas]$ java -jar arthas-boot.jar 
   [INFO] arthas-boot version: 3.4.6
   [INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
   * [1]: 32416 docker-demo.jar
     [2]: 15888 arthas-demo.jar
   2
   [INFO] arthas home: /home/ligen/.arthas/lib/3.4.6/arthas
   [ERROR] The telnet port 3658 is used by process 32416 instead of target process 15888, you will connect to an unexpected process.
   [ERROR] 1. Try to restart arthas-boot, select process 32416, shutdown it first with running the 'stop' command.
   [ERROR] 2. Or try to stop the existing arthas instance: java -jar arthas-client.jar 127.0.0.1 3658 -c "stop"
   [ERROR] 3. Or try to use different telnet port, for example: java -jar arthas-boot.jar --telnet-port 9998 --http-port -1
   ```

   此时解决办法如下：

   1. 启动`arthas`时不使用默认端口号，指定新的端口号启动

      ```sh
      java -jar arthas-boot.jar --telnet-port 9999 --http-port -1
      ```

   2. 重新`attach`占用`3658`端口的java进程，并执行`stop`，终止arthas对该java进程的attach。

      ```sh
      [arthas@32416]$ stop
      Resetting all enhanced classes ...
      Affect(class count: 0 , method count: 0) cost in 1 ms, listenerId: 0
      Arthas Server is going to shut down...
      ```

# 4. 通过浏览器访问arthas

Arthas目前支持Web Console，用户在attach成功之后，可以直接访问本机：http://127.0.0.1:3658/

默认情况下，arthas只会listen 127.0.0.1，如果需要远程，则需要在启动arthas时使用`--target-ip 'IP地址'`指定listen的IP

示例如下：

```sh
[ligen@center_server arthas]$ java -jar arthas-boot.jar --target-ip 10.8.0.3 --http-port 9999

# 会看到启动日志中有一行arthas-client，这个就是连接的ip(10.8.0.3)和端口(3658)
[INFO] arthas-client connect 10.8.0.3 3658
```

启动之后图示如下

![image-20210130161655252](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210130161655252.png)

# 5. 快速入门：常用命令

## 5.1 dashboard仪表板

输入`dashboard`，按`回车/enter`，会展示当前进程的信息，按`ctrl+c`可以中断执行。

注：输入前面部分字母，按tab可以自动补全命令

1. 第一部分是显示JVM中运行的所有线程：所在线程组，优先级，线程的状态，CPU的占用率，是否是后台进程等
2. 第二部分显示的JVM内存的使用情况
3. 第三部分是操作系统的一些信息和Java版本号，如下图所示：

![image-20210130162125337](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210130162125337.png)

## 5.2 通过thread命令来获取到`arthas-demo`进程的Main Class

输入thread命令会展示arthas-demo所有的线程

![image-20210130164210154](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210130164210154.png)

`thread 1`会打印线程ID 1的栈，通常是main函数的线程。

```sh
[arthas@15888]$ thread 1
"main" Id=1 TIMED_WAITING
    at java.lang.Thread.sleep(Native Method)
    at java.lang.Thread.sleep(Thread.java:340)
    at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
    at demo.MathGame.main(MathGame.java:17)
```

## 5.3 通过jad来反编译Main Class

输入 `jad [包名].[类名] ` 进行反编译

```sh
jad demo.MathGame
```

返回结果：

```sh
[arthas@15888]$ jad demo.MathGame 
# 类加载器
ClassLoader:                                                                                                                                                                                                                  
+-sun.misc.Launcher$AppClassLoader@70dea4e       # 程序类加载器                                                                                                                                                                                 
  +-sun.misc.Launcher$ExtClassLoader@7d4991ad    # 扩展类加载器
  
#jar包所在位置
Location:                                                                                                                                                                                                                     
/home/ligen/.arthas/lib/3.4.6/arthas/arthas-demo.jar                                                                                                                                                             


## 反编译代码结果
/*
 * Decompiled with CFR.
 */
package demo;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class MathGame {
    private static Random random = new Random();
    private int illegalArgumentCount = 0;

    public List<Integer> primeFactors(int number) {
        if (number < 2) {
            ++this.illegalArgumentCount;
            throw new IllegalArgumentException("number is: " + number + ", need >= 2");
        }
        ArrayList<Integer> result = new ArrayList<Integer>();
        int i = 2;
        while (i <= number) {
            if (number % i == 0) {
                result.add(i);
                number /= i;
                i = 2;
                continue;
            }
            ++i;
        }
        return result;
    }

    public static void main(String[] args) throws InterruptedException {
        MathGame game = new MathGame();
        while (true) {
            game.run();
            TimeUnit.SECONDS.sleep(1L);
        }
    }

    public void run() throws InterruptedException {
        try {
            int number = random.nextInt() / 10000;
            List<Integer> primeFactors = this.primeFactors(number);
            MathGame.print(number, primeFactors);
        }
        catch (Exception e) {
            System.out.println(String.format("illegalArgumentCount:%3d, ", this.illegalArgumentCount) + e.getMessage());
        }
    }

    public static void print(int number, List<Integer> primeFactors) {
        StringBuffer sb = new StringBuffer(number + "=");
        for (int factor : primeFactors) {
            sb.append(factor).append('*');
        }
        if (sb.charAt(sb.length() - 1) == '*') {
            sb.deleteCharAt(sb.length() - 1);
        }
        System.out.println(sb);
    }
}

Affect(row-cnt:1) cost in 606 ms.

```

## 5.4 watch监视

通过`watch`命令来查看`demo.MathGame#primeFactors`函数的返回值：

```sh
# watch [包].[类名] [方法] returnObj
watch demo.MathGame primeFactors returnObj
```

返回结果：

```sh
[arthas@15888]$ watch demo.MathGame primeFactors returnObj
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 82 ms, listenerId: 1
method=demo.MathGame.primeFactors location=AtExit
ts=2021-01-30 16:52:06; [cost=1.167615ms] result=@ArrayList[
    @Integer[3],
    @Integer[5],
    @Integer[17],
    @Integer[19],
    @Integer[19],
]
method=demo.MathGame.primeFactors location=AtExit
ts=2021-01-30 16:52:07; [cost=4.643083ms] result=@ArrayList[
    @Integer[128831],
]
method=demo.MathGame.primeFactors location=AtExceptionExit
ts=2021-01-30 16:52:08; [cost=0.260649ms] result=null
method=demo.MathGame.primeFactors location=AtExceptionExit
ts=2021-01-30 16:52:09; [cost=0.10574ms] result=null
method=demo.MathGame.primeFactors location=AtExceptionExit
ts=2021-01-30 16:52:10; [cost=0.096172ms] result=null
```

## 5.5 退出arthas

如果只是退出当前的连接，可以用`quit`或者`exit`命令。Attach到目标进程上的arthas还会继续运行，端口会保持开放，下次连接时可以直接连接上。

如果想完全退出arthas，可以执行`stop`命令。

# 6. 基础命令之一

## 6.1 `hlep` 帮助命令

1. **作用**

   查看命令帮助信息

2. **效果**

   ```sh
   [arthas@15888]$ help
    NAME         DESCRIPTION                                                                                                                                                                                                     
    help         Display Arthas Help                                                                                                                                                                                             
    keymap       Display all the available keymap for the specified connection.                                                                                                                                                  
    sc           Search all the classes loaded by JVM                                                                                                                                                                            
    sm           Search the method of classes loaded by JVM                                                                                                                                                                      
    classloader  Show classloader info                                                                                                                                                                                           
    jad          Decompile class                                                                                                                                                                                                 
    getstatic    Show the static field of a class                                       
   ```

## 6.2 `cat` 

1. **作用**

   打印文件内容，和linux里的cat命令类似

   如果没有写路径，则显示当前目录下的文件

2. **效果**

   ```sh
   [arthas@15888]$ cat /home/ligen/springbootdemo/target/log/app.log
   ```

   ​	![image-20210130170551078](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210130170551078.png)

## 6.3 `grep`

1. **作用**

   匹配查找，和linux里的grep命令类似，但是它只能用于管道命令

2. **语法**

   | 参数作用        | 作用                                 |
   | --------------- | ------------------------------------ |
   | -n              | 显示行号                             |
   | -i              | 忽略大小写查找                       |
   | -m 行数         | 最大显示行数，要与查询字符串一起使用 |
   | -e “正则表达式” | 使用正则表达式查找                   |

3. **举例**

   ```sh
   # 只显示包含'java'(字符串)的行系统信息
   [arthas@15888]$ sysprop | grep java
    java.specification.version                  1.8
    java.class.path                             arthas-demo.jar
    java.vm.vendor                              Oracle Corporation
    java.vendor.url                             http://java.oracle.com/
    java.vm.specification.version               1.8
    sun.java.launcher                           SUN_STANDARD
   ...
   
   # 只显示3 行 包含'java'(字符串)的行系统信息,并带所在位置行号
   [arthas@15888]$ sysprop | grep java -m 3 -n
   5: java.specification.version                  1.8
   8: java.class.path                             arthas-demo.jar
   9: java.vm.vendor                              Oracle Corporation
   ```

   

## 6.4 `pwd`

1. 作用

   返回当前的工作目录，和linux命令类似

   pwd：Print Work Directory 打印当前工作目录 

2. 效果

   ```sh
   [arthas@15888]$ pwd
   /home/ligen/.arthas/lib/3.4.6/arthas
   ```

## 6.5 `cls`

作用：

清空当前屏幕区域，与windows命令一致。

# 7. 基础命令之二

## 7.1 `session`

1. 作用

   查看当前会话的信息

2. 效果

   ```sh
   [arthas@15888]$ session
    Name        Value                                                                                                                                                                                                            
   --------------------------------------------------                                                                                                                                                                            
    JAVA_PID    15888                                                                                                                                                                                                            
    SESSION_ID  cfa322a1-d619-4c69-8c12-42ecbe175c84  
   ```

## 7.2 `reset`

1. 作用

   重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类

2. 语法

   ```sh
   # 还原指定类
   reset Test
   # 还原所有List结尾的类
   rest *List
   # 还原所有的类
   reset
   # reset也支持正则表达式
   reset -E regex
   ```

3. 示例

   还原指定的类

   ```sh
   [arthas@15888]$ trace demo.MathGame print
   Press Q or Ctrl+C to abort.
   Affect(class count: 1 , method count: 1) cost in 56 ms, listenerId: 3
   `---ts=2021-01-30 17:37:49;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@70dea4e
       `---[0.577801ms] demo.MathGame:print()
   
   `---ts=2021-01-30 17:37:53;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@70dea4e
       `---[0.116217ms] demo.MathGame:print()
   [arthas@15888]$ reset demo.MathGame
   Affect(class count: 1 , method count: 0) cost in 44 ms, listenerId: 0
   
   ```

   还原所有的类

   ```sh
   [arthas@15888]$ trace demo.MathGame print
   Press Q or Ctrl+C to abort.
   Affect(class count: 1 , method count: 1) cost in 50 ms, listenerId: 4
   `---ts=2021-01-30 17:39:51;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@70dea4e
       `---[0.379101ms] demo.MathGame:print()
   
   `---ts=2021-01-30 17:39:52;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@70dea4e
       `---[0.118382ms] demo.MathGame:print()
   
   `---ts=2021-01-30 17:39:53;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@70dea4e
       `---[0.113563ms] demo.MathGame:print()
   
   [arthas@15888]$ reset
   Affect(class count: 1 , method count: 0) cost in 40 ms, listenerId: 0
   ```

## 7.3 `version`

1. 作用

   输出当前目标 Java 进程所加载的 Arthas 版本号

2. 效果

   ```sh
   [arthas@15888]$ version
   3.4.6
   ```

## 7.4 `history`

1. 作用

   打印输入过的历史命令

2. 效果

   ```sh
   [arthas@15888]$ history
       1  exit
       2  session | plaintext
       3  quit
       4  session | plaintext
       5  quit
       6  stop
       7  [arthas@32416]$ stop
       8  Resetting all enhanced classes ...
       9  Affect(class count: 0 , method count: 0) cost in 1 ms, listenerId: 0
      10  Arthas Server is going to shut down...
      11  stop
      12  stop
      13  dashboard 
      14  cls
   ......
   ```

## 7.5 `quit`

1. 作用

   退出当前 Arthas 客户端，其他 Arthas 客户端不受影响

## 7.6 `stop`

1. 作用

   关闭 Arthas 服务端，所有 Arthas 客户端全部退出

2. 效果：结束所有会话

   ```sh
   [arthas@15888]$ stop
   Resetting all enhanced classes ...
   Affect(class count: 0 , method count: 0) cost in 2 ms, listenerId: 0
   Arthas Server is going to shut down...
   [arthas@15888]$ session (0d2ff8e6-488f-4c41-8b7e-a42ee18d4fe0) is closed because server is going to shutdown.
   ```

   

## 7.7 `keymap`

1. 作用

   Arthas快捷键列表及自定义快捷键

2. 效果：展示所有快捷键

   ```sh
   [arthas@15888]$ keymap
    Shortcut                                                  Description                                              Name                                                                                                              
   --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    "\C-a"                                                    Ctrl + a                                                  beginning-of-line                                                                                                
    "\C-e"                                                    Ctrl + e                                                  end-of-line                                                                                                      
    "\C-f"                                                    Ctrl + f                                                  forward-word                                                                                                     
    "\C-b"                                                    Ctrl + b                                                  backward-word                                                                                                    
    "\e[D"                                                    Left arrow                                                backward-char                                                                                                    
    "\e[C"                                                    Right arrow                                               forward-char                                                                                                     
    "\e[A"                                                    Up arrow                                                  history-search-backward                                                                                          
    "\e[B"                                                    Down arrow                                                history-search-forward                                                                                           
    "\C-h"                                                    Ctrl + h                                                  backward-delete-char                                                                                             
    "\C-?"                                                    Ctrl + ?                                                  backward-delete-char                                                                                             
    "\C-u"                                                    Ctrl + u                                                  undo                
    。。。。
   ```

3. 常用快捷键汇总

   | 快捷键说明       | 命令说明                         |
   | ---------------- | -------------------------------- |
   | ctrl+a           | 跳至行首                         |
   | ctrl+e           | 跳到行尾                         |
   | ctrl+f           | 光标向前移动一个单词             |
   | ctrl+b           | 光标向后移动一个单词             |
   | 键盘左方向键     | 光标向前移动一个字符             |
   | 键盘右方向键     | 光标向后移动一个字符             |
   | 键盘下方向键     | 下翻显示下一个命令               |
   | 键盘上方向键     | 上翻显示上一个命令               |
   | ctrl+h           | 向后删除一个字符                 |
   | ctrl + shift + / | 向后删除一个字符                 |
   | ctrl + u         | 撤销上一个命令，相当于清空当前行 |
   | ctrl + d         | 删除当前光标所在字符             |
   | ctrl + k         | 删除当前光标到行尾所有的字符     |
   | ctrl + i         | 自动补全，相当于敲`tab`          |
   | ctrl + j         | 结束当前行，相当于敲回车         |
   | ctrl + m         | 结束当前行，相当于敲回车         |

   任何时候`tab`键，会根据当前的输入给出提示

   命令后敲`-`或`--` ，然后按`tab`键，可以展示出此命令具体的选项

## 7.8 后台异步命令相关快捷键

* ctrl + c：终止当前命令
* ctrl + z：挂起当前命令，后序可以 bg/fg 重新支持此命令，或kill掉
* ctrl + a：回到行首
* ctrl + e：回到行尾