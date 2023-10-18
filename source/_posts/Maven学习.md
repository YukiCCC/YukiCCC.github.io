---
title: Maven学习
date: 2023-08-07 09:16:39
tags: Maven
category: Java
---

## 一、Maven简介

### 1. 什么是Maven

​	maven [ˈmeɪvn] 专家、内行

​	Apache Maven 是一个软件项目管理和构建工具，可以帮助创建和管理项目

​	基于项目对象模型（POM：Project Object Model）的概念，帮助开发者构建一个项目的完整生命周期

​	官网：http://maven.apache.org/

### 2. 为什么使用Maven

- 项目的管理工具

  项目规模很大时一定会将项目进行拆分，拆分成多个工程，使用Maven在多个工程之间建立依赖关系

- jar包的管理工具

  通过仓库管理jar包、解决jar包的依赖、自动下载jar包

- 自动化的构建工具

  编译代码、执行测试、打包、部署等

### 3. 术语

- 中央仓库

  是一个网络仓库，用于存放jar包和maven插件

  https://repo1.maven.org/maven2 

  https://mvnrepository.com

- 本地仓库

  从中央仓库下载的jar包的存放位置，也是一个仓库，只不过是存放在本地电脑上

- 镜像仓库

  对中央仓库做的镜像（mirror）

  阿里云提供的镜像仓库 https://maven.aliyun.com/repository/public

- 私服

  局域网内部搭建的maven服务器

## 二、安装Maven

### 1. 下载安装包

​	从maven官网下载安装包，这里使用apache-maven-3.6.0-bin.tar.gz

### 2. 解压安装包

​	将安装包解压到无中文、无空格的路径下，如：D:\software\apache-maven-3.6.0

​	配置环境变量，将bin目录添加到Path变量中，如：D:\software\apache-maven-3.6.0\bin

​	测试，在DOS窗口中执行以下命令：

```bash
mvn  -version
```

### 3. 配置本地仓库

本地仓库的默认位置： ~/.m2/repository（如 C:/Users/登录用户/.m2/repository）

修改本地仓库的位置：编辑conf/setting.xml文件

```xml
<settings>
  <!--指定本地仓库的位置-->
  <localRepository>D:\maven-repos</localRepository>
</settings>
```

### 4. 配置镜像仓库

​	使用maven时默认从中央仓库下载所需的包(插件)，比较慢，可以配置使用阿里云提供的镜像仓库

​	编辑maven主目录下的/conf/setting.xml文件，在`<mirrors></mirrors>`标签中添加如下内容：

```xml
<mirror>
  <id>aliyunmaven</id> <!-- 名称自定义，必须唯一 -->
  <mirrorOf>*</mirrorOf> <!-- 所有访问都使用该镜像仓库 -->
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### 5. 配置Maven的JDK版本

​	修改maven默认使用的jdk版本，编辑conf/setting.xml文件，在profiles标签里面添加如下内容

```xml
<profile>
  <id>jdk‐1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

## 三、使用Maven

### 1. 在IDEA中集成maven

指定Maven主目录和配置文件：Settings——>搜索Maven——>Maven Home directory和User settings file

注：切换Porject后要重新配置maven

### 2. 创建maven项目

File——>New——>Module——>Maven Archetype

- Archetype：maven-archetype-quickstart（Java项目） / maven-archetype-webapp（Java Web项目） 
- GroupId : net.wanho.shop（组织域名反向+项目名称）
- ArtifactId : shop-product（模块名称）
- Version : 1.0.1（版本）

### 3. 目录结构

Maven项目的目录结构如下：

```java
|-项目名称
	|-src				//程序代码
		|-main			//主代码
			|-java		//源代码
				|-用于存放源代码，相当于传统项目中的src，传统项目的包名.类名，如net.wanho.shop.dao
			|-resources	//配置文件
				|-用于存放配置文件
			|-webapp	//网站根目录
        		|-WEB-INF
        			|-web.xml
		|-test			//测试代码，目录结构和main中完全一致
			|-java
			|-resources
	|-pom.xml			//maven核心配置文件
```

如果没有对应的目录，可以自己创建，但必须符合该目录结构

在IDEA中目录是分类型的，常用的有四种：

- Sources Root：主代码的目录——>src/main/java
- Test Sources Root：测试代码的目录——>src/test/java
- Resources Root：主代码所需资源的目录——>src/main/resources
- Test Resources Root：测试代码所需资源的目录——>src/test/resources

默认情况下新建的目录是普通的Directory，创建后可以设置目录的类型：

- 右击目录——>Mark Directory As
- 每种目录的图标有所不同

### 4. 执行maven操作

在IDEA中管理所有Maven项目：View——>Tool Windows——>Maven

Maven项目的生命周期：

| 命令    | 作用 | 描述                                                         |
| ------- | ---- | ------------------------------------------------------------ |
| clean   | 清理 | 删除target目录                                               |
| compile | 编译 | 将main/中的源代码编译成字节码文件，放在target/classes目录下  |
| test    | 测试 | 执行测试类（使用JUnit），并生成测试报告，放在target/surefire-reports目录下 |
| package | 打包 | 将java项目打包成jar，将web项目打包成war，放在target目录下    |
| install | 安装 | 将项目的jar包安装到本地仓库，供其他项目使用                  |

注意：

- 问题：在Maven Projects中项目显示为灰色，表示该maven项目未被管理，不可用
- 解决：在Maven Projects里点击"+"，选择项目对应的pom.xml文件	

## 四、pom.xml文件

### 1. 简介

​	pom:project object model 项目对象模型

​	pom.xml是Maven的核心配置文件，与项目构建相关的所有配置都在该文件中

### 2. 坐标

​	用来唯一的标识每个项目，必须为项目定义坐标，且坐标必须唯一

​	Maven坐标是通过一些元素定义的：groupId、artifactId、version

```xml
<!--
    坐标：
        groupId：定义组织id，表示当前模块隶属的项目，采用"组织域名反向+项目名称"
        artifactId：定义模块id
        version：定义当前的版本
-->
<groupId>net.wanho.shop</groupId>
<artifactId>shop-product</artifactId>
<version>1.0.1</version>
```

### 3. dependency

​	如何查找一个jar包的坐标?  https://mvnrepository.com

```xml
<!-- dependency基本配置 -->
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.31</version>
  <scope>compile</scope>
</dependency>
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.38</version>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.24</version>
  <scope>provided</scope>
</dependency>
```

​	scope表示依赖的作用域，用来配置依赖的jar包的可作用范围，即在什么地方可以使用

| 取值     | 含义                                                 | 举例                                    |
| -------- | ---------------------------------------------------- | --------------------------------------- |
| compile  | 表示该依赖可以在整个项目中使用，参与打包部署，默认值 | fastjson                                |
| test     | 表示该依赖只能在测试程序中使用，不参与打包和部署     | junit                                   |
| provided | 表示编写源代码的时候需要，不参与打包部署             | lombok、servlet-api（因为tomcat中已有） |
| runtime  | 表示运行时需要，编译代码时不需要                     | mysql-connector（通过接口反射加载）     |

### 4. properties

​	全局属性，一般情况下用于定义全局的jar的版本

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <!-- 
       可以自定义标签名，然后使用 ${标签名} 获取标签中的值
       一般情况下用于定义全局的jar的版本，相当于定义全局变量
  -->
  <spring.version>5.3.27</spring.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${spring.version}</version>
  </dependency>
</dependencies>
```

​	注：快速将jar包的版本添加到properties中：右击版本号——>Refactor——>Property

### 5. repositories

​	用于配置当前工程使用的远程仓库

​	查找顺序：本地仓库、pom.xml中配置的远程仓库、maven主录下的conf/setting.xml中配置的远程仓库

```xml
<repositories>
  <!-- 有些最新版本的jar包，在中央仓库中可能并没有，此时可以指定其他可用的远程仓库 -->
  <repository>
    <id>springio</id>
    <url>http://repo.spring.io/milestone/</url>
  </repository>
</repositories>
```

### 6. plugins

插件是一种工具，如

- maven-clean-plugin插件是用来清理项目的工具
- maven-compile-plugin插件是用来编译代码的工具
- tomcat7-maven-plugin插件是用来将web项目自动打包并部署到tomcat的工具

```xml
<build>
    <plugins>
        <!-- tomcat插件 -->
        <plugin>
          <groupId>org.apache.tomcat.maven</groupId>
          <artifactId>tomcat7-maven-plugin</artifactId>
          <version>2.2</version>
          <configuration>
            <path>/</path>
            <port>8888</port>
          </configuration>
        </plugin>
    </plugins>
</build>
```

### 7. 案例

Web应用的开发，模块名称：shop-user

步骤：

1. 添加依赖

   ```xml
   <!-- servlet-api -->
   <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>javax.servlet-api</artifactId>
       <version>4.0.1</version>
       <scope>provided</scope>
   </dependency>
   <!-- jstl -->
   <dependency>
       <groupId>jstl</groupId>
       <artifactId>jstl</artifactId>
       <version>1.2</version>
   </dependency>
   ```

2. 创建HelloServlet类

   ```java
   @WebServlet("/hello")
   public class HelloServlet extends HttpServlet {
       @Override
       protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           req.setAttribute("name","tom");
   
           List<Integer> nums = Arrays.asList(13, 25, 38);
           req.setAttribute("nums",nums);
   
           Properties p=new Properties();              p.load(HelloServlet.class.getClassLoader().getResourceAsStream("stu.properties"));
           System.out.println(p);
           req.setAttribute("age",p.getProperty("age"));
           req.setAttribute("sex",p.getProperty("sex"));
   
           req.getRequestDispatcher("/index.jsp").forward(req,resp);
       }
   }
   ```

3. 创建index.jsp

    ```jsp
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%--默认生成的web.xml文件使用的是web-app_2_3.dtd，会忽略EL表达式，需要启用EL表达式--%>
    <%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
    <html>
    <head>
        <title></title>
    </head>
    <body>
        姓名：${name} <br>
        <ul>
            <c:forEach items="${hobbies}" var="hobby">
                <li>${hobby}</li>
            </c:forEach>
        </ul>
    </body>
    </html>
    
    ```

    stu.properties

    ```properties
    name=alice
    age=21
    sex=male
    ```

## 五、Maven中的关系

### 1. 继承

含义：一个Maven工程继承自另一个Maven工程，分别称为子工程、父工程

场景：实际开发中一个大项目会拆分为多个子项目（子模块/子工程），多个子工程使用的技术基本都相同，即多个子工程中使用的是相同的依赖或插件等配置，此时可以把相同配置抽取到一个父工程中，进行统一管理，保持一致性，简化pom.xml配置

步骤：

1. 创建三个工程：子工程child01和child02、父工程parent

2. 将父项目的打包方式设置为pom

   ```xml
   <!-- 
     打包方式
        jar：java项目的打包方式，默认值
        war：web项目的打包方式
        pom：父项目的打包方式
     注：将父工程打包方式设置为pom后，父工程将不会被打包，因此不要在父工程中写java代码，父工程只是用来简化POM配置
       -->
   <packaging>pom</packaging>
   ```

3. 在子项目中引用父项目，指定父项目的坐标，并指定父项目pom.xml文件的路径

   ```xml
   <!-- 引用父项目，指定父项目的坐标 -->
   <parent>
     <groupId>net.wanho.study</groupId>
     <artifactId>parent</artifactId>
     <version>0.0.1-SNAPSHOT</version>
     <!-- 指定父项目pom.xml文件的相对物理路径 -->
     <relativePath>../parent/pom.xml</relativePath>
   </parent>
   ```

   注：如果子项目的位置是在父项目所在的目录中，则可以省略不配置relativePath项   



**问题：有时并不是父项目的所有ja包都需要被子项目继承，但又希望能够对依赖进行统一管理，如：jar包版本的控制，怎么办？**

解决：配置dependencyManagement

步骤：   

1. 在父项目中配置dependencyManagement
   此时父项目只进行jar包的管理，父项目的jar包默认并不会被子项目继承  

   ```xml
   <!-- dependencyManagement表示父项目只进行依赖的管理，依赖默认不会被子项目继承 -->
   <dependencyManagement>
     <dependencies>
       <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>4.12</version>
         <scope>test</scope>
       </dependency>
       <dependency>
         <groupId>javax.servlet</groupId>
         <artifactId>javax.servlet-api</artifactId>
         <version>4.0.1</version>
         <scope>provided</scope>
       </dependency>
       <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>5.1.38</version>
       </dependency>
     </dependencies>
   </dependencyManagement>
   ```


2. 在子项目中引用父项目中的依赖

   如果子项目想继承父项目的jar包，需要在子项目中手动引用，且引用时只需要配置groupId和artifactId，**无需指定版本version**

   ```xml
   <dependencies>
     <!-- 
      父项目配置dependencyManagement后默认并不会继承过来，需要手动在子项目中引用
      只需要指定groupId和artifactId，无需指定版本version
      可以指定要引用的依赖，并不一定要使用父项目中所有的依赖
      -->
     <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
     </dependency>
     <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
       </dependency>
   </dependencies>
   ```


### 2. 聚合

将多个子项目聚合到一个父项目中，然后通过对该父项目进行操作，从而实现对所有的聚合项目的操作

在父项目中聚合子项目：

```xml
<!-- 聚合子项目，指定子项目的根目录-->
<modules>
    <module>../child01</module>
    <module>../child02</module>
</modules>
```

### 3. 依赖

项目C —> 项目B  —> 项目A 

概念：如果项目C依赖于项目B，项目B依赖于项目A，则项目C也依赖于项目A，这叫依赖的传递

步骤：

1. 配置依赖关系  

   child03——>child02——>child01

2. 在child01中添加依赖时，child02和child03会传递该依赖，也会出现该依赖

#### 3.1 控制依赖的传递

并不是所有的依赖都会被传递：

- scope为compile的依赖会被传递
- scope为test和provided的依赖不会被传递
- 配置optional为true的依赖不会被传递

```xml
<dependency>
    <groupId>com.alibaba</groupId>
  	<artifactId>fastjson</artifactId>
  	<version>1.2.31</version>
    <optional>true</optional> <!-- 表示该jar包不传递 -->
</dependency>
```

#### 3.2 继承和依赖的区别

继承：

- 使用`<parent>`
- 继承是指子项目可以继承父项目的数据和配置，从而简化配置
- 继承时父项目一般都是自己创建的项目，也可以是第三方的

依赖：

- 使用`<dependencies>`
- 依赖是指jar包可以通过依赖的方式引入
- 依赖时所依赖的jar包一般多是第三方的，也可以是自己创建的依赖项目

## 六、Maven综合应用

### 1. 分析

将项目分为多个工程，可以按层次分，也可以按模块分，或者同时按层次和模块分

以ums为例，使用Maven创建和管理项目：

- 父工程：ums-parent
- dao工程：ums-dao
- service工程：ums-service
- web工程：ums-web

### 2. 步骤

#### 2.1 创建工程

1. 创建父工程：ums-parent

   File——>New——>Module——>Maven Archetype——>maven-archetype-quickstart

   - GroupId：net.wanho.ums
   - ArtifactId：ums-parent
   - Context root：~/IdeaProjects/framework/ums-parent

2. 创建子工程：ums-dao

   右键ums-parent父工程——>New——>Module——>Maven Archetype——>maven-archetype-quickstart

   - ArtifactId：ums-dao
   - Content root：~/IdeaProjects/framework/**ums-parent/ums-dao**

3. 创建子工程：ums-service

4. 创建子工程：ums-web，web工程

#### 2.2 配置依赖

1. 在父工程中配置依赖管理

   ```xml
   <properties>
     <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
     <maven.compiler.source>1.8</maven.compiler.source>
     <maven.compiler.target>1.8</maven.compiler.target>
     <junit.version>4.12</junit.version>
     <javax.servlet-api.version>4.0.1</javax.servlet-api.version>
     <jstl.version>1.2</jstl.version>
     <mysql-connector-java.version>5.1.38</mysql-connector-java.version>
     <fastjson.version>1.2.31</fastjson.version>
     <lombok.version>1.18.24</lombok.version>
     <hutool-all.version>5.8.15</hutool-all.version>
   </properties>
   
   <dependencyManagement>
     <dependencies>
       <!-- junit -->
       <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>${junit.version}</version>
         <scope>test</scope>
       </dependency>
   
       <!-- servlet-api -->
       <dependency>
         <groupId>javax.servlet</groupId>
         <artifactId>javax.servlet-api</artifactId>
         <version>${javax.servlet-api.version}</version>
         <scope>provided</scope>
       </dependency>
       <!-- jstl -->
       <dependency>
         <groupId>jstl</groupId>
         <artifactId>jstl</artifactId>
         <version>${jstl.version}</version>
       </dependency>
   
       <!-- mysql -->
       <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>${mysql-connector-java.version}</version>
       </dependency>
       <dependency>
         <groupId>com.alibaba</groupId>
         <artifactId>druid</artifactId>
         <version>${druid.version}</version>
       </dependency>
   
      <!-- lombok  -->
     <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <scope>provided</scope>
     </dependency>
     <!-- hutool -->
     <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>${hutool-all.version}</version>
     </dependency>
     </dependencies>
   </dependencyManagement>
   ```
   
2. 在各个子工程中引用依赖

   ums-dao

   ```xml
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
   </dependency>
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid</artifactId>
   </dependency>
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
   </dependency>
   ```
   
   ums-service
   
   ```xml
   <!-- 依赖于dao -->
   <dependency>
       <groupId>net.wanho.ums</groupId>
       <artifactId>ums-dao</artifactId>
       <version>1.0-SNAPSHOT</version>
   </dependency>
   ```
   
   ums-web
   
   ```xml
   <dependencies>
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>javax.servlet-api</artifactId>
       </dependency>
       <dependency>
           <groupId>jstl</groupId>
           <artifactId>jstl</artifactId>
       </dependency>
   
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
       </dependency>
   
       <!-- 依赖于service -->
       <dependency>
           <groupId>net.wanho.ums</groupId>
           <artifactId>ums-service</artifactId>
           <version>1.0-SNAPSHOT</version>
       </dependency>
   </dependencies>
   ```
   
3. 部署访问

   先对其他模块进行install操作，然后再对ums-web进行部署访问

#### 2.3 配置dao子工程

​	create.sql

```sql
drop database if exists ums;
create database ums charset utf8;
use ums;
create table t_user(
	id int primary key auto_increment,
	username varchar(100),
	password varchar(100),
	age int
) engine=Innodb charset utf8;
```

​	datasource.properties

```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ums?useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=root
```

#### 2.3 配置service子工程

#### 2.4 配置web子工程

#### 2.5 测试

​	只要修改了其他模块，都需要先对这些模块进行install操作，然后再对ums-web进行部署访问







