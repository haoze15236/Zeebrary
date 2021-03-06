# SpringMVC执行流程

​		当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/7A49C57D2B984A1EABE723B1A45E9873/E75F3823251F4CB693635E55C66B71FF/2580](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/7A49C57D2B984A1EABE723B1A45E9873/E75F3823251F4CB693635E55C66B71FF/2580)

DispatcherServlet： 前端调度器 ， 负责将请求拦截下来分发到各控制器方法中

HandlerMapping: 负责根据请求的URL和配置@RequestMapping映射去匹配， 匹配到会返回Handler（具体控制器的方法）

HandlerAdaper: 负责调用Handler-具体的方法-  返回视图的名字  Handler将它封装到ModelAndView(封装视图名，request域的数据）

ViewReslover: 根据ModelAndView里面的视图名地址去找到具体的jsp封装在View对象中

View：进行视图渲染（将jsp转换成html内容） 最终response到的客户端

1、DispatcherServlet表示前端控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。

2、HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler。

3、返回处理器执行链，根据url查找控制器，并且将解析后的信息传递给DispatcherServlet

4、HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。

5、执行handler找到具体的处理器

6、Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。

7、HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。

8、DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。

9、视图解析器将解析的逻辑视图名传给DispatcherServlet。

10、DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图，进行试图渲染

11、将响应数据返回给客户端

# 项目部署

## 添加依赖

```xml
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-webmvc</artifactId>
           <version>5.2.3.RELEASE</version>
       </dependency> 
```

## web.xml

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
           <param-value>classpath:applicationContext.xml</param-value>
       </init-param>
        <load-on-startup>1</load-on-startup>
   </servlet>
   <!--匹配servlet的请求，/标识匹配所有请求-->
   <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
       <url-pattern>/</url-pattern>
   </servlet-mapping>
    
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--解决post请求乱码-->
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

## applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.tulingxueyuan"></context:component-scan>
</beans>
```

定义一个controller,访问hello能正常返回则说明部署成功

```java
package springMvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
public class HelloSpringMvc {
    @RequestMapping("/hello")
    public String hello(String name){
        System.out.println("hello springmvc:"+name);
        return "hello";
    }
}

```

# 注解使用



### @RequestParam

如何获取SpringMVC中请求中的信息，默认情况下，可以直接在方法的参数中填写跟请求一样的名称，此时会默认接受参数
如果有值，直接赋值，如果没有，那么直接给空值

@RequestParam:获取请求中的参数值,使用此注解之后，参数的名称不需要跟请求的名称一致，但是**必须要写**
public String request(@RequestParam("user") String username){}

此注解包含三个参数：
**value**:表示要获取的参数值
**required**：表示此参数是否必须，默认是true，如果不写参数那么会报错，如果值为false，那么不写参数不会有任何错误
**defaultValue**:如果在使用的时候没有传递参数，那么定义默认值即可

建议所有参数都使用包装数据类型。

### @RequestHeader

获取请求头中的某个属性使用Map接受则可以接收到请求头所有的内容

### @CookieValue              

获取cookie中的某个属性,使用Map接受则可以接收到cookie所有的内容

```java
@RequestMapping("/params")
public String paramsRequest(@CookieValue Map cookie){
    System.out.println(cookie);
    return cookie.toString();
}
```

### @PathVariable（restful参数）

```java
@RequestMapping("/user/{id}/{username}")
public String path01(@PathVariable("id") Integer id,@PathVariable("username") String name){
    System.out.println(id);
    System.out.println(name);
    return "/index.jsp";
}
```

若是对象，{属性名}与对象属性名对应上则会自动匹配



### @SessionAttribute

用在类上面的，写入session的：

```java
@Controller
// 通过model中指定的属性去写入到session,同时也会从session中写入指定的属性到model,
// 所以使用SessionAttributes的情况下 model和session是共同的
// 使用该方式设置session是依赖model
@SessionAttributes("type")
public class DTVController {
   @RequestMapping("output1")
   public String output1(Model model){
       model.addAttribute("type","hello,Springmvc");//此model的type值会被设置到session中
       return "output";
  }
}
```

用在参数上面的，负责读取session，默认指定的属性是必须要存在的，如果不存在则会报错，可以设置required =false 不需要必须存在，不存在默认绑定null。

```java
@RequestMapping("/getSession")
public String getSession(@SessionAttribute(value="type",required = false) String type){
    System.out.println(type);
    return "main";
}
```

### **@ModelAttribute**

**用在方法上:**@ModelAttribute的方法会在当前处理器中所有的处理方法之前调用

**用在参数上:**可以省略，加上则会从model中获取一个指定的属性和参数进行合并，因为model和sessionAttribute具有共通的特性，所以如果session中有对应的属性也会进行合并

注意:若通过@ModelAttribute来设置**单例,类级别的变量**存在线程安全问题。



# 面试题

1. springmvc 控制器是不是单例的？如果是单例的会出现什么问题？怎么解决？

spring MVC依赖spring容器，默认是单例的，单例类若存在类级别变量可能存在线程安全问题,可以通过

- 声明成方法级别变量
- 使用ThreadLocal存储变量