前面了解了springboot快速搭建，并学习了配置文件的使用，但是内置的配置文件可以控制哪些行为，又应该在application.properties文件中通过设置什么属性来控制还是没有头绪，springboot非常贴心的提供了一份[spring场景启动器可配置属性清单](https://docs.spring.io/spring-boot/docs/2.4.5/reference/html/appendix-application-properties.html#common-application-properties-web)供我们来查询，日常使用时不知道就查一查，但是他底层如何实现的？这些属性又是怎么工作的呢？下面让我们来一起看看springboot的自动配置原理初窥一二。

![image-20210517004944617](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210517004944617.png)

在[快速搭建](./00_springboot快速搭建#@SpringBootApplication)中，我们了解了简单介绍了@SpringBootApplication这个最顶层的注解,但是没有详细解释@EnableAutoConfiguration注解，而这个就是springboot自动配置的关键

# <span style="color:#ffa100">@EnableAutoConfiguration</span>

- @AutoConfigurationPackage：把配置类当前所在包加载到spring容器中供后续使用。
- @Import(AutoConfigurationImportSelector.class) ：导入自动配置选择器，获取所有的自动配置类

## AutoConfigurationImportSelector

自动配置选择器(AutoConfigurationImportSelector.class),这个类继承了**DeferredImportSelector**，也就是继承了**ImportSelector**。在spring ioc使用javaconfig方式配置的时候有说过**ImportSelector**，通过实现**ImportSelector**的selectImports()方法，可以自定义注入bean。

而**DeferredImportSelector**不同，它通过实现getImportGroup()方法,返回类**AutoConfigurationGroup**，通过调用**AutoConfigurationGroup**类的process()方法来获取所有的自动配置类。

```java
@Override
		public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
			Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector,
					() -> String.format("Only %s implementations are supported, got %s",
							AutoConfigurationImportSelector.class.getSimpleName(),
							deferredImportSelector.getClass().getName()));
            //getAutoConfigurationEntry(annotationMetadata);就是获取所有自动配置类
			AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
					.getAutoConfigurationEntry(annotationMetadata);
            //加入到装配集合
			this.autoConfigurationEntries.add(autoConfigurationEntry);
			for (String importClassName : autoConfigurationEntry.getConfigurations()) {
				this.entries.putIfAbsent(importClassName, annotationMetadata);
			}
		}
```

### getAutoConfigurationEntry

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		//获取了初始化时所有的自动配置类 debugger出来有130个
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
    	//通过场景控制器规则过滤之后，web场景下有23个自动配置类，返回之后被自动装配进spring容器
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

- **getCandidateConfigurations** 

  ```java
  protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
  		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
  				getBeanClassLoader());
  		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
  				+ "are using a custom packaging, make sure that file is correct.");
  		return configurations;
  	}
  ```

这里是从缓存中获取所有的自动配置类，而缓存中的自动配置类是在**SpringBootApplication** 初始化的时候，通过扫描项目引入的所有jar包中的META-INF/spring.factories定义的：[所有自动配置类详解](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-auto-configuration-classes.html#auto-configuration-classes)

![image-20210517003234153](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210517003234153.png)

### getConfigurationClassFilter

获取到在spring-boot-autoconfigure.jar的META-INF/spring.factories中定义的过滤规则,对上面初始化中缓存的所有配置类进行过滤，只留下当前场景控制器需要的自动配置类:

```
# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition
```

过滤规则是在每个自动配置类上通过@Conditional派生注解来定义的:

@Conditional派生了很多注解，下面给个表格列举一下派生注解的用法。

| @Conditional派生注解            | 作用(都是判断是否符合指定的条件)               |
| ------------------------------- | ---------------------------------------------- |
| @ConditionalOnJava              | 系统的java版本是否符合要求                     |
| @ConditionalOnBean              | 有指定的Bean类                                 |
| @ConditionalOnMissingBean       | 没有指定的bean类                               |
| @ConditionalOnExpression        | 符合指定的SpEL表达式                           |
| @ConditionalOnClass             | 有指定的类                                     |
| @ConditionalOnMissingClass      | 没有指定的类                                   |
| @ConditionalOnSingleCandidate   | 容器只有一个指定的bean，或者这个bean是首选bean |
| @ConditionalOnProperty          | 指定的property属性有指定的值                   |
| @ConditionalOnResource          | 路径下存在指定的资源                           |
| @ConditionalOnWebApplication    | 系统环境是web环境                              |
| @ConditionalOnNotWebApplication | 系统环境不是web环境                            |
| @ConditionalOnjndi              | JNDI存在指定的项                               |

# HttpEncodingAutoConfiguration

以其中一个自动配置类为例:

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ServerProperties.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {
}
```

- **@Configuration(proxyBeanMethods = false)**：标记了@Configuration Spring底层会给配置创建cglib动态代理。 作用：就是防止每次调用本类的Bean方法而重新创建对象，Bean是默认单例的。

  

- **@EnableConfigurationProperties(ServerProperties.class)**：启用可以在配置类设置的属性 对应的类

  

- **@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)**

  **@ConditionalOnClass(CharacterEncodingFilter.class)**

  **@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)**

  用来判断自动配置类是否生效，也就是前面过滤队则里面对应的过滤器。

## ServerProperties

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {}
```

可以看到，在配置属性类上使用到了前面讲的@ConfigurationProperties，定义了一个注解，这也是为什么可以用server.port来指定tomcat端口的原因。



**总结**：通过以上的学习，可以了解到springboot内置的默认行为流程，之后我们需要引入第三方框架，但是不知道如何配置属性的时候，就可以按照这个流程，找到对应的自动配置类,查看指定的配置属性类，自然一切可配置属性都在掌握中，且这些属性可以导致那些行为也能做到心中有底。