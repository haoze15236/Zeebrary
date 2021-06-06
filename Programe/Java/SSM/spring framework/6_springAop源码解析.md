# AOP功能如何加入到IOC容器?

学习spring的时候，都学过Spring的AOP功能，在XML配置文件中配置或者通过注解启用AOP，那么AOP功能是何时在IOC容器中生效的呢？

## 注册核心类

在spring基于javaconfig(注解)的方式，我们是在配置类上使用@EnableAspectJAutoProxy注解开启AOP功能，点开这个注解可以看到：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
	boolean proxyTargetClass() default false;
	boolean exposeProxy() default false;
}
```

其实内部是使用@Import注解导入了AspectJAutoProxyRegistrar类，而这个类实现了ImportBeanDefinitionRegistrar接口。

前面IOC源码解析的时候，我们了解到，在配置类上使用@Import注解的话，会在ConfigurationClassPostProcessor调用BeanDefinitionRegistryPostProcessor接口的processConfigBeanDefinitions方法实现扫描配置类时，回调ImportBeanDefinitionRegistrar接口的registerBeanDefinitions方法，我们看下这个类的实现：

```java
@Override
public void registerBeanDefinitions(
		AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	//在这里面注册了AnnotationAwareAspectJAutoProxyCreator类的bean定义到bean工厂
	AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
	//.......
	}

```

然后在IOC容器refresh方法中的`registerBeanPostProcessors(beanFactory);`中创建了此核心类的实例。

- **AnnotationAwareAspectJAutoProxyCreator**

这个类就是AOP的核心类，查看UML类图

![image-20210602230619001](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210602230619001.png)

## 解析通知类

在IOC容器创建bean的`finishBeanFactoryInitialization(beanFactory)`方法中，创建第一个bean实例的时候，调用bean工厂中所有的实现了`InstantiationAwareBeanPostProcessor`接口的类的**postProcessBeforeInstantiation**方法，可以看到AOP核心类的父类**AbstractAutoProxyCreator**实现了此接口，因此调用的是父类的方法。

其中又通过AspectJAwareAdvisorAutoProxyCreator类，重写了父类shouldSkip方法，在这个方法里调用了findCandidateAdvisors方法.

AspectJAwareAdvisorAutoProxyCreator又重写了父类findCandidateAdvisors方法,在这个方法里实际上还是调用的AbstractAdvisorAutoProxyCreator的findCandidateAdvisors方法。

这一连串的父类，子类调用稍微有点绕，可以看下面的调用链:

![image-20210605235423932](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210605235423932.png)



## 创建AOP代理增强bean

在上面的调用类中也有说明，其实是在bean实例创建完成之后，调用的bean后置处理器来创建AOP代理对象，增强bean。(第10步)

# 声明式事务功能

![课上图(1)](https://gitee.com/Zeebrary/PicBed/raw/master/img/课上图(1).png)
