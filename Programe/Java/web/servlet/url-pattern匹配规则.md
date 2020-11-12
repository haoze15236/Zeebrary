# servlet和filter的url-pattern匹配规则详细描述

首先需要明确几容易混淆的规则：

1. servlet容器中的匹配规则既不是简单的通配，也不是正则表达式，而是特定的规则。所以不要用通配符或者正则表达式的匹配规则来看待servlet的url-pattern。
2. Servlet 2.5开始，一个servlet可以使用多个url-pattern规则，标签声明了与该servlet相应的匹配规则，每个标签代表1个匹配规则；
3. 当servlet容器接收到浏览器发起的一个url请求后，容器会用url减去当前应用的上下文路径，以剩余的字符串作为servlet映射，假如url是http://localhost:8080/appDemo/index.html，其应用上下文是appDemo，容器会将http://localhost:8080/appDemo去掉，用剩下的/index.html部分拿来做servlet的映射匹配
4. url-pattern映射匹配过程是有优先顺序的
5. 而且当有一个servlet匹配成功以后，就不会去理会剩下的servlet了。

### 四种匹配规则

#### 1 精确匹配

  中配置的项必须与url完全精确匹配。

```
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/user/users.html</url-pattern>
    <url-pattern>/index.html</url-pattern>
    <url-pattern>/user/addUser.action</url-pattern>
</servlet-mapping>
复制代码
```

当在浏览器中输入如下几种url时，都会被匹配到该servlet  http://localhost:8080/appDemo/user/users.html  http://localhost:8080/appDemo/index.html  http://localhost:8080/appDemo/user/addUser.action

  

  注意：

  http://localhost:8080/appDemo/user/addUser/ 是非法的url，不会被当作http://localhost:8080/appDemo/user/addUser识别

  另外上述url后面可以跟任意的查询条件，都会被匹配，如

  http://localhost:8080/appDemo/user/addUser?username=Tom&age=23 会被匹配到MyServlet。

#### 2 路径匹配

以“/”字符开头，并以“/*”结尾的字符串用于路径匹配

```
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/user/*</url-pattern>
</servlet-mapping>

复制代码
```

路径以/user/开始，后面的路径可以任意。比如下面的url都会被匹配。  http://localhost:8080/appDemo/user/users.html  http://localhost:8080/appDemo/user/addUser.action  http://localhost:8080/appDemo/user/updateUser.actionl

#### 3 扩展名匹配

以“*.”开头的字符串被用于扩展名匹配

```
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>*.jsp</url-pattern>
    <url-pattern>*.action</url-pattern>
</servlet-mapping>

复制代码
```

则任何扩展名为jsp或action的url请求都会匹配，比如下面的url都会被匹配  http://localhost:8080/appDemo/user/users.jsp  http://localhost:8080/appDemo/toHome.action

#### 4 缺省匹配

```
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

复制代码
```

### 二、匹配顺序

1. 精确匹配，servlet-mapping1：/user/users.html，servlet-mapping2：/*。当一个请求http://localhost:8080/appDemo/user/users.html来的时候，servlet-mapping1匹配到，不再用servlet-mapping2匹配
2. 路径匹配，先最长路径匹配，再最短路径匹配servlet-mapping1：/user/*，servlet-mapping2：/*。当一个请求http://localhost:8080/appDemo/user/users.html来的时候，servlet-mapping1匹配到，不再用servlet-mapping2匹配
3. 扩展名匹配，servlet-mapping1：/user/*，servlet-mapping2：*.action。当一个请求http://localhost:8080/appDemo/user/addUser.action来的时候，servlet-mapping1匹配到，不再用servlet-mapping2匹配
4. 缺省匹配，以上都找不到servlet，就用默认的servlet，配置为/

### 三、需要注意的问题

#### 1 路径匹配和扩展名匹配无法同时设置

匹配方法只有三种，要么是路径匹配（以“/”字符开头，并以“/*”结尾），要么是扩展名匹配（以“*.”开头），要么是精确匹配，三种匹配方法不能进行组合，不要想当然使用通配符或正则规则。

  如/user/*.action是非法的

  另外注意：/aa/*/bb**是精确匹配，合法，这里的*不是通配的含义

#### 2 "/*"和"/"含义并不相同

1. “/*”属于路径匹配，并且可以匹配所有request，由于路径匹配的优先级仅次于精确匹配，所以“/*”会覆盖所有的扩展名匹配，很多404错误均由此引起，所以这是一种特别恶劣的匹配模式，一般只用于filter的url-pattern
2. “/”是servlet中特殊的匹配模式，切该模式有且仅有一个实例，优先级最低，不会覆盖其他任何url-pattern，只是会替换servlet容器的内建default servlet ，该模式同样会匹配所有request。
3. 配置“/”后，一种可能的现象是myServlet会拦截诸如http://localhost:8080/appDemo/user/addUser.action、http://localhost:8080/appDemo/user/updateUser的格式的请求，但是并不会拦截http://localhost:8080/appDemo/user/users.jsp、http://localhost:8080/appDemo/index.jsp，这是应为servlet容器有内置的“*.jsp”匹配器，而扩展名匹配的优先级高于缺省匹配，所以才会有上述现象。

Tomcat在%CATALINA_HOME%\conf\web.xml文件中配置了默认的Servlet，配置代码如下

```
<servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
<servlet>
        <servlet-name>jsp</servlet-name>
        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
        <init-param>
            <param-name>fork</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>xpoweredBy</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>3</load-on-startup>
 </servlet>
<servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
</servlet-mapping>

    <!-- The mappings for the JSP servlet -->
<servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
</servlet-mapping>
复制代码
```

1. 可以阅读http://stackoverflow.com/questions/4140448/difference-between-and-in-servlet-mapping-url-pattern
2. “/*”和“/”均会拦截静态资源的加载，需要特别注意

### 四、举例



![img](https://user-gold-cdn.xitu.io/2018/5/10/163482dc7430fc7b?imageView2/0/w/1280/h/960/ignore-error/1)



### filter

注意Filter的匹配规则与servlet一样，但对于filter，不会像servlet那样只匹配一个servlet，因为filter的集合是一个链，所以只会有处理的顺序不同，而不会出现只选择一个filter。Filter的处理顺序和filter-mapping在web.xml中定义的顺序相同。