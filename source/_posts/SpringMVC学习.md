---
title: SpringMVC学习
date: 2023-08-14 14:41:15
tags: Spring
category: Java
---


## 一、SpringMVC简介

### 1. 什么是MVC

​	M：model   数据模型，封装了业务逻辑，对业务数据进行处理

​	V：view  视图，封装了显示逻辑，如HTML、JSP、Excel、PDF等

​	C：controller 控制器，控制整个网站的处理流程，协调视图与模型

​	MVC是一种Web应用架构，是一种代码设计思想

​	思想：将所有客户端请求全部交由控制器，由控制器将其分发，并将结果响应回客户端

​	注：区别MVC和三层架构

### 2. SpringMVC优点

​	简单，使用注解配置替代原生XML配置

​	效率高，单例的，将Controller层对象交给IoC容器管理

​	扩展性好，方便用户自定义

​	SpringMVC和Spring无缝衔接

### 3. 实现原理

#### 3.1 流程图

![springMVC](assets/springMVC.JPG)

#### 3.2 执行过程

分为六步：

- DispatcherServlet

    SpringMVC核心控制器：主要作用是用来分发，不进行任何处理

- HandlerMapping

    映射处理器：根据请求URL映射到具体的处理器Handler

    Handler就是Controller层实现类，也可称为Controller或Action

- HandlerAdapter

    适配器：用来适配不同的处理器Handler

    处理器有两种实现方式：实现接口、基于注解，所以执行之前需要先适配，这样才能知道如何执行

- Handler

    处理器：执行处理具体业务，并产生数据模型Model和视图名View

    Handler将数据模型和视图封装成ModelAndView对象并返回

- ViewResolver

    视图解析器：根据视图名解析为具体的视图，一般多为jsp页面，然后封装为View对象

- View

    视图：使用具体视图技术进行渲染，结合Model展示数据

    视图有很多种形式，如jsp、freemarker、velocity、pdf、excel等

## 二、第一个SpringMVC程序

### 1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
</dependency>
```

### 2. 创建Controller

```java
@Controller
public class HelloController {
	@RequestMapping("/hello")
	public ModelAndView sayHello(String name){
		ModelAndView mav = new ModelAndView();
		mav.addObject("msg","Hello "+name);
		mav.setViewName("hello");
		return mav;
	}
}  
```

### 3. 核心配置文件	

名称自定义，如springmvc.xml

```xml
<!-- 扫包 -->
<context:component-scan base-package="controller"/>

<!-- mvc的注解驱动 -->
<mvc:annotation-driven/>

<!-- 配置ViewResolver -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="/view/"/>
  <property name="suffix" value=".jsp"/>
</bean>
```

### 4. 配置核心控制器

在web.xml中配置SpringMVC核心控制器，需要指定配置文件的路径

```xml
<!-- 配置DispatcherServlet，核心控制器，本质上就是一个Servlet -->
<servlet>
  <servlet-name>springMVC</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>springMVC</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 5. 静态资源处理

当配置DispatcherServlet的url-pattern为`/`时，会拦截所有请求（包括静态资源），导致所有静态资源都无法访问

两种处理方式：

- 使用tomcat提供的默认Servlet	

    ```xml
    <mvc:default-servlet-handler/>
    ```

- 使用SpringMVC提供的处理方式

    ```xml
    <mvc:resources mapping="/imgs/**" location="/WEB-INF/imgs/" />
    ```

## 三、方法的返回值

### 1. 返回值类型

共有四种类型：

- ModelAndView 表示数据模型和视图
- String 三种形式（写法）：
  - 普通字符串 ——> 表示视图名称，转发到指定视图
  - forward:url ——>  转发到指定url  
  - redirect:url ——>  重定向到指定url                         			
  
- void 表示使用HttpServletResponse处理响应
- Object 一般需要结合@ResponseBody使用

### 2. @ResponseBody

将方法返回值写到响应体中，一般用来处理Ajax请求，返回JSON数据

SpringMVC默认使用`jackson`处理json数据的转换，所以需要引入jackson的依赖

可以使用`@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")`，设置响应时的日期格式

```xml
<!--jackson-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.0</version>
</dependency>
```

注：也可以使用@RestController，相当于@Controller+@ResponseBody

常用注解：

- @JsonFormat 指定序列化和反序列化时的格式，一般用于指定日期、时间和数字的格式
- @JsonProperty 指定序列化和反序列化时的名称
- @JsonAlias 指定反序列化时的备用别名
- @JsonIgnore 指定序列化和反序列化时忽略属性

注：

- json序列化：将Java对象转换为json字符串
- json反序列化：将json字符串转换为Java对象

### 3. 跨域访问

允许跨域访问

- 方式一：使用@CrossOrigin注解，可以在类或方法上添加该注解

- 方式二：在xml中添加如下配置

    ```xml
    <mvc:cors>
      <mvc:mapping path="/**"/>
    </mvc:cors>
    ```

## 四、方法的参数

### 1. JavaEE组件

HttpServletRequest

HttpServletResponse

HttpSession

### 2. String、基本类型及包装类

@RequestParam	表示参数来源于请求参数，默认为所有参数添加该注解，参数值来源于同名称的请求参数

@PathVariable		表示参数来源于URL（URL就是请求路径）

@RequestHeader  表示参数来源于请求头

@CookieValue		表示参数来源于Cookie

@RequestBody	  表示参数来源于请求体，用来接收前端传递给后端的 json 格式的数据

- 请求的方式必须为post，post请求才会有请求体

- 请求的内容类型必须为 json格式，需要设置`contentType`为`application/json;charset=utf8`

### 4. 自定义对象

例如：User、UserDTO、UserVo等

@ModelAttribute	将请求数据转换为对象，默认为所有自定义类型添加该注解

- 要求对象的属性名必须与参数名相同
- 可以使用`@DateTimeFormat(pattern="yyyy-MM-dd")`，设置接收时的日期格式

注意：

- 可以使用SpringMVC提供的`CharacterEncodingFilter`解决POST请求中文乱码问题
- 控制台输出有乱码时，可以设置tomcat的VM options为`-Dfile.encoding=utf8`

## 五、请求路径

### 1. @RequestMapping

该注解可以定义在方法上，也可以定义在类上，表示层级关系

请求映射路径的写法：

- 固定写法

  value和path属性互为别名，其值是一个数组，可以指定多个值，允许通过多个路径访问

- Rest风格

  {变量}表示URL中的占位符，URL中必须有对应的值，可以结合@PathVariable获取值

### 2. 限定请求方式

method属性

- 限定请求方式：GET、POST、PUT、DELETE等	
- 也可使用@GetMapping、@PostMapping、@PutMapping、@DeleteMapping等注解限定请求方式

## 六、全局异常处理

步骤：

1. 定义一个异常处理类，添加@ControllerAdvice或@RestControllerAdvice
2. 定义异常处理方法，添加@ExceptionHandler

## 七、拦截器

拦截器Interceptor对请求进行拦截处理，类似于过滤器Filter

步骤：

1. 定义一个类，实现HandlerInterceptor接口

```java
public class HelloInterceptor implements HandlerInterceptor {

    // 调用目标处理方法之前执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("HelloInterceptor.preHandle");
        
        return true; // true表示放行，继续调用目标处理方法，false表示不放行
    }

    // 调用目标处理方法之后执行，渲染视图之前
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("HelloInterceptor.postHandle");
    }

    // 渲染视图之后
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("HelloInterceptor.afterCompletion");
    }
}

```

2. 配置拦截器

```xml
<mvc:interceptors>
  <mvc:interceptor>
    <mvc:mapping path="/hello" />
    <mvc:mapping path="/path/**" />
    <mvc:exclude-mapping path="/path/path1"/> <!-- 排除拦截该请求 -->
    <bean class="interceptor.HelloInterceptor"></bean>
  </mvc:interceptor>
</mvc:interceptors>
```

## 八、文件上传

SpringMVC为文件上传提供了支持，基于`commons-fileupload`

步骤：

1. 添加依赖

   ```xml
   <dependency>
     <groupId>commons-fileupload</groupId>
     <artifactId>commons-fileupload</artifactId>
     <version>1.3.1</version>
   </dependency>
   ```

2. 配置文件解析器

   ```xml
   <!-- 配置文件解析器，id名称必须为multipartResolver -->
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
     <property name="defaultEncoding" value="utf-8"/>
     <property name="maxUploadSize" value="5000000"/>
   </bean>
   ```

3. 编写页面

   ```jsp
   <form id="uploadForm">
       姓名：<input type="text" name="name" id="name"> <br>
       头像：<input type="file" name="avatar" id="avatar"> <br>
       <input type="button" value="提交" onclick="doUpload()">
   </form>
   ```
   
4. 编写FileController类

   ```java
   /*
    * 通过参数CommonsMultipartFile接收文件
    * 必须在参数前添加@RequestParam注解，否则无法接收文件
    */
   @PostMapping("/upload")
   public AjaxResult upload2(String name, @RequestParam CommonsMultipartFile avatar, HttpServletRequest req) throws IOException {
       System.out.println(name);
       System.out.println(avatar);
       System.out.println(avatar.getOriginalFilename()); // 文件名
       System.out.println(avatar.getSize()); // 文件大小
       System.out.println(avatar.getInputStream()); // 文件的输入流
   
       String uploadPath = req.getServletContext().getRealPath("upload");
   
       String filename = UUID.randomUUID() + "." + FileNameUtil.getSuffix(avatar.getOriginalFilename());
       String filePath = uploadPath + File.separator + filename;
       System.out.println(filePath);
       avatar.transferTo(new File(filePath));
   
       return AjaxResult.success(filename);
   }
   ```

​	注：maven打包时默认会忽略空目录，不对空目录进行打包，可以先手动创建一个文件

## 九、综合案例

基于SSM的员工管理系统（ssm-ems）

- 后端 SSM
- 前端 LayUI
