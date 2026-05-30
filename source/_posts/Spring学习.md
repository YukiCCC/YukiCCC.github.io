---
title: Spring学习
date: 2023-08-10 10:57:52
tags: Spring
category: Java
---

## 一、Spring简介

### 1. Spring是什么？

​	spring单词本义是“春天”，程序员的春天

​	是一个开源的控制反转(IoC)和面向切面(AOP)的容器框架，用来简化企业开发

​	官网：https://spring.io

### 2. 为什么使用Spring

​	降低组件之间的耦合度，实现软件各层之间的解耦

​	提供了众多的技术支持

​	对主流框架提供了集成

### 3. 核心概念

IoC：Inversion Of Control 控制反转

- 控制反转(IoC)就是应用本身不负责依赖对象的创建及维护，依赖对象的创建及维护由外部容器负责，这样控制权就由应用转移到了外部容器，控制权的转移就是所谓反转。
- 外部容器/IoC容器：存放对象（bean）的容器

```java
//Service依赖于DAO
public class UserServiceImpl{
      //UserDaoImpl由应用内部(Service)创建及维护
      private UserDao userDao = new UserDaoImpl();
      public void regist(User user){
            userDao.insert(user);
     }
}
```

DI：Dependency Injection 依赖注入

- 依赖注入(DI)就是指在运行期，由外部容器动态地将依赖对象注入到组件中。

```java
//依赖对象的创建及维护由外部容器负责
public class UserServiceImpl{
      private UserDao userDao;
      //通过setter方法，让容器把创建好的依赖对象注入到Service中
      public void setUserDao(UserDao userDao){
				this.userDao=userDao
      }	
      public void regist(User user){
            userDao.insert(user);
     }
}
```

## 二、第一个Spring程序

### 1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.27</version>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.24</version>
  <scope>provided</scope>
</dependency>
```

### 2. 核心配置文件

​	用来进行bean的配置，文件名可自定义，一般默认为applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd ">
	<!-- 
		定义一个bean
			id：指定bean的名称
			class：指定bean的类型
	-->
	<bean id="helloSpring" class="ioc01.HelloSpring">
		<!-- 为bean中的属性注入值 -->
        <property name="name" value="tom"/>
	</bean>
</beans>
```

### 3. 操作

```java
public static void main(String[] args) {
  // 获取IoC容器，读取配置文件，初始化Spring上下文
  ApplicationContext ac = new ClassPathXmlApplicationContext("ioc01/applicationContext.xml");
  // 根据id名称获取bean的实例
  HelloSpring helloSpring=(HelloSpring) ac.getBean("helloSpring");
  helloSpring.show();
}
```

### 4. 案例

用户登陆

```xml
<bean id="userDao" class="ioc02.dao.impl.UserDaoImpl"/>

<bean id="userService" class="ioc02.service.impl.UserServiceImpl">
    <!-- 通过ref属性注入bean -->
    <property name="userDao" ref="userDao"/>
</bean>

<bean id="userController" class="ioc02.controller.UserController">
    <property name="userService" ref="userService"/>
</bean>
```

## 三、数据装配

### 1. 简介

为bean中的属性注入值，称为数据的装配，可装配不同类型的数据：

- 简单类型 ——> 使用value

  ​	基本类型及包装类型、String

- 其他bean的引用 ——> 使用ref

- 集合类型 ——>使用相对应的子元素

  数组 List Set Map Properties


注：数据装配是通过setter方法进行的，所以必须有setter方法

### 2. 基本用法

```xml
<bean id="springBean" class="ioc.SpringBean">
	<property name="name" value="tom"/>
  <property name="otherBean" ref="otherBean"/>
  <property name="arrays">
    <array>
      <value>1</value>
      <value>2</value>
      <value>3</value>
    </array>
  </property>
  <property name="lists">
      <list>
          <ref bean="otherBean"/>
          <bean class="ioc03.OtherBean">
              <property name="sex" value="male"/>
          </bean>
      </list>
  </property>

  <property name="map">
      <map>
          <entry key="aaa" value-ref="otherBean"/>
          <entry key="bbb" value-ref="otherBean"/>
      </map>
  </property>

  <property name="properties">
      <props>
          <prop key="driver">com.mysql.jdbc.Driver</prop>
          <prop key="url">jdbc:mysql://localhost:3306/test</prop>
      </props>
  </property>
</bean>
```

### 3. 自动装配

对于**其他bean的引用**的装配，IoC容器可以根据bean的名称、类型或构造方法进行自动的注入，称为自动装配

通过bean元素的autowire来配置

```xml
<!-- 
  自动装配 
  autowire 可取值如下：
   	no：不进行自动装配，默认值，同default
   	byName：根据属性名自动装配，自动查找与属性名相同的bean
   	byType：根据属性类型自动装配，自动查找与属性类型相同的bean
		constructor：根据构造方法自动装配
     		根据构造方法参数的名称或类型进行自动装配，只要有一个可以装配就行
     		此时与setter方法无关，可以没有setter方法   	
 -->
<bean id="springBean" class="ioc.SpringBean" autowire="constructor"/>
```

## 四、bean管理

### 1. bean生命周期

​	生命周期：实例化-->数据装配-->初始化方法-->使用-->销毁方法-->从容器中销毁

```xml
<!-- 生命周期的扩展 init destory -->
<bean id="springBean" class="ioc.SpringBean" init-method="init" destroy-method="destory">
  <property name="name" value="admin"></property>
</bean>
```

​	在bean实例化并进行数据装配后，会调用初始化方法，执行初始化操作

​	在bean销毁之前会调用销毁方法，执行销毁前的操作

### 2. bean实例化时机

​	默认预先实例化，即在容器启动时实例化，可以设置懒实例化，在第一次使用bean时实例化

```xml
<bean id="springBean" class="ioc.SpringBean" lazy-init="true">
  <property name="name" value="admin"></property>
</bean>
```

### 3. bean作用域

​	在IoC容器中bean默认是单例的

​	通过bean元素的scope属性来设置bean的作用域，即是否为单例模式

```xml
<!--  
  scope属性 可取值如下：
   singleton：单例，在容器启动时初始化，默认值
   prototype：多例，在每次获取bean时初始化
 -->
<bean id="springBean" class="ioc.SpringBean" scope="prototype"/>
```

### 4. 在普通类中获取bean

实际开发中，并不是所有类都需要交由IoC容器管理，某些类并不需要交由容器管理，如各种工具类，本身都提供了静态方法

- 问题：没有交由容器管理的类，容器是无法进行数据装配的，如何在这些类中获取容器中的bean呢？
- 解决：使用ApplicationContextAware获取容器，然后从容器中获取bean

定义IoC容器工具类，步骤：

1. 定义一个类，实现ApplicationContextAware接口
2. 将该类添加到IoC容器中，当实例化时会自动注入当前的ApplicationContext
3. 调用IoC容器工具类，从容器中获取bean

```java
public class ApplicationContextHolder implements ApplicationContextAware{

    private static ApplicationContext ac;

    /**
     * 会自动注入当前的IoC容器
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.ac = applicationContext;
    }

    /**
     * 根据名称获取bean
     * @param beanName
     * @return
     */
    public static Object getBean(String beanName){
        return ac.getBean(beanName);
    }

    /**
     * 根据类型获取bean
     * @param clazz
     * @return
     * @param <T>
     */
    public static <T> T getBean(Class<T> clazz){
        return ac.getBean(clazz);
    }

}
```

## 五、注解配置

### 1. 简介

​	Spring提供了一系列注解来替代配置文件，简化配置

​	使用注解时，需要扫描注解所在的包

```xml
<context:component-scan base-package="net.wanho.dao.impl"/>
<context:component-scan base-package="net.wanho.service.impl"/>
<context:component-scan base-package="net.wanho.controller"/>
```

### 2. 常用注解

#### 2.1 定义组件

@Component 定义组件Bean，添加到IoC容器中，不区分组件类型

区分组件类型的注解：

- @Repository 表示Dao组件
- @Service 表示Service组件
- @Controller 表示Controller组件

注：以上三个注解和@Component的作用相同，只是用来表示不同的组件类型

#### 2.2 数据装配

注解方式的数据装配是直接使用`属性`进行注入，不是使用setter方法，所以可以没有setter方法

简单类型

```java
@Value("666")
private int num;

@Value("true")
private Boolean flag;

// @Value("tom")
@Value("${username}")
private String username;
```

```xml
<!-- 引用属性文件 -->
<context:property-placeholder location="classpath:ioc/info.properties"/>
```

其他bean的引用，使用`@Autowired` 或 `@Resource`

```java
// @Autowired // Spring提供的注解，默认按类型注入，如果有多个同类型的，则按名称注入
@Resource // Java提供的注解，默认按名称注入，如果没找到，则按类型注入
private OtherBean otherBean;

/*
 * 方式1：基于field注入，即在属性上面添加@Autowired或@Resource
 */
@Service
public class UserServiceImpl implements UserService {
    @Resource
    private UserDao userDao;
}

/*
 * 方式2：基于constructor注入，即在构造方法上面添加@Autowired或@Resource
 *      优点：可以为final属性注入、质量高（Spring官方推荐）
 */
@Controller
@RequiredArgsConstructor // 可以使用lombok的@RequiredArgsConstructor注解来生成带参构造方法（属性要使用final修饰）
public class UserController {

    private final UserService userService;

    // @Autowired // 如果类中只有一个构造方法，可以省略该注解
    // public UserController(UserService userService) {
    //     this.userService = userService;
    // }
}
```

#### 	2.3 bean生命周期

```java
// 相当于init-method
@PostConstruct
public void init() {
  System.out.println("SpringBean.init()");
}

// 相当于destroy-method
@PreDestroy
public void destroy() {
  System.out.println("SpringBean.destroy()");
}
```

#### 2.4 bean实例化时机

```java
// 默认是预实例化，配置为懒初始化
@Lazy
public class SpringBean {}
```

#### 2.5 scope作用域

```java
//默认是单例，配置为非单例
@Scope("prototype")
public class SpringBean {}
```

## 六、AOP

### 1. 简介

​	AOP：Aspect Oriented Programming 面向切面编程，是OOP面向对象编程的一种补充

​	将程序中的交叉业务逻辑（事务、日志、异常等）代码提取出来，封装成切面，由AOP容器在适当的时机（位置）将封装的切面动态的织入到具体业务逻辑中。

### 2. 作用

- 在不改变原有代码的基础上动态添加新的功能
- 模块化，将分散在各层中的相同代码，通过横向切割的方式抽取到单独的模块中
- 方便维护
- 可扩展性强

### 3. 原理

**AOP的原理是使用动态代理技术**

动态代理的含义：

- 代理类是在程序运行期间由JVM根据反射等机制动态生成的，自动生成代理类和代理对象
- 所谓动态是指在程序运行前不存在代理类的字节码文件，代理类和委托类的关系是在程序运行时确定

动态代理的两种技术：

- jdk技术：适用于有接口时使用，目标对象必须实现一个或多个接口，否则无法使用jdk动态代理
- cglib技术：适用于无接口时使用（有接口时也可以使用）

注：Spring默认使用的是jdk动态代理，SpringBoot默认使用的是cglib动态代理

### 4. 用法

步骤：

1. 添加依赖

   ```xml
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-aspects</artifactId>
   </dependency>
   ```
   
2. 配置Advice

   定义增强类并添加@Component和@Aspect注解，表示其为一个切面

3. 配置Pointcut

   定义切点表达式

   - 在一个空方法上添加@Pointcut注解，配置切点表达式

   为方法添加通知类型注解并指定切点

   - @Before   前置通知，在方法执行前添加功能
   - @AfterReturning 后置通知，在方法执行后添加功能
   - @AfterThrowing 异常通知，在方法抛出异常后添加功能
   - @Around 环绕通知 在方法执行前后添加功能

4. 配置自动创建代理并织入

   ```xml
   <aop:aspectj-autoproxy />
   ```

## 七、Spring整合JDBC

```sql
drop database if exists spring;
create database spring charset utf8;
use spring;

create table t_user
(
  id int primary key auto_increment,
  username varchar(50) not null unique,
  password varchar(50) not null
) charset utf8;
```

### 1. 基本用法

步骤：

1. 添加依赖

   ```xml
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-jdbc</artifactId>
   </dependency>
   <!--mysql-->
   <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>5.1.46</version>
   </dependency>
   <dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>druid</artifactId>
     <version>1.1.10</version>
   </dependency>
   ```
   
2. 配置

   DataSource-->JdbcTemplate-->Dao-->Service-->Controller

   ```java
   @Repository
   public class UserDaoJdbcImpl implements UserDao {
   	@Autowired
   	private JdbcTemplate jdbcTemplate;
   	// ...
   }
   ```

   ```xml
   <context:property-placeholder location="classpath:datasource.properties"/>
   
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
     <property name="driverClassName" value="${jdbc.driverClassName}"></property>
     <property name="url" value="${jdbc.url}"></property>
     <property name="username" value="${jdbc.username}"></property>
     <property name="password" value="${jdbc.password}"></property>
     <property name="initialSize" value="${jdbc.initialSize}"></property>
     <property name="maxActive" value="${jdbc.maxActive}"></property>
   </bean>
   
   <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
     <property name="dataSource" ref="dataSource"></property>
   </bean>
   ```

### 2. 事务操作

​	JDBC默认是自动提交事务的，每执行完一条SQL语句就提交事务

​	问题：如果Service层出现异常，并不会回滚，怎么办？

​	解决：配置事务操作（使用的是环绕通知）

```xml
<!-- 定义事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 事务的注解驱动 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

```java
// @Transactional也可以配置在类上，如果方法上没有配置@Transactional，则使用类上的事务配置
@Transactional(propagation = Propagation.REQUIRED,rollbackFor = Exception.class)
public class UserServiceImpl implements UserService {
  	@Autowired
	private UserDao userDao;
  
	@Override
	@Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
	public User login(String username, String password) {
	}

	@Override
	//@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.READ_COMMITTED, noRollbackFor = ArithmeticException.class, timeout = 3000)
    public void regist(UserVo userVo) {
    }
}  
```

### 3. 事务属性和事务特性

#### 3.1 事务属性

​	五个事务属性：传播属性、隔离级别、回滚条件、只读优化、超时处理

1. 传播属性

    propagation：定义事务的边界，用来配置当前方法是否需要有事务，常用取值：

    - REQUIRED 必须添加事务，如果当前没有事务则创建一个新的事务，一般用于增删改
    - SUPPORTS 可以没有事务，如果当前有事务则运行，如果没有事务也可以运行，一般用于查询

2. 隔离级别

    isolation：用来解决事务并发时会出现的一些问题，四种隔离级别，由低到高依次为：

    - READ_UNCOMMITTED 读未提交——>可能出现脏读、不可重复读、幻读
    - READ_COMMITTED 读已提交——>避免脏读，但可能出现不可重复读、幻读
    - REPEATABLE_READ 可重复读——>避免脏读、不可重复读，但可能出现幻读（MySQL默认隔离级别）
    - SERIALIZABLE 可序列化（串行）——>避免脏读、不可重复读、幻读，相当于是单并发，没意义
    

 事务并发时会出现的三个问题（脏读、不可重复读、幻读或虚读）：

- 脏读: 一个事务读取到另一个事务中没有提交的数据，一般不会发生，如MySQL、Oracle底层默认都是只读取提交的数据

    ```sql
    -- 打开两个session，即同时登陆两个账户
    mysql -uroot -p
    set autocommit=off;  
    insert into t_user values(null,'alice','123');
    select * from t_user;
    ```

- 不可重复读 : 一个事务已经读取数据，另一个事务在修改数据，可能导致使用的数据与数据库不同步

- 幻读或虚读 : 一个事务已经读取数据量，另一个事务在添加或删除数据，可能导致使用的数据量与数据库不一致

    **注：**不可重复读和虚读都是小概率事件，实际开发中一般不需要配置隔离级别，大多是通过定时任务来检查+人工审核

3. 回滚条件

    rollback：默认抛出RuntimeException时才进行回滚

    ​	rollbackFor=""	表示发生该异常时回滚

    ​	noRollbackFor="" 表示发生该异常时不回滚

4. 只读优化

    readOnly：在该事务中只能读取，一般用于查询

5. 超时处理

    timeout：配置事务超时的时间，一般不配置

#### 3.2 事务特性

​	四个事务特性：原子性(Atomicity)、一致性(Consistency)、隔离性(Isolation)、永久性(Durability)

​	简称为ACID

## 八、Spring整合MyBatis

​	步骤：

1. 添加依赖

   ```xml
   <!-- Spring整合MyBatis -->
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>3.5.13</version>  
   </dependency>
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis-spring</artifactId>
     <version>1.3.1</version>
   </dependency>
   <dependency>
     <groupId>com.github.pagehelper</groupId>
     <artifactId>pagehelper</artifactId>
     <version>5.1.2</version>
   </dependency>
   ```
   
2. 创建Mapper映射文件

   映射文件存放位置，两种形式：

   - 将映射文件放在resources目录下

     映射文件会进行打包部署

   - 将映射文件放在java目录下

     默认只会对该目录下的java代码进行打包部署，如果希望对该目录下的配置文件也进行打包，需要添加额外的配置

     编辑pom.xml文件：

   ````xml
   <build>
   	<resources>
         <!-- 将java目录下的配置文件也进行打包 -->
         <resource>
             <directory>src/main/java</directory>
             <includes>
                 <include>**/*.xml</include>
                 <include>**/*.properties</include>
             </includes>
             <filtering>false</filtering>
           </resource>
           <resource>
               <directory>src/main/resources</directory>
               <includes>
                 <include>**/*.xml</include>
                 <include>**/*.properties</include>
             	</includes>
               <filtering>false</filtering>
           </resource>
       </resources>
   </build>
   ````

3. 配置

   ```xml
   <!-- 配置SqlSessionFactoryBean -->
   <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
     <!-- 可以引用独立的mybatis配置文件，也可以不用 -->
     <!-- <property name="configLocation" value="classpath:mybatis-config.xml"></property> -->
     <property name="dataSource" ref="dataSource"></property>
     <!-- 指定映射文件所在路径 -->
     <property name="mapperLocations" value="classpath:org/wanho/mapper/*Mapper.xml"></property>
     <!-- 为映射的类指定别名 -->
     <property name="typeAliasesPackage" value="net.wanho.entity"></property>
     <!-- 自定义配置 -->
     <property name="configuration">
         <bean class="org.apache.ibatis.session.Configuration">
             <!-- 打印sql -->
             <property name="logImpl" value="org.apache.ibatis.logging.stdout.StdOutImpl"/>
             <!-- 下划线映射为驼峰 -->
             <property name="mapUnderscoreToCamelCase" value="true"/>
         </bean>
     </property>
     <!-- 分页插件 -->
     <property name="plugins">
         <list>
             <bean class="com.github.pagehelper.PageInterceptor">
                 <property name="properties">
                     <props>
                         <prop key="helperDialect">mysql</prop>
                     </props>
                 </property>
             </bean>
         </list>
     </property>
   </bean>
   
   <!-- 通过反射创建Dao实现类，然后放到IoC容器中 -->
   <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
     <!-- 指定Dao接口所在的包 -->
     <property name="basePackage" value="net.wanho.dao"/>
   	<!--注入sqlSessionFactory，通过value注入String类型的名称-->
     <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
   </bean>
   ```