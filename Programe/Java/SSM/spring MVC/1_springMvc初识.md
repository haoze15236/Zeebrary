**提示**： spring MVC是一个在web容器中发挥作用的框架,基于spring，所以在学习之前需要先了解spring 和javaWeb的知识,

# 项目搭建

创建一个maven项目

## 添加依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.6.RELEASE</version>
</dependency> 
```

maven项目中，并没有web项目的web文件夹,选中项目,右键`add frameworks support`,添加web application（4.0）,Idea会帮我们自动创建web文件夹。

## web.xml

首先，在web.xml文件中配置spring mvc的核心:前端控制器`DispatcherServlet`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
        version="4.0">
   <!--配置DispatcherServlet-->
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
        <!--解决post请求乱码-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
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

使用tomcat容器，在容器启动时会加载web.xml中配置的DispatcherServlet,同时找到配置的spring配置文件（此处配置的是:`spring-mvc.xml`）,进而加载spring容器

## spring-mvc.xml

配置spring容器，设置扫描包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <context:component-scan base-package="mvc.**"></context:component-scan>
    
    <!--mvc:annotation-drive 启动spring mvc注解-->
    <mvc:annotation-driven>
	<!--解决使用@ResponseBody返回报文中文乱码-->
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
    <mvc:resources mapping="/resources/**" location="/resources/"></mvc:resources>
</beans>
```

在mvc文件夹下定义一个controller

```java
package mvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloSpringMvc {

	@RequestMapping("/hello")
	@ResponseBody
	public String hello(String name){
		System.out.println("hello springmvc:"+name);
		return "hello";
	}
}
```

以上，基于spring mvc的web容器配置就完成了，配置好tomcat启动，可能会出现找不到类的报错，或者前端访问，抛出404，我们需要添加lib。如图，在WEB-INF下新建lib文件夹，将左边的依赖项选中，右键put into WEB-INF/lib 即可

![image-20210427213800949](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210427213800949.png)

