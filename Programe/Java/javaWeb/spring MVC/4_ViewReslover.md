# InternalResourceViewResolver

springMvc默认视图解析器,如果想使用的话,需要注册到spring容器中,即配置到`spring-mvc.xml`中即可

```xml
 <!--定义视图解析器,设置前缀和后缀-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" name="viewResolver">
        <!--定义视图前缀,就可以不写文件路径名,web文件夹为根目录-->
        <property name="prefix" value="/WEB-INF/view/"></property>
        <!--定义视图后缀,就可以不写文件后缀名-->
        <property name="suffix" value=".jsp"></property>
    </bean>
```

按如上配置好之后,当我们的controller返回一个视图名称，视图解析器会自动帮我们添加前缀和后缀，并进行请求转发，解析视图

```java
package mvc.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class ViewResolverController {

	@RequestMapping(value = {"/helloResolver"})
	public String helloResolver(){
		return "index";
	}
}
```

如果我们有某个视图想可以直接访问，还可以在`spring-mvc.xml`中配置视图控制器

```java
<mvc:view-controller path="/" view-name="index"></mvc:view-controller>
```

此时，当我们访问项目根目录,会自动转发到index，并通过视图解析器添加前缀，后缀,即访问到了`/WEB-INF/view/index.jsp`页面