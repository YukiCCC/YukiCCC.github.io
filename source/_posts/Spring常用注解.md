---
title: Spring常用注解
date: 2023-08-29 08:52:13
tags: Spring
category: Java
---
## @Param
@Param()注解，标注在参数前，为参数指定占位符名称
```
public User selectByUsernameAndPassword(@Param("username") String username, @Param("password") String password);
```

## @Component 及@Repository、@Service、@Controller
@Component 定义组件Bean，添加到IoC容器中，不区分组件类型
区分组件类型的注解：
@Repository 表示Dao组件
@Service 表示Service组件
@Controller 表示Controller组件


## @RequestMapping：
@RequestMapping：



## @ResponseBody
@ResponseBody：将方法返回值写到响应体中，一般用来处理Ajax请求，返回JSON数据
也可以使用@RestController，相当于@Controller+@ResponseBody
注意：ResponseBody与RequestBody区别

## @RequiredArgsConstructor：

该注解作用于类上

标记为final的对象,会自动进行注入

使用lombok的 @NonNull 注解标记的对象,会自动进行注入

## @Resource：

@Resource和@Autowired注解都是用来实现依赖注入的。只是@AutoWried按by type自动注入，而@Resource默认按byName自动注入。

@Resource有两个重要属性，分别是name和type

## @Json衍生
### @JsonFormat 
指定序列化和反序列化时的格式，一般用于指定日期、时间和数字的格式
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")，设置响应时的日期格式
### @JsonProperty 
指定序列化和反序列化时的名称
### @JsonAlias 
指定反序列化时的备用别名
### @JsonIgnore 
指定序列化和反序列化时忽略属性


## 参数来源注解
### @RequestParam 
表示参数来源于请求参数，默认为所有参数添加该注解，参数值来源于同名称的请求参数
### @PathVariable 
表示参数来源于URL（URL就是请求路径）

@PathVariable：spring3.0的一个新功能：接收请求路径中占位符的值

@PathVariable("xxx")

通过 @PathVariable 可以将URL中占位符参数{xxx}绑定到处理器类的方法形参中@PathVariable(“xxx“) 
@RequestMapping(value=”user/{id}/{name}”)
请求路径：http://localhost:8080/hello/show5/1/james

### @RequestHeader 
表示参数来源于请求头
### @CookieValue
表示参数来源于Cookie
### @RequestBody 
表示参数来源于请求体，用来接收前端传递给后端的 json 格式的数据


## @RequestMapping及其衍生变体
@RequestMapping表示共享映射,用于将任意HTTP 请求映射到控制器方法上。如果没有指定请求方式，将接收GET、POST、HEAD、OPTIONS、PUT、PATCH、DELETE、TRACE、CONNECT所有的HTTP请求方式。@GetMapping、@PostMapping、@PutMapping、@DeleteMapping、@PatchMapping 都是HTTP方法特有的快捷方式@RequestMapping的变体，分别对应具体的HTTP请求方式的映射注解。


## @Transactional
@Transactional也可以配置在类上，如果方法上没有配置@Transactional，则使用类上的事务配置

使用@Transactional 注解管理事务的实现步骤分为两步。

第一步，在 xml 配置文件中添加如清单 1 的事务配置信息。除了用配置文件的方式，@EnableTransactionManagement 注解也可以启用事务管理功能。这里以简单的 DataSourceTransactionManager 为例。

第二步，将@Transactional 注解添加到合适的方法上，并设置合适的属性信息。

 

 



 

 

## SpringBoot

### @SpringBootApplication

标注在类上，表示这是一个SpringBoot应用，通过运行该类的main方法来启动SpringBoot应用

 

### @SpringBootConfiguration

 

标注在类上，表示这个类是SpringBoot的配置类

 

层级关系：@SpringBootConfiguration——>@Configuration——>@Component

 

### @Configuration
标注在类上，表示这个类是Spring的配置类，相当于是一个xml配置文件

 

### @EnableAutoConfiguration

 

开启自动配置功能，SpringBoot会自动完成许多配置，简化了以前繁琐的配置

 

### @ComponentScan

 

标注在类上，指定要扫描的包，默认只扫描主程序类所在的包及其子包

 

可以使用@ComponentScan手动指定要扫描的包

 

### @MapperScan：指定Dao接口所在的包

 

## API：

### @Api
标注在类上，对类进行说明

### @ApiOperation 
标注在方法上，对方法进行说明

### @ApiParam 
标注在参数上，对方法的参数进行说明

### @ApiModel 
标注在模型Model上，对模型进行说明

### @ApiModelProperty 
标注在属性上，对模型的属性进行说明

### @ApiIgnore 
标注在类或方法上，表示忽略这个类或方法

## Mybatis注解：
### @Insert、@Update、@Delete、@Select
这四个注解分别代表将会被执行的 SQL 语句。它们用字符串数组（或单个字符串）作为参数。如果传递的是字符串数组，字符串之间先会被填充一个空格再连接成单个完整的字符串。这有效避免了以 Java 代码构建 SQL 语句时的“丢失空格”的问题。然而，你也可以提前手动连接好字符串。属性有：value，填入的值是用来组成单个 SQL 语句的字符串数组。
 





 

 

