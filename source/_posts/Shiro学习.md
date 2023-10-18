---
title: Shiro学习
date: 2023-08-24 11:15:48
tags: Shiro
category: Java
---

## 一、Shiro简介

### 1. Shiro是什么？

Apache Shiro是一个强大且易用的开源Java安全框架，执行身份认证、授权、密码学和会话管理等

Spring Security也是一个开源的权限管理框架，Spring Security 和 Shiro 的比较如下：

- Spring Security 是一个重量级的安全管理框架； Shiro 则是一个轻量级的安全管理框架
- Spring Security 概念复杂，配置繁琐； Shiro 概念简单、配置简单
- Spring Security 功能强大； Shiro 功能简单，但在一般的 SSM 项目中也够用了
- Spring Security 依赖于Spring运行； Shiro则相对独立
- Spring Security 一般与 Spring Boot/Spring Cloud 项目组合使用； Shiro 一般与 SSM 项目结合使用
- 虽然在 Spring Boot 项目中一般使用 Spring Security ，但也可以使用 Shiro 

### 2. 基本功能

​	身份认证、授权、加密、会话管理

​	Web支持、缓存、多线程、测试、允许一个用户假装为另一个用户的身份进行访问、记住我

​			<img src="https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/1.基本功能.png" width="500px" />

​			

### 3. 单词

​	authentication  [ɔ:ˌθentɪ'keɪʃn] 认证、身份验证

​	authorization [ˌɔ:θəraɪˈzeɪʃn] 授权

​	cryptography [krɪpˈtɒgrəfi] 密码学

​	subject [ˈsʌbdʒɪkt] 主题、学科、主体

​	token [ˈtəʊkən] 令牌

​	strategy [ˈstrætədʒi] 策略

​	realm 领域、范围

​	principal [ˈprɪnsəpl] 当事人、用户

​	credentials [krəˈdenʃlz] 凭证

## 二、身份认证

### 1. 认证流程图

​		<img src="https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/4.身份认证流程.png" width="500px"/>				

### 2. 执行过程

分为五步：

- Subject

  用户主体：请求的发起者，即访问应用的用户

- Security Manager

  安全管理器：Shiro的核心，用来分发请求，对Shiro中的其他对象进行管理

- Authenticator

  认证器：用来进行认证操作

- Authentication Strategy

  认证策略 ：对于多个realm，可以对认证realm的个数进行配置

  三种认证策略：AtLeastOneSuccessfulStrategy、FirstSuccessfulStrategy、AllSuccessfulStrategy

- Realm

  安全数据源：用来进行数据匹配的，可以通过多种数据源进行匹配认证，如文件、数据库、QQ、微信、手机号等

### 3. url过滤

场景：有些url的访问需要登录才能访问，如后台管理界面，未登录时不允许访问，自动跳转到登录页面

解决：使用Shiro过滤器，配置url过滤规则

常用的过滤规则：

- anon 表示url不需要验证
- authc 表示url需要登录验证，如果未登录，默认跳转到/login.jsp，参考FormAuthenticationFilter类
- roles 表示url需要角色验证
- perms 表示url需要权限验证

​	注：默认所有url都不需要验证，相当于是anon

## 三、SpringBoot整合Shiro

![image-20230606081203830](https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230606081203830.png)

### 1. 搭建项目环境

步骤：

1. 创建一个springboot工程，选择以下模块：Web、Lombok、DevTools

2. 添加依赖 

    ```xml
    <!--jsp-->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
    </dependency>
    ```
    
3. 配置application.yml

    ```yml
    server:
      port: 8080
      servlet:
        context-path: /shiro
    
    spring:
      mvc:
        view:
          prefix: /
          suffix: .jsp
    ```

4. 创建前端页面

​	在webapp文件夹中创建index.jsp和login.jsp

- index.jsp	

    ```jsp
    <%@page contentType="text/html;UTF-8" pageEncoding="UTF-8" isErrorPage="false" %>
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <%--受限资源--%>
        <h1>系统主页</h1>
    
        <ul>
            <li><a href="#">用户管理</a></li>
            <li><a href="#">商品管理</a></li>
            <li><a href="#">订单管理</a></li>
            <li><a href="#">物流管理</a></li>
        </ul>
    </body>
    </html>
    ```
    
- login.jsp

    ```jsp
    <%@page contentType="text/html;UTF-8" pageEncoding="UTF-8" isErrorPage="false" %>
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <h1>登录界面</h1>
        <form action="${pageContext.request.contextPath}/user/login" method="post">
            用户名：<input type="text" name="username" > <br/>
            密码：<input type="password" name="password"> <br>
            <input type="submit" value="登录">
        </form>
    </body>
    </html>
    ```

5. 配置SpringBoot，兼容JSP

    ![image-20230615095532594](https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230615095532594.png)

6. 测试

    访问 http://localhost:8080/shiro/login.jsp 、 http://localhost:8080/shiro/index.jsp

### 2. 整合Shiro

步骤：

1. 添加依赖 

    ```xml
    <!--shiro-->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring-boot-starter</artifactId>
        <version>1.5.3</version>
    </dependency>
    ```

2. 自定义Realm

    ```java
    /**
     * 自定义Realm，继承AuthorizingRealm
     */
    public class ShiroRealm extends AuthorizingRealm {
    
        /**
         * 授权
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            return null;
        }
    
        /**
         * 认证
         */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            return null;
        }
    }
    ```

3. 创建配置类ShiroConfig

    ```java
    @Configuration
    public class ShiroConfig {
    
        /**
         * ShiroFilter，对资源进行过滤处理
         */
        @Bean
        public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager) {
            ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
            // 设置安全管理器
            shiroFilterFactoryBean.setSecurityManager(securityManager);
          
            // 设置url过滤规则
            Map<String, String> map = new LinkedHashMap<>();
            map.put("/index.jsp", "authc");// 表示这个资源需要登录认证
            shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
          
            // 设置认证界面
            shiroFilterFactoryBean.setLoginUrl("/login.jsp"); // 默认就是跳转到/login.jsp
    
            return shiroFilterFactoryBean;
        }
    
        /**
         * 创建安全管理器
         */
        @Bean
        public DefaultWebSecurityManager getDefaultWebSecurityManager(Realm realm) {
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            securityManager.setRealm(realm);
            return securityManager;
        }
    
        /**
         * 创建自定义Realm
         */
        @Bean
        public Realm getRealm() {
            ShiroRealm realm = new ShiroRealm();
            return realm;
        }
    }
    ```

4. 测试

​	访问 http://localhost:8080/shiro/index.jsp，发现会自动跳转到login.jsp，因为这个资源需要登录认证

### 3. 认证和退出

步骤：

1. 在index.jsp添加欢迎信息和退出

    ```jsp
    欢迎您：<shiro:principal/>
    <a href="${pageContext.request.contextPath}/user/logout">退出登录</a>
    ```

2. 编写UserController

    ```java
    @Controller
    @RequestMapping("/user")
    public class UserController {
    
        @RequestMapping("/login")
        public String login(String username, String password) {
            // 获取主体对象
            Subject subject = SecurityUtils.getSubject();
            try {
                subject.login(new UsernamePasswordToken(username,password));
                return "index";
            } catch (UnknownAccountException e) {
                e.printStackTrace();
                System.out.println("用户不存在！");
            } catch (IncorrectCredentialsException e) {
                e.printStackTrace();
                System.out.println("密码错误！");
            } catch (AuthenticationException e){
                e.printStackTrace();
                System.out.println("认证失败！");
            }
            return "redirect:/login.jsp";
        }
    
        @RequestMapping("logout")
        public String logout() {
            Subject subject = SecurityUtils.getSubject();
            subject.logout();
            return "redirect:/login.jsp";
        }
    
    }
    ```

3. 修改自定义的ShiroRealm

    ```java
    public class ShiroRealm extends AuthorizingRealm {
    
        /**
         * 授权
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            return null;
        }
    
        /**
         * 认证
         */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            String username = (String) authenticationToken.getPrincipal(); // 用户名
     
            if (!"admin".equals(username)) {
              	throw new UnknownAccountException();
            }
          	// 返回AuthenticationInfo，然后交由凭证匹配器（CredentialsMatcher）进行凭证的判断，默认是对密码进行判断
           	return new SimpleAuthenticationInfo(username,"123",this.getName()); // 参数：用户信息、密码、realm名称
        }
    }
    ```

4. 修改配置类ShiroConfig

    ```java
    @Configuration
    public class ShiroConfig {
    
        /**
         * ShiroFilter，对资源进行过滤处理
         */
        @Bean
        public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager) {
            ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
            // 设置安全管理器
            shiroFilterFactoryBean.setSecurityManager(securityManager);
    
            // 设置url过滤规则
            Map<String, String> map = new LinkedHashMap<>();
            map.put("/user/login","anon");// 表示这个为公共资源，一定是在受限资源上面
            map.put("/**","authc");//表示这个为受限资源，需要登录认证
            shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
    
            // 设置认证界面
            shiroFilterFactoryBean.setLoginUrl("/login.jsp"); // 默认就是跳转到/login.jsp
    
            return shiroFilterFactoryBean;
        }
    
        /**
         * 创建安全管理器
         */
        @Bean
        public DefaultWebSecurityManager getDefaultWebSecurityManager(Realm realm) {
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            securityManager.setRealm(realm);
            return securityManager;
        }
    
        /**
         * 创建自定义Realm
         */
        @Bean
        public Realm getRealm() {
            ShiroRealm realm = new ShiroRealm();
            return realm;
        }
    }
    ```

5. 测试

    登录和退出的功能正常

    未登录时不能访问index.jsp

### 4. 加密的认证

通常需要对密码进行加密，常用的散列算法有md5、sha等，都是非对称的算法

- 对于普通的散列加密，如果知道加密后的值，可以通过穷举算法，暴力破解出对应的明文
- 所以建议进行散列加密时可以添加salt（盐），相当于对`原始密码+盐`进行散列加密，盐值一般放在数据库中

- 同时可以配置散列次数，次数一般放在配置文件中

步骤：

1. 创建数据库

    ```sql
    drop database if exists shiro;
    create database shiro charset utf8;
    use shiro;	
    
    create table t_user (
      id int primary key auto_increment comment '编号',
      username varchar(40) comment '用户名',
      password varchar(40) comment '密码',
      salt varchar(255) comment '盐值'
    ) engine=innodb default charset=utf8 comment '用户表';
    ```

2. 添加依赖

    ```xml
    <!--mybatis-->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>2.2.2</version>
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

    ```yml
    server:
      port: 8080
      servlet:
        context-path: /shiro
    
    spring:
      mvc:
        view:
          prefix: /
          suffix: .jsp
      datasource:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/shiro?useUnicode=true&characterEncoding=utf8
        username: root
        password: root
    
    mybatis:
      type-aliases-package: net.wanho.entity
      mapper-locations: classpath:mapper/*.xml
    ```

4. 创建注册页面register.jsp

    ```jsp
    <%@page contentType="text/html;UTF-8" pageEncoding="UTF-8" isErrorPage="false" %>
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <h1>注册界面</h1>
        <form action="${pageContext.request.contextPath}/user/register" method="post">
            用户名：<input type="text" name="username" > <br/>
            密码：<input type="password" name="password"> <br>
            <input type="submit" value="立即注册">
        </form>
    </body>
    </html>
    ```

5. 创建Entity、Mapper、Service、Controller

6. 修改自定义的ShiroRealm

    ```java
    public class ShiroRealm extends AuthorizingRealm {
    
        @Autowired
        private UserService userService;
    
        /**
         * 授权
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            return null;
        }
    
        /**
         * 认证
         */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            String username = (String) authenticationToken.getPrincipal(); // 用户名
    
            User user = userService.findByUsername(username);
            if (ObjectUtils.isEmpty(user)) {
                throw new UnknownAccountException();
            }
    
            // 返回AuthenticationInfo，交由AuthenticatingRealm进行密码匹配（使用密码匹配器CredentialsMatcher）
            return new SimpleAuthenticationInfo(
                    user.getUsername(), // 用户信息
                    user.getPassword(), // 密码
                    ByteSource.Util.bytes(user.getSalt()), // salt盐值
                    this.getName()); //realm名称
        }
    }
    ```

7. 修改配置类ShiroConfig

    ```java
    @Configuration
    public class ShiroConfig {
    
        /**
         * ShiroFilter，对资源进行过滤处理
         */
        @Bean
        public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager) {
            ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
            // 设置安全管理器
            shiroFilterFactoryBean.setSecurityManager(securityManager);
    
            // 设置url过滤规则
            Map<String, String> map = new LinkedHashMap<>();
            map.put("/user/login","anon");// 表示这个为公共资源，一定是在受限资源上面
            map.put("/user/register","anon");
            map.put("/register.jsp","anon");
            map.put("/**","authc");//表示这个为受限资源，需要登录认证
            shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
    
            // 设置认证界面
            shiroFilterFactoryBean.setLoginUrl("/login.jsp"); // 默认就是跳转到/login.jsp
    
            return shiroFilterFactoryBean;
        }
    
        /**
         * 创建安全管理器
         */
        @Bean
        public DefaultWebSecurityManager getDefaultWebSecurityManager(Realm realm) {
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            securityManager.setRealm(realm);
            return securityManager;
        }
    
        /**
         * 创建自定义Realm
         */
        @Bean
        public Realm getRealm() {
            ShiroRealm realm = new ShiroRealm();
    
            // 创建密码匹配器，支持散列算法
            HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
            credentialsMatcher.setHashAlgorithmName("md5"); // 设置加密算法
            credentialsMatcher.setHashIterations(1024); // 设置散列次数
            // 设置密码匹配器
            realm.setCredentialsMatcher(credentialsMatcher);
    
            return realm;
        }
    }
    ```

8. 测试

​	注册和登录的功能正常

## 五、授权

### 1. 简介

授权也称为访问控制，控制用户对资源的访问

- 权限：增删改查 CRUD

- 角色：权限的集合，如系统管理员、老师、学生

    ![image-20230607211534866](https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230607211534866.png)			

### 2. 授权流程图

​		<img src="https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/6.授权流程.png" width="500px" />			

执行过程，分为4步：

- Subject

  发送请求，对角色和权限进行判断  `hasRole()、isPermitted()` 

- SecurityManager

  接收Subject的请求

- Authorizer

  授权器

- Realm

  查询角色和权限信息

### 3. 基本用法

步骤：

1. 创建数据库

    ```sql
    drop database if exists shiro;
    create database shiro charset utf8;
    use shiro;	
    
    create table t_user (
      id int primary key auto_increment comment '编号',
      username varchar(40) comment '用户名',
      password varchar(40) comment '密码',
      salt varchar(255) comment '盐值'
    ) engine=innodb default charset=utf8 comment '用户表';
    
    create table t_role (
      id int not null primary key auto_increment,
      name varchar(60) default null
    ) engine=innodb default charset=utf8 comment '角色表';
    
    create table t_perms (
      id int not null primary key auto_increment,
      name varchar(80) default null,
      url varchar(255) default null
    ) engine=innodb default charset=utf8 comment '权限表';
    
    create table t_user_role (
      id int not null primary key auto_increment,
      userid int default null,
      roleid int default null
    ) engine=innodb default charset=utf8 comment '用户角色表';
    
    create table t_role_perms (
      id int not null primary key auto_increment,
      roleid int default null,
      permsid int default null
    ) engine=innodb default charset=utf8 comment '角色权限表';
    
    insert into t_user values (null, 'aaa', '37c80a50a975580fb2fb287bee20a04a', 'K2*LWFB#'); -- 密码为123
    insert into t_user values (null, 'bbb', '93a2dc457a2f9c50fa0a9f2b9de9f456', 'DwOkv&WW'); -- 密码为123
    insert into t_user values (null, 'ccc', '37c80a50a975580fb2fb287bee20a04a', 'K2*LWFB#'); -- 密码为123
    
    insert into t_role values (null, 'admin');
    insert into t_role values (null, 'user');
    insert into t_role values (null, 'stu');
    
    insert into t_perms values (null, 'admin:*:*', null);
    insert into t_perms values (null, 'user:*:*', null);
    insert into t_perms values (null, 'user:find:*', null);
    
    insert into t_user_role values (null, 1, 1);
    insert into t_user_role values (null, 1, 2);
    insert into t_user_role values (null, 2, 2);
    insert into t_user_role values (null, 3, 3);
    
    insert into t_role_perms values (null, 1, 1);
    insert into t_role_perms values (null, 2, 2);
    insert into t_role_perms values (null, 2, 3);
    insert into t_role_perms values (null, 3, 3);
    ```

2. 创建Entity、Mapper、Service

3. 修改自定义的ShiroRealm

    ```java
    public class ShiroRealm extends AuthorizingRealm {
    
        @Resource
        private UserService userService;
        @Resource
        private PermsService permsService;
    
        /**
         * 授权
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            String username = (String) principalCollection.getPrimaryPrincipal();
    
            // 获取角色信息
            User user = userService.findRolesByUsername(username);
            List<String> roles = user.getRoles().stream().map(Role::getName).collect(Collectors.toList());
    
            // 获取权限信息
            List<String> perms = user.getRoles().stream().flatMap(role -> {
                List<Perms> list = permsService.findPermsByRoleId(role.getId());
                return list.stream().map(Perms::getName);
            }).collect(Collectors.toList());
    
    
            SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
            info.addRoles(roles);
            info.addStringPermissions(perms);
            System.out.println(roles);
            System.out.println(perms);
            return info;
        }
    
        /**
         * 认证
         */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            String username = (String) authenticationToken.getPrincipal(); // 用户名
    
            User user = userService.findByUsername(username);
            if (ObjectUtils.isEmpty(user)) {
                throw new UnknownAccountException();
            }
    
            // 返回AuthenticationInfo，然后交由凭证匹配器（CredentialsMatcher）进行凭证的判断，默认是对密码进行判断
            return new SimpleAuthenticationInfo(
                    user.getUsername(), // 用户信息
                    user.getPassword(), // 密码
                    ByteSource.Util.bytes(user.getSalt()), // salt盐值
                    this.getName()); //realm名称
        }
    }
    
    ```

7. 测试

​	不同用户由于角色和权限的不同，登录后看到的系统主页是不一样的！

## 六、JWT

### 1. 简介

 JWT（JSON Web Token）是目前最流行的跨域认证解决方案。

传统的认证流程，使用session：

```
1、用户向服务器发送用户名和密码
2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等
3、服务器向用户返回一个 session_id，写入用户的 Cookie
4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器
5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份

存在的问题：扩展性差、服务器内存占用较高
```

JWT的认证流程，使用token：

```
1、用户向服务器发送用户名和密码
2、服务器验证通过后，会生成一个token（JWT），表示用户的身份
3、服务器向用户返回该token，客户端存储token，可以存储在Cookie、localStorage或sessionStorage中
4、用户随后的每一次请求，都要带上这个token，一般放请求头中携带
5、服务器收到token并验证是否有效，由此得知用户的身份。
```

### 2. JWT原理

JWT 原理：服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。

```json
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2023年6月12日0点0分"
}
```

- 以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。 
- 为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。
- 优点：
    - 服务器不再保存任何 session 数据，服务器变成无状态了
    - 由客户端保存身份信息，即JWT令牌 token，一般存储在客户端的localStorage中
    - 客户端每次请求时，JWT令牌随着请求头一起提交

### 3. JWT数据结构

实际的 JWT 大概就像下面这样。

![bg2018072304](https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/bg2018072304.jpeg)

它是一个很长的字符串，中间用点`.`分隔成三个部分。

JWT的三个部分如下：

- Header（头部）
- Payload（载荷）
- Signature（签名）

![](https://raw.githubusercontent.com/YukiCCC/figure/main/https://raw.githubusercontent.com/YukiCCC/figure/main/8.jpg)

#### 3.1 头部（Header）

Header 部分是一个 JSON 对象，用于描述该JWT的基本信息，例如签名所用的算法及令牌的类型等。

```java
{
  "alg":"HS256",
  "typ":"JWT"
}
```

上面代码中，`alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`type`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

最后，将上面的 JSON 对象使用 Base64URL 算法转成字符串。

```shell
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9        
```

注： Base64URL 算法和 Base64 算法基本类似，但有一些小的不同，其会将个别特殊符合替换掉。

#### 3.2 载荷（Playload）

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

```
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

除了官方字段，你还可以在这个部分自定义私有字段，下面就是一个例子。

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

这个 JSON 对象也要使用 Base64URL 算法转成字符串。

```java
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

注意：由于Base64URL是对称算法，可以被解密为明文信息，所以一般不建议存放敏感数据。

#### 3.3 签名（Signature）

Signature 部分是对前两部分的签名，防止数据篡改。

- 首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。


```json
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

- 算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

- 注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。

### 4. 优缺点

优点：

1. 因为json的通用性，所以JWT是可以跨语言的

2. 因为有了payload部分，所以JWT可以在自身存储一些其他业务所必要的非敏感信息

3. 便于传输，JWT的构成非常简单，字节占用很小，所以它是非常便于传输的

4. 它不需要在服务端保存会话信息，所以易于扩展，例如集群或微服务环境下。

5. JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

缺点：

1. JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次

2. JWT 不加密的情况下，不能将敏感数据写入 JWT。

3. 由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

4. JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

5. 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

### 5. 在Java中使用JWT

步骤：

1. 添加依赖

```xml
<!--java-jwt-->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.3.0</version>
</dependency>
```

2. 基本用法

```java
public class JwtTest {

    /**
     * 生成令牌
     */
    @Test
    public void generate(){
        Map<String, Object> payload = new HashMap<>();
        payload.put("id",1001);
        payload.put("name","tom");

        Calendar c = Calendar.getInstance();
        c.add(Calendar.SECOND,30);

        // JWTCreator.Builder builder = JWT.create();
        String token = JWT.create()
                .withPayload(payload) // 载荷
                .withExpiresAt(c.getTime()) // 过期时间
                .sign(Algorithm.HMAC256("secret")); // 签名：算法和密钥
        System.out.println(token);
    }

    /**
     * 校验令牌
     *  如果令牌过期，会抛异常TokenExpiredException
     *  一般会返回boolean，表示校验是否通过
     */
    @Test
    public void valid(){
        String token ="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoidG9tIiwiaWQiOjEwMDEsImV4cCI6MTY4NjU3NTczNn0.W4CdE-upxcAGK4uVD-awRXIl0Y6Q8NHTS8PG4dhjJeo";
        try {
            JWT.require(Algorithm.HMAC256("secret")).build().verify(token);
        } catch (Exception e) {
            // 如果令牌过期，会抛异常TokenExpiredException
            e.printStackTrace();
        }
    }

    /**
     * 解析令牌
     *  如果令牌过期，会抛异常TokenExpiredException
     */
    @Test
    public void parse(){
        String token ="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoidG9tIiwiaWQiOjEwMDEsImV4cCI6MTY4NjU3NTczNn0.W4CdE-upxcAGK4uVD-awRXIl0Y6Q8NHTS8PG4dhjJeo";
        DecodedJWT decodedJWT = JWT.require(Algorithm.HMAC256("secret")).build().verify(token);
        // 获取载荷中的信息
        Long id = decodedJWT.getClaim("id").asLong();
        String name = decodedJWT.getClaim("name").asString();
        System.out.println(id+"-"+name);
    }

    /**
     * 反码令牌
     *  直接获取载荷信息，不会校验是否过期
     */
    @Test
    public void decode(){
        String token ="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoidG9tIiwiaWQiOjEwMDEsImV4cCI6MTY4NjU3NTczNn0.W4CdE-upxcAGK4uVD-awRXIl0Y6Q8NHTS8PG4dhjJeo";
        DecodedJWT decodedJWT = JWT.decode(token);
        Long id = decodedJWT.getClaim("id").asLong();
        String name = decodedJWT.getClaim("name").asString();
        System.out.println(id+"-"+name);
    }

}
```

## 七、整合

步骤：

1. 创建一个springboot工程，选择以下模块：Web、Lombok、DevTools、Spring Data Redis

2. 添加依赖 

    ```xml
    <!--mysql、druid、mybatis-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
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
    
    <!--knife4j-->
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-spring-boot-starter</artifactId>
        <version>3.0.3</version>
    </dependency>
    
    <!--shiro-->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring-boot-starter</artifactId>
        <version>1.5.3</version>
    </dependency>
    
    <!--java-jwt-->
    <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
        <version>4.3.0</version>
    </dependency>
    
    <!-- easy-captcha -->
    <dependency>
        <groupId>com.github.whvcse</groupId>
        <artifactId>easy-captcha</artifactId>
        <version>1.6.2</version>
    </dependency>
    
    <!-- hutool -->
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.15</version>
    </dependency>
    ```

3. 配置application.yml

    ```yml
    server:
      port: 8080
    
    spring:
      # DruidDataSource
      datasource:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/project?useUnicode=true&characterEncoding=utf8
        username: root
        password: root
      # knife4j
      mvc:
        pathmatch:
          matching-strategy: ant_path_matcher
      # redis
      redis:
        host: localhost
        port: 6379
        database: 0
    
    # MyBatis
    mybatis:
      type-aliases-package: net.wanho.po
      mapper-locations: classpath:mapper/*.xml
      configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
        map-underscore-to-camel-case: true
    
    # PageHelper
    pagehelper:
      helper-dialect: mysql
    ```

4. 数据库

    ```sql
    drop database if exists project;
    create database project charset utf8;
    use project;
    
    create table user
    (
        id int primary key auto_increment comment '编号',
        username varchar(50) unique comment '用户名',
        password varchar(50) comment '密码',
        status tinyint comment '帐号状态（0正常 1停用）'
    ) engine innodb default charset utf8 comment '用户表';
    
    create table student
    (
        id int primary key auto_increment comment '编号',
        name varchar(50) comment '姓名',
        age int comment '年龄',
        gender varchar(50) comment '性别',
        address varchar(50) comment '地址',
        birth date comment '生日'
    ) engine innodb default charset utf8 comment '学生表';
    ```
    

