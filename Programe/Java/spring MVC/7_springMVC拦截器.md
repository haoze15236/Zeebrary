# 自定义拦截器

## 定义拦截器类实现HandlerInterceptor接口

```java
package mvc.interceptors;

import org.springframework.util.ObjectUtils;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor implements HandlerInterceptor {

	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		System.out.println("--------------------拦截器preHandle-------------------");
        //类似这种可以做登录权限拦截
		if(request.getSession().getAttribute("user")==null){
			response.sendRedirect(request.getContextPath()+"index.jsp");
			return false;
		}
		return true;
	}

	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
		System.out.println("--------------------拦截器postHandle-------------------");
        //此处使用modelAndView可以设置参数
		modelAndView.setViewName("model");
	}

	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
		System.out.println("--------------------拦截器afterCompletion-------------------");
        //捕获异常记录到日志
		if(!ObjectUtils.isEmpty(ex)){
			System.out.println(ex.getMessage());
		}
	}
}

```

## 添加容器配置

```xml
<mvc:interceptors>
    
    <!--全局拦截器,拦截所有controller的请求-->
    <bean class="mvc.interceptors.MyInterceptor" />
    
    <!--定制拦截器,拦截指定controller请求-->
    <mvc:interceptor>
        <!--拦截器映射请求-->
        <mvc:mapping path="/**"/>
        <!--拦截器排除请求-->
        <mvc:exclude-mapping path="/login"/>
        <bean class="com.ns.interceptors.CheckLoginInterceptor"></bean>
    </mvc:interceptor>
    
</mvc:interceptors>
```

## 拦截器执行顺序

先执行拦截器的preHandle方法----》执行目标方法----》执行拦截器的postHandle方法----》执行页面跳转----》执行拦截器的afterCompletion方法

多个拦截器执行顺序：

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/598C2461DF604566A60D02319D89BE62/D455889CE70D41B3821DFE1869A5E752/3628](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/598C2461DF604566A60D02319D89BE62/D455889CE70D41B3821DFE1869A5E752/3628)

拦截器的preHandle是按照顺序执行的

拦截器的postHandle是按照逆序执行的

拦截器的afterCompletion是按照逆序执行的

如果执行的时候核心的业务代码出问题了，那么已经通过的拦截器的afterCompletion会接着执行。

## **拦截器跟过滤器的区别**

1、过滤器是基于函数回调的，而拦截器是基于java反射的

2、过滤器依赖于servlet容器，而拦截器不依赖与Servlet容器，拦截器依赖SpringMVC

3、过滤器几乎对所有的请求都可以起作用，而拦截器只能对SpringMVC请求起作用。

