# 配置文件类型

查看springboot的版本仲裁中心(**2.4.2版本**)，在其pom文件中可以看到默认支持三种配置文件类型。

![image-20210215224329948](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210215224329948.png)



-----



# 配置文件加载顺序

[springboot 配置文件官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)：springboot 有多种设置配置文件的方法，<span style="color:red">本文按照优先级顺序依次讲解，相同属性在多个地方配置,以优先级高的为准,不同配置文件配置的属性互补,下面依次介绍。</span>

## <span style="color:#ffa100">@PropertySource</span>

在springboot的使用了<span style="color:#ffa100">@Configuration</span>的配置上通过注解指定配置文件，只能指定`.properties`的配置文件，无法使用yml，ymal。

```java
package com.example.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.PropertySource;
import java.io.IOException;

@SpringBootApplication
@PropertySource(value = {"app.properties"})
public class DemoApplication {
	public static void main(String[] args) throws IOException {
		SpringApplication.run(DemoApplication.class);
	}
}
```



## SpringApplication.setDefaultProperties

在springboot启动类的main方法中手动指定配置文件,只能指定`.properties`的配置文件，无法使用yml，ymal。

```java
package com.example.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) throws IOException {
		SpringApplication springApplication = new SpringApplication(DemoApplication.class);
		Properties properties = new Properties();
		InputStream is = DemoApplication.class.getClassLoader().getResourceAsStream("app.properties");
		properties.load(is);
		springApplication.setDefaultProperties(properties);
		springApplication.run(args);
	}
}

```



## 默认及外部配置文件

[springboot默认配置文件官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-files)：官网介绍了5个默认路径可存放配置文件，**以下说明的优先级依次增高**,相同属性在多个地方配置,以优先级高的为准,不同配置文件配置的属性互补,下面依次介绍。

>官网提示: It is recommended to stick with one format for your entire application. If you have configuration files with both `.properties` and `.yml` format in the same location, `.properties` takes precedence.
>
>翻译:**若properties和yml文件位于一个路径下，以properties文件配置为准**

### classpath root 

也可以表示成`optional:classpath:/` : 代表项目/resources目录下，编译之后会自动打包到classes目录下，。在springboot快速搭建中，我们使用的是内置的默认配置就是在classpath root下

### classpath /config

也可以表示成`optional:classpath:/config/` ：代表在项目/resources文件夹下新建config文件夹，然后添加properties,yml,yaml文件。

### current directory

也可以表示成:`optional:file:./` ：代表当前项目的根目录，就是pom.xml文件所在目录。需要注意的是：如果当前项目是继承/耦合 关系maven项目的话，**current directory为父maven项目的根目录(与父项目pom.xml文件同级)**,（maven打包成jar之后，不包含此目录的内容，可以在跟jar包同级的文件下新建config文件夹，那么通过java -jar启动时会自动加载）

### current directory /config

也可以表示成:`optional:file:./config/` ：代表当前项目根目录(与pom.xml文件同级)，新建config文件夹,然后可添加properties,yml,yaml文件,跟上面一样,若项目是继承/耦合 关系的maven项目,则是在父项目根目录下(父项目pom.xml文件同级)创建config文件夹。（maven打包成jar之后，不包含此目录的内容，可以在跟jar包同级的文件下新建config文件夹，那么通过java -jar启动时会自动加载）

### /config subdirectory

也可以表示成`optional:file:./config/*/` ：代表当前项目根目录(与pom.xml文件同级)新建的config文件夹下的任何一个配置文件，由于支持通配符，所以可以建立不同用途的文件夹来区分配置文件。（maven打包成jar之后，不包含此目录的内容，可以在跟jar包同级的文件下新建config文件夹，那么通过java -jar启动时会自动加载）。



## spring.config.location

以上都是springboot默认的可存放配置文件的路径，另外我们还可以通过修改`spring.config.location`参数来指定自定义的配置文件存放路径，可以在使用命令行启动时指定，或者在某个配置文件中修改。<span style="color:red">注意:使用这个会改变springboot的默认行为，导致上面5中默认路径下的配置文件不会被读取到，也就不存在互补属性了</span>

### 指定classpath下配置文件夹

`optional:classpath:custom-config/` :代表指定编译好的项目的classpath路径下的文件夹，比如在项目/resource 目录下创建两个环境配置文件:

![image-20210515211055620](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210515211055620.png)

通过`mvn package`打包之后，使用命令行指定不同环境配置文件加载

```shell
#按prod环境配置文件加载启动
java -jar demo-0.0.1-SNAPSHOT.jar --spring.config.location=classpath:/profiels/prod/
#按dev环境配置文件加载启动
java -jar demo-0.0.1-SNAPSHOT.jar --spring.config.location=classpath:/profiels/dev/
```

### 指定任意配置文件夹

`optional:file:./custom-config/` ：表示指定jar包所在电脑上的任意路径下的配置文件夹，比如还是上面图中，我们建立了/demo/config文件夹，虽然maven打包的时候没有把这个文件夹下加载到jar包中，但是还是可以通过指定来加载这里面的配置文件

```shell
#因为config文件夹在jar包所在目录的上级目录，当然也可以通过绝对路径来指定
java -jar demo-0.0.1-SNAPSHOT.jar --spring.config.location=file:../config/dev/
```



---



# Profile配置文件

上面介绍了通过修改`spring.config.location`配置来加载不同配置文件，但是由于会修改springboot默认加载配置文件的行为，导致不同配置文件之间无法互补，所以还有另外一种方式用于实际开发中来区分不同环境的配置文件，就是指定profile，通过这种方式就可以保留springboot默认加载配置文件的属性互补。

[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-files-profile-specific)中介绍了，默认文件名为`application-{profile}`,然后通过`spring.profiles.active`参数来设置有效的profile。<span style="color:#ff0011">注意:此时若不同路径里还有多个配置文件，依然按上面的配置文件路径顺序加载。</span>

![image-20210515215526929](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210515215526929.png)

如图，加载优先级最高的/resources/config/application.properties配置文件，确定端口为8082，激活prod的profile,然后加载application-prod.properties，属性互补。

当然也可以在使用命令行方式指定

```shell
#指定spring.profiles.active设置profile
java -jar demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
#另外一种写法
java -jar demo-0.0.1-SNAPSHOT.jar -Dspring.profiles.active=prod
```

通过这种指定profile的方式，还可以影响使用了@Profile注解的bean是否加载，详细可查看官方文档 [Profile](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-profiles)

## 多profile设置

**yml配置文件通过---来分割不同的profile块:**

```properties
spring:
  profiles:
    active: dev
---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 8989
---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8080
```



---



# 配置文件值注入

按照前面的方式，我们创建好了自己的配置文件，不过有时候可能需要把项目中一些内容也写到配置文件中，然后在bean中引入配置文件中的值(如果想和其他配置区分开，可以使用<span style="color:#ffa100">@PropertySource</span>来单据指定配置文件)

## <span style="color:#ffa100">@Value</span>

前面在学习spring ioc容器中也介绍过，支持SpEL表达式直接注入属性值

```java
@Component
public class User {
    //注入算式
	@Value("${user.id}")
	private Long id;
    //直接注入常量
	@Value("${user.name}")
	private String name;
    //注入配置文件中的属性值
	@Value("${user.age}")
	private Long age;

}
```

properties配置文件中定义SpEL计算表达式: 测试可以正确计算出id=4

```properties
user.id = #{2*2}
user.name = 测试
user.age = 10
```

yml配置文件中定义，测试无法正确计算出id，注入的属性值为null，不会报错。

```yml
user:
  id: #{2*3}
  name: yml
  age: 12
```

通过这种方式注入，需要注意两个问题：

- user.name输出的是环境变量里面的值，所以注意<span style="color:red">属性名不要跟环境变量冲突</span>，避免莫名奇妙的问题。

- 配置文件中的中文注入之后乱码，注意修改IDEA配置,否则打包出来的jar也注入也是乱码的。

![image-20210515234226932](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210515234226932.png)

## <span style="color:#ffa100">@ConfigurationProperties</span>

通过value注解，需要一个一个给bean的属性定义对应的配置文件属性名，太麻烦了，所以引入@ConfigurationProperties批量注入bean的所有属性，此处设置的prefix对应yml文件中的前缀

```java
@Component
@ConfigurationProperties(prefix = "user")
public class User {
	private Long id;
	private String name;
	private Long age;
}
```

配置文件同上，但是需要注意，使用这种方式，<span style="color:red">properties配置文件中无法定义SpEL计算表达式</span>，直接报错了。

以上属性都可以使用松散绑定，可参考 [relaxed bingding](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-relaxed-binding)

value 和 ConfigurationPropertis的对比可参考 [@ConfigurationProperties vs. @Value](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-vs-value)

## [配置文件占位符](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-placeholders-in-properties)

在yml配置文件中可以使用占位符赋值:

```yaml
user:
  username: ${user.friends.20} #配置其他属性的引用
  birthday: 2020/01/02	#自动注入日期数据
  hobbies: [cat,dog,book]	#自动注入List类型数据
  friends: {18: zxcasd, 20: zxcz} #自动注入map类型数据
  age: ${random.int} 
#还有类似 ${random.value}、${random.int}、${random.long}、${random.int(10)}、${random.int[1024,65536]}等随机占位符

```

详细可参考[随机占位符](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-random-values)

## **[@Validated](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-validation)**

**可支持JSR303数据校验,需要导入校验启动器**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```



