[toc]


# 1 sc（Search Class）

## 1.1 作用

查看JVM已加载的类信息，“Search-Class” 的简写，这个命令能搜索出所有已经加载到 JVM 中的 Class 信息，这个命令支持的参数有 `[d]`、`[E]`、`[f]` 和 `[x:]`。

## 1.2 参数说明

| 参数名称              | 参数说明                                                     |
| --------------------- | ------------------------------------------------------------ |
| *class-pattern*       | 类名表达式匹配                                               |
| *method-pattern*      | 方法名表达式匹配                                             |
| [d]                   | 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的ClassLoader等详细信息。 如果一个类被多个ClassLoader所加载，则会出现多次 |
| [E]                   | 开启正则表达式匹配，默认为通配符匹配                         |
| [f]                   | 输出当前类的成员变量信息（需要配合参数-d一起使用）           |
| [x:]                  | 指定输出静态变量时属性的遍历深度，默认为 0，即直接使用 `toString` 输出 |
| `[c:]`                | 指定class的 ClassLoader 的 hashcode                          |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name                   |
| `[n:]`                | 具有详细信息的匹配类的最大数量（默认为100）                  |

* class-pattern支持全限定名，如com.taobao.test.AAA，也支持com/taobao/test/AAA这样的格式，这样，我们从异常堆栈里面把类名拷贝过来的时候，不需要在手动把`/`替换为`.`啦。
* sc 默认开启了子类匹配功能，也就是说所有当前类的子类也会被搜索出来，想要精确的匹配，请打开`options disable-sub-class true`开关

## 1.3 使用参考

1. 打印出包下所有的类名

   ```sh
   [arthas@28896]$ sc demo.*
   demo.MathGame
   Affect(row-cnt:1) cost in 7 ms.
   ```

2. 打印类的详细信息

   ```sh
   [arthas@28896]$ sc -d demo.MathGame 
    class-info        demo.MathGame                                                                                                                                                                                          
    code-source       /home/ligen/.arthas/lib/3.4.6/arthas/arthas-demo.jar                                                                                                                                                   
    name              demo.MathGame                                                                                                                                                                                          
    isInterface       false         #是否接口                                                                                                                                                                                         
    isAnnotation      false         #是否注解
    isEnum            false         #是否枚举                                                                                                                                                                                         
    isAnonymousClass  false                                                                                                                                                                                                  
    isArray           false         #是否数组                                                                                                                                                                                       
    isLocalClass      false                                                                                                                                                                                                  
    isMemberClass     false                                                                                                                                                                                                  
    isPrimitive       false                                                                                                                                                                                                  
    isSynthetic       false                                                                                                                                                                                                  
    simple-name       MathGame       #类名                                                                                                                                                                                        
    modifier          public         #访问控制符                                                                                                                                                                                            
    annotation                                                                                                                                                                                                               
    interfaces                                                                                                                                                                                                               
    super-class       +-java.lang.Object                                                                                                                                                                                     
    class-loader      +-sun.misc.Launcher$AppClassLoader@70dea4e      #类加载器                                                                                                                                                         
                        +-sun.misc.Launcher$ExtClassLoader@7d4991ad                                                                                                                                                          
    classLoaderHash   70dea4e              #类加载器hash                                                                                                                                                                                       
   
   Affect(row-cnt:1) cost in 8 ms.
   
   ```

3. 打印出类的Field信息

   ```sh
   [arthas@28896]$ sc -d -f demo.MathGame 
    class-info        demo.MathGame                                                                                                                                                                                          
    code-source       /home/ligen/.arthas/lib/3.4.6/arthas/arthas-demo.jar                                                                                                                                                   
    name              demo.MathGame                                                                                                                                                                                          
    isInterface       false                                                                                                                                                                                                  
    isAnnotation      false                                                                                                                                                                                                  
    isEnum            false                                                                                                                                                                                                  
    isAnonymousClass  false                                                                                                                                                                                                  
    isArray           false                                                                                                                                                                                                  
    isLocalClass      false                                                                                                                                                                                                  
    isMemberClass     false                                                                                                                                                                                                  
    isPrimitive       false                                                                                                                                                                                                  
    isSynthetic       false                                                                                                                                                                                                  
    simple-name       MathGame                                                                                                                                                                                               
    modifier          public                                                                                                                                                                                                 
    annotation                                                                                                                                                                                                               
    interfaces                                                                                                                                                                                                               
    super-class       +-java.lang.Object                                                                                                                                                                                     
    class-loader      +-sun.misc.Launcher$AppClassLoader@70dea4e                                                                                                                                                             
                        +-sun.misc.Launcher$ExtClassLoader@7d4991ad                                                                                                                                                          
    classLoaderHash   70dea4e                                                                                                                                                                                                
    fields            name     random            #random成员变量信息                                                                                                                                                                            
                      type     java.util.Random                                                                                                                                                                              
                      modifier private,static                                                                                                                                                                                
                      value    java.util.Random@2f971242                                                                                                                                                                     
                                                                                                                                                                                                                             
                      name     illegalArgumentCount    #illegalArgumentCount成员变量信息                                                                                                                                                          
                      type     int                                                                                                                                                                                           
                      modifier private                                                                                                                                                                                       
                                                                                                                                                                                                                             
   
   Affect(row-cnt:1) cost in 8 ms.
   
   ```

   

# 2 sm（Search Method）

## 2.1 作用

> 查看已加载类的方法信息

“Search-Method” 的简写，这个命令能搜索出所有已经加载了 Class 信息的方法信息。

`sm` 命令只能看到由当前类所声明 (declaring) 的方法，父类则无法看到。

## 2.2 参数说明

| 参数名称              | 参数说明                                    |
| --------------------- | ------------------------------------------- |
| *class-pattern*       | 类名表达式匹配                              |
| *method-pattern*      | 方法名表达式匹配                            |
| [d]                   | 展示每个方法的详细信息                      |
| [E]                   | 开启正则表达式匹配，默认为通配符匹配        |
| `[c:]`                | 指定class的 ClassLoader 的 hashcode         |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name  |
| `[n:]`                | 具有详细信息的匹配类的最大数量（默认为100） |

## 2.3 使用参考

1. 查看类中的所有方法名

   ```sh
   [arthas@28896]$ sm demo.MathGame
   demo.MathGame <init>()V
   demo.MathGame primeFactors(I)Ljava/util/List;
   demo.MathGame main([Ljava/lang/String;)V
   demo.MathGame run()V
   demo.MathGame print(ILjava/util/List;)V
   Affect(row-cnt:5) cost in 9 ms.
   ```

2. 查看所有方法的详细信息

   ```sh
   [arthas@28896]$ sm -d demo.MathGame
    declaring-class   demo.MathGame
    constructor-name  <init>
    modifier          public
    annotation              
    parameters              
    exceptions              
    classLoaderHash   70dea4e
    declaring-class  demo.MathGame
    method-name      primeFactors 
    modifier         public            
    annotation             
    parameters       int   
    return           java.util.List
    exceptions                     
    classLoaderHash  70dea4e      
    declaring-class  demo.MathGame
    method-name      main                               #方法名  
    modifier         public,static                      #访问修饰符
    annotation                                          #注解
    parameters       java.lang.String[]                 #入参
    return           void                               #返回值类型
    exceptions       java.lang.InterruptedException     #是否抛出异常
    classLoaderHash  70dea4e                            #类加载器hash
    declaring-class  demo.MathGame
    method-name      run          
    modifier         public       
    annotation                    
    parameters                    
    return           void         
    exceptions       java.lang.InterruptedException
    classLoaderHash  70dea4e
    declaring-class  demo.MathGame
    method-name      print
    modifier         public,static
    annotation                    
    parameters       int          
                     java.util.Lis
                     t
    return           void
    exceptions
    classLoaderHash  70dea4e
   Affect(row-cnt:5) cost in 9 ms.
   
   ```

   

# 3 jad 

## 3.1 作用

反编译指定已加载类的源码

`jad` 命令将 JVM 中实际运行的 class 的 byte code 反编译成 java 代码，便于你理解业务逻辑；

- 在 Arthas Console 上，反编译出来的源码是带语法高亮的，阅读更方便
- 当然，反编译出来的 java 代码可能会存在语法错误，但不影响你进行阅读理解

## 3.2 参数说明

| 参数名称              | 参数说明                                   |
| --------------------- | ------------------------------------------ |
| *class-pattern*       | 类名表达式匹配                             |
| `[c:]`                | 类所属 ClassLoader 的 hashcode             |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |
| [E]                   | 开启正则表达式匹配，默认为通配符匹配       |

## 3.3 使用参考

1. 编译java.lang.String

   ```java
   [arthas@23916]$ jad java.lang.String
   
   ClassLoader:                                                                                                                                                                                                              
   
   Location:                                                                                                                                                                                                                 
                                                                                                                                                                                                                             
   
   /*
    * Decompiled with CFR.
    */
   package java.lang;
   
   import java.io.ObjectStreamField;
   import java.io.Serializable;
   import java.io.UnsupportedEncodingException;
   import java.nio.charset.Charset;
   import java.util.ArrayList;
   import java.util.Arrays;
   import java.util.Comparator;
   import java.util.Formatter;
   import java.util.Locale;
   import java.util.Objects;
   import java.util.StringJoiner;
   import java.util.regex.Matcher;
   import java.util.regex.Pattern;
   
   public final class String
   implements Serializable,
   Comparable<String>,
   CharSequence {
       private final char[] value;
       private int hash;
       private static final long serialVersionUID = -6849794470754667710L;
       private static final ObjectStreamField[] serialPersistentFields = new ObjectStreamField[0];
       public static final Comparator<String> CASE_INSENSITIVE_ORDER = new CaseInsensitiveComparator();
   
       public String(byte[] arrby, int n, int n2) {
           String.checkBounds(arrby, n, n2);
           this.value = StringCoding.decode(arrby, n, n2);
       }
   
     ...
   }
   
   ```
   
2. 编译时仅展示源码

   默认情况下，反编译结果里会带有`ClassLoader`信息，通过`--source-only`选项，可以只打印源代码。方便和[mc](https://arthas.gitee.io/mc.html)/[retransform](https://arthas.gitee.io/retransform.html)命令结合使用。

   ```java
   [arthas@23916]$ jad --source-only demo.MathGame
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
   ......
   }
   ```
   
3. 反编译指定的函数

   ```java
   [arthas@23916]$ jad demo.MathGame primeFactors 
   
   ClassLoader: 
   +-sun.misc.Launcher$AppClassLoader@70dea4e
     +-sun.misc.Launcher$ExtClassLoader@7d4991ad
   Location: 
   /home/ligen/.arthas/lib/3.4.6/arthas/arthas-demo.jar
   
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
   
   Affect(row-cnt:1) cost in 179 ms.
   ```

4. 反编译时指定ClassLoader

   当有多个 `ClassLoader` 都加载了这个类时，`jad` 命令会输出对应 `ClassLoader` 实例的 `hashcode`，然后你只需要重新执行 `jad` 命令，并使用参数 `-c <hashcode>` 就可以反编译指定 ClassLoader 加载的那个类了；

   ```sh
   $ jad org.apache.log4j.Logger
    
   Found more than one class for: org.apache.log4j.Logger, Please use jad -c hashcode org.apache.log4j.Logger
   HASHCODE  CLASSLOADER
   69dcaba4  +-monitor's ModuleClassLoader
   6e51ad67  +-java.net.URLClassLoader@6e51ad67
               +-sun.misc.Launcher$AppClassLoader@6951a712
               +-sun.misc.Launcher$ExtClassLoader@6fafc4c2
   2bdd9114  +-pandora-qos-service's ModuleClassLoader
   4c0df5f8  +-pandora-framework's ModuleClassLoader
    
   Affect(row-cnt:0) cost in 38 ms.
   $ jad org.apache.log4j.Logger -c 69dcaba4
    
   ClassLoader:
   +-monitor's ModuleClassLoader
    
   Location:
   /Users/admin/app/log4j-1.2.14.jar
    
   package org.apache.log4j;
   
   import org.apache.log4j.spi.*;
   
   public class Logger extends Category
   {
       private static final String FQCN;
    
       protected Logger(String name)
       {
           super(name);
       }
    
   ...
    
   Affect(row-cnt:1) cost in 190 ms.
   ```

   对于只有唯一实例的ClassLoader还可以通过`--classLoaderClass`指定class name，使用起来更加方便：

   `--classLoaderClass` 的值是ClassLoader的类名，只有匹配到唯一的ClassLoader实例时才能工作，目的是方便输入通用命令，而`-c <hashcode>`是动态变化的。

# 4 mc  （在内存中把源代码编译成字节码文件）

## 4.1 作用

Memory Compiler/内存编译器，编译`.java`文件生成`.class`。

## 4.2 使用参考

1. 编译Test.java文件，不指定目录的情况下，默认编译至arthas启动目录下

   ```sh
   [arthas@23916]$ mc /home/ligen/Test.java 
   Memory compiler output:
   /home/ligen/.arthas/lib/3.4.6/arthas/demo/Test.class
   Affect(row-cnt:1) cost in 163 ms.
   ```

2. 通过`-d`命令指定输出目录

   ```sh
   [arthas@2803]$ mc /home/ligen/Test.java -d /home/ligen
   Memory compiler output:
/home/ligen/demo/Test.class
   Affect(row-cnt:1) cost in 480 ms.
   ```
   
3. 通过`-c`参数指定classloader：

   ```sh
   [arthas@9188]$ mc -c 70dea4e /home/ligen/Test.java
   Memory compiler output:
/home/ligen/.arthas/lib/3.4.6/arthas/demo/Test.class
   Affect(row-cnt:1) cost in 34 ms.
   ```
   
   也可以通过`--classLoaderClass`参数指定ClassLoader：
   
   ```sh
   [arthas@9188]$ mc --classLoaderClass sun.misc.Launcher$AppClassLoader /home/ligen/Test.java -d /home/ligen/tMemory compiler output:
   /home/ligen/temp/demo/Test.class
   Affect(row-cnt:1) cost in 410 ms.
   ```
   
   

# 5 redefine

## 5.1 作用

加载外部的`.class`文件，重定义 jvm已加载的类。推荐使用 [retransform](https://arthas.gitee.io/retransform.html) 命令

## 5.2 常见问题

- redefine的class不能修改、添加、删除类的field和method，包括方法参数、方法名称及返回值
- 如果mc失败，可以在本地开发环境编译好class文件，上传到目标系统，使用redefine热加载class
- 目前redefine 和`watch/trace/jad/tt`等命令冲突，以后重新实现redefine功能会解决此问题

> 注意， redefine后的原来的类不能恢复，redefine有可能失败（比如增加了新的field），参考jdk本身的文档。

> `reset`命令对`redefine`的类无效。如果想重置，需要`redefine`原始的字节码。

> `redefine`命令和`jad`/`watch`/`trace`/`monitor`/`tt`等命令会冲突。执行完`redefine`之后，如果再执行上面提到的命令，则会把`redefine`的字节码重置。 原因是jdk本身redefine和Retransform是不同的机制，同时使用两种机制来更新字节码，只有最后修改的会生效。

## 5.3 参数说明

| 参数名称              | 参数说明                                   |
| --------------------- | ------------------------------------------ |
| [c:]                  | ClassLoader的hashcode                      |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |

## 5.4 使用案例-结合jad和mc

1. 反编译MathGame.class并输出到MathGame.java文件中

   ```sh
   [arthas@22297]$ jad --source-only demo.MathGame >> /home/ligen/MathGame.java
   ```

2. 修改MathGame.java文件

   ```java
   //在四个方法中分别加入如下输出信息
   System.out.println("prime中日志信息添加成功了。。。");
   
   System.out.println("main中日志信息添加成功了。。。");        
   
   System.out.println("run中日志信息添加成功了。。。");
   
   System.out.println("print中日志信息添加成功了。。。");   
   ```

3. 使用mc命令对java文件进行编译

   ```sh
   [arthas@22297]$ mc /home/ligen/MathGame.java -d /home/ligen/ 
   Memory compiler output:
   /home/ligen/demo/MathGame.class
   Affect(row-cnt:1) cost in 440 ms.
   ```

4. 使用redefine命令重新加载MathGame.class文件

   ```sh
   [arthas@22297]$ redefine /home/ligen/demo/MathGame.class 
   redefine success, size: 1, classes:
   demo.MathGame
   ```

   输出结果:

   ![image-20210309105341588](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210309105341588.png)

## 5.5 redefine的限制

- 不允许新增加field/method，只能基于已有的方法和成员变量
- 正在跑的函数，没有退出不能生效，比如下面新增加的`System.out.println`，只有`main()`函数里的不会生效

# 6 retransform

## 6.1 作用

加载外部的`.class`文件，retransform jvm已加载的类。

功能与redefine类似，但是解决了与`watch/trace/jad/tt`的问题，在每次触发retransform的时候会生成一个retransform entry.

> 如果多次执行 retransform 加载同一个 class 文件，则会有多条 retransform entry.

## 6.2 使用参考

1. retransform指定的.class文件(使用redefine中的文件)

   ```sh
   [arthas@22297]$ retransform /home/ligen/demo/MathGame.class 
   retransform success, size: 1, classes:
   demo.MathGame
   ```

   此时使用jad可以看到重新加载的class文件

   ​	![image-20210309112007277](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210309112007277.png)

2. 查看retransform entry

   ```sh
   [arthas@22297]$ retransform -l
   Id              ClassName       TransformCount  LoaderHash      LoaderClassName 
   1               demo.MathGame   2               null            null    
   ```

   - TransformCount 统计在 ClassFileTransformer#transform 函数里尝试返回 entry对应的 .class文件的次数，但并不表明transform一定成功。

3. 删除指定的entry

   需要指定id

   ```sh
   [arthas@22297]$ retransform -d 1
   ```

4. 显式触发retransform

   在删除entry之后，触发retransform才会生效

   ```sh
   [arthas@22297]$ retransform --classPattern demo.MathGame
   retransform success, size: 1, classes:
   demo.MathGame
   ```

   >  注意：对于同一个类，当存在多个 retransform entry时，如果显式触发 retransform ，则最后添加的entry生效(id最大的)。

5. 消除 retransform 的影响

   如果对某个类执行 retransform 之后，想消除影响，则需要：

   - 删除这个类对应的 retransform entry
   - 重新触发 retransform

   > 如果不清除掉所有的 retransform entry，并重新触发 retransform ，则arthas stop时，retransform过的类仍然生效。

# 7 dump

## 7.1 作用

> dump 已加载类的 bytecode 到特定目录，默认路径为`logs/arthas/classdump`下

## 7.2 参数说明

| 参数名称              | 参数说明                                   |
| --------------------- | ------------------------------------------ |
| *class-pattern*       | 类名表达式匹配                             |
| `[c:]`                | 类所属 ClassLoader 的 hashcode             |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |
| `[d:]`                | 设置类文件的目标目录                       |
| [E]                   | 开启正则表达式匹配，默认为通配符匹配       |

## 7.3 使用参考

1. 提取`java.lang.String的class文件

   ```sh
   [arthas@22297]$ dump java.lang.String
    HASHCODE  CLASSLOADER  LOCATION                                                                                                                                                                                          
    null                   /home/ligen/logs/arthas/classdump/java/lang/String.class                                                                                                                                    
   Affect(row-cnt:1) cost in 55 ms.
   ```

2. 提取demo包下所有的class文件并指定输出目录

   ```sh
   [arthas@22297]$ dump demo.* -d /home/ligen
   ```

   

# 8 classloader

## 8.1 作用

> 查看classloader的继承树，urls，类加载信息

`classloader` 命令将 JVM 中所有的classloader的信息统计出来，并可以展示继承树，urls等。

可以让指定的classloader去getResources，打印出所有查找到的resources的url。对于`ResourceNotFoundException`比较有用。

## 8.2 参数说明

| 参数名称              | 参数说明                                   |
| --------------------- | ------------------------------------------ |
| [l]                   | 按类加载实例进行统计                       |
| [t]                   | 打印所有ClassLoader的继承树                |
| [a]                   | 列出所有ClassLoader加载的类，请谨慎使用    |
| `[c:]`                | ClassLoader的hashcode                      |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |
| `[c: r:]`             | 用ClassLoader去查找resource                |
| `[c: load:]`          | 用ClassLoader去加载指定的类                |

## 8.3 使用参考

1. 按类加载类型查看统计信息

   ```sh
   [arthas@3680]$ classloader
   ```

   ​	![image-20210309141703816](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210309141703816.png)

2. 按类加载实例查看统计信息

   ```sh
   [arthas@3680]$ classloader  -l
   ```

   ​	![image-20210309141901178](https://gitee.com/frank1246/typora-image-table/raw/master/uPic/image-20210309141901178.png)

3. 查看ClassLoader的继承树

   ```sh
   [arthas@3680]$ classloader  -t
   +-BootstrapClassLoader             
   +-sun.misc.Launcher$ExtClassLoader@7d4991ad  
     +-com.taobao.arthas.agent.ArthasClassloader@1d4dd539
     +-sun.misc.Launcher$AppClassLoader@70dea4e
   Affect(row-cnt:4) cost in 3 ms.
   
   ```

4. 查看URLClassLoader实际的urls

   ```sh
   [arthas@3680]$ classloader  -c 1d4dd539
   file:/home/ligen/.arthas/lib/3.4.6/arthas/arthas-core.jar             
   Affect(row-cnt:2) cost in 0 ms.
   ```
   
5. 使用ClassLoader去查找resource

   ```sh
   [arthas@3680]$ classloader  -c 1d4dd539 -r META-INF/MANIFEST.MF
    jar:file:/home/ligen/.arthas/lib/3.4.6/arthas/arthas-core.jar!/META-INF/MANIFEST.MF
   Affect(row-cnt:1) cost in 1 ms.
   ```

   查找加载的`java.lang.String.class`

   ```sh
   [arthas@3680]$ classloader  -c 1d4dd539 -r java/lang/String.class
    jar:file:/usr/java/jdk1.8.0_271-amd64/jre/lib/rt.jar!/java/lang/String.class
   Affect(row-cnt:1) cost in 1 ms.
   ```

6. 使用ClassLoader去查看类的详细信息

   ```sh
   [arthas@3680]$ classloader  -c 70dea4e --load demo.MathGame
   load class success.
   class-info        demo.MathGame
    code-source       /home/ligen/.arthas/lib/3.4.6/arthas/arthas-demo.jar  
    name              demo.MathGame
    isInterface       false 
    isAnnotation      false 
    isEnum            false 
    isAnonymousClass  false 
    isArray           false 
    isLocalClass      false 
    isMemberClass     false 
    isPrimitive       false 
    isSynthetic       false 
    simple-name       MathGame
    modifier          public
    annotation
    interfaces
    super-class       +-java.lang.Object
    class-loader      +-sun.misc.Launcher$AppClassLoader@70dea4e
                        +-sun.misc.Launcher$ExtClassLoader@7d4991ad
    classLoaderHash   70dea4e
   ```

   
