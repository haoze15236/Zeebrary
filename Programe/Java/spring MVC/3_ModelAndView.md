# BindingAwareModelMap

在controller接收到请求之后,经过一系列的业务处理,我们需要把数据返回前端,首先介绍spring Mvc中的的`BindingAwareModelMap`。

在controller层，我们可以使用Map，ModelMap，Model将数据传入request域：

```java
package mvc.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.Map;

@Controller
public class ModelAndViewController {

	@RequestMapping(value = {"/map"})
	public String map(Map map){
		System.out.println(map.getClass().getName());
		map.put("type","haoze");
		return "model";
	}

	@RequestMapping(value = {"/modelMap"})
	public String modelMap(ModelMap modelMap){
		System.out.println(modelMap.getClass().getName());
		modelMap.addAttribute("type","郝泽");
		modelMap.put("type","haoze");
		return "model";
	}

	@RequestMapping(value = {"/model"})
	public String modelMap(Model model){
		System.out.println(model.getClass().getName());
		model.addAttribute("type","郝泽");
		return "model";
	}
}
```

其底层都是使用的实现类`BindingAwareModelMap`，看UML图很容易明白。

![image-20210428232349046](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210428232349046.png)

# ModelAndView

看名字就知道，是model+view的结合，因此可以同时设置返回的视图和数据，使用方法很简单：

```java
@Controller
public class ModelAndViewController {
    @RequestMapping(value = {"/modelandview"})
	public ModelAndView modelandview(Model model){
		ModelAndView mv = new ModelAndView("model");
		mv.addObject("type","郝泽");
		return mv;
	}
}
```

其中view的名称会被视图解析器解析(在[ViewReslover](4_ViewReslover.md)章节会介绍视图解析器)，单体应用下，使用jsp可以看到效果，`web\WEB-INF\view\model.jsp`下代码如下:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${requestScope.type}
</body>
</html>

```

## **@ModelAttribute**

**用在方法上:**@ModelAttribute的方法会在当前处理器中所有的处理方法之前调用。因此可以用于初始化设置一些公共数据到model中。

**用在参数上:**可以从model中获取一个指定的属性和参数进行合并。

```java
@Controller
public class ModelAndViewController {
    @RequestMapping(value = {"/session"})
	public ModelAndView session(@ModelAttribute("type") String type){
        //获取model中某个属性的值
		ModelAndView mv = new ModelAndView("model");
		System.out.println("type="+type);
		return mv;
	}

	@ModelAttribute
	public void modelAttribute(Model model){
        //设置属性到model中
		model.addAttribute("type","郝泽modelandview");
	}
}
```

