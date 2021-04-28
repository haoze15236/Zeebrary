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
    
	<!--解决使用@ResponseBody返回报文中文乱码-->
    <mvc:annotation-driven>
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

# SpringMVC执行流程

​		当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/7A49C57D2B984A1EABE723B1A45E9873/E75F3823251F4CB693635E55C66B71FF/2580](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/7A49C57D2B984A1EABE723B1A45E9873/E75F3823251F4CB693635E55C66B71FF/2580)

- **DispatcherServlet**： 前端调度器 ， 负责将请求拦截下来分发到各控制器方法中
- **HandlerMapping**: 负责根据请求的URL和配置@RequestMapping映射去匹配， 匹配到会返回Handler（具体控制器的方法）
- **HandlerAdaper**: 负责调用Handler-具体的方法-  返回视图的名字  Handler将它封装到ModelAndView(封装视图名，request域的数据）
- **ViewReslover**: 根据ModelAndView里面的视图名地址去找到具体的jsp封装在View对象中
- **View**：进行视图渲染（将jsp转换成html内容） 最终response到的客户端

## 执行流程

> - DispatcherServlet表示前端控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。
> - HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler。
> - 返回处理器执行链，根据url查找控制器，并且将解析后的信息传递给DispatcherServlet
> - HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。
> - 执行handler找到具体的处理器
> - Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。
> - HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。
> - DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。
> - 视图解析器将解析的逻辑视图名传给DispatcherServlet。
> - DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图，进行视图渲染
> - 将响应数据返回给客户端

