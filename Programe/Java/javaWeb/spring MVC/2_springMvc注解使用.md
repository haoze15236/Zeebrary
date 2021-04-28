# http请求相关注解

## @RequestParam

获取http请求中的键值对参数，如get/post请求中url后面的参数,或者post使用x-www-form-urlencoded格式的请求体中键值对参数。

```java
@Controller
public class HelloSpringMvc {
	@RequestMapping("/hello")
	@ResponseBody
	public String hello(@RequestParam("userName") String name){
		System.out.println("hello springmvc:"+name);
		System.out.println("hello springmvc:"+header);
		return "hello";
	}
}
```

查看此注解源码如下:

```java
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam {
    @AliasFor("name")
    String value() default "";//可定义参数名称,跟请求中的参数名一致即可获取到参数

    @AliasFor("value")
    String name() default "";//可定义参数名称,跟请求中的参数名一致即可获取到参数

    boolean required() default true;//参数是否必输,如必输，但是请求没有传则会抛出400的错误

    String defaultValue() default "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n";
}
```

## @RequestHeader

获取请求头的参数

```java
@Controller
public class HelloSpringMvc {
	@RequestMapping("/hello")
	@ResponseBody
	public String hello(@RequestHeader Map header,@RequestHeader("agent") String agent){
        //@RequestHeader Map header 			获取请求头内的所有请求头参数
        //@RequestHeader("agent") String agent  获取请求头内中agent的参数值
		System.out.println("hello springmvc:"+name);
		System.out.println("hello springmvc:"+header);
		return "hello";
	}
}
```

## @CookieValue              

获取cookie中的某个属性,使用方式同@RequestHeader

```java
@RequestMapping("/params")
public String paramsRequest(@CookieValue Map cookie){
    System.out.println(cookie);
    return cookie.toString();
}
```

## @PathVariable（restful参数）

```java
@RequestMapping("/user/{id}/{username}")
public String path01(@PathVariable("id") Integer id,@PathVariable("username") String name){
    System.out.println(id);
    System.out.println(name);
    return "/index.jsp";
}
```

若是对象，{属性名}与对象属性名对应上则会自动匹配

## @SessionAttribute

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

## **@ModelAttribute**

**用在方法上:**@ModelAttribute的方法会在当前处理器中所有的处理方法之前调用

**用在参数上:**可以省略，加上则会从model中获取一个指定的属性和参数进行合并，因为model和sessionAttribute具有共通的特性，所以如果session中有对应的属性也会进行合并

注意:若通过@ModelAttribute来设置**单例,类级别的变量**存在线程安全问题。

## @RequestMapping

@RequestMapping用来匹配客户端发送的请求，可以在方法上使用，也可以在类上使用。

- 方法：表示用来匹配要处理的请求。
- 类上：表示为当前类的所有方法的请求地址添加一个前置路径，访问的时候必须要添加此路径。

注意：在整个项目的不同方法上不能包含相同的@RequestMapping值。

```java
/**
* @RequestMapping包含三种模糊匹配的方式，分别是：
* ？：能替代任意一个字符
* *: 能替代任意多个字符和一层路径
* **：能代替多层路径
*/
 @RequestMapping(value = "/**/h*llo?")
 public String hello5(){}
```



# 拓展

1. springmvc 控制器是不是单例的？如果是单例的会出现什么问题？怎么解决？

spring MVC依赖spring容器，默认是单例的，单例类若存在类级别变量可能存在线程安全问题,可以通过

- 声明成方法级别变量
- 使用ThreadLocal存储变量



# 接收复杂对象类型参数

spring mvc中的controller默认只能自动解析填充表单提交的http请求里面的数据到复杂对象类型

```java
package mvc.dto;

import java.util.Date;
import java.util.List;
import java.util.Map;

public class Student {
	private String name;
	private Integer age;
	private Date date;
	private Map<String,String> habbits;
	private List<Role> role;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Integer getAge() {
		return age;
	}

	public void setAge(Integer age) {
		this.age = age;
	}

	public Date getDate() {
		return date;
	}

	public void setDate(Date date) {
		this.date = date;
	}

	public Map<String, String> getHabbits() {
		return habbits;
	}

	public void setHabbits(Map<String, String> habbits) {
		this.habbits = habbits;
	}

	public List<Role> getRole() {
		return role;
	}

	public void setRole(List<Role> role) {
		this.role = role;
	}

	@Override
	public String toString() {
		return "Student{" +
				"name='" + name + '\'' +
				", age=" + age +
				", date=" + date +
				", habbits=" + habbits +
				", role=" + role +
				'}';
	}
}

```

```java
package mvc.dto;

public class Role {
	private String name;
	private String roleName;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getRoleName() {
		return roleName;
	}

	public void setRoleName(String roleName) {
		this.roleName = roleName;
	}

	@Override
	public String toString() {
		return "Role{" +
				"roleName='" + roleName + '\'' +
				'}';
	}
}


```

controller如下

```java
package mvc.controller;
import mvc.dto.Role;
import mvc.dto.Student;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import java.util.Map;
@Controller
public class HelloSpringMvc {
	/**
	 * 测试复杂类型参数自动装配
	 * @param student
	 * @return
	 */
	@RequestMapping("/Student")
	@ResponseBody
	public String Student(Student student, Role role){
		System.out.println("hello springmvc:"+student);
		System.out.println("hello springmvc:"+role);
		return "郝泽";
	}
}
```

使用postman调用测试:

![image-20210427235141780](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210427235141780.png)

