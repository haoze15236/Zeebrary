# **类型转换器**

在日常的企业开发需求中，我们输入文本框的内容全部都是字符串类型，但是在后端处理的时候我们可以用其他基本类型来接受数据，也可以使用实体类来接受参数，这个是怎么完成的呢？就是通过SpringMVC提供的类型转换器，SpringMVC内部提供了非常丰富的类型转换器的支持，但是有些情况下有可能难以满足我们的需求，因此我们自定义一个类型转换器来试试看。

- 定义类型转换器

```java
package cn.tulingxueyuan.converter;

import cn.tulingxueyuan.bean.User;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;

@Component
public class MyConverter implements Converter<String, User> {
    public User convert(String source) {
        User user = null;
        String[] split = source.split("-");
        if (source!=null && split.length==4){
            user = new User();
            user.setId(Integer.parseInt(split[0]));
            user.setName(split[1]);
            user.setAge(Integer.parseInt(split[2]));
            user.setGender(split[3]);
        }
        return user;
    }
}
```

- 配置自定义转换器到spring容器

  在spring的xml配置中添加

  ```xml
     <bean class="cn.tulingxueyuan.view.MyConverter">
         <property name="order" value="1"></property>
     </bean>
  <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
     <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
         <property name="converters">
             <set>
                 <ref bean="myConverter"></ref>
             </set>
         </property>
     </bean>
  ```

  

# **数据格式化**

Spring 提供了两个可以用于格式化数字、日期和时间的注解@NumberFormat和@DateTimeFormat，这两个标签可以用于javabean的属性或方法参数上。@NumberFormat可以用来格式化任何的数字的基本类型（如int，long）或java.lang.Number的实例（如 BigDecimal, Integer)。@DateTimeFormat可以用来格式化java.util.Date、java.util.Calendar和 java.util.Long类型.

要指定数字或日期/时间类型的属性，只需要在其上添加 @NumberFormat或@DateTimeFormat注解接可以了。

## @NumberFormat

- - **pattern**。类型为String，使用自定义的数字格式化字符串，"##,###.##"。
  - **style**。类型为NumberFormat.Style，常用值：

- - - Style.NUMBER正常数字类型
    - Style.PERCENT百分数类型
    - Style.CURRENCY 货币类型

## @DateTimeFormat

- - **iso**。类型为DateTimeFormat.ISO

- - - DateTimeFormat.ISO.DATE: 格式yyyy-MM-dd。
    - DateTimeFormat.ISO.DATE_TIME: 格式yyyy-MM-dd HH:mm:ss .SSSZ。
    - DateTimeFormat.ISO.TIME: 格式HH:mm:ss .SSSZ。
    - DateTimeFormat.ISO.NONE: 表示不使用ISO格式的时间。

- - **pattern**。类型为String，使用自定义的时间格式化字符串。
  - **style**。类型为String，通过样式指定日期时间的格式，由两位字符组成，第1位表示日期的样式，第2位表示时间的格式：

- - - S: 短日期/时间的样式；
    - M: 中日期/时间的样式；
    - L: 长日期/时间的样式；
    - F: 完整日期/时间的样式；
    - -: 忽略日期/时间的样式；

# 数据校验

JSR303是 Java 为 Bean 数据合法性校验提供的标准，它已经包含在 JavaEE 6.0 中 。JSR 303 (Java Specification Requests意思是Java 规范提案)通过**在** **Bean** **属性上标注**类似于 @NotNull、@Max 等标准的注解指定校验规则，并通  j过标准的验证接口对 Bean 进行验证。

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/726DF2BA29D843119C9EA3F8CF7326BC/3180](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/726DF2BA29D843119C9EA3F8CF7326BC/3180)

Hibernate Validator 扩展注解:

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/1FBB2006046A4CAAA0B255ACD7E9AB33/3181](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/1FBB2006046A4CAAA0B255ACD7E9AB33/3181)

添加依赖：

```xml
<!--JSR349数据认证依赖-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.1.0.Final</version>
        </dependency>
```

# JSON数据处理

## @RequestBody

解析请求中json转换成注解修饰的参数

**以RequestParam接收**

前端传来的是json数据不多时：

```java
@PostMapping("/json/request01")
@ResponseBody
public  User responseJson(@RequestBody String name){
    User user = new User(1,"12346",new Date());
    System.out.println(name);
    return user;

}
```

**以实体类方式接收**

前端传来的是一个json对象时：{ id:1,name:xx},可以用实体类直接进行自动绑定

```java
@PostMapping(value="/json/request02",consumes = "application/json")
@ResponseBody
public  User requestJson02(@RequestBody User user){
    User user2 = new User(1, "12346",new Date());
    System.out.println(user);
    return user2;

}
```

**以Map接收**

前端传来的是一个json对象时：{ id:1,name:xx},可以用Map来获取

```java
@PostMapping(value="/json/request03",consumes = "application/json")
@ResponseBody
public  User requestJson03(@RequestBody Map<String,String> map){
    User user2 = new User(1, "徐庶","12346",new Date());
    System.out.println(map);
    return user2;

}
```

**以List接收**

当前端传来这样一个json数组：[{ id:1,name:xx},{ id:1,name:xx},{ id:1,name:xx},...]时，用List接收 

```java
@PostMapping(value="/json/request04",consumes = "application/json")
@ResponseBody
public  User requestJson04(@RequestBody List<User> list){
    User user2 = new User(1, "徐庶","12346",new Date());
    System.out.println(list);
    return user2;

}
```

## @ResponseBody

将返回内容转换成json格式

```java
@RequestMapping("/params")
@ResponseBody
public String paramsRequest(@NumberFormat Map username){
System.out.println(username);
return username.toString();
}
```

