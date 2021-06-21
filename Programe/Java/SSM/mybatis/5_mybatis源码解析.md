mybatis源码从创建SqlSessionFactory跟进去开始看。

```java
String resource = "mybatis-config.xml";
InputStream inputStream = null;
try {
inputStream = Resources.getResourceAsStream(resource);
} catch (IOException e) {
e.printStackTrace();
}
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

# **XMLConfigBuilder** 

> 用于解析mybatis的xml配置文件

继承 **BaseBuilder**,可以看到父类中有三个成员属性:

```java
public abstract class BaseBuilder {
  //配置类解析出来的对象
  protected final Configuration configuration;
  //类型别名注册机
  protected final TypeAliasRegistry typeAliasRegistry;
  //类型处理器注册机
  protected final TypeHandlerRegistry typeHandlerRegistry;
  //....
}
```

初始化的时候创建一个Configutation类的实例并赋值给父类,调用<span style="color:green">parse()</span>开始解析mybatis配置类，解析过程可自行看源码，本文中挑选几个重点的讲解一下:

- **pluginElement(root.evalNode("plugins"))**

  解析配置文件中的plugins标签，获取所有的拦截器，通过反射创建实例对象保存在**interceptorChain**属性中

- **settingsElement(settings)**

  初始化mybatis默认配置，其中包括configuration.setDefaultEnumTypeHandler(...)设置了很多默认的类型处理器，保存在属性typeAliasRegistry的defaultEnumTypeHandler属性中。

- **mapperElement(root.evalNode("mappers"))**

  创建**XMLMapperBuilder**对象(同样继承BaseBuilder)来解析配置文件中的mappers标签，读取所有的mapper.xml文件，分为package,resource,url,class四种方式，最后把mapper.xml文件中的sql封装成MappedStatement对象，保存在mappedStatements属性中

# Configuration

> mybatis配置类解析出来的对象,包含整个项目关于mybatis的所有内容

# MappedStatement

> mapper文件解析出来的对象，包含了sql,参数，返回值类型等等,

- **SqlSource** ：获取sql的接口

  其实现类一般都包含属性SqlNode，在实际调用过程中，是使用装饰器模式层层解析节点，最后拼接出来要执行的sql，一般动态sql使用的实现类是**DynamicSqlSource**

执行流程:从MappedStatement的getBoundSql进入找到对应的SqlSource，以DynamicSqlSource为例，调用了SqlNode的apply(context)方法。sqlNode

