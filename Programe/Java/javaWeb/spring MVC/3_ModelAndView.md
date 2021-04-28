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

其中view的名称会被视图解析器解析，单体应用下，使用jsp可以看到效果，`web\WEB-INF\view\model.jsp`下代码如下:

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

