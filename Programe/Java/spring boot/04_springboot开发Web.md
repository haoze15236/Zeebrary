# rest 风格接口

```java
//查询
@GetMapping("/{id}")
//新增
@PostMapping("/add")
//修改
@PutMapping("/{id}")
//删除
@DeleteMapping("/{id}")
```

## [RestTemplate](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.resttemplate)

```java
// url: 请求的远程rest url
// object : post请求的参数
// Class<T>:返回的类型
// ...Object: 是@PathVariable 占位符的参数
ResponseEntity<Result> resultResponseEntity = restTemplate.postForEntity("http://localhost:8080/user/add", user, Result.class);
resultResponseEntity.getBody().toString();
//put请求调用修改
ResponseEntity<Result> resultResponseEntity = restTemplate.exchange("http://localhost:8080/user/{id}", HttpMethod.PUT, httpEntity, Result.class, 1);
//delete请求调用删除
ResponseEntity<Result> resultResponseEntity = restTemplate.exchange("http://localhost:8080/user/{id}", HttpMethod.DELETE, null, Result.class, 1);
```

# [Mock MVC](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.with-mock-environment)

> MockMvc是由spring-test包提供，实现了对Http请求的模拟，能够直接使用网络的形式，转换到Controller的调用，使得测试速度
> 快、不依赖网络环境。同时提供了一套验证的工具，结果的验证十分方便。

```java
@SpringBootTest
@AutoConfigureMockMvc
class DemoApplicationTests {
	@Test
	void exampleTest(@Autowired MockMvc mvc) throws Exception {
		String webClient = "{\n" +
				"  \"id\": 1,\n" +
				"  \"url\": \"www.baidu.com\",\n" +
				"  \"host\": \"8080\"\n" +
				"}";
		mvc.perform(MockMvcRequestBuilders.post("/postUser")
		.accept(MediaType.APPLICATION_JSON_UTF8)
		.contentType(MediaType.APPLICATION_JSON_UTF8)
		.content(webClient))
		.andExpect(MockMvcResultMatchers.status().isOk())
		.andExpect(MockMvcResultMatchers.jsonPath("$.id").value(1))
		.andDo(MockMvcResultHandlers.print());
	}

}
```

通过构建MockMvcRequestBuilders来设置请求的参数,包括

- 接受类型accept
- 请求内容类型contentType
- 请求内容content
- 返回结果断言**andExpect**
  - MockMvcResultMatchers.status().isOk() ：代表返回状态是200
  - MockMvcResultMatchers.jsonPath("$.id").value(1) ：代表返回的json中存在id属性，且值为1

- 断言通过之后执行操作**andDo**
  - MockMvcResultHandlers.print() : 打印请求内容

# swagger

> 根据后端申明的注解，自动生成接口说明文档，让后端开发人员，从繁重的接口说明文档中解脱出来

- 添加依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

- 添加swagger配置类

```java
@EnableSwagger2
@Configuration
public class SwaggerConfig {
	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.SWAGGER_2) //定义生成swagger2规范的文档
				.pathMapping("/") //设置哪些接口会映射到swagger中
				.select() //接口选择器
				.apis(RequestHandlerSelectors.basePackage("com.example.demo")) //api扫描包
				.paths(PathSelectors.any())//路径过滤
				.build()
				//swagger文档信息描述
				.apiInfo(new ApiInfoBuilder()
				.title("测试swagger") //标题
				.description("springboot整合swagger")  //描述
				.version("1.0") //版本
				.contact(new Contact("郝泽","","136910472@qq.com")) //联系人
				.build());
	}
}
```

## **controller** 

| 注解 | 说明           |
| ---- | -------------- |
| @Api | 对请求类的说明 |

@API 其它属性配置:

| 属性名称    | 备注                               |
| ----------- | ---------------------------------- |
| value       | url 的路径值                       |
| tags        | 如果设置这个值、value 的值会被覆盖 |
| description | 对 api 资源的描述                  |
| basePath    | 基本路径                           |

| position | 如果配置多个 Api 想改变显示的顺序位置   |
| -------- | --------------------------------------- |
| produces | 如, “application/json, application/xml” |
| consumes | 如, “application/json, application/xml” |

| protocols      | 协议类型，如: http, https, ws, wss. |
| -------------- | ----------------------------------- |
| authorizations | 高级特性认证时配置                  |
| hidden         | 配置为 true ，将在文档中隐藏        |

## **方法**

| 注解                                  | 说明                                                        |
| ------------------------------------- | ----------------------------------------------------------- |
| @ApiOperation                         | 方法的说明                                                  |
| @ApiImplicitParams、@ApiImplicitParam | 方法的入参的说明；@ApiImplicitParams 用于指定单个参数的说明 |
| @ApiResponses、@ApiResponse           | 方法返回值的说明 ；@ApiResponses 用于指定单个参数的说明     |

- @ApiOperation

  value="说明方法的作用"    

  notes="方法的备注说明"    

- @ApiImplicitParam

  name: 参数名

  value: 参数的汉字说明, 解释

  required: 参数是否必须传

  paramType: 参数放在哪个地方

  . header --> 请求参数的获取:@RequestHeader

  . query --> 请求参数的获取:@RequestParam

  . path(用于 restful 接口)--> 请求参数的获取:@PathVariable

  . body(请求体)-->  @RequestBody User user            . form(普通表单提交)              

  dataType: 参数类型, 默认 String, 其它值 dataType="Integer"

  defaultValue: 参数的默认值

- @ApiResponse

  code: 数字, 例如 400

  message: 信息, 例如 "请求参数没填好"

  response: 抛出异常的类

## **对象**

| 注解              | 说明                                           |
| ----------------- | ---------------------------------------------- |
| @ApiModel         | 用在 JavaBean 类上，说明 JavaBean 的 用途      |
| @ApiModelProperty | 用在 JavaBean 类的属性上面，说明此属性的的含议 |

# spring mvc自动配置原理

