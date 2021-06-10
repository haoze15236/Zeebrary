整合SSM也就是把spring，springMVC，mybatis合并到一起，来搭建快速开发web项目的基础框架。需要明确一个概念就是,是在web容器里包含spring容器。所以我们先来构建web容器的配置，也就是类似spring MVC部署步骤:

# pom依赖

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <spring.version>5.2.8.RELEASE</spring.version>
        <servlet.version>4.0.1</servlet.version>
        <jackson.version>2.12.3</jackson.version>
        <hibernate.version>5.1.0.Final</hibernate.version>
        <commons-fileupload.version>1.4</commons-fileupload.version>
        <aspectjweaver.version>1.9.5</aspectjweaver.version>
        <mybatis.version>3.5.6</mybatis.version>
    </properties>

    <dependencies>
        <!--spring MVC依赖 会自动加载spring IOC相关依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--servlet-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.version}</version>
        </dependency>
        <!--JSR349数据认证依赖-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <!--jackson依赖-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <!--文件上传-->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>${commons-fileupload.version}</version>
        </dependency>
        <!--spring依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--spring aop依赖-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectjweaver.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--spring 声明式事务-jdbc依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--spring 声明式事务-事务管理器依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--配置druid数据源依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.23</version>
        </dependency>
        <!--mybatis整合spring-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>
        <!--mybatis依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <!--slf4j日志接口-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <!--slf4j 具体实现logback-->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        <!--junit依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

# 基于XML整合

## web.xml

- 定义DispatcherServlet,同时指定spring MVC的xml配置文件
- 设置请求编码
- 设置请求拦截映射匹配
- 设置spring容器全局配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--定义spring全局配置文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-core.xml</param-value>
    </context-param>
    <!--spring初始化监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    
    <servlet>
       <servlet-name>springmvc</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!--关联springmvc的配置文件 不配置会自动找[servlet-name]-servler.xml的配置文件-->
       <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:spring-mvc.xml</param-value>
       </init-param>
    <load-on-startup>1</load-on-startup>
     </servlet>
     <!--匹配servlet的请求，/标识匹配所有请求-->
     <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <!--
       /                        会拦截除了 .jsp的所有请求
       /*                       会拦截所有请求
       *.do  *.action           匹配url结尾以.do,.action的请求
       /request/*               匹配所有/request/开头的请求     -->
       <url-pattern>/</url-pattern>
     </servlet-mapping>

    
    <!--请求编码设置-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--解决请求中文乱码-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!--解决响应乱码-->
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

## spring-mvc.xml

只扫描controller，配置请求映射，视图解析器,文件上传

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--定义spring Bean扫描规则:扫描所有controller-->
    <context:component-scan base-package="com.**" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!--动态资源扫描-->
    <mvc:annotation-driven></mvc:annotation-driven>
    <!--静态资源扫描-->
    <mvc:default-servlet-handler/>

    <!--定义静态资源文件路径,通过/resource/资源路径名即可访问静态资源-->
    <mvc:resources mapping="/resources/**" location="/resources/"/>

    <!--视图控制器,可直接通过path路径访问视图-->
    <mvc:view-controller path="/" view-name="index"/>

    <!--定义视图解析器,设置前缀和后缀-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" name="viewResolver">
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>


    <mvc:annotation-driven conversion-service="conversionService">
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>application/json;charset=UTF-8</value>
                        <value>text/plain;charset=UTF-8</value>
                        <value>text/html;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <!--请求数据转换器-->
        <bean class="org.springframework.format.support.FormattingConversionServiceFactoryBean" id="conversionService">
            <property name="converters">
                <set>
                    <ref bean="stringToDateConverter"/>
                </set>
            </property>
        </bean>

    <!--文件上传解析器-->
    <bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">
        <property name="defaultEncoding" value="utf-8"/>
        <property name="maxUploadSize" value="10240000000"/>
    </bean>

    <!--    <mvc:interceptors>-->
    <!--        <bean class="mvc.interceptors.MyInterceptor" />-->
    <!--    </mvc:interceptors>-->
</beans>
```

## spring-core.xml

加载除了controller之外所有需要注入到spring容器中的bean,配置数据源,事务管理器,AOP。

整合mybatis，注入所有的mapper接口类，配置所有的mapper.xml文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd">

    <!--定义spring Bean扫描规则:扫描除了controller外的所有bean-->
    <context:component-scan base-package="com.**">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!--定义spring配置资源文件路劲-->
    <context:property-placeholder location="classpath:spring-config.properties"/>
    <!--开启AOP注解事务-->
    <aop:aspectj-autoproxy/>

    <!--配置数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
    </bean>

    <!--配置spring jdbc事务管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--开启基于注解的事务控制模式，依赖tx名称空间-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sessionFactory">
        <property name="dataSource" ref="dataSource"/>
        <!--配置mybatis配置文件,mybatis的setting属性只能在配置文件里定义-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!--指定mapper文件路径-->
        <property name="mapperLocations" value="classpath*:/**/*Mapper.xml"/>
    </bean>

    <!--spring管理mapper接口 两种方式任选其一-->
    <!-- 方式1：使用mybatis包扫描所有的mapper接口-->
    <mybatis:scan base-package="com.**.mapper"/>
    <!--方式2：通过MapperScannerConfigurer来扫描所有mapper接口注入spring容器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="*.**.mapper" />
    </bean>
</beans>
```

# spring&springMVC整合源码

在web.xml文件中配置了

```xml
<listener>
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

该类的UML类图如下:

![image-20210610000854377](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210610000854377.png)

由于继承了ServletContextListener接口，在tomcat容器启动时,会回调contextInitialized方法，初始化XmlWebApplicationContext,并将初始好的web应用上下文放入ServletContext的属性中。

重点关注其中的**configureAndRefreshWebApplicationContext**方法，默认读取contextConfigLocation参数定义的配置文件，进入XML解析初始化spring容器的逻辑，跟IOC容器初始化一样，调用的refresh()方法。

## DispatcherServlet初始化

> 初始化了springMVC的bean工厂，并设置父容器为spring初始化的上下文

![image-20210610002803174](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210610002803174.png)

从UML类图中可以看到，DispatcherServlet本质还是一个Servlet,不过他实现了很多接口和抽象类，我们重点关注以下几个:

### HttpServletBean

```java
public final void init() throws ServletException {
    // Set bean properties from init parameters.
    //从web.xml文件中配置的init-parameters属性读取配置文件
    PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
    if (!pvs.isEmpty()) {
        try {
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
            initBeanWrapper(bw);
            //设置配置文件属性
            bw.setPropertyValues(pvs, true);
        }
        catch (BeansException ex) {
            if (logger.isErrorEnabled()) {
                logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
            }
            throw ex;
        }
    }
    // 抽像方法交给子类实现
    initServletBean();
}
```

重写了Servlet的init方法，因此在DispatcherServlet初始化的时候首先会执行这一段逻辑。

### FrameworkServlet

作为HttpServletBean的子类，实现了上面的initServletBean()方法，重点关注调用的initWebApplicationContext方法

```java
protected WebApplicationContext initWebApplicationContext() {
    //从ApplicationContext中获取上下文，其中包含初始化好的spring容器
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    //其他代码....
    //创建springMVC容器上下文，跟第一步一样调用的configureAndRefreshWebApplicationContext方法，并设置了父容器为这里传进去的上下文
    wac = createWebApplicationContext(rootContext);
    //其他代码....
}
```

从这里可以看到，springMVC容器初始化在spring容器之后，并且是spring容器上下文的子容器，所以从springmvc容器中管理的bean，如controller,是可以自动注入spring容器中的bean(如service)，但是反过来就不行。

另外需要注意的是，在FrameworkServlet类中，有个内部类：

```java
private class ContextRefreshListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        FrameworkServlet.this.onApplicationEvent(event);
    }
}
```

实现了spring的ApplicationListener接口，作为ContextRefreshedEvent事件的监听者，在springMVC容器初始化的最后阶段——finishRefresh()方法中，发布了这个事件，也就回调到了这里:

```java
public void onApplicationEvent(ContextRefreshedEvent event) {
    //设置状态,之后不会重复调用
    this.refreshEventReceived = true;
    synchronized (this.onRefreshMonitor) {
        //空的方法，交给子类实现
        onRefresh(event.getApplicationContext());
    }
}
```

在这个地方，就调用了子类DispatcherServlet的onRefresh方法,初始化了springMVC中老生常谈的几个核心组件

```java
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

默认内置对象如下维护在spring-webmvc的jar包属性资源文件DispatcherServlet.properties中,各个组件详细使用可参考 [10_springMVC核心组件源码分析](./spring MVC/10_springMVC核心组件源码分析)

# 整合mybatis源码解析

在spring-core.xml文件中配置了mybatis的相关类，其中比较重要的如下。

## SqlSessionFactoryBean

> 创建sqlSession工厂，解析mybatis配置类。

从spring-core.xml中可以看到，定义了一个SqlSessionFactoryBean，这是用来解析myabtis配置文件的类,实现了InitializingBean接口，在bean初始化之后，回调<span style="color:green">afterPropertiesSet()</span>方法。

```java
@Override
  public void afterPropertiesSet() throws Exception {
    //.....
    //创建sqlSessionFactory
    this.sqlSessionFactory = buildSqlSessionFactory();
  }
```

过程和mybatis手动创建工厂大同小异，需要注意的是：

- **创建事务工厂**

```java
targetConfiguration.setEnvironment(new Environment(this.environment,
        this.transactionFactory == null ? new SpringManagedTransactionFactory() : this.transactionFactory,
        this.dataSource));
```

可以看到这里使用了**SpringManagedTransactionFactory** 的实例,这个工厂创建的是**SpringManagedTransaction** 事务，也就是spring的声明式事务。

## MapperScannerConfigurer

> 为mapper接口在Spring容器中创建Bean实例

实现了BeanDefinitionRegistryPostProcessor接口,在spring IOC容器初始化的时候，调用所有的bean定义的BeanFactoryPostProcessor里面会回调这个方法。

- **ClassPathMapperScanner**

  > 扫描mapper接口，并注册bean定义到bean工厂中。

  创建了此对象，用于扫描所有的mapper接口。scan方法调用的是父类ClassPathBeanDefinitionScanner的，然后又重写了父类的doScan方法。最重要的是重写了父类的isCandidateComponent方法，由于spring默认是通过这个方法判断类如果是接口则跳过注册bean定义，所有重写之后，可以进入候选的注册bean定义集合——`Set<BeanDefinitionHolder> beanDefinitions`。

  在 **<span style="color:green">processBeanDefinitions</span>**方法中修改了原接口的bean定义:

  ```java
  definition.setBeanClass(this.mapperFactoryBeanClass);
  ```

- **MapperFactoryBean**

  > mapper接口真正实例化的类，使用jdk动态代理

继承了FactoryBean接口，在实例化bean的时候回调getObject()方法

```java
  @Override
  public T getObject() throws Exception {
    return getSqlSession().getMapper(this.mapperInterface);
  }
```

实际上是从mybatis的configutation对象中的MapperRegistry中获取到了mapper的代理对象。
