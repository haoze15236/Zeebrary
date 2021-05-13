# http请求相关注解

> 此部分是按照[10_web基础](../10_web基础/http请求模型.md)中request请求来逐个讲解平常开发中会用到的注解,其他注解后面涉及到再说。

## 请求行(url)

### @RequestMapping

@RequestMapping用来匹配客户端发送的请求，可以在方法上使用，也可以在类上使用。

- 方法：表示用来匹配要处理的请求行。
- 类上：表示为当前类的所有方法的请求地址添加一个前置路径，访问的时候必须要添加此路径。

注意：在整个项目的不同方法上不能包含相同的@RequestMapping值。

```java
@Controller
public class HelloSpringMvc {
@RequestMapping(value = {"/requestMapping"}
	,method = {RequestMethod.POST,RequestMethod.GET}
	,params = {"name"}
	,headers = {"haoze"}
	,consumes = {MediaType.TEXT_PLAIN_VALUE}
	,produces= {MediaType.TEXT_PLAIN_VALUE})
	@ResponseBody
	public String requestMapping(String name){
		System.out.println("hello springmvc:"+name);
		return "郝泽";
	}
}
```

- **value** :映射的url路径,支持模糊匹配：

  - ？：能替代任意一个字符
  - *: 能替代任意多个字符和一层路径
  - **：能代替多层路径

  同时可以使用{}占位来支持restful风格请求，与`@PathVariable`一起使用

- **method** : 映射请求方法，可选值可参考`RequestMethod`定义的枚举

- **params** ：映射请求参数

  - = ： `name = 1` 表示请求中必须要有name参数且值必须等于1
  - != :  `name != 1` 表示请求中name参数不能不等于1，可以没有name参数
  - 参数 ： `name` 表示请求中必须要有name参数,不限制值
  - !参数：`!name` 表示请求中必须不包含name参数

- **headers** ：映射请求头,用法跟params一样，有一点不同的是仅当`content-type`作为参数时，其值支持*模糊匹配

  - content-type=text/*  ：匹配所有的text/开头的参数值

- **consumes** ：映射请求数据格式:即限制请求头中`content-type`的值，支持*模糊匹配
- **produces** ：映射返回数据格式,即限制请求头中`Accept`的值，支持*模糊匹配，意思是本接口只返回此处定义的格式的数据，你能处理的数据格式匹配不上，就不处理你的请求。

在使用reseful风格的后端接口时,springmvc提供了更细分的Mapping注解

| 注解           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| @GetMapping    | 只接受Get请求的映射，其他同@RequestMapping,一般用于查询数据  |
| @PostMapping   | 只接受Post请求的映射，其他同@RequestMapping,一般用于新增数据 |
| @PutMapping    | 只接受Put请求的映射，其他同@RequestMapping,一般用于修改数据  |
| @DeleteMapping | 只接受delete请求的映射，其他同@RequestMapping,一般用于删除数据 |

## 请求头

### @RequestHeader

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

### @CookieValue              

获取cookie中的某个属性,使用方式同@RequestHeader

```java
@RequestMapping("/params")
public String paramsRequest(@CookieValue Map cookie){
    System.out.println(cookie);
    return cookie.toString();
}
```

### @SessionAttribute&@SessionAttributes

@SessionAttributes是用在类上面的，加上之后,此controller里面所有model方式(`Map`,`ModelMap`,`Model`,`ModelAndView`)写入的同名属性，都会同步**写入**session：

```java
@Controller
// 通过model中对应的属性去写入到session,同时也会从session中写入指定的属性到model,
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

@SessionAttribute用在参数上面的，用来**读取**session中对应的值，默认指定的属性是必须要存在的，如果不存在则会报错，可以设置required =false 不需要必须存在，不存在默认绑定null。

```java
@RequestMapping("/getSession")
public String getSession(@SessionAttribute(value="type",required = false) String type){
    System.out.println(type);
    return "main";
}
```

由于`@SessionAttributes`会影响controller下所有接口的model行为，有时需要更细粒度的控制的话，还是得采用耦合servlet的方式，首先添加servlet依赖

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

然后在controller上通过自动注入的方式来操作session:

```java
@Controller
//@SessionAttributes({"type","scope"})
public class SessionController {

	@Autowired
	private HttpSession httpSession;
	
	@RequestMapping(value = {"/setSession"})
	public ModelAndView modelandview(Model model){
		ModelAndView mv = new ModelAndView("model");
        //手动设置session
		httpSession.setAttribute("scope","modelandview");
		mv.addObject("type","郝泽");
		return mv;
	}
    
    @RequestMapping(value = {"/getSession"})
	public ModelAndView session(@SessionAttribute("scope")String scope){
        //此处还是可以使用@SessionAttribute获取到session
		ModelAndView mv = new ModelAndView("model");
		System.out.println(scope);
		return mv;
	}
}
```

这种方式下,session的数据跟model的数据不会一致。spring mvc底层自动注入`HttpSession`时使用的ThreadLocal来存储不同请求的session值，因此并不会有线程安全问题。

## 请求数据

### @RequestParam

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

### 接收复杂对象类型参数

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

### @PathVariable

用于rest风格的参数获取，

```java
@RequestMapping("/user/{id}/{username}")
public String path01(@PathVariable("id") Integer id,@PathVariable("username") String name){
    System.out.println(id);
    System.out.println(name);
    return "/index.jsp";
}
```

若是对象，{属性名}与对象属性名对应上则会自动匹配



