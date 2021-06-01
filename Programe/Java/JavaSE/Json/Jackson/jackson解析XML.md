- 添加依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.8.11</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.11</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.8.11</version>
</dependency>
```

# 核心注解

## @JacksonXmlRootElement

> 指定XML根元素

- namespace属性：用于指定XML根元素命名空间的名称。
- localname属性：用于指定XML根元素节点标签的名称。

## @JacksonXmlProperty

- namespace和localname属性用于指定XML命名空间的名称

- isAttribute指定该属性作为XML的属性还是作为子标签.

## @JacksonXmlElementWrapper

列表循环注解，将集合属性转换成多行。

```xml
<elXmlIniPlus>
	<Section SectionName="APPAYSAVZ">
	</Section>
	<Section SectionName="SYCOMRETZ">
	</Section>
</elXmlIniPlus>
```

对于上面这个xml,需要在dto类上使用此注解，同时配合 @JacksonXmlProperty来为每行标签定义名称。

```java
@Data
@JacksonXmlRootElement(localName = "elXmlIniPlus")
public class PaymentHandleIniplusRequestDTO {

    @JacksonXmlElementWrapper(useWrapping=false)
    @JacksonXmlProperty(localName = "Section")
    private List<Section> section;
    
     @Data
    public static class Section{
        @JacksonXmlProperty(localName = "SectionName",isAttribute = true)
        private String sectionName;
    }
}
```

## @JacksonXmlText

@JacksonXmlText注解将属性直接作为未被标签包裹的普通文本表现

## @JacksonXmlCData

@JacksonXmlCData将属性包裹在CDATA标签中

## @JsonPropertyOrder

@JsonPropertyOrder和@JsonProperty的index属性类似，指定属性序列化时的顺序

## @JsonIgnore

用于排除某个属性，这样该属性就不会被Jackson序列化和反序列化

## @JsonIgnoreProperties

类注解。在序列化为JSON的时候，@JsonIgnoreProperties({“prop1”, “prop2”})会忽略pro1和pro2两个属性。在从JSON反序列化为Java类的时候，@JsonIgnoreProperties(ignoreUnknown=true)会忽略所有没有Getter和Setter的属性。该注解在Java类和JSON不完全匹配的时候很有用。

# Xml转bean对象

```java
XmlMapper xmlMapper = new XmlMapper();
try {
    //postXmlDate 为XML报文  CbsIniplusRequestDto为bean对象
    CbsIniplusRequestDto rootXml = xmlMapper.readValue(postXmlDate, CbsIniplusRequestDto.class);
    System.out.println();
    System.out.println(rootXml.toString());
} catch (Exception e) {
    System.out.println("xml转Bean时发生错误:" + e.getMessage());
    throw new RuntimeException(e);
}
```

# bean对象转XML

```java
XmlMapper xmlMapper = new XmlMapper();
String result = null;
try {
    //cbsIniplusRequestDto为bean对象
	result = xmlMapper.writeValueAsString(cbsIniplusRequestDto);
} catch (JsonProcessingException e) {
	e.printStackTrace();
}
```

