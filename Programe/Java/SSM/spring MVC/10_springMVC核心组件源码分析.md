# SpringMVC执行流程

​		当发起http请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

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

在`DispatcherServlet.properties`中可以看到spring定义的一些默认类：

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver
org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver
#处理映射器
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping
#处理适配器
org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter
#处理异常解析器
org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator
#视图解析器
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

# MultipartResolver

> 文件解析器接口，默认解析客户端发送过来的post请求，其中( `Content-Type` )为 `multipart/*` 格式，也就是文件上传的请求

# LocaleResolver

> 语言解析器接口，默认使用AcceptHeaderLocaleResolver，从请求头的Accept-Language属性获取语言信息

还有几个默认实现:

- SessionLocaleResolver，通过session来获取语言环境。
- CookieLocaleResolver,通过cookie来获取语言环境。

# ThemeResolver

> 主题解析器,默认使用FixedThemeResolver，固定主题

# HandlerMapping

> 请求处理映射器,用于解析请求，获取对应的处理器和拦截器调用链

请求发送到服务器之后，首先通过RequestMappingHandlerMapping

默认包含两个实现

BeanNameUrlHandlerMapping

通过bean的name与请求的url匹配来指定调用的controller，**一般不使用**。可参考:https://www.cnblogs.com/suanshun/p/6700200.html

## RequestMappingHandlerMapping

> 我们使用@RequestMapping注解都是通过这个映射器实现类来记录如何映射的。

解析spring MVC的xml文件生成spring mvc容器时，读取到`<mvc:annotation-driven/>`标签时，会调用AnnotationDrivenBeanDefinitionParser注册此实现类。

# HandlerAdapter

> 处理器适配器，判断处理器是否可以处理请求，然后执行处理，返回ModeAndView

默认包含三个实现类：

- HttpRequestHandlerAdapter

  使用此适配器需要controller实现HttpRequestHandler接口

- SimpleControllerHandlerAdapter

  使用此适配器需要controller实现Controller接口

- RequestMappingHandlerAdapter

  跟上面RequestMappingHandlerMapping一起被注册创建的，需要controller使用@Controller注解。

# HandlerExceptionResolver

> 处理器异常解析器

- ResponseStatusExceptionResolver
- DefaultHandlerExceptionResolver
- ExceptionHandlerExceptionResolver





[spring MVC中返回json源码](https://www.jianshu.com/p/66dd08df166e)

![image-20210616210344365](C:/Users/zee/AppData/Roaming/Typora/typora-user-images/image-20210616210344365.png)

找到MappingJackson2HttpMessageConverter来zh
