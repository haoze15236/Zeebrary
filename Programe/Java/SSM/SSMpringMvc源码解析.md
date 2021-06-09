# 上下文初始化

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

# DispatcherServlet初始化

![image-20210610002803174](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210610002803174.png)

从UML类图中可以看到，DispatcherServlet本质还是一个Servlet,不过他实现了很多接口和抽象类，我们重点关注以下几个:

## HttpServletBean

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

## FrameworkServlet

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

默认内置对象如下维护在spring-webmvc的jar包属性资源文件DispatcherServlet.properties中。

