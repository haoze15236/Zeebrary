# SpringApplication

> springboot应用类，记录了应用启动需要的信息

构造方法中初始化了几个比较重要的属性:

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

- **bootstrapRegistryInitializers** : 启动注册初始化类

- **initializers** : ApplicationContextInitializer的所有实现类
- **listeners** : ApplicationListener的所有实现类

## SpringFactoriesLoader

> 每个springboot项目只有一个的final class ,提供静态方法供springboot内部获取当前项目的所有META-INF/spring.factories 资源文件的键值对

在上面springApplication初始化时反复调用了此类来从默认的META-INF/spring.factories获取指定接口的实现类。

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    //从缓存中获取指定类加载器的配置资源
    Map<String, List<String>> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }
    //第一次进入此方法时加载一次当前项目的所有META-INF/spring.factories
    result = new HashMap<>();
    try {
        Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            UrlResource resource = new UrlResource(url);
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryTypeName = ((String) entry.getKey()).trim();
                String[] factoryImplementationNames =
                    StringUtils.commaDelimitedListToStringArray((String) entry.getValue());
                for (String factoryImplementationName : factoryImplementationNames) {
                    result.computeIfAbsent(factoryTypeName, key -> new ArrayList<>())
                        .add(factoryImplementationName.trim());
                }
            }
        }

        // Replace all lists with unmodifiable lists containing unique elements
        result.replaceAll((factoryType, implementations) -> implementations.stream().distinct()
                          .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList)));
        cache.put(classLoader, result);
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                                           FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
    return result;
}
```

这一步在初始化时找到了默认的配置文件并将:

```
org/springframework/boot/spring-boot/2.4.5/spring-boot-2.4.5.jar!/META-INF/spring.factories
org/springframework/boot/spring-boot-autoconfigure/2.4.5/spring-boot-autoconfigure-2.4.5.jar!/META-INF/spring.factories
```

**后续如果我们自定义场景启动器时，一些默认配置就可以参考此方法，在项目的resource目录下新建/META-INF/spring.factories文件来定义**。

项目启动时，会首先从默认配置文件中获取<span style="color:#ffa100">**Bootstrapper**</span>和<span style="color:#ffa100">**BootstrapRegistryInitializer**</span>接口的实现类，会调用**Bootstrapper**接口实现类的initialize方法(已废弃),放入SpringApplication的bootstrapRegistryInitializers成员变量中

```java
private List<BootstrapRegistryInitializer> getBootstrapRegistryInitializersFromSpringFactories() {
    ArrayList<BootstrapRegistryInitializer> initializers = new ArrayList<>();
    getSpringFactoriesInstances(Bootstrapper.class).stream()
        .map((bootstrapper) -> ((BootstrapRegistryInitializer) bootstrapper::initialize))
        .forEach(initializers::add);
    initializers.addAll(getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    return initializers;
}
```

## 启动应用

> springboot启动的核心逻辑

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    // 就是记录了启动开始时间
    stopWatch.start();
    // 它是任何spring上下文的接口， 所以可以接收任何ApplicationContext实现
    DefaultBootstrapContext bootstrapContext = createBootstrapContext();
    ConfigurableApplicationContext context = null;
    // 开启了Headless模式：
    configureHeadlessProperty();
    // 去spring.factroies中读取了SpringApplicationRunListener 的组件， 就是用来发布事件或者运行监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 发布1.ApplicationStartingEvent事件，在运行开始时发送
    listeners.starting(bootstrapContext, this.mainApplicationClass);
    try {
        // 根据命令行参数 实例化一个ApplicationArguments
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 预初始化环境： 读取环境变量，读取配置文件信息（基于监听器）
        ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        // 忽略beaninfo的bean
        configureIgnoreBeanInfo(environment);
        // 打印Banner 横幅
        Banner printedBanner = printBanner(environment);
        // 根据webApplicationType创建Spring上下文
        context = createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
        //预初始化spring上下文
        prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        // 加载spring ioc 容器 **相当重要 由于是使用AnnotationConfigServletWebServerApplicationContext 启动的spring容器所以springboot对它做了扩展：
        // 加载自动配置类：invokeBeanFactoryPostProcessors ， 创建servlet容器onRefresh
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

在refeashContext中初始化了spring容器，由于springboot启动类上使用了类似<span style="color:#ffa100">@EnableAutoConfiguration</span>这样的注解，在解析注解的时候，就会把相应的场景启动器的配置类加载,详细加载流程可参考 [02_springboot自动配置原理](./02_springboot自动配置原理.md)

