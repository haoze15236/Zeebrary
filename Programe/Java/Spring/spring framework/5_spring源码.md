# **源码项目构建**

- **源码下载** 

https://github.com/spring-projects/spring-framework 笔者下载的是5.2.8releaase版本

- **配置文件修改**

**build.gradle**  

 在解压出来文件夹的根目录,添加aliyun依赖下载地址,可显著提高jar包下载速度

```gradle
repositories {
    maven{ url 'https://maven.aliyun.com/nexus/content/groups/public/'}
    maven{ url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'}
    mavenCentral()
    maven { url "https://repo.spring.io/libs-spring-framework-build" }
}
```

- **初编译**

window环境下，cmd进入命令行，在项目跟目录下进行预编译

```shell
gradlew :spring-oxm:compileTestJava
```

笔者构建时报错，查找关键字 id 'io.spring.gradle-enterprise-conventions' version '0.0.2' ，在**build.gradle** 文件中找到对应代码，注释。

```
plugins {
	id 'io.spring.dependency-management' version '1.0.8.RELEASE' apply false
	id 'org.jetbrains.kotlin.jvm' version '1.3.72' apply false
	id 'org.jetbrains.dokka' version '0.10.1' apply false
	id 'org.asciidoctor.jvm.convert' version '2.4.0'
//	id 'io.spring.gradle-enterprise-conventions' version '0.0.2'
	id 'io.spring.nohttp' version '0.0.5.RELEASE'
	id 'de.undercouch.download' version '4.0.0'
	id 'com.gradle.build-scan' version '3.2'
	id "com.jfrog.artifactory" version '4.12.0' apply false
	id "io.freefair.aspectj" version '4.1.1' apply false
	id "com.github.ben-manes.versions" version '0.24.0'
}
```

等待构建成功，若出现build successful代表构建成功。预编译成功之后，导入idea等待下载依赖(大概需要10分钟左右)即可

# **源码解析**

本次源码解析从`AnnotationConfigApplicationContext`类开始,一步步剖析spring初始化过程中的知识点,领略spring强大的拓展性，学习编程思维。

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		//调用构造函数
		this();
		//注册我们的配置类
		register(componentClasses);
		//IOC容器刷新接口
		refresh();
	}
```

在开始创建Spring容器之前,我们先简单了解2个顶层接口:

- **BeanFactory**：bean工厂,用于生产bean的抽象接口。
- **BeanDefination**: bean定义,用于说明bean的所有属性的抽象接口。

## this()

this()首先隐式调用了父类`GenericApplicationContext`的构造方法,为spring容器初始化了`BeanFactory`,使用的是最底层实现类`DefaultListableBeanFactory`。

### DefaultListableBeanFactory

**bean工厂**：最底层实现的子类，功能最齐全,重点关注两个成员变量。

- **beanDefinitionNames**:存储bean定义的名称
- **beanDefinitionMap**:存储bean定义名称和对应的bean定义

### AnnotatedBeanDefinitionReader

**bean定义读取器**：用于读取bean定义进bean工厂。在初始化的时候，会将spring内置的一些**后置处理器**的bean定义初始化进容器(保存在bean工厂的**beanDefinitionNames**，**beanDefinitionMap**中),主要关注以下两个

- <span style="color:red">**BeanFactoryPostProcessor**</span> :**bean工厂后置处理器接口**,是spring容器的拓展点。bean定义读取器在初始化时,是使用的其子类接口`BeanDefinitionRegistryPostProcessor`的具体实现类`ConfigurationClassPostProcessor`。

```java
if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
}
```

- <span style="color:red">**BeanPostProcessor**</span> ：**bean后置处理器接口**，spring容器的扩展点。bean定义读取器在初始化时,使用其子类`PersistenceAnnotationBeanPostProcessor`

```java
if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition();
			try {
				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
						AnnotationConfigUtils.class.getClassLoader()));
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
			}
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
```

## register()

注册配置类的bean定义到bean工厂中。

```java
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {
		//转为AnnotatedGenericBeanDefinition数据结构，里面有一个getMetadata方法，可以拿到类上的注解
		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
		//判断是否需要跳过注册，spring中有一个@Condition注解，当不满足条件，这个bean就不会被解析
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}

		abd.setInstanceSupplier(supplier);
		//解析bean的作用域，如果没有设置的话，默认为单例
		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		//获得beanName
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
		//解析通用注解，填充到AnnotatedGenericBeanDefinition，解析的注解为Lazy，Primary，DependsOn，Role，Description
		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		//限定符处理，不是特指@Qualifier注解，也有可能是Primary,或者是Lazy，或者是其他（理论上是任何注解，这里没有判断注解的有效性），如果我们在外面，以类似这种
		//AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplic ationContext(Appconfig.class);常规方式去初始化spring，
		//qualifiers永远都是空的，包括上面的name和instanceSupplier都是同样的道理
		//但是spring提供了其他方式去注册bean，就可能会传入了
		if (qualifiers != null) {
			//可以传入qualifier数组，所以需要循环处理
			for (Class<? extends Annotation> qualifier : qualifiers) {
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				//其他，AnnotatedGenericBeanDefinition有个Map<String,AutowireCandidateQualifier>属性，直接push进去
				else {
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}
		if (customizers != null) {
			for (BeanDefinitionCustomizer customizer : customizers) {
				customizer.customize(abd);
			}
		}
		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);

		//注册，最终会调用DefaultListableBeanFactory中的registerBeanDefinition方法去注册，
		//DefaultListableBeanFactory维护着一系列信息，比如beanDefinitionNames，beanDefinitionMap
		//beanDefinitionNames是一个List<String>,用来保存beanName
		//beanDefinitionMap是一个Map,用来保存beanName和beanDefinition
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}
```

## refresh()

上面两步完成之后，在spring的bean工厂中已经注册了内置的一些bean定义和配置类的bean定义，现在开始

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			//1:准备刷新上下文环境,主要做了一些刷新前的准备工作，和主流程关系不大，主要是保存了容器的启动时间，启动标志等
			prepareRefresh();

			//2:获取告诉子类初始化Bean工厂  不同工厂不同实现 简单的认为，就是把beanFactory取出来而已。XML模式下会在这里读取BeanDefinition
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			//3: 主要做了如下的操作：
            //设置了一个类加载器
            //设置了bean表达式解析器
            //添加了属性编辑器的支持
            //添加了一个后置处理器：ApplicationContextAwareProcessor，此后置处理器实现了BeanPostProcessor接口
            //设置了一些忽略自动装配的接口
            //设置了一些允许自动装配的接口，并且进行了赋值操作
            //在容器中还没有XX的bean的时候，帮我们注册beanName为XX的singleton bean
			prepareBeanFactory(beanFactory);

			try {
				// 第四:留个子类去实现该接口
				postProcessBeanFactory(beanFactory);

				// 调用我们的bean工厂的后置处理器. 1. 会在此将class扫描成beanDefinition  2.bean工厂的后置处理器调用
				invokeBeanFactoryPostProcessors(beanFactory);

				// 注册我们bean的后置处理器
				registerBeanPostProcessors(beanFactory);

				// 初始化国际化资源处理器.
				initMessageSource();

				// 创建事件多播器
				initApplicationEventMulticaster();

				// 这个方法同样也是留个子类实现的springboot也是从这个方法进行启动tomcat的.
				onRefresh();

				//把我们的事件监听器注册到多播器上
				registerListeners();

				// 实例化我们剩余的单实例bean.
				finishBeanFactoryInitialization(beanFactory);

				// 最后容器刷新 发布刷新事件(Spring cloud也是从这里启动的)
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception  encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

