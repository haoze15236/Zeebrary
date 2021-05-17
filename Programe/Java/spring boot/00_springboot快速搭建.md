# spring boot从零搭建

为了更好的理解springboot,下面我们从一个空的maven项目，一步步加入springboot内容，来构建一个最基础的springboot,只需要简单的几步，开始吧！

- [pom依赖](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-first-application-pom)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--设置springboot为父项目-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.5</version>
    </parent>
    <dependencies>
        <!--添加springboot web项目依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--导入配置文件处理器，属性配置文件进行绑定就会有提示-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        <!--spring-boot 热部署-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

- 添加启动类,<span style="color:red">配置类放在跟controller文件夹一级，因为默认是扫描配置类当前路径下的所有包。</span>

```java
package com.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class);
	}
}

```

- 编写controller,注意文件路径哦

```java
package com.springboot.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloSpringBootController {

	@RequestMapping("/helloSpringBoot")
	public String helloSpringBoot(){
		System.out.println("hello");
		return "hello spring boot!";
	}
}

```

从启动类main方法开始执行，即可启动项目，通过浏览器已经能正常访问啦！当然也可以参考官方文档操作，通过`mvn spring-boot:run`来启动项目。

[官方说明](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-first-application-code)

以上是在idea里面启动项目，而实际上，我们是开发好项目直接通过jar包部署，或者不使用springboot自带的tomcat,打成war包，放在tomcat下部署。

## [jar包发布](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-first-application-executable-jar)

- pom文件中引入maven插件

  ```xml
  <build>
      <!--设置项目打包之后的名称-->
      <finalName>helloSpringBoot</finalName>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
  </build>
  ```

- 在命令行中使用`mvn package` 命令打包项目成jar包

- 使用`java -jar`命令启动项目

```shell
java -jar springboot-1.0-SNAPSHOT.jar
```

## war包发布

- pom文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.4.2</version>
          <relativePath/> <!-- lookup parent from repository -->
      </parent>
      <groupId>com.haoze</groupId>
      <artifactId>hello-spring-boot</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <name>hello-spring-boot</name>
      <!--修改打包方式为war-->
      <packaging>war</packaging>
      <description>Demo project for Spring Boot</description>
      <properties>
          <java.version>1.8</java.version>
      </properties>
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-tomcat</artifactId>
              <!--修改tomcat启动依赖的范围,即打包时排除此依赖-->
              <scope>provided</scope>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
      </dependencies>
  
      <build>
          <!--设置项目打包之后的名称-->
          <finalName>helloSpringBoot</finalName>
          <plugins>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
              </plugin>
          </plugins>
      </build>
  </project>
  ```

- 修改启动配置类

  ```java
  package com.haoze.hellospringboot;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.boot.builder.SpringApplicationBuilder;
  import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
  
  @SpringBootApplication
  public class HelloSpringBootApplication extends SpringBootServletInitializer {
  
      public static void main(String[] args) {
          SpringApplication.run(HelloSpringBootApplication.class, args);
      }
  	/**
  	*打war包需要启动类继承SpringBootServletInitializer 并修改重写configure方法
  	*/
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
          return builder.sources(HelloSpringBootApplication.class);
      }
  
  }
  
  ```

将打好的war包放入tomcat下启动tomcat即可,不过这种方式一般不使用，了解即可。

# @SpringBootApplication

前面我们通过简单的几行代码，加上pom文件的配置，就搭建好了一个web项目的框架，那么springboot是如何实现的呢？首先我们来看spring的启动配置类，从一切发生的源头**<span style="color:#ffa100">@SpringBootApplication</span>**注解开始。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	...
}
```

- **<span style="color:#ffa100">@SpringBootConfiguration</span>** :声明配置类。这个注解点进源码可以看到，它被<span style="color:#ffa100">@Configuration</span>修饰，前面spring文件中有介绍javaconfig方式配置IOC容器，就是用这个注解来声明配置类，其实是一个作用
- **<span style="color:#ffa100">@EnableAutoConfiguration</span>** ：根据依赖jar来自动配置你的项目，[官方简介](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-first-application-code)
- **<span style="color:#ffa100">@ComponentScan</span>** : 包扫描配置，默认扫描当前配置类路径下的所有包，所以前面springboot快速搭建中要注意和controller文件夹放在一层路径里。
  - TypeExcludeFilter.class :	通过继承这个类，可以自定义排除方式
  - AutoConfigurationExcludeFilter.class :不扫描配置类

# pom文件简单说明

## springboot版本仲裁中心

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.5</version>
</parent>
```

点进引入的父maven项目pom文件，可以发现它有引入一个父maven项目,这个是用来管理所有的第三方依赖，俗称**springboot版本仲裁中心**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.4.5</version>
</parent>
```

## [场景启动器](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter)

而我们为了启动web项目，我们引入了这个依赖，然后所有需要的jar包就自动导入了项目，这个俗称**场景启动器**,可查看官网查看所有支持的场景启动器，不同场景启动器维护了不同场景下所有需要的依赖。

```xml
<dependencies>
    <!--添加springboot web项目依赖,使用spring mvc 构建web(包含restful)应用程序
		使用tomcat为默认容器-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

## springboot部署插件

前面介绍了，只有使用这个插件，才可以打包项目成可直接启动的jar

```xml
<build>
    <plugins>
        <!--添加maven打包插件-->
        <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

# [SpringApplication](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-application)

上面启动springboot是使用SpringApplication.rum()方法，当我们需要对springboot做一些全局配置的时候，可以按照官方文档，自定义SpringApplication。

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    //关闭启动横幅
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

