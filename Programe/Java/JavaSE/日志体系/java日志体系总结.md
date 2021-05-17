# Java常用日志框架历史



> - **<span style="color:#ff8cef">Log4j</span>：**1996年早期，欧洲安全电子市场项目组决定编写它自己的程序跟踪API(Tracing API)。经过不断的完善，这个API终于成为一个十分受欢迎的Java日志软件包，即Log4j。后来Log4j成为Apache基金会项目中的一员。期间Log4j近乎成了Java社区的日志标准。据说Apache基金会还曾经建议sun引入Log4j到java的标准库中，但Sun拒绝了。
> - **<span style="color:#ff8cef">JUL</span>：**2002年Java1.4发布，JDK内置了日志库JUL(**Java.Util.Logging**),其实现基本模仿了Log4j的实现。在JUL出来以前，log4j就已经成为一项成熟的技术，使得log4j在选择上占据了一定的优势。
> - **<span style="color:#118cef">JCL</span>：**Apache推出了**Commons Logging** 也就是**JCL**，JCL只是定义了一套日志接口(其内部也提供一个Simple Log的简单实现)，支持运行时动态加载日志组件的实现，也就是说，在你应用代码里，只需调用Commons Logging的接口，底层实现可以是log4j，也可以是Java.Util.Logging。
> - **<span style="color:#118cef">slf4j</span>：**2006年Ceki Gülcü不适应Apache的工作方式，离开了Apache。然后先后创建了slf4j(日志门面接口，类似于JCL)和**<span style="color:#ff8cef">Logback</span>**(Slf4j的实现)两个项目，并回瑞典创建了QOS公司，QOS官网上是这样描述Logback的：The Generic，Reliable Fast&Flexible Logging Framework(一个通用，可靠，快速且灵活的日志框架)。
> - **<span style="color:#ff8cef">Log4j2</span>：**Apache于2012-07重写了log4j 1.x，成立了新的项目Log4j2，Log4j2具有logback的所有特性。

## 直接使用日志实现

### JUL

 JDK自带日志实现，使用起来很方便，无需引入任何外部jar

```java
package com.example.springbootlog.logInterface;
import java.util.logging.Logger;
public class Jul {
	public static void main(String[] args) {
		Logger logger = Logger.getLogger(Jcl.class.getName());
		logger.info("测试jul日志");
	}
}
```

### log4j

开源日志实现，现在已经停止更新，此处只做了解

1. 引入pom依赖

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.12</version>
</dependency>
```

2. 配置日志属性

```properties
### 设置日志级别 trace<debug<info<warn<error<fatal
log4j.rootLogger = debug,stdout
### 输出信息到控制抬 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

```

3. 使用日志

```java
package com.example.springbootlog.logInterface;
import org.apache.log4j.Logger;
public class log4j {
	public static void main(String[] args) {
		Logger logger =Logger.getLogger(Slf4j.class);
		logger.debug("测试log4j日志");
	}
}
```

# JCL日志门面

上面介绍了直接使用日志实现的方式，虽然使用方便，但是不便于后期日志功能的扩展，所以规范上，一般都是定义日志门面，来调用具体日志实现，首先介绍JCL日志门面。

1. 引入pom依赖.

```xml
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.1.3</version>
</dependency>

```

2. 使用

```java
package com.example.springbootlog.logInterface;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
public class Jcl {
	public static void main(String[] args) {
		Log log = LogFactory.getLog(Jcl.class);
		System.out.println(log.getClass());
		log.debug("测试jcl日志");
	}
}
```

需要注意的是：JCL动态查找机制进行日志实例化，执行顺序为：

>commons­-logging.properties ­­­­> 系统环境变量 ­­­­­­­> log4j ­­­> jul­­­ > simplelog ­­­­> nooplog

因此，我们可以使用commons­-logging.properties来修改jcl的某些行为：

```properties
#使用jdk自带日志实现
org.apache.commons.logging.Log = org.apache.commons.logging.impl.Jdk14Logger
```

# slf4j日志门面

1.引入pom依赖

```java
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.30</version>
</dependency>
```

2.使用**桥接器**确定具体实现，此处就用上面引入的log4j日志实现。

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.30</version>
</dependency>
```

否则会报错没有日志实现。

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

3.使用sfl4j日志门面

```java
package com.example.springbootlog.logInterface;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class Slf4j {
	public static void main(String[] args) {
		Logger logger = LoggerFactory.getLogger(Slf4j.class);
		System.out.println(logger.getClass());
		logger.debug("测试slf4j日志");
	}
}
```

## 桥接器

从上面的例子可以看到使用sfl4j,必须使用桥接器来链接具体的日志实现，不同日志实现桥接器说明如下:

![image-20210517221158555](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210517221158555.png)

## [适配器](http://www.slf4j.org/legacy.html)

当我们使用sfl4j时，如果想不改动使用JCL日志的代码，但是又想统一日志格式输出的时候，就可以通过sfl4j提供的适配器来把jcl日志适配到sfl4j上。

- 引入pom依赖

```java
<groupId>org.slf4j</groupId>
<artifactId>slf4j-parent</artifactId>
<version>1.7.30</version>
```

