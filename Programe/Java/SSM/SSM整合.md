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
        <!--mybatis-redis缓存-->
        <dependency>
            <groupId>org.mybatis.caches</groupId>
            <artifactId>mybatis-redis</artifactId>
            <version>1.0.0-beta2</version>
        </dependency>
        <!--mybatis-pageHelper分页插件-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.2.0</version>
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

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.22</version>
        </dependency>
    </dependencies>
</project>
```

# web.xml

- 定义DispatcherServlet,同时指定spring MVC的xml配置文件
- 设置请求编码
- 设置请求拦截映射匹配
- 设置spring容器初始化配置

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

# spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--定义spring Bean扫描规则:扫描所有controller-->
    <context:component-scan base-package="**" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    
    <!--动态资源扫描-->
    <mvc:annotation-driven></mvc:annotation-driven>
    <!--静态资源扫描-->
    <mvc:default-servlet-handler/>

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
    <!--定义静态资源文件路径,通过/resource/资源路径名即可访问静态资源-->
    <mvc:resources mapping="/resources/**" location="/resources/"/>

    <!--定义视图解析器,设置前缀和后缀-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" name="viewResolver">
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--视图控制器,可直接通过path路径访问视图-->
    <mvc:view-controller path="/" view-name="index"/>


    <!--请求数据转换器-->
<!--    <bean class="org.springframework.format.support.FormattingConversionServiceFactoryBean" id="conversionService">-->
<!--        <property name="converters">-->
<!--            <set>-->
<!--                <ref bean="stringToDateConverter"/>-->
<!--            </set>-->
<!--        </property>-->
<!--    </bean>-->

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

# spring-core.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--定义spring Bean扫描规则:扫描除了controller外的所有bean-->
    <context:component-scan base-package="**">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <context:property-placeholder location="classpath:spring-config.properties"/>

    <!--开启AOP注解事务-->
    <aop:aspectj-autoproxy/>
</beans>
```

