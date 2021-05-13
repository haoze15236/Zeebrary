# [mybatis-plugins](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)

mybatis预留了4个拓展点来对它的行为做出合理的修改,通过合理配置pluguns插件，即可做到如下内容的修改。

![image-20210511234251752](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210511234251752.png)分页功能

插件一般用于，查询数据分页,性能监控，公共字段统一赋值等。

## 源码解析

### Configuration类和interceptorChain属性

`org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration` 解析mybatis-config.xml配置文档 到**Configuration**对象中,其中就有把所有配置`<plugins/>`通过反射创建类的实例，并保存在属性interceptorChain中。

```java
private void parseConfiguration(XNode root) {
    try {
        ...
      //加载所有配置的插件类,通过反射创建类的实例,保存在Configuration的interceptorChain属性中
      pluginElement(root.evalNode("plugins"));
        ...
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

### Interceptor接口

在`org.apache.ibatis.session.Configuration#newExecutor(Transaction,ExecutorType)`方法中创建**Executor**的时候，通过第一步Configuration中的InterceptorChain来调用每个实现了`Interceptor`接口的插件类的`plugin`方法:

```java
public interface Interceptor {
  Object intercept(Invocation invocation) throws Throwable;
  default Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }
  default void setProperties(Properties properties) {
    // NOP
  }
}
```

此接口定义的很简单，plugin方法用于创建目标类代理对象，详细过程下面会说。intercept方法就是具体的增强方法，注意参数 

- **Invocation**

```java
public class Invocation {
  //实际的目标类对象
  private final Object target;
  //被拦截器拦截的方法
  private final Method method;
  //方法执行时的参数值数组
  private final Object[] args;
}
```

自定义拦截器类需要用到两个注解<span style="color:#ffa100">@Intercepts</span>和<span style="color:#ffa100">@Signature</span>，解释下@Signature参数：

- **type**：需要拦截的目标类
- **method** ：目标类的方法名
- **args** ：目标类的方法参数类型数组

通过这样的注解定义，就可以确定此拦截器是用来拦截什么方法的。

```java
@Intercepts(
        {
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
        }
)
public class MyInterceptor implements Interceptor {
	...
}
```

### Plugin

插件类的作用就是用来创建目标类的动态代理对象,来执行实现了**Interceptor**接口的拦截器跟进去看，调用的`org.apache.ibatis.plugin.Plugin#wrap`，将拦截器定义的目标类做了JDK代理增强。

```java
public class Plugin implements InvocationHandler {

  private final Object target;//待拦截器增强的目标类
  private final Interceptor interceptor;//拦截器对象
  private final Map<Class<?>, Set<Method>> signatureMap;//拦截器拦截的类和方法

  private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
    this.target = target;
    this.interceptor = interceptor;
    this.signatureMap = signatureMap;
  }

public static Object wrap(Object target, Interceptor interceptor) {
    //获取拦截器类上的注解配置,@Intercepts和@Signature,返回能拦截的类和方法的集合。
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    //获取目标类的Class对象
    Class<?> type = target.getClass();
    //获取目标类的所有接口
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      //jdk动态代理增强方法
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap));
    }
    return target;
  }

@Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      Set<Method> methods = signatureMap.get(method.getDeclaringClass());
      if (methods != null && methods.contains(method)) {
        //执行拦截器的拦截方法,传入对象Invocation
        return interceptor.intercept(new Invocation(target, method, args));
      }
      return method.invoke(target, args);
    } catch (Exception e) {
      throw ExceptionUtil.unwrapThrowable(e);
    }
  }
}
```

# [mybatis-pageHelper](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md)

测试：

```java
@Test
public void testPageHelper(){
    try (SqlSession session = sqlSessionFactory.openSession(true)) {
        ExpReportLineMapper mapper = session.getMapper(ExpReportLineMapper.class);
        //通过ThreadLocal<Page>保存分页参数,线程安全，只对下当前线程中下一个执行sql有效
        PageHelper.startPage(1,2);
        System.out.println(mapper.selectMap());
        PageHelper.startPage(1,2);
        System.out.println(mapper.selectOne(1));
        PageHelper.startPage(1,2);
        System.out.println(mapper.selectLines(1));
    }
}
```

输出结果：

```
{1={DESCRIPTION=测试1, EXP_REPORT_LINE_ID=1, PAT_RATIO=10, EXP_REPORT_HEADER_ID=1}, 2={DESCRIPTION=测试2, EXP_REPORT_LINE_ID=2, PAT_RATIO=20, EXP_REPORT_HEADER_ID=1}}

BillLine(billLineId=1, billHeaderId=1, description=测试1)ExpReportLine{payRatio=10}

Page{count=true, pageNum=1, pageSize=2, startRow=0, endRow=2, total=4, pages=2, reasonable=false, pageSizeZero=false}[BillLine(billLineId=1, billHeaderId=1, description=测试1)ExpReportLine{payRatio=10}, BillLine(billLineId=2, billHeaderId=1, description=测试2)ExpReportLine{payRatio=20}]
```

可以看到，返回的如果是List集合，会被自动包装到`com.github.pagehelper.Page`类中，所以返回的是Page对象。同时在官方说明中，有提到可以再把Page封装到`com.github.pagehelper.PageInfo`里以获取更多友好的支持。

```java
PageInfo page = new PageInfo(list);
```

