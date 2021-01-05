# Idea 项目引入外部SDK依赖报错

报错信息:

```shell
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'springAddtionalModelService': Unsatisfied dependency expressed through field 'typeResolver'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.fasterxml.classmate.TypeResolver' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

查看报错类所在包为:`com.github.xiaoymin.swaggerbootstrapui.service` 

在Idea中使用插件 Maven Helper，点开pom.xml文件可以看到依赖树

![image-20201215192644756](C:/Users/zee/AppData/Roaming/Typora/typora-user-images/image-20201215192644756.png)

可以右键排除不需要使用的模块