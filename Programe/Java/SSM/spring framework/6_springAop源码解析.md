在spring基于javaconfig的方式，我们是在配置类上使用@EnableAspectJAutoProxy注解开启AOP功能，点开这个注解可以看到：

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

# AnnotationAwareAspectJAutoProxyCreator

这个类就是AOP的核心类，查看UML类图

![image-20210602230619001](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210602230619001.png)

