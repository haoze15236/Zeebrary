# [Jackson 的 基本用法](https://www.cnblogs.com/guanbin-529/p/11488869.html)

------

Jackson 是当前用的比较广泛的，用来序列化和反序列化 json 的 Java 的开源框架。Jackson 社 区相对比较活跃，更新速度也比较快， 从 Github 中的统计来看，Jackson 是最流行的 json 解析器之一 。 Spring MVC 的默认 json 解析器便是 Jackson。 Jackson 优点很多。 Jackson 所依赖的 jar 包较少 ，简单易用。与其他 Java 的 json 的框架 Gson 等相比， Jackson 解析大的 json 文件速度比较快；Jackson 运行时占用内存比较低，性能比较好；Jackson 有灵活的 API，可以很容易进行扩展和定制。

Jackson 的 1.x 版本的包名是 org.codehaus.jackson ，当升级到 2.x 版本时，包名变为 com.fasterxml.jackson，本文讨论的内容是基于最新的 Jackson 的 2.9.1 版本。

### Jackson 的核心模块由三部分组成。

- jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
- jackson-annotations，注解包，提供标准注解功能；
- jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

##### 在 pom.xml 的 Jackson 的配置信息

```xml
<dependency> 
<groupId>com.fasterxml.jackson.core</groupId> 
<artifactId>jackson-databind</artifactId> 
<version>2.9.1</version> 
</dependency>
```

jackson-databind 依赖 jackson-core 和 jackson-annotations，当添加 jackson-databind 之后， jackson-core 和 jackson-annotations 也随之添加到 Java 项目工程中。在添加相关依赖包之后，就可以使用 Jackson。



### ObjectMapper 的 使用

Jackson 最常用的 API 就是基于"对象绑定" 的 ObjectMapper。下面是一个 ObjectMapper 的使用的简单示例。

##### ObjectMapper 使用示例

```java
ObjectMapper mapper = new ObjectMapper(); 
Person person = new Person(); 
person.setName("Tom"); 
person.setAge(40); 
String jsonString = mapper.writerWithDefaultPrettyPrinter() 
.writeValueAsString(person); 
Person deserializedPerson = mapper.readValue(jsonString, Person.class);
```

ObjectMapper 通过 writeValue 系列方法 将 java 对 象序列化 为 json，并 将 json 存 储成不同的格式，String（writeValueAsString），Byte Array（writeValueAsString），Writer， File，OutStream 和 DataOutput。

ObjectMapper 通过 readValue 系列方法从不同的数据源像 String ， Byte Array， Reader，File，URL， InputStream 将 json 反序列化为 java 对象。

### 信息配置

在调用 writeValue 或调用 readValue 方法之前，往往需要设置 ObjectMapper 的相关配置信息。这些配置信息应用 java 对象的所有属性上。示例如下：

##### 清单 3 . 配置信息使用示例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//在反序列化时忽略在 json 中存在但 Java 对象不存在的属性 
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,
   false); 
//在序列化时日期格式默认为 yyyy-MM-dd'T'HH:mm:ss.SSSZ 
mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,false) 
//在序列化时忽略值为 null 的属性 
mapper.setSerializationInclusion(Include.NON_NULL); 
//忽略值为默认值的属性 
mapper.setDefaultPropertyInclusion(Include.NON_DEFAULT);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

更多配置信息可以查看 Jackson 的 DeserializationFeature，SerializationFeature 和 I nclude。

 

### Jackson 的 注解的使用

Jackson 根据它的默认方式序列化和反序列化 java 对象，若根据实际需要，灵活的调整它的默认方式，可以使用 Jackson 的注解。常用的注解及用法如下。

##### 表 1. Jackson 的 常用注解

| 注解               | 用法                                                         |
| ------------------ | ------------------------------------------------------------ |
| @JsonProperty      | 用于属性，把属性的名称序列化时转换为另外一个名称。示例：  @JsonProperty("birth_ d ate")  private Date birthDate; |
| @JsonFormat        | 用于属性或者方法，把属性的格式序列化时转换成指定的格式。示例：  @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm")  public Date getBirthDate() |
| @JsonPropertyOrder | 用于类， 指定属性在序列化时 json 中的顺序 ， 示例：  @JsonPropertyOrder({ "birth_Date", "name" })  public class Person |
| @JsonCreator       | 用于构造方法，和 @JsonProperty 配合使用，适用有参数的构造方法。 示例：  @JsonCreator  public Person(@JsonProperty("name")String name) {…} |
| @JsonAnySetter     | 用于属性或者方法，设置未反序列化的属性名和值作为键值存储到 map 中  @JsonAnySetter  public void set(String key, Object value) {  map.put(key, value);  } |
| @JsonAnyGetter     | 用于方法 ，获取所有未序列化的属性  public Map<String, Object> any() { return map; } |

###  Jackson示例

[回到顶部](https://www.cnblogs.com/guanbin-529/p/11488869.html#_labelTop)

## Jackson ObjectMapper Example

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ObjectMapper objectMapper = new ObjectMapper();

String carJson =
    "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

try {
    Car car = objectMapper.readValue(carJson, Car.class);

    System.out.println("car brand = " + car.getBrand());
    System.out.println("car doors = " + car.getDoors());
} catch (IOException e) {
    e.printStackTrace();
}

public class Car {
    private String brand = null;
    private int doors = 0;

    public String getBrand() { return this.brand; }
    public void   setBrand(String brand){ this.brand = brand;}

    public int  getDoors() { return this.doors; }
    public void setDoors (int doors) { this.doors = doors; }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/guanbin-529/p/11488869.html#_labelTop)

## 从Reader读取对象

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ObjectMapper objectMapper = new ObjectMapper();

String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 4 }";
Reader reader = new StringReader(carJson);

Car car = objectMapper.readValue(reader, Car.class);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[回到顶部](https://www.cnblogs.com/guanbin-529/p/11488869.html#_labelTop)

## 从File中读取对象

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ObjectMapper objectMapper = new ObjectMapper();

File file = new File("data/car.json");

Car car = objectMapper.readValue(file, Car.class);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[回到顶部](https://www.cnblogs.com/guanbin-529/p/11488869.html#_labelTop)

## 从URL中读取对象

```
ObjectMapper objectMapper = new ObjectMapper();

URL url = new URL("file:data/car.json");

Car car = objectMapper.readValue(url, Car.class);
```

[回到顶部](https://www.cnblogs.com/guanbin-529/p/11488869.html#_labelTop)

## 从InputStream读取对象

```
ObjectMapper objectMapper = new ObjectMapper();

InputStream input = new FileInputStream("data/car.json");

Car car = objectMapper.readValue(input, Car.class);
```

## 从字节数组中读取对象

```
ObjectMapper objectMapper = new ObjectMapper();

String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

byte[] bytes = carJson.getBytes("UTF-8");

Car car = objectMapper.readValue(bytes, Car.class);
```

## 从JSON数组字符串中读取对象数组 

```
String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";

ObjectMapper objectMapper = new ObjectMapper();

Car[] cars2 = objectMapper.readValue(jsonArray, Car[].class);
```

## 从JSON数组字符串中读取对象列表

```
String jsonArray =“[{\”brand \“：\”ford \“}，{\”brand \“：\”Fiat \“}]”;

ObjectMapper objectMapper = new ObjectMapper（）;

List <Car> cars1 = objectMapper.readValue（jsonArray，new TypeReference <List <Car >>（）{}）;
```

## 从JSON字符串中读取映射为map

```
String jsonObject =“{\”brand \“：\”ford \“，\”doors \“：5}”;

ObjectMapper objectMapper = new ObjectMapper（）;
Map <String，Object> jsonMap = objectMapper.readValue（jsonObject，
    new TypeReference <Map <String，Object >>（）{}）;
```

## 树模型

```
String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

ObjectMapper objectMapper = new ObjectMapper();

try {

    JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);

} catch (IOException e) {
    e.printStackTrace();
}
```

JSON字符串被解析为`JsonNode`对象而不是`Car`对象，只需将`JsonNode.class`第二个参数传递给`readValue()`方法而不是`Car.class`本教程前面的示例中使用的方法。

该`ObjectMapper`班也有一个特殊的`readTree()`，它总是返回一个方法 `JsonNode`。以下是`JsonNode`使用该`ObjectMapper` `readTree()`方法将JSON解析为a的示例：

```
String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

ObjectMapper objectMapper = new ObjectMapper();

try {

    JsonNode jsonNode = objectMapper.readTree(carJson);

} catch (IOException e) {
    e.printStackTrace();
}
```

JsonNode类

```
String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 5," +
        "  \"owners\" : [\"John\", \"Jack\", \"Jill\"]," +
        "  \"nestedObject\" : { \"field\" : \"value\" } }";

ObjectMapper objectMapper = new ObjectMapper();


try {

    JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);

    JsonNode brandNode = jsonNode.get("brand");
    String brand = brandNode.asText();
    System.out.println("brand = " + brand);

    JsonNode doorsNode = jsonNode.get("doors");
    int doors = doorsNode.asInt();
    System.out.println("doors = " + doors);

    JsonNode array = jsonNode.get("owners");
    JsonNode jsonNode = array.get(0);
    String john = jsonNode.asText();
    System.out.println("john  = " + john);

    JsonNode child = jsonNode.get("nestedObject");
    JsonNode childField = child.get("field");
    String field = childField.asText();
    System.out.println("field = " + field);

} catch (IOException e) {
    e.printStackTrace();
}
```

### 将Object转换为JsonNode

```
ObjectMapper objectMapper = new ObjectMapper();

Car car = new Car();
car.brand = "Cadillac";
car.doors = 4;

JsonNode carJsonNode = objectMapper.valueToTree(car);
```

### 将JsonNode转换为Object

```
ObjectMapper objectMapper = new ObjectMapper();

String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

JsonNode carJsonNode = objectMapper.readTree(carJson);

Car car = objectMapper.treeToValue(carJsonNode);
```

## 使用Jackson ObjectMapper读取和编写YAML

 

**1.示例1(只是yaml字符串和对象的互转，不涉及yaml文件的处理)**

```
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;

import java.io.IOException;

public class YamlJacksonExample {

    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper(new YAMLFactory());

        Employee employee = new Employee("John Doe", "john@doe.com");

        String yamlString = null;
        try {
            yamlString = objectMapper.writeValueAsString(employee);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            // normally, rethrow exception here - or don't catch it at all.
        }

    }
}
```

该`yamlString`变量包含`Employee`在执行此代码后序列化为YAML数据格式的对象。

以下是`Employee`再次将YAML文本读入对象的示例：

```
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;

import java.io.IOException;

public class YamlJacksonExample {

    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper(new YAMLFactory());

        Employee employee = new Employee("John Doe", "john@doe.com");

        String yamlString = null;
        try {
            yamlString = objectMapper.writeValueAsString(employee);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            // normally, rethrow exception here - or don't catch it at all.
        }

        try {
            Employee employee2 = objectMapper.readValue(yamlString, Employee.class);

            System.out.println("Done");
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

**2. 示例2 （yaml文件的读取和写入）**

 **2.1定义Employee实体类**

```
package com.example.jackjson;

import lombok.Data;

/**
 * @author: GuanBin
 * @date: Created in 上午10:18 2020/6/15
 */
@Data
public class Employee {

    public Employee() {
    }

    public Employee(String name, String email) {
        this.name = name;
        this.email = email;
    }

    String name;

    String email;
}
```



**2.2创建要读取的yml EmployeeYaml.yml文件，并初始化一条数据**

```
name: test
email: test@qq.com
```

**2.3创建要写入的yml文件,EmployeeYamlOutput.yml （空文件）**

 

**2.4 测试类**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.example.jackjson;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;
import com.fasterxml.jackson.dataformat.yaml.YAMLGenerator;

import java.io.File;
import java.io.IOException;

/**
 * @author: GuanBin
 * @date: Created in 上午10:17 2020/6/15
 */
public class YamlJacksonExample {
    public static void main(String[] args) {


        try {
            //从yaml文件读取数据
            reaedYamlToEmployee();
            //写入yaml文件
            reaedEmployeeToYaml();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }


    /**
     * 从yaml文件读取数据
     * @throws IOException
     */
    private static void reaedYamlToEmployee() throws IOException {
        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
        Employee employee = mapper.readValue(new File("src/test/java/com/example/jackjson/EmployeeYaml.yml"), Employee.class);
        System.out.println(employee.getName() + "********" + employee.getEmail());

    }

    /**
     * 写入yaml文件
     * @throws IOException
     */
    private static void reaedEmployeeToYaml() throws IOException {
        //去掉三个破折号
        ObjectMapper  mapper = new ObjectMapper(new YAMLFactory().disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER));
        //禁用掉把时间写为时间戳
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

        Employee employee = new Employee("test2", "999@qq.com");
        mapper.writeValue(new File("src/test/java/com/example/jackjson/EmployeeYamlOutput.yml"), employee);
    }
}
```

读取文件的打印输出

```
test********test@qq.com

Process finished with exit code 0
```

写入文件的输出

![img](https://img2020.cnblogs.com/blog/1089423/202006/1089423-20200615111708246-1003443113.png)

 