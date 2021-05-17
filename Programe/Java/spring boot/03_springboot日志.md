# springboot日志体系

在[javeSE日志体系](../javaSE/日志体系)中我们对各种日志门面和具体日志实现有了了解，那么在springboot中，他的日志体系是怎么样的呢？

![image-20210517233433703](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210517233433703.png)

可以看到，它使用了**slf4j**日志门面,通过 **logback-classic**适配器来支持具体的日志实现 **logback**，通过 **log4j-to-slf4j**适配器来支持具体的日志实现 **log4j-api**同时，它通过 **jul-to-slf4j** 桥接器来支持jul日志门面。

# [springboot日志](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging)

[springboot可配置日志属性](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties-testing)

## [日志级别](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-custom-log-levels)

可以设置 TRACE < DEBUG < INFO < WARN < ERROR < FATAL,或者设置为OFF关闭日志。在实际代码中记录日志使用方式如下:

```java
package com.example.springbootlog;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootLogApplication {
	private static Logger logger = LoggerFactory.getLogger(SpringbootLogApplication.class);
	public static void main(String[] args) {
		SpringApplication.run(SpringbootLogApplication.class, args);
		logger.trace("跟踪");
		logger.debug("调试");
		logger.info("信息");
		logger.warn("警告");
		logger.error("错误");
	}

}

```

在属性文件中设置日志级别：springboot默认就是info级别

```properties
logging.level.root = info
#单独为com.example.springbootlog包下设置日志级别为trace
logging.level.com.example.springbootlog = trace
```

## [日志格式](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging-format)

| 属性                       | 默认值                                                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| logging.pattern.console    | `%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}` | Appender模式，用于输出到控制台。仅在默认的Logback设置中受支持。 |
| logging.pattern.dateformat | `yyyy-MM-dd HH:mm:ss.SSS`                                    | 记录日期格式的附加模式。仅在默认的Logback设置中受支持。      |
| logging.pattern.file       | `%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}` | 用于输出到文件的附加模式。仅在默认的Logback设置中受支        |

在配置文件中通过属性`logging.pattern.console`来设置控制台日志格式,默认值为：

```properties
logging.pattern.console = %clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}
```

- **%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint}**  : 按无颜色格式输出时间 。

  - **%clr(){faint}** ：表示设置当前信息的输出颜色，可参考 [颜色输出设置](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging-format)

  - **%d{}** : 是logback的用法，表示按格式输出日期，有兴趣的可以学习[logback中文文档-patternlayout](https://logbackcn.gitbook.io/logback/06-di-liu-zhang-layouts#patternlayout)
  - **${value1:value2}** ：spring占位符null表达式，当value1为null则取value2
  - **LOG_DATEFORMAT_PATTERN** ：表示系统环境变量，springboot提供了把属性文件中的值转移到环境变量中([支持属性](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-custom-log-configuration))

- **%clr(${LOG_LEVEL_PATTERN:-%5p})** ：按spring默认日志级别颜色输出日志等级。
  - **%5p** ：是logback里[格式修改器](https://logbackcn.gitbook.io/logback/06-di-liu-zhang-layouts#ge-shi-xiu-gai-qi)的用法， p表示输出日志级别，**%5**表示按最少5位输出，不够5位左边补空格 ，超过5位正常输出。

- **%clr([%15.15t]){faint}** :按无颜色格式输出线程名。
  - **%15.15t** : t表示输出线程名(logback用法)，**%15.15** 表示按最少15位输出，不够15位左边补空格，超过15位则截取最后15位。

- **%clr(%-40.40logger{39}){cyan}** ：按cyan颜色输出日志记录器名字
  - **%-40.40logger{39}** :logger{39}表示输出日志记录器类的名字，全限定类名如果超过39位，则包名全部简写成1位，类名全部输出。然后按照**%-40.40**格式化。

- **%m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}** :
  - **%m** : 表示日志信息 ，(logback用法)
  - **%n** : 表示换行符 (logback用法)
  - **${LOG_EXCEPTION_CONVERSION_WORD:-}** : LOG_EXCEPTION_CONVERSION_WORD是异常转换字,如果为空则用-
  - **%wEx** ：表示输出异常堆栈

## [日志文件输出](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging-file-output)

前面的日志输出默认都是输出到控制台,如果我们想把日志输出到文件中，可以在配置文件中配置`logging.file.name`或者`logging.file.path`来指定。

```properties
#默认输出到当前目录生成out.log日志文件，也可以指定绝对路径
logging.file.name=out.log
#输出到当前目录,也可以指定绝对路径，默认生成spring.log日志文件
logging.file.path=./
```

### [日志归档](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging-file-rotation)

日志过多之后，可以进行日志归档，以下是springboot默认配置

```properties
#是否启用日志归档
logging.logback.rollingpolicy.clean-history-on-start = true
#归档日志名称
logging.logback.rollingpolicy.file-name-pattern = ${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz
#归档日志阈值,达到10MB就归档
logging.logback.rollingpolicy.max-file-size = 10MB
#归档日志保留时间 7天
logging.logback.rollingpolicy.max-history = 7
#归档日志总大小 0表示不限制
logging.logback.rollingpolicy.total-size-cap = 0B
```

- **${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz** 
  - **${LOG_FILE}** ：日志文件名，可以通过logging.file.name设置
  - **%d{yyyy-MM-dd}** :获取当前时间拼在归档日志文件名中
  - **%i** :归档日志序列号，可能一天会生成多个归档日志文件，依次递增区分
  - **gz** ：归档日志后缀，gz表示压缩包。

## [logback拓展支持](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logback-extensions)

spring默认的日志行为只包含了logback最基础的功能，所以springboot也提供了使用logback高级功能的拓展方式，这部分内容需要先学习logback的使用。

# 切换日志实现

## logback - log4j2

1. 将logback的场景启动器排除（slf4j只能运行有1个桥接器）
2. 添加log4j2的场景启动器
3. 添加log4j2的配置文件

```xml
<dependencies>
    <dependency>
        <!--starter-web里面自动添加starter-logging 也就是logback的依赖-->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <!--排除starter-logging 也就是logback的依赖-->
            <exclusion>
                <artifactId>spring-boot-starter-logging</artifactId>
                <groupId>org.springframework.boot</groupId>
            </exclusion>
        </exclusions>
    </dependency>

    <!--Log4j2的场景启动器-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-log4j2</artifactId>
    </dependency>
</dependencies>
```

## logback - log4j 

1. 要将logback的桥接器排除

```xml
<dependency>
<!--starter-web里面自动添加starter-logging 也就是logback的依赖-->
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
<exclusions>
<exclusion>
<artifactId>logback-classic</artifactId>
<groupId>ch.qos.logback</groupId>
</exclusion>
</exclusions>
</dependency>
```

2. 添加log4j的桥接器

```xml
<dependency>
<groupId>org.slf4j</groupId>
<artifactId>slf4j-log4j12</artifactId>
</dependency>
```

3. 添加log4j的配置文件

```properties
#trace<debug<info<warn<error<fatal
log4j.rootLogger=trace, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
```

