---
title: SpringBoot学习
date: 2023-08-17 16:36:01
tags: Spring
category: Java
---
## 一、SpringBoot简介

### 1. SpringBoot是什么？

产生背景：Spring开发变的越来越笨重，大量的XML文件、繁琐的配置、复杂的部署流程、整合第三方框架难度大等，导致开发效率低下

SpringBoot是一个用来简化Spring应用的初始创建和开发过程的框架，简化配置，实现快速开发

融合了整个Spring技术栈，JavaEE开发的一站式解决方案	

参考：Spring官网 https://spring.io/projects

### 2. 为什么使用SpringBoot？	

优点：

- 快速创建独立运行的Spring项目并与主流框架集成
- 内置Servlet容器，应用无需打成WAR包
- 使用starter(启动器)管理依赖并进行版本控制 
- 大量的自动配置，简化开发
- 提供准生产环境的运行时监控，如指标、健康检查、外部配置等
- 无需配置XML，没有冗余代码生成，开箱即用

## 二、第一个SpringBoot程序

### 1. 环境要求

- SpringBoot 2.x
- JDK 8及以上
- Maven 3.5及以上
- Tomcat 9及以上

### 2. 操作步骤

步骤：

1. 创建一个maven的java工程

   传统web应用需要创建一个web工程，后期要打成war包，然后放到tomcat中，太麻烦

   而SpringBoot应用只需要创建一个java工程，后期直接打成可执行的jar包，其内置tomcat

2. 导入SpringBoot的相关依赖

   ````xml
   <!-- 继承SpringBoot的父工程 -->
   <parent>
       <groupId>org.springframework.boot</groupId>
    	 	<artifactId>spring-boot-starter-parent</artifactId>
     	<version>2.7.13</version>
   </parent>
   <dependencies>
     	<!-- 添加web应用的starter -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   </dependencies>
   ````

3. 编写Controller

   ```java
   @Controller
   public class HelloController {
   
       @RequestMapping("/hello")
       @ResponseBody
       public String hello() {
           return "Hello World";
       }
   }
   ```

4. 编写主程序类，用来启动SpringBoot应用

   ```java
   /**
    * 使用@SpringBootApplication标注主程序类，表示这是一个SpringBoot应用
    */
   @SpringBootApplication
   public class MainApplication {
       public static void main(String[] args) {
           // 启动SpringBoot应用
           SpringApplication.run(MainApplication.class, args); //传入主程序类的Class对象
       }
   }
   ```

   注：主程序类必须放到其他类的上层包中，因为默认只扫描主程序类所在的包及其子包

5. 运行主程序并访问测试

   http://localhost:8080/hello

6. 部署打包

   添加spring-boot-maven-plugin插件，将应用打成可执行的jar包，然后直接执行`java -jar springboot01-helloworld-1.0-SNAPSHOT.jar`

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```

### 3. 分析HelloWorld

POM文件：

- 父项目是spring-boot-starter-parent

  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.7.13</version>
  </parent>
  ```

  父项目的父项目是spring-boot-dependencies，用来管理SpringBoot应用中依赖的版本，进行版本控制	

  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.7.13</version>
  </parent>
  ```
  
- 通过启动器starter指定依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```


主程序类：

- @SpringBootApplication

  标注在类上，表示这是一个SpringBoot应用，通过运行该类的main方法来启动SpringBoot应用

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @SpringBootConfiguration
  @EnableAutoConfiguration
  @ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
  public @interface SpringBootApplication {
  ```

- @SpringBootConfiguration

  标注在类上，表示这个类是SpringBoot的配置类

  层级关系：@SpringBootConfiguration——>@Configuration——>@Component

  @Configuration标注在类上，表示这个类是Spring的配置类，相当于是一个xml配置文件

- @EnableAutoConfiguration

  开启自动配置功能，SpringBoot会自动完成许多配置，简化了以前繁琐的配置

- @ComponentScan

  标注在类上，指定要扫描的包，默认只扫描主程序类所在的包及其子包

  可以使用@ComponentScan手动指定要扫描的包


## 三、基本用法

### 1. 快速创建项目

步骤：

1. 使用Spring Initializer快速创建SpringBoot项目

​	选择SpringBoot 2.x 版本

​	勾选Web、Lombok模块

2. 基本操作

​	默认生成的.mvn、.gitignore、mvnw、mvnw.cmd，可以直接删除

​	resources文件夹的目录结构

```java
|-resources
	|-static // 存放静态资源，如css、js、imgs等
	|-templates // 存放模板页面，可以使用模板引擎，如thymeleaf、freemarker等
	|-application.properties // SpringBoot应用的配置文件，可以修改默认设置
```

​	注：SpringBoot生成的可执行jar包使用嵌入式的Tomcat，默认不支持JSP

3. 修改banner图案

​	在resources/目录下创建名为`banner.txt`的文件，内容如下生成

​	在线生成：https://patorjk.com/software/taag

​	经典图案 ：https://blog.csdn.net/vbirdbest/article/details/78995793

```java
::Spring Boot Version:: ${spring-boot.version}
////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//            佛祖保佑       永不宕机     永无BUG                    //
////////////////////////////////////////////////////////////////////
```

### 2. 配置文件

#### 2.1 简介

SpringBoot的默认全局配置文件有两个：

- application.properties
- application.yml

注：文件名固定，存放在`classpath:/` 或 `classpath:/config/`目录下

#### 2.2 YAML文件

YAML是专门用来写配置文件的语言，文件的后缀名为`.yml`，语法规则：

- 使用缩进表示层级关系，相同层级的元素左侧要对齐
- 键和值之间以冒号+空格隔开 `key: value`
- `#` 表示注释

```yaml
server:
  port: 8081
  servlet:
    context-path: /springboot02
```

#### 2.3 多环境切换

针对不同的环境，可以提供不同的配置，例如，不同环境下使用的数据库配置大多是不同的

常用的环境：

- 开发环境 dev

- 测试环境 test
- 生产环境 prod

可以通过命名约定来定义多个配置文件，格式：`application-{profile}.yml`

然后在application.yml文件中使用`spring.profiles.active` 来激活某一个环境配置

```yml
spring:
  profiles:
    active: dev
```

### 3. 热部署

​	使用SpringBoot提供的devtools实现热部署

​	原理：实时监控classpath下文件的变化（即编译后的target目录），如果发生变化则自动重启

​	配置：添加devtools的依赖即可（需要开启IDEA的自动编译）

```xml
<!-- devtools热部署 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
  	<scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**补充：开启IDEA的自动编译，IDEA默认是不自动编译的**

- Settings——>搜索Compiler——>勾选Build project automatically
- Help——>Find Action——>搜索Registry——>勾选compiler.automake.allow.parallel

### 4. 定义配置类

#### 4.1 添加组件

通过定义**配置类** 向容器中添加组件，使用注解：@Configuration和@Bean

```java
// 标注在类上，表示这是一个配置类，相当于以前编写的Spring配置文件
@Configuration
public class SpringConfig {

    // 标注在方法上，向容器中添加一个组件，将方法的返回值添加到容器中，方法名作为组件id
    @Bean
    public Date date(){
        Calendar c = Calendar.getInstance();
        c.set(2008,5,10);
        return c.getTime();
    }
}
```

#### 4.2 跨域访问

定义一个配置类，实现WebMvcConfigurer接口，重写addCorsMappings方法

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowCredentials(true)
                .allowedMethods("*")
                .maxAge(3600);
    }
}
```

### 5. 数据校验

#### 5.1 简介

数据校验就是数据的合法性检查，在服务器端也可以对数据进行校验，一般使用` JSR303` 校验

- JSR303是Java为Bean数据合法性校验提供的标准框架，是一种声明式校验
- JSR303通过在Bean属性上标注类似于@NotNull、@Max等注解来指定校验规则，并通过标准的验证接口对Bean进行验证

| 注解           | 功能                                                 |
| -------------- | ---------------------------------------------------- |
| @Null          | 必须为null                                           |
| @NotNull       | 不能为null                                           |
| @NotBlank      | 字符串不能为null，且长度大于 0，会去掉前后空格       |
| @Max(value)    | 数字必须小于等于指定值                               |
| @Min(value)    | 数字必须大于等于指定值                               |
| @Size(min,max) | 长度必须在指定的范围内（可以是字符串、数组、集合等） |
| @Past          | 时间必须是过去的时间                                 |
| @Future        | 时间必须是将来的时间                                 |
| @Pattern       | 必须符合指定的正则表达式                             |

JSR303的扩展： Hibernate Validator扩展注解

- Hibernate Validator是JSR303的一个参考实现，除支持所有标准的校验注解外，它还支持以下的扩展注解

| 注解                    | 功能                         |
| ----------------------- | ---------------------------- |
| @Length(min,max)        | 字符串长度必须在指定范围之间 |
| @NotEmpty               | 字符串不能为空               |
| @Email                  | 必须是合法的邮箱             |
| @Range(min,max,message) | 数值必须在指定的范围内       |

#### 5.2 基本用法

步骤：

1. 添加依赖 

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    ```

2. 在Bean上添加校验注解

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class User implements Serializable {
        private Integer id;
    
        @NotBlank(message = "用户名不能为空")
        private String username;
    
        @NotBlank(message = "密码不能为空")
        @Length(min = 6,max = 12,message = "密码长度必须在6~12之间")
        private String password;
    
        @Range(min = 18,max = 30,message = "年龄只能在18~30之间")
        private int age;
    
        @NotNull(message = "生日不能为空")
        @Past(message = "生日必须是过去的时间")
        @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
      	@DateTimeFormat(pattern = "yyyy-MM-dd")
        private Date birthday;
    }
    ```

3. 在方法形参前添加`@Valid`注解

    ```java
    @RestController
    public class AjaxController {
      
        @RequestMapping("/addUser")
        public String addUser(@Valid User user){ // @Valid 开启对User对象的数据校验
            System.out.println(user);
            return "success";
        }
    
    }
    ```

### 6. 定时任务

#### 6.1 简介

在项目开发中，经常需要编写定时任务来执行一些周期性的任务。

- 比如定时备份数据库、定时发送邮件、定时清理数据、定时提醒或通知等。
- 例如生活中网上购物时如果用户收到了商品，但一直没有确认收货，过了一个星期这个订单就需要自动转变为已收货状态，这种实时性不高的情景就可以用定时任务实现。

定时任务的实现：

- Quartz框架：是一个开源的任务调度框架

- Spring Boot内置定时任务

Cron表达式是一个字符串，由6个字段组成，每个字段使用空格隔开 ，用于指定定时任务的执行时间。也称为`七子表达式`

| 字段 | 取值范围  | 说明                 |
| ---- | --------- | -------------------- |
| 秒   | 0-59      |                      |
| 分   | 0-59      |                      |
| 时   | 0-23      |                      |
| 日   | 1-31      |                      |
| 月   | 1-12      |                      |
| 星期 | 0-7       | 0和7都表示星期日     |
| 年   | 1970~2099 | 此项非必需，可以省略 |

使用以下特殊字符来指定执行时间：

- 星号（*）：表示匹配该字段的所有可能值
- 问号（?）：表示不关心该字段具体的值
- 斜线（/）：表示指定一个间隔
- 逗号（,）：表示列举多个值
- 连字符（-）：表示指定一个范围

#### 6.2 基本用法

我们可以使用Spring Boot提供的定时任务，非常简单。

步骤：

1. 定制任务，使用`@Scheduled`

    ```java
    @Component
    public class MyScheduledTasks {
    
        @Scheduled(fixedRate = 3000) // 每隔3秒执行一次
        public void task1() {
            System.out.println("定时任务1执行了！");
        }
    
        @Scheduled(cron = "0 0 3 * * ?") // 每天凌晨3点执行
        public void task2() {
            System.out.println("定时任务2执行了！");
        }
    
    }
    ```

2. 启用定时任务，使用`@EnableScheduling`

    ```java
    @SpringBootApplication
    @EnableScheduling // 开启定时任务
    public class Springboot02QuickApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(Springboot02QuickApplication.class, args);
        }
    
    }
    ```

### 7. 日志

SLF4j是一个日志标准，提供了日志接口，并不是日志系统的具体实现，其具体实现有：log4j、log4j2、slf4j-simple、logback等。

SpringBoot默认使用的日志是logback

常见的日志级别： debug、info、warn、error

使用步骤：

1. 创建logback.xml日志配置文件，默认会自动加载该文件

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <configuration scan="true" scanPeriod="10 seconds">
    
        <!--日志存放目录-->
        <property name="LOG_HOME" value="./logs" />
    
        <!-- 控制台输出 -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <!-- 日志输出格式：%d表示日期，%thread表示线程名，%-5level表示级别从左显示5个字符宽度，%logger{50}表示logger名字最长50个字符，%msg表示日志消息，%n表示换行符 -->
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </encoder>
        </appender>
    
        <!-- 每天生成日志文件 DEBUG -->
        <appender name="DEBUG"  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <!--日志文件名-->
                <FileNamePattern>${LOG_HOME}/debug/manage-remote-debug.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
                <!--日志文件保留天数-->
                <MaxHistory>16</MaxHistory>
                <!--日志文件切割大小-->
                <maxFileSize>20MB</maxFileSize>
            </rollingPolicy>
            <!--日志输出格式-->
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </encoder>
            <!--过滤日志级别-->
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>DEBUG</level>
                <OnMatch>ACCEPT</OnMatch>
                <OnMismatch>DENY</OnMismatch>
            </filter>
        </appender>
    
        <!-- 每天生成日志文件 INFO -->
        <appender name="INFO"  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <FileNamePattern>${LOG_HOME}/info/manage-remote-info.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
                <MaxHistory>16</MaxHistory>
                <maxFileSize>20MB</maxFileSize>
            </rollingPolicy>
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </encoder>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>INFO</level>
                <OnMatch>ACCEPT</OnMatch>
                <OnMismatch>DENY</OnMismatch>
            </filter>
        </appender>
    
        <!-- 每天生成日志文件 WARN -->
        <appender name="WARN"  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <FileNamePattern>${LOG_HOME}/warn/manage-remote-warn.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
                <MaxHistory>24</MaxHistory>
                <maxFileSize>20MB</maxFileSize>
            </rollingPolicy>
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </encoder>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>WARN</level>
                <OnMatch>ACCEPT</OnMatch>
                <OnMismatch>DENY</OnMismatch>
            </filter>
        </appender>
    
        <!-- 每天生成日志文件 ERROR -->
        <appender name="ERROR"  class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <FileNamePattern>${LOG_HOME}/error/manage-remote-error.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
                <MaxHistory>32</MaxHistory>
                <maxFileSize>20MB</maxFileSize>
            </rollingPolicy>
            <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </encoder>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>ERROR</level>
                <OnMatch>ACCEPT</OnMatch>
                <OnMismatch>DENY</OnMismatch>
            </filter>
        </appender>
    
    
        <!-- 设置logger -->
        <logger name="net.wanho" level="debug"/>
    
        <!-- 设置root -->
        <root level="warn">
            <appender-ref ref="STDOUT" />
            <appender-ref ref="DEBUG" />
            <appender-ref ref="INFO" />
            <appender-ref ref="WARN" />
            <appender-ref ref="ERROR" />
        </root>
    
    </configuration>
    ```

2. 记录日志

    ```java
    @Slf4j // lombok注解，自动创建log对象
    public class LogTest {
    
        // private static final Logger log = LoggerFactory.getLogger(LogTest.class);
    
        @Test
        public void test() {
            log.debug("debug消息");
            log.info("info消息");
            log.warn("warn消息");
            log.error("error消息");
    
            try {
                System.out.println(5/0);
            } catch (Exception e) {
                log.error(e.getMessage());
                throw new RuntimeException(e);
            }
        }
    
    }
    ```

注：可以在IDEA中安装Grep Console插件，便于在控制台以不同颜色区别日志级别



## 四、整合MyBatis

```sql
drop database if exists springboot;
create database springboot charset utf8;
use springboot;

create table t_user(
    id int primary key auto_increment comment '编号',
    username varchar(100) not null unique comment '用户名',
    password varchar(50) comment '密码'
)engine innodb default charset utf8 comment '用户表';
```

### 1. MyBatis

步骤：

1. 创建一个工程，选择以下模块：Web、MyBatis、Lombok

   ```xml
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.38</version>
      	<scope>runtime</scope>
   </dependency>
   <dependency>
     	<groupId>com.alibaba</groupId>
     	<artifactId>druid-spring-boot-starter</artifactId>
     	<version>1.2.16</version>
   </dependency>
   <dependency>
       <groupId>com.github.pagehelper</groupId>
       <artifactId>pagehelper-spring-boot-starter</artifactId>
       <version>1.4.6</version>
   </dependency>
   ```

   注：SpringBoot默认使用的连接池是HikariCP，是一款轻量、高性能的连接池。

2. 配置application.yml

   ```yaml
   # 配置MyBatis
   mybatis:
     type-aliases-package: net.wanho.entity
     mapper-locations: classpath:mapper/*.xml
     configuration:
       # 打印sql
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
       # 下划线映射为驼峰
       map-underscore-to-camel-case: true
   
   # 配置PageHelper
   pagehelper:
     helper-dialect: mysql
   ```

3. 使用`@MapperScan`指定Dao接口所在的包

    ```java
    @SpringBootApplication
    //指定Dao接口所在的包，动态创建Dao实现类
    @MapperScan("net.wanho.mapper")
    public class Springboot04MybatisApplication {
        public static void main(String[] args) {
            SpringApplication.run(Springboot04MybatisApplication.class, args);
        }
    }
    ```

4. 配置Dao、Service、Controller等

    可以在Dao接口上添加`@Repository` 或 `@Mapper`，以解决在Service层注入时提示找不到bean的问题

### 2. MyBatis-Plus

参考：http://mp.baomidou.com

#### 2.1 基本用法

步骤：

1. 创建一个springboot工程，选择以下模块：Web、Lombok

2. 添加依赖 

    ```xml
    <!--mbyatis-plus-->
    <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.5.2</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.38</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      	<groupId>com.alibaba</groupId>
      	<artifactId>druid-spring-boot-starter</artifactId>
      	<version>1.2.16</version>
    </dependency>
    ```

3. 配置application.yml

    ```yaml
    # 配置DataSource
    spring:
      datasource:
      	type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/springboot?useUnicode=true&characterEncoding=utf8
        username: root
        password: root
        initialSize: 5
        maxActive: 50
        minIdle: 3
        maxWait: 5000
    
    # 配置MyBatis-Plus
    mybatis-plus:
      type-aliases-package: net.wanho.entity
      mapper-locations: classpath:mapper/*.xml
      configuration:
        # 打印sql
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
        # 下划线映射为驼峰
        map-underscore-to-camel-case: true
      global-config:
        db-config:
          # id-type: auto # 使用数据库的自动增长，不可作为分布式ID使用
          id-type: assign_id # 使用雪花算法自动生成long类型的数字，分布式的情况下可使用
    ```

4. 配置MybatisPlusConfig

    ```java
    @Configuration
    @MapperScan("net.wanho.mapper")
    public class MyBatisPlusConfig {
    
        /**
         * 配置MP的分页插件
         */
        @Bean
        public MybatisPlusInterceptor mybatisPlusInterceptor() {
            MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
            interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
            return interceptor;
        }
    }
    ```

5. 定义Mapper，继承BaseMapper

    ```java
    /**
     * 继承BaseMapper接口，通用Mapper
     */
    public interface UserMapper extends BaseMapper<User> {
    }
    ```


注：当数据库`表名和字段名` 与 `类名和属性名`不一致时，需要使用@TableName、@TableId、@TableField注解对实体类进行标注

6. 定义Service，使用通用Service

    ```java
    /**
     * 继承IService接口，通用Service
     */
    public interface UserService extends IService<User> {
    }
    
    /**
     * 继承ServiceImpl类，通用Service实现类
     */
    @Service
    @Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
    public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    }
    ```
    
7. 使用Mybatis-plus提供的接口



#### 2.2 逻辑删除

物理删除和逻辑删除：

- 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除的数据
- 逻辑删除：假删除，将对应数据中代表是否被删除字段的状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录

步骤：

1. 在表中添加逻辑删除标识列

    ```sql
    is_deleted tinyint default 0 comment '是否删除（0表示正常，1表示删除）'
    ```

2. 在实体类中添加逻辑删除属性 

```sql
@TableLogic
@TableField(value = "is_deleted")
private Integer isDeleted;
```

​	也可以进行全局配置

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: isDeleted  # 全局逻辑删除字段
      logic-delete-value: 1        # 标识删除的值(默认为1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为0)
```

3. 测试

    测试删除：实际上执行的是更新操作

```sql
-- 实际执行的SQL
update t_user set is_deleted=1 where id = 1 and is_deleted=0
```

​		测试查询：不会查询被逻辑删除的数据

```sql
-- 实际执行的SQL
select id,username,password,is_deleted t_from user where is_deleted=0
```

#### 2.3 自动填充

在对数据库表进行操作时经常会遇到一些字段，每次都使用相同的方式填充值，比如创建时间、更新时间等。

在阿里巴巴的开发手册中建议每个数据库表都必须要有create_time 和 update_time字段，以便记录每次操作的时间。

此时我们可以使用MyBatis Plus的自动填充功能，完成这些字段的自动填充赋值。

步骤：

1. 在表中添加列

    ```sql
    create_time datetime comment '创建时间',
    update_time datetime comment '更新时间'
    ```

2. 在实体类中添加属性 

```java
@TableField(fill = FieldFill.INSERT)
private Date createTime;

@TableField(fill = FieldFill.INSERT_UPDATE)
private Date updateTime;
```

3. 创建元对象处理器，实现MetaObjectHandler接口，用于对字段进行填充

```java
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ....");
        this.strictInsertFill(metaObject, "createTime", () -> new Date(), Date.class);
        this.strictInsertFill(metaObject, "updateTime", () -> new Date(), Date.class);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.strictUpdateFill(metaObject, "updateTime", () -> new Date(), Date.class);
    }
}
```

4. 测试

​		测试新增：会自动对create_time和update_time进行填充赋值

​		测试修改：会自动对update_time进行填充赋值

## 五、Restful API

### 1.简介

Representational State Transfer，简称为REST， 即表现层状态转化

Restful是一种网络应用程序的设计方式与开发方式，基于HTTP

- 表现层 Representational

    资源的表现层，指的是资源的具体呈现形式，如HTML、JSON等

- 状态转化 State Transfer

    指的是状态变化，通过HTTP方法来实现

    通过不同的请求方式实现不同的功能，实现表现层状态转化

    > GET		获取资源，即查询操作
    >
    > POST 	新建资源，即添加操作
    >
    > PUT		更新资源，即修改操作
    >
    > DELETE	删除资源，即删除操作 

### 2.设计原则

Restful 是目前最流行的 API 设计规范，用于 Web 数据接口的设计

Restful API设计原则：

- 尽量将API 部署在一个专用的域名下，如 `http://wanho.net/api` 或 `http://api.wanho.net` 

- API的版本应该在URL中体现，如 `http://wanho.net/api/v2` 

- URL中不要使用动词，应使用资源名词，且使用名词的复数形式，如：

    | 功能说明       | 请求类型 | URL                              |
    | -------------- | -------- | -------------------------------- |
    | 获取用户列表   | GET      | http://wanho.net/api/v2/users    |
    | 根据id获取用户 | GET      | http://wanho.net/api/v2/users/id |
    | 添加用户       | POST     | http://wanho.net/api/v2/users    |
    | 根据id删除用户 | DELETE   | http://wanho.net/api/v2/users/id |
    | 修改用户       | PUT      | http://wanho.net/api/v2/users    |

    注：简单来说，可以使用同一个 URL ，通过约定不同的 HTTP 方法来进行不同的业务操作

- 服务器响应时返回JSON对象，包含业务逻辑状态码、响应的消息、响应的查询结果


### 3. 基本用法

​	@GetMapping

​	@PostMapping

​	@PutMapping

​	@DeleteMapping

### 4. 接口测试Postman

​	Postman是一款非常优秀的调试工具，可以用来模拟发送各类HTTP请求，进行接口测试。

![postman](assets/postman.png)

## 六、API接口文档

### 1. 简介

通常情况下，我们会创建一份API文档来记录所有的接口细节，供其他开发人员使用提供的接口服务，但会存在以下的问题：

- 接口众多，并且细节复杂
- 需要根据接口的变化，不断修改API文档，非常麻烦，费时费力

Swagger的出现就是为了解决上述的这些问题，减少创建API文档的工作量

- 后端人员在代码里添加接口的说明内容，就能够生成可预览的API文档，无须再维护Word文档
- 让维护文档和修改代码整合为一体，在修改代码逻辑的同时方便的修改文档说明
- 提供了页面测试功能，便于对接口进行测试

Knife4j是对Swagger的封装，对接口文档UI进行了优化，并做了增强，用起来更方便

### 2. 使用步骤

使用步骤：

1. 添加依赖

    ```xml
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-spring-boot-starter</artifactId>
        <version>3.0.3</version>
    </dependency>
    ```

2. 创建配置类

    ```java
    @Configuration
    @EnableKnife4j // 启用Knife4j
    public class Knife4jConfig {
    
        /**
         * 创建Restful API文档内容
         */
        @Bean
        public Docket Api() {
            return new Docket(DocumentationType.SWAGGER_2)
                    .apiInfo(apiInfo())
                    .groupName("商城模块")
                    .select()
                    // 指定要暴露的接口所在包
                    .apis(RequestHandlerSelectors.basePackage("net.wanho.controller"))
                    .paths(PathSelectors.any())
                    .build();
        }
    
        /**
         * API的基本信息
         */
        private ApiInfo apiInfo() {
            return new ApiInfoBuilder()
                    .title("商城项目后端API接口文档")
                    .description("欢迎访问后端API接口文档")
                    .contact(new Contact("tangxiaoyang","https://github.com/tangyang8942","tangxiaoyang@qq.com"))
                    .version("1.0")
                    .build();
        }
    
    }
    ```

3. 配置application.yml

    ```yaml
    spring:
      mvc:
        pathmatch:
          matching-strategy: ant_path_matcher # 用于解决SpringBoot2.6+版本与Swagger的兼容性问题
    ```

4. 添加文档内容

    使用Swagger提供的注解对接口进行说明，常用注解：

    - @Api 标注在类上，对类进行说明
    - @ApiOperation  标注在方法上，对方法进行说明
    - @ApiParam 标注在参数上，对方法的参数进行说明
    - @ApiModel  标注在模型Model上，对模型进行说明
    - @ApiModelProperty  标注在属性上，对模型的属性进行说明
    - @ApiIgnore  标注在类或方法上，表示忽略这个类或方法

5. 查看接口文档页面，并测试接口

    启动SpringBoot程序，访问`http://localhost:端口/doc.html`，查看接口文档

    可以在接口文档中对接口进行测试

## 七、代码生成器Generator

### 1. 简介

MyBatis-Plus代码生成器，快速生成 Entity、Mapper、Service、Controller 层代码，支持模板引擎自定义配置

```sql
drop database if exists springboot;
create database springboot charset utf8;
use springboot;

create table t_user(
    id int primary key auto_increment comment '编号',
    username varchar(100) not null unique comment '用户名',
    password varchar(50) comment '密码',
  	is_deleted tinyint default 0 comment '是否删除（0表示正常，1表示删除）',
  	create_time datetime comment '创建时间',
		update_time datetime comment '更新时间'
)engine innodb default charset utf8 comment '用户表';

create table t_dept(
  id int primary key auto_increment comment '编号',
  name varchar(20) comment '部门名称'
)engine innodb default charset utf8 comment '部门表';

create table t_emp(
  id int primary key auto_increment comment '编号',
  name varchar(20) comment '姓名',
  salary double comment '工资',
  dept_id int comment '部门编号',
  foreign key (dept_id) references t_dept(id)
)engine innodb default charset utf8 comment '员工表';
```

### 2. 基本用法

步骤：

1. 创建一个SpringBoot工程，选择以下模块：Web、Lombok

2. 添加依赖 

    ```xml
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.2.16</version>
    </dependency>
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-spring-boot-starter</artifactId>
        <version>3.0.3</version>
    </dependency>
    <!--mbyatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.2</version>
    </dependency>
    <!--mybatis-plus-generator-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
        <version>3.4.1</version>
    </dependency>
    <!--beetl模板引擎-->
    <dependency>
        <groupId>com.ibeetl</groupId>
        <artifactId>beetl</artifactId>
        <version>3.15.4.RELEASE</version>
    </dependency>
    <!--guava-->
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>32.0.0-jre</version>
    </dependency>
    ```

3. 快速生成

    ```java
    public class CodeGenerator {
    
        private static String projectPath = "/Users/txy/code/IdeaProjects/framework/springboot05-generator";
    
        public static void main(String[] args) {
            // 代码生成器
            AutoGenerator generator = new AutoGenerator();
    
            // 1、全局配置
            GlobalConfig gc = new GlobalConfig();
            gc.setOutputDir(projectPath + "/src/main/java");
            gc.setAuthor("汤小洋");
            gc.setOpen(false);
            gc.setFileOverride(true); // 重新生成时文件是否覆盖
            gc.setServiceName("%sService"); // Service接口的命名方式
            gc.setDateType(DateType.ONLY_DATE);
            gc.setSwagger2(true);
            gc.setBaseResultMap(true);
            gc.setBaseColumnList(true);
            generator.setGlobalConfig(gc);
    
            // 2、数据源配置
            DataSourceConfig dsc = new DataSourceConfig();
            dsc.setUrl("jdbc:mysql://localhost:3306/springboot?useUnicode=true&useSSL=false&characterEncoding=utf8");
            dsc.setDriverName("com.mysql.jdbc.Driver");
            dsc.setUsername("root");
            dsc.setPassword("root");
            generator.setDataSource(dsc);
    
            // 3、包配置
            PackageConfig pc = new PackageConfig();
            pc.setParent("net.wanho");
            pc.setEntity("po"); // entity 层
            pc.setMapper("mapper"); // mapper 层
            pc.setService("service"); // service层
            pc.setController("controller"); // controller 层
            generator.setPackageInfo(pc);
    
            // 4、自定义配置
            InjectionConfig cfg = new InjectionConfig() {
                @Override
                public void initMap() {
                    // to do nothing
                }
            };
    
            // 5、策略配置
            StrategyConfig strategy = new StrategyConfig();
            strategy.setTablePrefix("t_"); // 过滤表前辍
            strategy.setNaming(NamingStrategy.underline_to_camel); //数据库表映射到实体的命名策略
            strategy.setColumnNaming(NamingStrategy.underline_to_camel); //数据库列映射到实体属性的命名策略
            strategy.setEntityLombokModel(true);  // lombok 模型
            strategy.setRestControllerStyle(true); //restful api风格控制器
            strategy.setControllerMappingHyphenStyle(true); //url中驼峰转连字符
            // 逻辑删除
            strategy.setLogicDeleteFieldName("is_deleted");
            // 自动填充
            List<TableFill> tableFillList = new ArrayList<>();
            tableFillList.add(new TableFill("create_time", FieldFill.INSERT));
            tableFillList.add(new TableFill("update_time", FieldFill.INSERT_UPDATE));
            strategy.setTableFillList(tableFillList);
    
            generator.setStrategy(strategy);
    
            // 6、设置解析模板
            generator.setTemplateEngine(new BeetlTemplateEngine() {
    
                // 获取主键
                String primaryKey = "";
                String primaryKeyUnderline="";
    
                @Override
                public AbstractTemplateEngine init(ConfigBuilder configBuilder) {
                    AbstractTemplateEngine engine = super.init(configBuilder);
                    // 映射文件放到resources目录下
                    configBuilder.getPathInfo().put("xml_path", projectPath + "/src/main/resources/mapper/" + pc.getModuleName());
                    return engine;
                }
    
                @Override
                public Map<String, Object> getObjectMap(TableInfo tableInfo) {
                    for (TableField tableField : tableInfo.getFields()) {
                        if (tableField.isKeyFlag()) {
                            primaryKey = tableField.getPropertyName();
                            primaryKeyUnderline = tableField.getName();
                            break;
                        }
                    }
    
                    // 获取源码中的map
                    Map objectMap = super.getObjectMap(tableInfo);
                    // 额外添加两个键值对
                    objectMap.put("entityCamel", CaseFormat.UPPER_CAMEL.to(CaseFormat.LOWER_CAMEL, tableInfo.getEntityName()));
                    objectMap.put("primaryKey", primaryKey);
                    objectMap.put("primaryKeyUnderline", primaryKeyUnderline);
                    return objectMap;
                }
            });
          
            // 7、执行生成
            generator.execute();
        }
    
    }
    ```

4. 自定义模板

    可以根据需求自定义模板

    - entity.java.btl
    - mapper.java.btl
    - mapper.xml.btl
    - service.java.btl
    - serviceImpl.java.btl
    - controller.java.btl

5. 测试

### 3. 多表查询

MyBatis-Plus默认只能完成单表的操作，多表操作仍然需要自己写。

步骤：

1. 在Emp实体类中添加属性

    ```java
    @ApiModelProperty(value = "部门")
    @TableField(exist = false) // 表示该属性不是数据库表中的字段
    private Dept dept;
    ```

2. 编辑EmpMapper.java和EmpMapper.xml

    ```java
    public interface EmpMapper extends BaseMapper<Emp> {
    
        List<Emp> selectAll();
    
        IPage<Emp> selectByPage(IPage<Emp> page, @Param("ew") Wrapper<Emp> queryWrapper);
    }
    ```

    ```xml
    <resultMap id="EmpMap" type="Emp" extends="BaseResultMap">
      <association property="dept" javaType="Dept">
        <id property="id" column="dept_id"/>
        <result property="name" column="dept_name"/>
      </association>
    </resultMap>
    
    <select id="selectAll" resultMap="EmpMap">
        select e.*, d.name as dept_name
        from t_emp e left join t_dept d on e.dept_id = d.id
    </select>
    
    <select id="selectByPage" resultMap="EmpMap">
        select e.*, d.name as dept_name
        from t_emp e left join t_dept d on e.dept_id = d.id
        ${ew.customSqlSegment}
    </select>
    ```

3. 编辑EmpService和EmpServiceImpl

    ```java
    public interface EmpService extends IService<Emp> {
    
        List<Emp> findAll();
    
        IPage<Emp> findByPage(IPage<Emp> page, @Param("ew") Wrapper<Emp> queryWrapper);
    }
    ```

    

    ```java
    @Service
    public class EmpServiceImpl extends ServiceImpl<EmpMapper, Emp> implements EmpService {
    
        @Override
        public List<Emp> findAll() {
            return this.baseMapper.selectAll();
        }
    
        @Override
        public IPage<Emp> findByPage(IPage<Emp> page, Wrapper<Emp> queryWrapper) {
            return this.baseMapper.selectByPage(page,queryWrapper);
        }
    }
    ```

4. 测试

    ```java
    @SpringBootTest
    class EmpServiceImplTest {
    
        @Resource
        private EmpService empService;
    
        @Test
        public void findAll() {
            System.out.println(empService.findAll());
        }
    
        @Test
        public void findByPage() {
            Page<Emp> page = new Page<>(1, 3);
            QueryWrapper<Emp> wrapper = new QueryWrapper<>();
            wrapper.likeRight("e.name", "1");
            empService.findByPage(page, wrapper);
    
            System.out.println(page.getRecords());
        }
    }
    ```

## 八、MyBatis-Plus-Join

### 1. 简介

官网 https://mybatisplusjoin.com/

### 2. 基本用法

步骤：

1. 添加依赖 

    ```xml
    <!--mybatis-plus-join-->
    <dependency>
       <groupId>com.github.yulichang</groupId>
       <artifactId>mybatis-plus-join-boot-starter</artifactId>
       <version>1.4.5</version>
    </dependency>
    ```

2. 编辑EmpMapper.java，继承自MPJBaseMapper

    ```java
    public interface EmpMapper extends MPJBaseMapper<Emp> {
    }
    ```

3. 测试

    ```java
    @SpringBootTest
    class EmpMapperTest {
    
        @Resource
        private EmpMapper empMapper;
    
        @Test
        public void selectAll(){
            MPJLambdaWrapper<Emp> wrapper = new MPJLambdaWrapper<Emp>()
                    .selectAll(Emp.class) // 查询Emp类的所有字段
                    .selectAs(Dept::getName, EmpDTO::getDeptName)// 查询Dept类的name字段
                    .leftJoin(Dept.class, Dept::getId, Emp::getDeptId) // 左连接
                    .orderByDesc(Emp::getId);
    
            List<EmpDTO> list = empMapper.selectJoinList(EmpDTO.class, wrapper);
            list.forEach(System.out::println);
        }
    
      
        @Test
        public void selectByPage(){
            Page<EmpDTO> page = new Page<>(1, 3);
    
            empMapper.selectJoinPage(page, EmpDTO.class,
                    new MPJLambdaWrapper<Emp>()
                        .selectAll(Emp.class)
                        .selectAs(Dept::getName, EmpDTO::getDeptName)
                        .leftJoin(Dept.class, Dept::getId, Emp::getDeptId)
                        .eq(Dept::getId, 1));
    
            page.getRecords().forEach(System.out::println);
        }
    
    }
    ```

## 九、综合案例

基于SpringBoot的日记管理系统（diary）

- 后端 SpringBoot

- 前端 LayUI

    
