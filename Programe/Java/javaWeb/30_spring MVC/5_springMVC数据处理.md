在[2_springMvc注解使用](2_springMvc注解使用.md)中有介绍，在controller层可以获取到请求中的数据，如请求头,请求数据,session,cookie中的数据。但是前端传这些数据的格式可能和后端接受的格式不同，比如传输了`2021-04-28`的字符串，后端需要转换成Date类型。针对这个问题,spring mvc提供了一系列的类型转换器，并且允许自定以类型转换器。

# **类型转换器**

## GenericConversionService

- spring mvc默认通过`GenericConversionService`来进行数据的类型转换,它实现了`ConversionService`,`ConverterRegistry`两个接口，其中实现`ConverterRegistry`接口的`addConverter`方法，在容器初始化时将初始的类型转换器保存到converters:

```java
private final GenericConversionService.Converters converters = new GenericConversionService.Converters();

 
public void addConverter(GenericConverter converter) {
    //debug断点达到这里,可以跟踪堆栈查看spring mvc加载流程
    this.converters.add(converter);
    this.invalidateCache();
}
```

debug跟踪spring mvc初始化流程可以看到,底层通过`FormattingConversionServiceFactoryBean`继承了`FactoryBean`接口和`InitializingBean`接口，在`afterPropertiesSet`实现中将类型转换器保存在converters中。

- 通过实现`ConversionService`的`convert`方法来进行类型转换:

```java
public Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType) {
    //debug断点打到这里,可以上去查看converters里面包含的类型转换器，一共有124个默认类型转换器
    Assert.notNull(targetType, "Target type to convert to cannot be null");
    if (sourceType == null) {
        Assert.isTrue(source == null, "Source must be [null] if source type == [null]");
        return this.handleResult((TypeDescriptor)null, targetType, this.convertNullSource((TypeDescriptor)null, targetType));
    } else if (source != null && !sourceType.getObjectType().isInstance(source)) {
        throw new IllegalArgumentException("Source to convert from must be an instance of [" + sourceType + "]; instead it was a [" + source.getClass().getName() + "]");
    } else {
        GenericConverter converter = this.getConverter(sourceType, targetType);
        if (converter != null) {
            Object result = ConversionUtils.invokeConverter(converter, source, sourceType, targetType);
            return this.handleResult(sourceType, targetType, result);
        } else {
            return this.handleConverterNotFound(source, sourceType, targetType);
        }
    }
}
```

跟踪上面的`this.getConverter`方法会发现，每次在获取具体的转换器之后spring都会缓存起来:

```java
private final Map<GenericConversionService.ConverterCacheKey, GenericConverter> converterCache = new ConcurrentReferenceHashMap(64);`
```

## 自定义类型转换器

```java
package mvc.service.conver;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;
import org.springframework.util.ObjectUtils;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

@Component
public class StringToDateConverter implements Converter<String, Date> {

	public Date convert(String s) {
		if(ObjectUtils.isEmpty(s)){
			return null;
		}
		if(s.split("-").length>2||s.split("/").length>2){
			try {
				SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
				return sdf.parse(s);
			} catch (ParseException e) {
				e.printStackTrace();
				return null;
			}
		}else{
			return null;
		}
	}
}

```

- 配置自定义转换器到spring容器

  在spring的xml配置中添加

  ```xml
  <mvc:annotation-driven conversion-service="conversionService"/>   
  <bean class="org.springframework.context.support.ConversionServiceFactoryBean" id="conversionService">
      <property name="converters">
          <set>
              <ref bean="stringToDateConverter"/>
          </set>
      </property>
  </bean>
  ```

其实底层同上面默认类型加载器初始化类似，只是这里是使用ConversionServiceFactoryBean实现了`FactoryBean`接口和`InitializingBean`接口，在`afterPropertiesSet`实现中将类型转换器保存在converters中。

- **数据格式化**

Spring 另外提供了两个可以用于格式化数字、日期和时间的类型转换器。通过注解@NumberFormat和@DateTimeFormat，这两个标签可以用于javabean的属性或方法参数上。@NumberFormat可以用来格式化任何的数字的基本类型（如int，long）或java.lang.Number的实例（如 BigDecimal, Integer)。@DateTimeFormat可以用来格式化java.util.Date、java.util.Calendar和 java.util.Long类型.

## @NumberFormat

- **pattern**。类型为String，使用自定义的数字格式化字符串，"##,###.##"。

- **style**。类型为NumberFormat.Style，常用值：
  - Style.NUMBER正常数字类型
  - Style.PERCENT百分数类型
  - Style.CURRENCY 货币类型

## @DateTimeFormat

- **iso**。类型为DateTimeFormat.ISO

  - DateTimeFormat.ISO.DATE: 格式yyyy-MM-dd。

  - DateTimeFormat.ISO.DATE_TIME: 格式yyyy-MM-dd HH:mm:ss .SSSZ。
  - DateTimeFormat.ISO.TIME: 格式HH:mm:ss .SSSZ。
  - DateTimeFormat.ISO.NONE: 表示不使用ISO格式的时间。

- **pattern**。类型为String，使用自定义的时间格式化字符串。

- **style**。类型为String，通过样式指定日期时间的格式，由两位字符组成，第1位表示日期的样式，第2位表示时间的格式：

  - S: 短日期/时间的样式；

  - M: 中日期/时间的样式；
  - L: 长日期/时间的样式；
  - F: 完整日期/时间的样式；
  - -: 忽略日期/时间的样式；

<span style="color:red">若自定义了类型转换器和这两个注解冲突，则注解会失效</span>

### JSR303

JSR303是 Java 为 Bean 数据合法性校验提供的标准，它已经包含在 JavaEE 6.0 中 。JSR 303 (Java Specification Requests意思是Java 规范提案)通过**在** **Bean** **属性上标注**类似于 @NotNull、@Max 等标准的注解指定校验规则，并通  j过标准的验证接口对 Bean 进行验证。

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/726DF2BA29D843119C9EA3F8CF7326BC/3180](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/726DF2BA29D843119C9EA3F8CF7326BC/3180)

Hibernate Validator 扩展注解:

![https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/1FBB2006046A4CAAA0B255ACD7E9AB33/3181](https://note.youdao.com/yws/public/resource/5c9055f0c6fff47263ddf0e0d37422d5/xmlnote/854CD1465613401C93050C2AE246EAF8/1FBB2006046A4CAAA0B255ACD7E9AB33/3181)

添加依赖：idea记得将jar包put into /WEB-INF/lib,否则不会生效

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

