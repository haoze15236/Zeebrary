# Spring IOC

## 容器

ApplicationContext是Spring IoC容器(The Spring Container)实现的代表，**它负责实例化，配置和组装Bean**。容器通过<span style="color:#ff11f5">读取配置元数据(Configuration Metadata)</span>获取有关实例化、配置和组装哪些对象(Your Business Objects)的说明 。配置元数据可以使用XML、Java注解或Java代码来呈现。它允许你处理应用程序的对象与其他对象之间的互相依赖关系。如图:

![container magic](https://docs.spring.io/spring-framework/docs/current/reference/html/images/container-magic.png)

### 初始化

- xml方式初始化IOC容器

首先在Resource目录下创建配置xml文件：`spring-context.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--在ioc.xml.dto包下创建User类-->
    <bean name="user" class="ioc.xml.dto.User"/>
</beans>
```

这里我们使用前面配置的junit来做单元测试,尝试加载XML配置初始化spring IOC容器，并获取User类的实例

```java
package ioc.xml;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class HelloSpring {
	private ClassPathXmlApplicationContext xmlApplicationContext;
	@Before
	public void initContext(){
        //加载XML配置文件
		xmlApplicationContext = new ClassPathXmlApplicationContext("spring-context.xml");
	}
	@Test
	public void testIocGetBean(){
        //通过name获取Bean,我们在User的无参构造方法中输出一句话,此处就可以看到在控制台的输出
		xmlApplicationContext.getBean("user");
	}
}
```

- javaconfig方式初始化IOC容器

首先创建配置类

```java
package ioc.javaconfig;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.context.annotation.PropertySource;

@Configuration
@ComponentScan(basePackages = {"ioc.**"})
@PropertySource("classpath:setting.properties")
@EnableAspectJAutoProxy
public class BeanConfig {

}

```

初始化容器并尝试获取User实例:

```java
package springframework.javaConfig;

import com.alibaba.druid.pool.DruidDataSource;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class TestJavaConfig {
    private  AnnotationConfigApplicationContext ioc;
    @Before
    public void before(){
        //初始化javaconfig配置类来创建IOC容器
        ioc = new AnnotationConfigApplicationContext(AppConfig.class);
    }
    @Test
    public void testJavaConfig(){
        DruidDataSource dataSource = ioc.getBean("dataSource",DruidDataSource.class);
        System.out.println(dataSource);
        dataSource = ioc.getBean("dataSource",DruidDataSource.class);
        System.out.println(dataSource);
    }

    @Test
    public void testPeople(){
        People people = ioc.getBean(People.class);
        System.out.println(people);
        people = ioc.getBean(People.class);
        System.out.println(people);
    }
}

```

获取bean的几种方式

```java
//使用容器通过bean的类名获取具体的对象
User user = context.getBean(User.class);
//使用容器通过bean的name或者id获取具体的对象
User user = (User)ioc.getBean("user1");
//使用容器通过bean的name+类名获取具体的对象,适用于同一个类在配置元数据中出现多个的情况
User user = ioc.getBean("user1",User.class);
```

## Bean

### 基于XML配置元数据

#### bean的定义

```xml
<!--
class:指向类所在相对项目root的路径
id:bean的id,唯一
name:bean的name,唯一，一个bean可起多个name,中间通过空格 , ;分隔开即可
-->
<bean class="com.init.User"  id="user1" name="user2 user3,user4;user5"></bean>
<!--为定义的bean起别名,可起多个-->
<alias name="user" alias="user6"></alias>
<alias name="user" alias="user5"></alias>
```

##### bean的作用域

| 作用域                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [singleton](https://github.com/DocsHome/spring-docs/blob/master/pages/core/IoC-container.md#beans-factory-scopes-singleton) | (默认) 每一Spring IOC容器都拥有唯一的实例对象。              |
| [prototype](https://github.com/DocsHome/spring-docs/blob/master/pages/core/IoC-container.md#beans-factory-scopes-prototype) | 一个Bean定义可以创建任意多个实例对象.                        |
| [request](https://github.com/DocsHome/spring-docs/blob/master/pages/core/IoC-container.md#beans-factory-scopes-request) | 将单个bean定义范围限定为单个HTTP请求的生命周期。 也就是说，每个HTTP请求都有自己的bean实例，它是在单个bean定义的后面创建的。 只有基于Web的Spring `ApplicationContext`的才可用。 |
| [session](https://github.com/DocsHome/spring-docs/blob/master/pages/core/IoC-container.md#beans-factory-scopes-session) | 将单个bean定义范围限定为HTTP `Session`的生命周期。 只有基于Web的Spring `ApplicationContext`的才可用。 |
| [application](https://github.com/DocsHome/spring-docs/blob/master/pages/core/IoC-container.md#beans-factory-scopes-application) | 将单个bean定义范围限定为`ServletContext`的生命周期。 只有基于Web的Spring `ApplicationContext`的才可用。 |
| [websocket](https://github.com/DocsHome/spring-docs/blob/master/pages/web/web.md#websocket-stomp-websocket-scope) | 将单个bean定义范围限定为 `WebSocket`的生命周期。 只有基于Web的Spring `ApplicationContext`的才可用。 |

```xml
<!--作用域scope
singleton 默认:单例  只会在Ioc容器种创建一次
prototype 多例（原型bean) 每次获取都会new一次新的bean-->
<bean class="cn.init.beans.Person" id="person3" scope="prototype">
    <property name="id" value="1"></property>
    <property name="realName" value="吴彦祖"></property>
    <property name="name" value="haoz"></property>
</bean>
```

#### 依赖注入

##### bean的基础属性注入

```xml
<!--基于setter方法的依赖注入
    1. 属性必须声明了set方法
    2. name是根据set方法的名字来的  比如方法名字是： setIdxx   ->  name="idxx"
-->
<bean class="com.init.User" id="user">
    <!--property:代表class类里面对应的属性
	-->
    <property name="idtuling" value="1"></property>
    <property name="username" value="zhangsan"></property>
    <property name="realname" value="张三"></property>
</bean>

<!--基于构造函数的依赖注入
    1. 将会调用自定义构造函数来实例化对象，就不会调用默认的无参构造函数
    2. name是根据构造函数的参数名来的，  比如：User(String idxx) -> name="idxx"
    3. name属性可以省略 但是要注意参数的位置 
    4. 如果非要把位置错开 可以使用 name 或者 index 或者 type
    5. index 是下标  从0开始
    6. type 在位置错开情况下只能在类型不一样的时候指定才有明显效果
-->
<bean class="com.init.User" id="user3">
    <constructor-arg name="username"  value="lisi"></constructor-arg>
    <constructor-arg name="id_xushu"  value="1"></constructor-arg>
    <constructor-arg name="realname" value="李四"></constructor-arg>
</bean>
```

##### bean的复杂类型属性注入

```xml
<!--复杂数据类型-->
<bean class="cn.init.beans.Person" id="person">
    <property name="id" value="1"></property>
    <property name="realName" value=""></property>
    <!--注入null值-->
    <property name="name">
        <null></null>
    </property>
    <!--当依赖其他bean: 内部bean (不想其他地方使用此bean的用内部bean)
    <property name="wife">
        <bean class="cn.init.beans.Wife" >
            <property name="age" value="18"></property>
            <property name="name" value="迪丽热巴"></property>
        </bean>
    </property>-->
    <!--当依赖其他bean: 引用外部bean
    <property name="wife" ref="wife"></property>-->
    <property name="birthday" value="2020/05/20"></property>
    <!--注入list类型属性-->
    <property name="hobbies">
        <list>
            <value>唱歌</value>
            <value>跳舞</value>
            <!--如果List的泛型对象使用<bean>-->
        </list>
    </property>
    <!--注入map类型属性-->
    <property name="course" >
        <map>
            <entry key="1" value="JAVA"> </entry>
            <entry key="2" value="HTML"> </entry>
            <!--
            如果Map的泛型对象使用<entry value-ref=""></entry>
            -->
        </map>
    </property>

</bean>

<!--idea选中p:或者c:通过快捷键alt+entry引入xml命名空间-->
<!--可以使用p命名空间来简化基于setter属性注入   它不支持集合
p:wife-ref="wife2"引用对象属性
-->
<bean class="cn.init.beans.Wife" id="wife" p:age="18" p:name="迪丽热巴" p:wife-ref="wife2">
</bean>

<!--可以使用c命名空间来简化基于构造函数属性注入   它不支持集合-->
<bean class="cn.init.beans.Wife" id="wife2"  c:age="20" c:name="xxx">
   <!-- <constructor-arg name="age" value="18"></constructor-arg>-->
</bean>
```

##### depend-on

```xml
<!--使用depends-on可以设置先加载的Bean   也就是控制bean的加载顺序 此处先加载wifi-->
<bean class="cn.init.beans.Person" id="person" depends-on="wife"></bean>
<bean class="cn.init.beans.Wife" id="wife"></bean>
```

##### 懒加载

```xml
<!--使用lazy-init设置懒加载
默认为false: 在spring容器创建的时候加载（实例化）
      true: 在使用的时候(getBean)才会去加载（实例化）-->
<bean class="cn.init.beans.Person" id="person2" lazy-init="true">
</bean>
```

#### Bean的实例化

##### 静态工厂实例化bean

```xml
<!--
factory-method:配置User类中的静态工厂方法
-->
<bean class="com.tuling.service.impl.UserServiceImpl" id="userService2"
      factory-method="createUserServiceInstance">
    <!--若静态工厂方式有参数,可通过constructor-arg注入参数的值
        <constructor-arg name="type" ref="user3"/>
    -->
</bean>
```

##### 抽象工厂实例化bean

通过定义一个工厂bean来初始化具体的bean

```xml
<!--此时UserFactory类的工厂方法createUserInstance就不是static-->
<bean class="com.init.UserFactory" id="userFactory"/>
<bean class="com.init.User" id="user5" factory-bean="userFactory" factory-method="createUserInstance">
    <constructor-arg name="type" ref="user3"/>
</bean>
```

#### 自动注入

前面我们需要注入bean对象时要么使用内部bean注入，要么使用ref注入外部bean,但是其实spring可以帮我们自动完成注入这个过程；

```xml
<!--自动注入：
1. bytype 根据类型自动注入(spring会根据bean里面的所有对象属性的类型，只要它匹配到bean里面某一个类型跟属性类型吻合就会自动注入
2. byname 会根据属性setxxx的名字来自动匹配 (spring会根据bean里面的所有对象属性的set的名字，只要它匹配到bean里面某一个名字跟属性名字吻合就会自动注入
3. constructor 优先根据名字来找， 如果名字没有匹配到根据类型来匹配，  如果类型匹配到多个则不会自动注入
注意：bytype  如果匹配到两个同样的类型会出现错误，所以一定要保证ioc容器里面只有一个对应类型的bean
     byname  最能匹配到唯一的那个bean
     constructor 保证构造函数不能包含多余的其他参数
 default:不会进行自动注入
-->
<bean class="cn.init.beans.Person" id="person6" autowire="constructor" >
</bean>
<!--primary:根据类型匹配自动注入时,以此bean为优先注入bean-->
<bean class="cn.init.beans.Wife" id="wife" p:age="18" p:name="迪丽热巴" primary="true">
</bean>
<!--aurowire-candidate:false为放弃自动注入-->
<bean class="cn.init.beans.Wife" id="wife2" p:age="60" p:name="乔碧螺" aurowire-candidate="false">
</bean>
```

#### bean的生命周期回调

- 使用接口实现的方式来实现生命周期的回调：
  - 初始化方法： bean实现接口： InitializingBean 重写afterPropertiesSet方法   初始化会自动调用的方法
  - 销毁的方法： bean实现接口： DisposableBean  重写destroy 方法   销毁的时候自动调用方法

什么时候销毁?

在spring容器关闭的时候 AbstractApplicationContext.close()或者 使用 ConfigurableApplicationContext.registerShutdownHook方法优雅的关闭

- 使用指定具体方法的方式实现生命周期的回调：

在**对应的bean里面创建对应的两个方法**init-method="init"  destroy-method="destroy"

```xml
<bean class="com.init.User" id="user3" init-method="initConfig" destroy-method="destroyConfig">
    <constructor-arg name="id" value="1"/>
    <constructor-arg name="name" value="haoze"/>
    <constructor-arg name="realName" value="test"/>
</bean>
```

<span style="color:#ff1f1f">使用接口的方式执行的回调方法优先级更高</span>

#### 引入外部配置文件

在resource中添加dbconfig.properties

```properties
username=root
password=123456
url=jdbc:mysql://localhost:3306/demo
driverClassName=com.mysql.jdbc.Driver
```

```xml
<bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
    <property name="username" value="${username}"></property>
    <property name="password" value="${password}"></property>
    <property name="url" value="${url}"></property>
    <property name="driverClassName" value="${driverClassName}"></property>
</bean>
<!--加载外部配置文件
    在加载外部依赖文件的时候需要context命名空间
    -->
<context:property-placeholder location="classpath:database.properties"></context:property-placeholder>
```

#### SpEL

SpEL:Spring Expression Language,spring的表达式语言，支持运行时查询操作对象,使用#{...}作为语法规则，所有的大括号中的字符都认为是SpEL.

```XML
<bean id="user" class="cn.init.entity.User">
       <!--支持任何运算符-->
       <property name="id" value="#{12*2}"></property>
       <!--可以引用其他bean的某个属性值-->
       <property name="name" value="#{address.province}"></property>
       <!--引用其他bean-->
       <property name="role" value="#{address}"></property>
       <!--调用静态方法 #{T(类名).静态方法名} -->
       <property name="hobbies" value="#{T(java.util.UUID).randomUUID().toString().substring(0,4)}"></property>
       <!--调用非静态方法-->
       <property name="gender" value="#{address.getCity()}"></property>
</bean>
```

### 基于注解配置元数据

#### bean定义

| 注解        | 说明                                 |
| ----------- | ------------------------------------ |
| @Controller | 控制器，推荐给controller层添加此注解 |
| @Service    | 业务逻辑，推荐给业务逻辑层添加此注解 |
| @Repository | 仓库管理，推荐给数据访问层添加此注解 |
| @Component  | 给不属于以上基层的组件添加此注解     |

spring底层并不区分bean的实际种类，区分4中只是为了更好的划分层次，提高可读性，利于spring管理。

原理：<span style="color:#ff1f1f">spring自动解析使用了注解的类名，将类名首字母小写作为bean的name</span>

如果不想使用默认bean的name，可以通过value设置bean的name

```java
//此时这个bean就被命名为userTest
@Service(value="userTest")
//当有多个同类的bean时,此bean会优先被注入
@Primary
public class UserServiceImpl implements UserService {
}
```

##### @Scope

设置bean的作用域,同xml配置，默认单例

#### 设置扫描包

```xml
<!--
    context:component-scan:扫描注解配置所在的包 
		use-default-filters="true":默认为true,表示会扫描所在包的注解配置元数据,false为不扫描
    context:exclude-filter:排除扫描
	context:include-filter:包含扫描
    -->
<context:component-scan base-package="com.*" use-default-filters="true">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```
上面的排除类型及说明如下：

| 过滤类型             | 表达式例子                   | 描述                                                         |
| -------------------- | ---------------------------- | ------------------------------------------------------------ |
| annotation (default) | `org.example.SomeAnnotation` | 要在目标组件中的类级别出现的注解。                           |
| assignable           | `org.example.SomeClass`      | 目标组件可分配给（继承或实现）的类（或接口）。               |
| aspectj              | `org.example..*Service+`     | 要由目标组件匹配的AspectJ类型表达式。                        |
| regex                | `org\.example\.Default.*`    | 要由目标组件类名匹配的正则表达式。                           |
| custom               | `org.example.MyTypeFilter`   | `org.springframework.core.type .TypeFilter`接口的自定义实现。 |

#### 依赖注入

##### 基础属性注入

```java
@Component
public class User {
    @Value("郝泽")
    private String name;
    @Value("${username}")
    private String username;
    @Value("#{role.name}")
    private String roleName;
}
```

- 直接注入属性值。
- ${}:注入外部资源文件的变量。

<span style="color:#ff1f1f">此时是优先获取系统变量的值 </span>可参考此博客解决优先级问题[https://www.cnblogs.com/d191/p/12566008.html](https://www.cnblogs.com/d191/p/12566008.html)

- #{}:SpEL表达式,注入外部bean的值。

##### @depend-on

```java
@Service
@DependsOn("userServiceImpl")
public class RoleServiceImpl{
}
```

此时spring就会先加载userServiceImpl这个bean

##### @Lazy

```java
@Service
@Lazy
public class RoleServiceImpl{
}
```

此时RoleServiceImpl这个bean就不会在spring容器初始化时加载，而是使用到时才加载

#### 自动注入

##### @Autowired

```java
@Controller
public class UserController {
    @Autowired
    private UserService userServiceImpl;
}
```

当使用AutoWired注解的时候，自动装配的时候是根据类型(byType)实现的。

​		1、如果只找到一个(UserService的实现类或者子类)，则直接进行赋值;

​		2、如果没有找到，则直接抛出异常;

​		3、如果找到多个(UserService的实现类或者子类)，那么会按照变量名(userServiceImpl)作为name继续byName匹配;

​				1、匹配上(一个bean的name为userServiceImpl)直接进行装配

​				2、如果匹配不上则直接报异常

扩展:

- 若需要自动注入的接口有泛型，那么通过泛型也可以帮助自动注入判断出应该注入那个接口

```java
@Controller
public class UserController {
	//比如此时若UserService有多个实现类,那么会自动注入泛型指定为User的那个实现类,若此时还出现找到多个，依然按Autowired逻辑走
    @Autowired
    private UserService<User> UserServiceImpl;

    public void getName(){
        userTest.getName();
    }
}
```

- @Resource

JDK自带注解,用法跟@Autowired一样，区别是@Resource默认优先通过byName匹配；

##### @Qualifier

```java
@Controller
public class UserController {

    @Autowired
    @Qualifier(value = "UserServiceImpl")
    private UserService userTest;
    public void getName(){
        userTest.getName();
    }
}
```

@Qualifier用于补充自动注入，value指定注入的bean的name,如上面的例子：

1. 如果找到只有一个bean的name为UserServiceImpl的直接赋值;
2. 如果没有找到bean的name为UserServiceImpl则直接报错；
3. 若找到多个,也直接报错

可以看到，此时@Autowired注解原有的逻辑被覆盖失效

#### bean的生命周期回调

在bean的类中使用注解修改方法

##### @PostConstruct

bean初始化时会调用被@PostConstruct修饰的方法

```java
@PostConstruct
public void init(){
System.out.println("bean已初始化");
}
```

##### @PreDestroy

bean销毁时会调用被@PreDestroy修饰的方法

```java
@PreDestroy
public void destory(){
System.out.println("bean已销毁");
}
```

### 基于javaConfig配置元数据

#### 配置元数据

```java
@Configuration
@ComponentScan(basePackages = {"com.javaConfig"})
@PropertySource("classpath:database.properties")
@Import({UserConfig.class})
public class AppConfig {

    @Value("${mysql.username}")
    private String name;
    @Value("${mysql.password}")
    private String password;
    @Value("${mysql.url}")
    private String url;
    @Value("${mysql.driverClassName}")
    private String driverName;
	
    @Bean
    public DruidDataSource dataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setName(name);
        dataSource.setPassword(password);
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverName);
        return dataSource;
    }
}
```

#### @Configuration

在之前XML配置元数据中,我们通过XML文件配置元数据，而通过javaConfig的方式配置元数据时，使用@Configuration来申明类为配置类，相当于原来的XML文件

#### @ComponentScan

相当于原来XML文件中的包扫描标签,<span style="color:Red">若没有配置扫描包路径，会把当前配置类所在的包当作扫描包进行扫描。</span>

#### @PropertySource

配置外部资源文件

#### @Import

- 引入其他配置类

```java
@Import({UserConfig.class})//引入其他配置类
```

- 直接注册bean

```java
@Import({User.class})//直接注册bean
```

- 引入ImportSelector的实现类,通过类的完整限定名字段串来同时注册多个bean

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{"com.javaConfig.People",People.class.getName()};//两种写法任选其一
    }
}
```

```java
@Import({MyImportSelector.class})
public class AppConfig {
}
```

此时通过加载AppConfig创建spring容器，只能通过byType的方式获取People这个bean。

```java
@Test
public void testPeople(){
    People people = ioc.getBean(People.class);
    System.out.println(people);
    people = ioc.getBean(People.class);
    System.out.println(people);
}
```

- 引入ImportBeanDefinitionRegistrar的实现类,手动注册类的定义态

```java
public class MyBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(People.class);
        //设置bean的作用域
        beanDefinition.setScope("prototype");
        registry.registerBeanDefinition("people",beanDefinition);
    }
}

```

可以看到，这种方式,相当于抛弃了注解的使用，所有bean定义的属性都需要通过设置beanDefinition来手动定义,可设置属性可以查看AbstractBeanDefinition，这其实也是spring源码中使用的方式

![image-20210212192915023](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210212192915023.png)

#### @Bean

类似XML配置元数据的`<bean/>`标签，是一个方法级别的注解。

注册的bean的name默认为注解配置的方法名称，也可以使用name属性设置bean的name。

可以使用`init-method="initByConfig" destroy-method="destroyByConfig"` 属性设置生命周期回调方法。

```java
@Configuration
public class UserConfig {

    @Bean(initMethod = "initByConfig",destroyMethod = "destroyByConfig")
    public User user(){
        User user = User.builder().id(1L).name("郝泽").build();
        return user;
    }

}
```

方法要定义在bean的类中

```java
public class User {
    private Long id;
    private String name;


    public void initByConfig(){
        System.out.println("user初始化");
    }

    public void destroyByConfig(){
        System.out.println("user销毁");
    }
}
```

@Bean注解在使用的时候与原来XML的`<bean/>`标签有一点区别的是这里是我们自己初始化创建bean的实例，将实例作为方法的返回值交给spring容器管理，因此我们可以**干预类的实例化过程**，而XML配置是由spring来初始化创建bean。

#### 自动注入

依赖外部bean(不同config类的bean)在@Bean修饰的方法中添加形参,形参可以byType自动注入

依赖内部bean(同一个config类的bean)直接调用方法即可