---
title: MyBatis学习
date: 2023-08-08 13:28:46
tags: MyBatis
category: Java
---

## 一、简介

### 1. MyBatis是什么

MyBatis的前身叫iBatis

是一个持久层框架，或称为 ORM框架

- 用来访问数据库，做数据持久化操作
- 本质上只是对JDBC进行封装，简化JDBC繁琐的操作

​	注：框架就是别人写好的，对某些技术进行的封装，封装成对应的jar、js、css等，我们可以直接拿过来使用，简化开发

### 2. 持久层

​	DAO：Data Access Object 数据访问对象

​	用来对数据进行持久化操作，如将数据存入数据库、硬盘等，可以永久保存

### 3. ORM

Object Relational Mapping 对象关系映射

Java程序和数据库之间的映射关系：

- 类	 ——>  表 
- 对象  ——> 一条数据
- 属性  ——> 列

### 4. 回顾JDBC

​	JDBC访问数据库的步骤

```java
Class.forName(driverClassName);
Connection conn = DriverManager.getConnection(url,user,password);
PrepareStatement ps = conn.preparedStatement(sql);
//ps.executeUpdate()
ResultSet rs = ps.executeQuery();
while(rs.next){
  //RM(RowMapper)行映射
}
rs.close();
ps.close();
conn.close();
```

数据库操作中的可变部分：

- 连接信息

    dirverClassName、 url、user、password（也称为数据源datasource）

- SQL语句

- RM行映射

## 二、第一个MyBatis程序

### 1. 创建项目并添加依赖

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.5.13</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.46</version>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.24</version>
  <scope>provided</scope>
</dependency>
```

### 2. 数据库设计

```sql
drop database if exists mybatis;
create database mybatis charset utf8;
use mybatis;

create table t_user(
  id int primary key auto_increment comment '编号',
  username varchar(20) unique not null comment '用户名',
  password varchar(50) comment '密码',
  phone varchar(20) comment '电话',
  address varchar(100) comment '地址'
)engine innodb default charset utf8 comment '用户表';
```

### 3. 创建主配置文件config

主配置文件，在一个mybatis工程中有且只有一个

用来配置与整个工程相关的信息，如环境配置、别名配置、插件配置、注册mapper文件等	

文件名可自定义，一般命名为mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--
        environments：配置当前工程中可能使用的所有数据库环境
            default属性：指定默认使用的环境，取值为某一个environment的id
    -->
    <environments default="hello">
        <!--
            envirinment：配置某一个数据库环境，可以有多个
                id属性：指定该环境的唯一标识符
        -->
        <environment id="hello">
            <!--
                transactionManager：配置事务管理器
                    type属性：指定事务管理器的类型，取值有两种：
                        jdbc：使用简单的jdbc事务操作，如开启、提交、回滚
                              在mybatis中，默认是关闭自动提交事务的，即conn.setAutoCommit(false)
                        managed：将事务交给其他框架/容器来处理，如spring
                                mybatis不负责事务，什么都不会做
            -->
            <transactionManager type="jdbc"></transactionManager>
            <!--
                dataSource：配置数据源
                    type属性：配置数据源的类型，取值有三种：
                        UNPOOLED：简单的JDBC配置，未使用连接池，相当于DriverManager.getConnection(url,username,password)
                        POOLED：使用连接池技术
                        JNDI：通过外部容器获取连接

            -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"></property>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?useUnicode=true&amp;characterEncoding=utf-8"></property>
                <property name="username" value="root"></property>
                <property name="password" value="root"></property>
            </dataSource>
        </environment>
        <!--<environment id="world">-->
            <!--<transactionManager type=""></transactionManager>-->
            <!--<dataSource type=""></dataSource>-->
        <!--</environment>-->
    </environments>

    <!--
        注册当前工程中使用的所有映射文件
    -->
    <mappers>
        <!--
             mapper：注册某一个mapper文件，可以有多个
                resource属性：指定映射文件的路径，写的是相对于src的路径，使用正斜杠分隔
        -->
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

小技巧：

- 创建配置文件的模板：Settings——>搜索template——>File and Code Templates——>Files——>Create Template

### 4. 创建映射文件mapper

映射配置文件，在一个mybatis工程中可以有多个mapper文件

用来配置dao功能相关的sql操作，如sql语句、CURD操作、字段映射等

每个实体类对应一个映射文件，每一个mapper文件相当于原来三层架构中dao实现类

文件名可自定义，一般命名为XxxMapper.xml，放到mapper文件夹中

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace属性：指定当前mapper配置文件的唯一标识符，取值为对应接口的全名
-->
<mapper namespace="dao.UserDao">
    <!--
        insert：用来执行添加操作
            id属性：表示当前的方法名，取值必须与接口中的方法名相同
            parameterType属性：表示方法的参数类型
                如果参数是对象，可以使用类的全名
                如果参数是普通数据，可以使用mybatis中的别名，见参考文档12页
            标签体：编写sql语句
                使用#{xxx}表示占位符
                如果参数是对象，则xxx为对象的属性
                如果参数是普通数据，则xxx为参数名
    -->
    <insert id="insertUser" parameterType="entity.User">
        insert into
          t_user
            (username, password, phone, address)
          values
            (#{username},#{password},#{phone},#{address})
    </insert>
</mapper>
```

小技巧：

- 去除背景：Settings——>Editor——>Color Scheme——>General——>Code——>Injected language fragment，取消勾选右边的Background
- XML注释风格：Settings——>Editor——>Code Style——>XML——>Code Generation，取消勾选下面的两个
- 编写SQL时提示表和列：
    1. 在右侧的工具栏，添加数据库连接
    2. Settings——>搜索SQL Dialects——>将右边的都选择为MySQL

### 5. 测试类

```java
public static void main(String[] args) {
    /**
     * 创建SqlSession，称为持久化管理器，是MyBatis操作的核心
     */
    // 1.创建SqlSessionFactoryBuilder
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();

    // 2.创建SqlSessionFactory，读取核心配置文件
    SqlSessionFactory factory = builder.build(Test01.class.getClassLoader().getResourceAsStream("mybatis-config.xml"));

    // 3.创建SqlSession
    SqlSession session = factory.openSession();

    // Connection connection = session.getConnection();
    // System.out.println(connection);


    User user = new User();
    user.setUsername("alice");
    user.setPassword("123");
    user.setPhone("110");
    user.setAddress("南京");

    /**
     * 获取DAO实现类的实例，并执行数据库操作
     */
    UserDao userDao = session.getMapper(UserDao.class); // 参数为接口的Class对象
    // System.out.println(userDao); // 代理对象，通过代理自动生成DAO的实现类
    userDao.insertUser(user);

    session.commit(); // 提交事务
}
```

### 6. MyBatisUtil工具类

MyBatisUtil工具类：

（1）MyBatisUtils主要职责是：● 帮助我们初始化SqlSessionFactory这个对象；同时让SqlSessionFactory全局唯一；● 获得SqlSession对象的方法；● 关闭SqlSession对象的方法；

（2）在实际开发中，会经常使用MyBatisUtils工具类；


添加模板，实现快捷操作：Settings——>搜索template——>Live Templates

```java
SqlSession session = null;
try {
    session = MyBatisUtil.getSession();

    $END$

    session.commit();
} catch (Exception e) {
    e.printStackTrace();
    session.rollback();
} finally {
    MyBatisUtil.close();
}
```

## 三、config文件

### 1. settings 

```xml
<!-- 自定义配置 -->
<settings>
    <!-- 打印sql -->
    <!--<setting name="logImpl" value="org.apache.ibatis.logging.stdout.StdOutImpl"/>-->
  	<setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

logImpl可以输出日志
注：setting一定要在environment的上面，否则会报错。

### 2. typeAliases

```xml
<!--
    配置别名，为当前工程中的某些类指定别名
-->
<typeAliases>
    <!--
        typeAlias：为某个类配置别名
            type属性：指定类名
            alias属性：指定类的别名
    -->
    <!--<typeAlias type="entity.User" alias="User"/>-->
    <!--
        package：为某个包下的所有类配置别名
            name属性：指定包名，该包下所有类的别名就是其类名（别名不区分大小写，但建议与类名完全一致）
    -->
    <package name="entity"/>
</typeAliases>
```

### 3. properties

```xml
<!--
    引用外部的properties文件
-->
<properties resource="datasource.properties"></properties>
```

```xml
<!--
    通过${key}访问properties文件中的值
-->
<property name="driver" value="${jdbc.driver}"></property>
<property name="url" value="${jdbc.url}"></property>
<property name="username" value="${jdbc.username}"></property>
<property name="password" value="${jdbc.password}"></property>
```

## 四、mapper文件

### 1. insert

​	保存返回主键

```xml
<!--
    useGeneratedKeys属性：设置保存时是否返回主键，取值有两个：
        false：表示不返回主键，默认值
        true：表示返回主键，会自动将返回的主键绑定到参数对象的主键属性中
    keyProperty属性：指定对象的哪个属性为主键属性，即主键所映射的属性，必须指定
-->
<insert id="insert" parameterType="User" useGeneratedKeys="true" keyProperty="id">
    insert into t_user
        (username, password, phone, address)
   	values
        (#{username},#{password},#{phone},#{address})
</insert>
```

### 2. update

```xml
<!--
    update：执行修改操作
-->
<update id="updateUser" parameterType="User">
  update t_user
  set username=#{username},
      password=#{password},
      phone=#{phone},
      address=#{address}
  where id=#{id}
</update>
```

### 3. delete

```xml
<!--
    delete：执行删除操作
-->
<delete id="deleteById" parameterType="int">
    delete from t_user
    where id=#{id}
</delete>
```

### 4. select

```xml
<!--
    select：执行查询操作
        resultType属性：表示返回的结果类型
            如果返回的是一个对象，会自动进行映射
            前提条件：查询结果的字段名必须与对象的属性名完全相同
-->
<select id="selectById" parameterType="int" resultType="User">
    select id,username,password,phone,address
    from t_user
    where id=#{id}
</select>

<!--
    当方法返回值为对象集合时，resultType指定的是集合中对象的类型，而非集合本身
-->
<select id="selectAll" resultType="User">
    <!--
        include：用于引用sql代码段
            refid属性：指定要引用的sql代码段的id值
     -->
    select <include refid="userColumn"/>
    from t_user
</select>

<!--
    模糊查询
-->
<select id="selectByUsername" resultType="User">
    select <include refid="userColumn"/>
    from t_user
    <!-- 方式1：使用concat()函数拼接 -->
    <!-- where username like concat('%',#{username},'%') -->

    <!-- 方式2：传参时直接拼接上% -->
    where username like #{username}
</select>
```

### 5. sql片段

```xml
<!--
    sql：定义sql代码段，便于复用
        id属性：指定该sql代码段的唯一标识符
-->
<sql id="UserColumn">
    id,username,password,phone,address
</sql>

<sql id="BaseQuery">
    select id,username,password,phone,address
    from t_user
</sql>
```

## 五、手动映射

​	当数据库查询结果的字段名与Java对象的属性名不同时，如何映射？

```sql
create table t_user2(
  user_id int primary key auto_increment comment '编号',
  user_username varchar(20) unique not null comment '用户名',
  user_password varchar(50) comment '密码',
  user_phone varchar(20) comment '电话',
  user_address varchar(100) comment '地址'
)engine innodb default charset utf8 comment '用户表';
```

### 1. 使用别名

```xml
<!--
    使用别名：为查询结果的每个字段指定别名，与对象的属性名相同，此时相当于自动映射
-->
<select id="selectById2" parameterType="int" resultType="User">
    select
      user_id id,
      user_username username,
      user_password password ,
      user_phone phone ,
      user_address address
    from
      t_user2
    where
      user_id=#{id}
</select>
```

### 2. 使用resultMap

```xml
<!--
    resultMap：定义结果映射，将数据库的字段与对象的属性进行映射
        id属性：指定该resultMap的唯一标识符
        type属性：映射的对象类型
-->
<resultMap id="UserMap" type="User">
    <!--
        id：配置主键映射
        result：配置其他映射
            property属性：映射的属性名
            column属性：映射的字段名
    -->
    <id property="id" column="user_id"/>
    <result property="username" column="user_username"/>
    <result property="password" column="user_password"/>
    <result property="phone" column="user_phone"/>
    <result property="address" column="user_address"/>
</resultMap>
```

```xml
<!--
    resultMap属性：引用一个resultMap，使用该resultMap进行手动映射
        其值为已存在的某一个resultMap标签的id值
-->
<select id="selectById3" parameterType="int" resultMap="UserMap">
    select
      user_id,
      user_username,
      user_password ,
      user_phone,
      user_address
    from t_user2
    where user_id=#{id}
</select>
```

## 六、多个参数

### 1. 使用@Param()注解

```java
// 使用@Param()注解，标注在参数前，为参数指定占位符名称
public User selectByUsernameAndPassword(@Param("username") String username, @Param("password") String password);
```

```xml
<!--
    使用@Param()定义的名称引用指定的参数
-->
<select id="selectByUsernameAndPassword" resultType="User">
    <include refid="BaseQuery"/>
    where username=#{username} and password=#{pwd}
</select>
```

​	注：如果方法只有一个参数，不需要加@Param注解，可以在占位符#{ }中使用任意名称

### 2. 将参数封装为对象

```xml
<!-- 将多个参数封装成一个对象，然后传递该对象 -->
<select id="selectByUsernameAndPassword2" parameterType="UserDTO" resultType="User">
    <include refid="BaseQuery"/>
    where username=#{username} and password=#{password}
</select>
```

​	注：可以自定义一个参数对象，如UserDTO、UserParam等

### 3. 将参数封装为Map

```xml
<!--
    将多个参数封装成一个Map集合，在#{}占位符中根据key获取value
-->
<select id="selectByUsernameAndPassword3" resultType="User">
    <include refid="BaseQuery"/>
    where username=#{username} and password=#{password}
</select>
```

​	注：parameterType属性可以省略

### 4. #{}与${}的区别

#{ }的含义

- #{ }表示一个占位符  即JDBC中的？
- 占位不会出现SQL注入的问题

${ }的含义

- ${}表示一个拼接符
- 拼接会导致SQL注入，如查询条件输入`'' or 1=1`就会有注入情况
- 如果排序时要指定排序列名和排序方式，此时就必须使用字符的拼接，即使用${ }

## 七、动态SQL

​	根据条件的不同，动态的拼接SQL语句，称为动态SQL

​	传统JDBC拼接：

```sql
StringBuffer sql=new StringBuffer();
sql.append("select * from t_user where 1=1");
if(username!=null && !"".equals(username)){
	sql.append("  and username=? ");
}
if(password!=null && !"".equals(password)){
	sql.append("  and password=? ");
}
```

### 1. if

```xml
<!--
    if标签：用来进行条件的判断
        test属性：判断表达式的值，如果为true，则拼接该sql片段，否则不拼接
-->
<select id="selectByParams" parameterType="User" resultType="User">
    <include refid="BaseQuery"/>
    where 1=1
    <if test="username != null and username != ''">
        and username = #{username}
    </if>
    <if test="password != null and password != ''">
        and password = #{password}
    </if>
    <if test="phone != null and phone != ''">
        and phone = #{phone}
    </if>
    <if test="address != null and address != ''">
        and address = #{address}
    </if>
</select>
```

### 2. choose

```xml
<!--
    choose标签：用来进行条件的选择，只会拼接一个SQL
        when标签：
            test属性：判断表达式的值，如果为true，则拼接该sql片段，此时不再判断其它when
        otherwise标签：当所有when都不成立时，则拼接该sql片段
-->
<select id="selectByParams2" parameterType="User" resultType="User">
    <include refid="BaseQuery"/>
    where
    <choose>
        <when test="username != null and username != ''">
            username = #{username}
        </when>
        <when test="password != null and password != ''">
            password = #{password}
        </when>
        <when test="phone != null and phone != ''">
            phone = #{phone}
        </when>
        <when test="address != null and address != ''">
            address = #{address}
        </when>
        <otherwise>
            1=1
        </otherwise>
    </choose>
</select>
```

### 3. where

```xml
<!--
    where标签：一般结合if或choose一起使用
        作用：1.添加where关键字
             2.删除sql片段的第一个连接关键字，如and、or等
             3.如果没有拼接任何sql片段，则不会添加where关键字
-->
<select id="selectByParams3" parameterType="User" resultType="User">
    <include refid="BaseQuery"/>
    <where>
        <if test="username != null and username != ''">
            and username = #{username}
        </if>
        <if test="password != null and password != ''">
            or password = #{password}
        </if>
        <if test="phone != null and phone != ''">
            and phone = #{phone}
        </if>
        <if test="address != null and address != ''">
            and address = #{address}
        </if>
    </where>
</select>
```

### 4. set

```xml
<!--
    set标签：一般结合if或choose一起使用
        作用：1.添加set关键字
             2.删除sql片段的末尾逗号
-->
<update id="updateUser2" parameterType="User">
    update t_user
    <set>
        <if test="username != null and username != ''">
            username = #{username},
        </if>
        <if test="password != null and password != ''">
            password = #{password},
        </if>
        <if test="phone != null and phone != ''">
            phone = #{phone},
        </if>
        <if test="address != null and address != ''">
            address = #{address},
        </if>
    </set>
    where id = #{id}
</update>
```

### 5. trim

```xml
<!--
    trim标签：
        作用：1.在开头或末尾添加特定的前缀prefix或后缀suffix
             2.删除开头prefixOverrides或末尾suffixOverrides的特定内容
         注：当属性值可能有多个时，可以使用竖杠|来表示或者的意思，且竖杠的后面不能有空格
-->
<insert id="insertUser2" parameterType="User">
    insert into t_user
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null">id,</if>
            <if test="username != null and username != ''">username,</if>
            <if test="password != null and password != ''">password,</if>
            <if test="phone != null and phone != ''">phone,</if>
            <if test="address != null and address != ''">address,</if>
        </trim>
    values
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null">#{id},</if>
            <if test="username != null and username != ''">#{username},</if>
            <if test="password != null and password != ''">#{password},</if>
            <if test="phone != null and phone != ''">#{phone},</if>
            <if test="address != null and address != ''">#{address},</if>
        </trim>
</insert>
```

### 6. foreach

```xml
<!--
    foreach标签：当参数是集合时，用来对集合进行遍历，一般用在in条件中
        collection属性：要遍历的集合，默认List集合指定为list，Map集合指定为map，数组指定为array
        item属性：迭代变量
        open属性：遍历前添加的字符串
        close属性：遍历后添加的字符串
        separator属性：元素分隔符
        index属性：当前迭代元素的索引
-->
<select id="selectByIds" resultType="User">
    <include refid="BaseQuery"/>
    where id in
    <foreach collection="list" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
</select>
```

## 八、多表关系映射

### 1. 简介

关联关系：

- 多对一，多个员工都在同一个部门中
- 一对多，一个部门中有多个员工
- 一对一，一个员工只能有一个身份证
- 多对多，一个员工可以同时开发多个项目，一个项目也可以同时有多个员工开发

数据库设计：

```sql
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

### 2. 保存操作

```xml
<mapper namespace="dao.DeptDao">
    <insert id="insertDept" parameterType="Dept" useGeneratedKeys="true" keyProperty="id">
        insert into t_dept
          (name)
        values
          (#{name})
    </insert>
</mapper>
```

```xml
<mapper namespace="dao.EmpDao">
    <insert id="insertEmp" parameterType="Emp">
      insert into t_emp
        (name, salary, dept_id)
      values
        (#{name},#{salary},#{dept.id})
    </insert>
</mapper>
```

### 3. 多对一

在一个对象中定义另一个对象的属性

两种实现方式：

- 使用关联属性，即直接使用association标签
- 使用嵌套查询，即使用association的select属性，引用其他select，通过多个单表查询来实现

```xml
<resultMap id="BaseMap" type="Emp">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <result property="salary" column="salary"/>
</resultMap>

<resultMap id="EmpMap" type="Emp" extends="BaseMap">
    <!--
        association：用于配置关联属性，多对一的关系
            property属性：当前需要映射的是对象中的哪个属性
            javaType属性：当前映射的属性的Java类型
            标签体：对当前映射的属性所在的表进行映射
    -->
    <association property="dept" javaType="Dept">
        <id property="id" column="deptId"/>
        <result property="name" column="deptName"/>
    </association>
</resultMap>

<resultMap id="EmpMap2" type="Emp" extends="BaseMap">
    <!--
        select属性：引用其他的select查询配置
                值为：select所在Mapper文件的namespace.select的id值
        column属性：当前查询的某列，作为查询条件，传递给引用的select查询配置的参数
    -->
    <association property="dept" javaType="Dept" select="dao.DeptDao.selectById" column="dept_id">
    </association>
</resultMap>

<select id="selectAll" resultMap="EmpMap">
 select <include refid="EmpColumn"/>
 from t_emp e
   left join t_dept d
   on e.dept_id=d.id
</select>

<select id="selectAll" resultMap="EmpMap2">
    select id,name,salary,dept_id
    from t_emp
</select>
```

嵌套查询的缺点：效率低，会进行多次查询，存在N+1问题

- 首先查询了1次emp表，获取到N条dept_id的记录
- 对于每一个不同的dep_id，都会去dept表中进行一次查询，可能会查询N次
- 所以，总查询次数可能为：1次emp表+N次dept表

### 4. 一对多

在一个对象中定义另一个对象的集合

两种实现方式：

- 使用集合属性，即直接使用collection标签
- 使用嵌套查询，即使用collection的select属性，引用其他select，通过多个单表查询来实现

```xml
<resultMap id="BaseMap" type="Dept">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
</resultMap>

<resultMap id="DeptMap" type="Dept" extends="BaseMap">
    <!--
        collection：用于配置集合属性，一对多的关系
            property属性：当前需要映射的集合属性
            ofType属性：集合属性中对象的类型
            标签体：对集合属性中对象所在的表进行映射
    -->
    <collection property="emps" ofType="Emp">
        <id property="id" column="empId"/>
        <result property="name" column="empName"/>
        <result property="salary" column="salary"/>
    </collection>
</resultMap>

<resultMap id="DeptMap2" type="Dept" extends="BaseMap">
    <!--
       select属性
    -->
    <collection property="emps" ofType="Emp" select="dao.EmpDao.selectByDeptId" column="id"/>
</resultMap>

<select id="selectAll" resultMap="DeptMap">
  select <include refid="DeptColumn"/>
  from t_dept d
    left join t_emp e
    on d.id=e.dept_id
</select>

<select id="selectAll" resultMap="DeptMap2">
    select id,name
    from t_dept
</select>
```

### 5. 懒加载

```xml
<resultMap id="DeptMap2" type="Dept" extends="BaseMap">
    <!--
        fetchType属性：指定集合属性的加载时机
            lazy：延迟加载，当使用到集合属性时，才会加载
            eager：立即加载，查询主表时，就会查询集合属性
        也可以在主配置文件中，通过setting标签，配置全局的fetchType属性
    -->
   	<collection property="emps" ofType="Emp" select="dao.EmpDao.selectByDeptId" column="id" fetchType="lazy"/>
</resultMap>

<settings>
  	<!-- 懒加载 -->
  	<setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

## 九、缓存

### 1. 简介

将从数据库中查询出来的数据放入缓存中，下次使用时不必从数据库查询，而是直接从缓存中读取，避免频繁操作数据库，减轻数据库的压力，同时提高系统性能。

合理使用缓存是优化中最常见的操作。

### 2. 一级缓存

一级缓存是`SqlSession`级别的，存储在SqlSession中，默认是开启的

一般来说，一个请求中的所有增删改查操作都是在 同一个sqlSession里面的，可以认为每个请求都有自己的一级缓存

如果同一个SqlSession会话中 2个查询中间有一个 insert 、update或delete 语句，那么之前查询的所有缓存都会清空

流程：

- 用户发起查询请求，查找某条数据，SqlSession 先去缓存中查找，是否有该数据，如果有，读取；
- 如果没有，从数据库中查询，并将查询到的数据放入一级缓存区域，供下次查找使用。
- 当SqlSession 执行commit，即增删改操作时会清空缓存。这么做的目的是避免脏读。

生效的条件：

- 必须使用同一SqlSession，执行同一个查询方法才会有效
- 同一个SqlSession，如果查询条件不同，则无效
- 同一个SqlSession，如果两次查询期间执行了任何一次的增删改操作，则无效

### 3. 二级缓存

二级缓存是 mapper 级别的缓存，多个SqlSession去操作同一个Mapper的sql语句时，多个SqlSession可以共用二级缓存

二级缓存是跨SqlSession的，因此二级缓存的作用范围更大

二级缓存默认是关闭的，需要手动开启

流程：

- 开启二级缓存后，用户查询时，会先去二级缓存中找，找不到了再去一级缓存中找
- 一级缓存也没有查询到，则查询数据库
- 当SqlSession会话提交或者关闭时，一级缓存的数据会刷新到二级缓存中

启用二级缓存：在 `XxxMapper.xml` 映射文件中，添加：`<cache/>`

## 十、分页插件

### 1. 简介

​	PageHelper是一款基于mybatis的分页插件

### 2. 用法

#### 2.1 添加依赖

```xml
<dependency>
  <groupId>com.github.pagehelper</groupId>
  <artifactId>pagehelper</artifactId>
  <version>5.1.2</version>
</dependency>
```

#### 2.2 插件的配置

​	在mybatis核心配置文件中配置插件

```xml
<plugins>
    <!-- 配置mybatis分页插件 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
       <property name="helperDialect" value="mysql"/>
    </plugin>
</plugins>
```

#### 2.3 使用

​	分为三步：

```java
// 1.配置分页信息，指定页码、页大小
int pageNum = 3;
int pageSize = 4;
PageHelper.startPage(pageNum, pageSize);

// 2.获取原始数据
UserDao userDao = session.getMapper(UserDao.class);
List<User> users = userDao.selectAll();
for (User user:users){
  System.out.println(user);
} 

// 3.将原始数据封装成分页数据
PageInfo<User> pageInfo=new PageInfo<>(users);
System.out.println(pageInfo);

System.out.println("页码："+pageInfo.getPageNum());
System.out.println("页大小："+pageInfo.getPageSize());
System.out.println("总页数："+pageInfo.getPages());
System.out.println("总条数："+pageInfo.getTotal());
System.out.println("分页数据："+pageInfo.getList());
```

## 十一、其他

### 1. MyBatisX

​	一款全免费且强大的 IDEA 插件，支持跳转，自动补全生成 SQL，代码生成。

```sql
create table t_product
(
    pro_id int primary key auto_increment comment '编号',
    pro_name varchar(100) comment '产品名称',
    pro_price decimal(10,2) comment '产品价格',
  	pro_number int comment '产品数量',
  	pro_introduce text comment '产品介绍',
  	pro_state tinyint comment '产品状态(0未通过,1审核中,2已审核)',
  	add_time datetime comment '添加时间',
  	is_del tinyint comment '是否删除(0正常,1删除)'
) engine innodb default charset utf8 comment '产品表';
```

```XML
<settings>
    <!-- 下划线映射为驼峰 -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

### 2. MyBatisCodeHelperPro

​	功能更强大，收费！

