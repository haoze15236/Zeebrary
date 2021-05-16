# AOP

AOP：Aspect Oriented Programming  面向切面编程

OOP：Object Oriented Programming  面向对象编程

​		面向切面编程：基于OOP基础之上新的编程思想，OOP面向的主要对象是类，而AOP面向的主要对象是切面，在处理日志、安全管理、事务管理等方面有非常重要的作用。AOP是Spring中重要的核心点，虽然IOC容器没有依赖AOP，但是AOP提供了非常强大的功能，用来对IOC做补充。通俗点说的话就是在程序运行期间。 

**在不修改原有代码的情况下** 增强**跟主要业务没有关系的公共功能代码**到  **之前写好的方法中的指定位置** 这种编程的方式叫AOP

AOP的底层用的代理，代理是一种设计模式

# 静态代理

运用多态特性，为每一个被代理的类创建一个“代理类”，两者实现同一个接口，代理类通过持有被代理类的引用。如图:

![image-20210213004412453](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210213004412453.png)

那么在proxy在实现接口方法(比如operation1)时，通过引用调用被代理类target的方法(operation1),然后就可以在之前或者之后做增强逻辑。

# 动态代理

AOP的底层是用的动态,动态代理分为2种。

## jdk动态代理

通过Proxy.newProxyInstance(ClassLoader loader,<?>[] interfaces,InvocationHandler h)获取代理对象,可以看到需要三个参数：

- ClassLoader:被代理类的类加载器
- Class<?>[]:被代理类接口的Class对象
- InvocationHandler:增强处理器,通过实现此接口的invoke方法，来实现具体的增强功能。

```java
package com.aop;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
/**
 * JDK动态代理工具类
 */
public class MyProxyUtil {
    /**
     * 获取代理对象静态方法
     * @param target
     * @return 返回代理对象
     */
    public static Object getProxy(Object target){
        ClassLoader classLoader = target.getClass().getClassLoader();
        Class<?>[] interfaces = target.getClass().getInterfaces();//需要这个参数决定了被代理类必须实现接口
        InvocationHandler invocationHandler = new InvocationHandler() {
            /*
             **
             *  执行目标方法
             * @param proxy 代理对象，给jdk使用，任何时候都不需要操作此对象
             * @param method 当前将要执行的目标对象的方法
             * @param args 这个方法调用时外界传入的参数值
             * @return
             * @throws Throwable
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("开始执行目标方法:"+method.getName());//前置增强
                //target:被代理类对象
                Object result = method.invoke(target, args);
                System.out.println("目标方法执行完毕结果为:"+result);//后置增强
                return result;
            }
        };
        return Proxy.newProxyInstance(classLoader,interfaces,invocationHandler);//返回代理对象
    }
}
```

调用工具类动态获取代理对象，由于返回的代理对象是被代理类的接口对象,所以请注意<span style="color:#ff11f5">只能转成接口才可以调用实现类方法</span>

```java
package com.aop;
import org.junit.Test;

public class jdkProxyTest {
    @Test
    public void test(){
        IGameCaculate gameCaculate = (IGameCaculate)MyProxyUtil.getProxy(new GameCaculate());
        gameCaculate.add(1,1);
    }
}
```

上面的例子可以看到，jdk动态代理还存在缺点：

- 被代理类必须实现接口，只用通过接口才可以调用方法

## cglib动态代理

解决了jdk动态代理的弊端，无需被代理类实现接口

通过Enhancer.create()获取代理对象

```java
package com.aop;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * cjlib动态代理工具类
 */
public class MyCjlibProxyUtil {

    public static Object getProxy(Object target){
        Enhancer enhancer = new Enhancer();
        //设置代理类的父类为代理类
        enhancer.setSuperclass(target.getClass());
        //设置增强方法
        enhancer.setCallback(new MethodInterceptor(){
            /**
             * o：cglib生成的代理对象
             * method：被代理对象方法
             * objects：方法入参
             * methodProxy: 代理方法
             */
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                System.out.println("开始执行目标方法:"+method.getName());
                Object result = methodProxy.invokeSuper(o, objects);
                System.out.println("目标方法执行完毕结果为:"+result);
                return result;
            }
        });
        //返回代理对象
        return enhancer.create();
    }
}

```

调用cjlib动态代理工具类获取代理类，做功能增强：

```java
package com.aop;
import org.junit.Test;

public class proxyTest {
    @Test
    public void cjlibProxyTest(){
        GameCaculate gameCaculate = (GameCaculate) MyCjlibProxyUtil.getProxy(new GameCaculate());
        gameCaculate.add(1,2);
    }
}
```

此时无需要求被代理类实现接口。

# Spring AOP

了解了上述代理的实现方式之后，大家应该可以看到3种代理方式的优劣,他们还有一个共同的缺点就是为了使用代理增强原业务逻辑,我们在调用业务逻辑时，都需要通过代理类对象来操作，而spring AOP就提供了更轻便的方式来帮助我们快速开发，只需要使用注解来声明增强方法，就可以快速帮助我们做到通过代理切面编程。

在开始之前我们还需要了解spring AOP中的几个基本概念：

## **AOP的核心概念及术语**

- 切面（Aspect）: 指关注点模块化，这个关注点可能会横切多个对象。事务管理是企业级Java应用中有关横切关注点的例子。 在Spring AOP中，切面可以使用通用类基于模式的方式（schema-based approach）或者在普通类中以@Aspect注解（@AspectJ 注解方式）来实现。
- 连接点（Join point）: 在程序执行过程中某个特定的点，例如某个方法调用的时间点或者处理异常的时间点。在Spring AOP中，一个连接点总是代表一个方法的执行。
- 通知（Advice）: 在切面的某个特定的连接点上执行的动作。通知有多种类型，包括“around”, “before” and “after”等等。通知的类型将在后面的章节进行讨论。 许多AOP框架，包括Spring在内，都是以拦截器做通知模型的，并维护着一个以连接点为中心的拦截器链。
- 切点（Pointcut）: 匹配连接点的断言。通知和切点表达式相关联，并在满足这个切点的连接点上运行（例如，当执行某个特定名称的方法时）。切点表达式如何和连接点匹配是AOP的核心：Spring默认使用AspectJ切点语义。
- 引入（Introduction）: 声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被通知的对象上。例如，可以使用引入来使bean实现 IsModified接口， 以便简化缓存机制（在AspectJ社区，引入也被称为内部类型声明（inter））。
- 目标对象（Target object）: 被一个或者多个切面所通知的对象。也被称作被通知（advised）对象。既然Spring AOP是通过运行时代理实现的，那么这个对象永远是一个被代理（proxied）的对象。
- AOP代理（AOP proxy）:AOP框架创建的对象，用来实现切面契约（aspect contract）（包括通知方法执行等功能）。在Spring中，AOP代理可以是JDK动态代理或CGLIB代理。
- 织入（Weaving）: 把切面连接到其它的应用程序类型或者对象上，并创建一个被被通知的对象的过程。这个过程可以在编译时（例如使用AspectJ编译器）、类加载时或运行时中完成。 Spring和其他纯Java AOP框架一样，是在运行时完成织入的。

**AOP的通知类型**

- 前置通知（Before advice）: 在连接点之前运行但无法阻止执行流程进入连接点的通知（除非它引发异常）。
- 后置返回通知（After returning advice）:在连接点正常完成后执行的通知（例如，当方法没有抛出任何异常并正常返回时）。
- 后置异常通知（After throwing advice）: 在方法抛出异常退出时执行的通知。
- 后置通知（总会执行）（After (finally) advice）: 当连接点退出的时候执行的通知（无论是正常返回还是异常退出）。
- 环绕通知（Around Advice）:环绕连接点的通知，例如方法调用。这是最强大的一种通知类型，。环绕通知可以在方法调用前后完成自定义的行为。它可以选择是否继续执行连接点或直接返回自定义的返回值又或抛出异常将执行结束。

## 搭建spring AOP

### 引入依赖

在项目的pom.xml文件中加入依赖：

```xml
<!--spring aop依赖-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.2.3.RELEASE</version>
        </dependency>
```

### spring容器开启AOP功能

#### xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--设置包扫描-->
    <context:component-scan base-package="com.aop"></context:component-scan>
    <!--开启aop功能-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

#### javaconfig配置

在配置类上使用@EnableAspectJAutoProxy开启aop功能，@ComponentScan设置包扫描在之前IOC章节已经学习过

```java
package com.javaConfig;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.*;


@Configuration
@ComponentScan(basePackages = {"com.javaConfig","com.aop"})
@EnableAspectJAutoProxy
public class AppConfig {
}
```

上述两种方式都可以设置两个属性：

- proxyTargetClass：默认为false，走jdk动态代理,此时需要注意<span style="color:#ff11f5">当我们通过spring容器获取被代理对象时，必须使用接口接收</span>，

当被代理类没有实现接口时，走cjlib动态代理，设置为true时强制走cglib动态代理

- exposeProxy：是否暴露代理对象，默认为false,当设置为true时，可以通过AopContext.currentProxy()在被代理对象中获取实际程序运行时创建的代理对象

### 申明切面

#### 切点表达式

##### 格式

<span style="color:#ff11f5">切点标识符</span>(<span style="color:#ff5511">访问修饰符</span> <span style="color:#1111f5">方法返回值</span> <span style="color:#2ff11f">包名</span>.<span style="color:#e3551f">类名</span>.<span style="color:#ff11f5">方法名</span>(<span style="color:#1411f5">参数类型</span>))

示例：![https://note.youdao.com/yws/public/resource/9ea06d3b1e9532e871342b8b7e7d9b0c/xmlnote/A668683C87AF489C88FA528F3552DB44/4C219A04474E44FC8F989840F41D917C/1752](https://note.youdao.com/yws/public/resource/9ea06d3b1e9532e871342b8b7e7d9b0c/xmlnote/A668683C87AF489C88FA528F3552DB44/4C219A04474E44FC8F989840F41D917C/1752)

**访问修饰符**:可以不写，可以匹配任意一个访问修饰符

**方法返回值**:可以使用通配符`*`来匹配所有的返回值类型

**包名**:可以使用使用通配符`.`匹配所有包路径

**类名**：可以使用通配符`*`来匹配所有类名

**方法名**:可以使用通配符`*`来匹配所有方法

**参数类型**:可以使用通配符`..`匹配所有参数类型

根据以上规则，最简写法如：`execution(* com.aop..*.*(..))` 代表匹配com.aop下所有子包的所有类的所有方法

- 切点表示符
  - **execution**: 用于匹配方法执行连接点。 这是使用Spring AOP时使用的主要切点标识符。  可以匹配到方法级别 ，细粒度
  - **within**: 只能匹配类这级，只能指定类， 类下面的某个具体的方法无法指定， 粗粒度
  - **this**:  匹配实现了某个接口
  - **target**: 限制匹配到连接点（使用Spring AOP时方法的执行），其中目标对象（正在代理的应用程序对象）是给定类型的实例。
  - **args**: 限制与连接点的匹配（使用Spring AOP时方法的执行），其中变量是给定类型的实例。
  - **@target**: 限制与连接点的匹配（使用Spring AOP时方法的执行），其中执行对象的类具有给定类型的注解。
  - **@args**: 限制匹配连接点（使用Spring AOP时方法的执行），其中传递的实际参数的运行时类型具有给定类型的注解。
  - **@within**: 限制与具有给定注解的类型中的连接点匹配（使用Spring AOP时在具有给定注解的类型中声明的方法的执行）。
  - **@annotation**:限制匹配连接点（在Spring AOP中执行的方法具有给定的注解）。

##### **合并切点表达式**

您可以使用 &&, || 和 !等符号进行合并操作。也可以通过名字来指向切点表达式,其他切点表示符同理

```java
//&&：两个表达式同时
execution( public int com.aop.inter.MyCalculator.*(..)) && execution(* *.*(int,int) )
//||：任意满足一个表达式即可
execution( public int com.aop.inter.MyCalculator.*(..)) || execution(* *.*(int,int) )
//！：只要不是这个位置都可以进行切入
!execution( public int com.aop.inter.MyCalculator.*(..))
```

#### 注解申明切面

切面类必须也在spring容器中注册成一个bean

```java
package com.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.Arrays;

@Component
@Aspect
public class LogUtil {
	//抽取相同的切点表达式,注册一个切点
    @Pointcut("execution(* com.aop..*.*(..))")
    public void logPointcut(){
    }
    /*
  设置下面方法在什么时候运行
      @Before:在目标方法之前运行：前置通知
      @After:在目标方法之后运行：后置通知
      @AfterReturning:在目标方法正常返回之后：返回通知
      @AfterThrowing:在目标方法抛出异常后开始运行：异常通知
      @Around:环绕：环绕通知

      当编写完注解之后还需要设置在哪些方法上执行，使用表达式
      execution(访问修饰符  返回值类型 方法全称)
   */
    // 前置通知
    @Before("execution(* com.aop..*.*(..))")
    //或者使用抽取出来的切点
    //@Before("logPointcut()")
    public static void start(JoinPoint joinPoint){
        //获取方法参数
        Object[] args = joinPoint.getArgs();
        //获取方法名
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法开始执行，参数是："+ Arrays.asList(args));
    }

    // 后置通知
    @After("execution(* com.aop..*.*(..))")
    public static void end(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法执行结束了......");
    }

    // 后置异常通知
    @AfterThrowing(value ="execution(* com.aop..*.*(..))",throwing = "exception")
    public static void afterException(JoinPoint joinPoint, Exception exception){
        String name = joinPoint.getSignature().getName();
        StringWriter sw = new StringWriter();
        exception.printStackTrace(new PrintWriter(sw,true));
        System.out.println(name+"方法出现异常："+sw.getBuffer().toString());
    }

    // 后置返回通知
    @AfterReturning(value = "execution(* com.aop..*.*(..))",returning = "result")
    public static void stop(JoinPoint joinPoint,Object result){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法执行完成，结果是："+result);

    }
    /*
    环绕通知
    */
    @Around("logPointcut()")
    public Object myAround(ProceedingJoinPoint proceedingJoinPoint){
        Object[] args = proceedingJoinPoint.getArgs();
        String name = proceedingJoinPoint.getSignature().getName();
        Object proceed = null;
        try {
            System.out.println("环绕前置通知:"+name+"方法开始，参数是"+Arrays.asList(args));
            //利用反射调用目标方法，就是method.invoke()
            proceed = proceedingJoinPoint.proceed(args);
            System.out.println("环绕返回通知:"+name+"方法返回，返回值是"+proceed);
        } catch (Throwable e) {
            System.out.println("环绕异常通知"+name+"方法出现异常，异常信息是："+e);
        }finally {
            System.out.println("环绕后置通知"+name+"方法结束");
        }
        return proceed;
    }
}

```

其中@annotation切点表达式还可以这样指定：

```java
@Component
@Aspect
public class LogUtil {

    //直接把注解设定为切面方法的参数,@annotation时传入注解形参logger
    @Before("@annotation(logger)")
    public static void start(JoinPoint joinPoint, Logger logger){
        //获取方法参数
        Object[] args = joinPoint.getArgs();
        //获取方法名
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法开始执行，参数是："+ Arrays.asList(args)+"使用日志注解时设定的名称为"+logger.getName());
    }
}
```

测试AOP效果

```java
package com.aop;
import org.junit.Test;
public class proxyTest {
    private AnnotationConfigApplicationContext ioc;
    @Before
    public void before(){
        ioc = new AnnotationConfigApplicationContext(AppConfig.class);
    }
    @Test
    public void springAopTest(){
        //默认走jdk动态代理时,必须使用接口IGameCaculate.class类获取bean对象,否则会出现代理对象无法强转的错误
        IGameCaculate gameCaculate = (IGameCaculate)ioc.getBean("gameCaculate", IGameCaculate.class);
        System.out.println(gameCaculate.add(1,2));
    }
}
```



#### xml申明切面（基于schema方式）

```xml
<!--定义切面bean-->
<bean id="logUtil" class="com.aop.LogUtil"></bean>
<aop:config>
    <!--申明全局切点表达式-->
    <aop:pointcut id="globalPoint" expression="execution(* com.aop..*.*(..))"/>
    <!--申明bean为切面-->
    <aop:aspect ref="logUtil">
        <!--申明局部切点表达式-->
        <aop:pointcut id="mypoint" expression="execution(* com.aop..*.*(..))"/>
        <!--申明通知-->
        <aop:before method="start" pointcut="@annotation(logger)" arg-names="logger"></aop:before>
        <aop:after method="end" pointcut-ref="mypoint"></aop:after>
        <aop:after-returning method="stop" pointcut-ref="mypoint"  returning="result"></aop:after-returning>
        <aop:after-throwing method="afterException" pointcut-ref="mypoint" throwing="exception"></aop:after-throwing>
        <aop:around method="myAround" pointcut-ref="mypoint"></aop:around>
    </aop:aspect>
</aop:config>
```



### <span style="color:#ff0000">通知执行顺序</span>

```java
//版本大于spring 5.2.7-------After后置通知最后执行,此时出现异常,不会被AfterThrowing通知捕获
环绕前置通知:divide方法开始，参数是[1, 1]
Before前置通知:方法开始执行，参数是：[1, 1]
AfterReturning后置返回通知:方法执行完成，结果是：1
After后置通知:方法执行结束了......
环绕返回通知:divide方法返回，返回值是1
环绕后置通知divide方法结束
//版本小于spring 5.2.7-------After后置通知的如果抛出异常,会被AfterThrowing通知捕获
环绕前置通知:divide方法开始，参数是[1, 1]
Before前置通知:方法开始执行，参数是：[1, 1]
环绕返回通知:divide方法返回，返回值是1
环绕后置通知divide方法结束
After后置通知:方法执行结束了......
AfterReturning后置返回通知:方法执行完成，结果是：1
//疑问点：当我通过xml配置aop时,测试spring5.3.3和spring 5.2.6结果都为
Before前置通知:方法开始执行，参数是：[2, 1]日志名称为：除法
环绕前置通知:divide方法开始，参数是[2, 1]
环绕返回通知:divide方法返回，返回值是2
环绕后置通知divide方法结束
AfterReturning后置返回通知:方法执行完成，结果是：2
After后置通知:方法执行结束了......
```

