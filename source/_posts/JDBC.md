---
title: JDBC
date: 2023-07-11 18:37:55
tags:
  - JDBC
  - Java
category: 数据库
---
JDBC API 允许用户访问任何形式的表格数据，尤其是存储在关系数据库中的数据。
## 1. JDBC

### 1.1 简介

![image-20220506202345258](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506202345258.png)

### 1.2 工作原理

![image-20220506202452003](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506202452003.png)

### 1.3 JDBC API

![image-20220506202537196](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506202537196.png)

### 1.4 JDBC 驱动

![image-20220506202623525](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506202623525.png)

![image-20220506202656291](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506202656291.png)

### 1.5 Connection[会话]

![image-20220506203007872](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506203007872.png)


### 1.6 Statement【执行SQL】

#### Statement

用于执行静态 SQL 语句

![image-20220506204152891](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506204152891.png)

![image-20220506204629191](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506204629191.png)

![image-20220506204512610](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506204512610.png)

#### PreparedStatement

表示预编译的 SQL 语句的对象。

![image-20220506204254289](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506204254289.png)

![image-20220506204400101](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506204400101.png)

#### CallableStatement

用于执行 SQL 存储过程的接口。

#### 存储过程定义

```sql
DELIMITER $$

CREATE PROCEDURE p_save_dept(v_deptno INT,v_dname VARCHAR(20),v_loc VARCHAR(20) ,OUT v_rs INT )
 BEGIN
   DECLARE cut INT(1) ;
   SELECT COUNT(0) INTO cut FROM dept WHERE deptno = v_deptno ;
   IF cut = 1 THEN 
     SET v_rs = -1 ; 
   END IF ;
   IF cut = 0 THEN 
     SET v_rs = 1 ;
     INSERT INTO dept (deptno,dname,loc) VALUES (v_deptno,v_dname,v_loc) ;
     COMMIT ;
   END IF ;
END  $$

DELIMITER ;
```

#### 数据库直接调用

```sql
CALL p_save_dept(11,'aa','NJ',@rs) ;
SELECT @rs ;
```

#### JDBC调用

```java
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
CallableStatement cs = conn.prepareCall("{call p_save_dept(?,?,?,?)}");
cs.setInt(1,12);
cs.setString(2,"dev");
cs.setString(3,"NJ");
cs.registerOutParameter(4, JDBCType.INTEGER);
cs.execute() ;
int out = cs.getInt(4);
System.out.println(out);
cs.close();
conn.close();
```



### 1.7 ResultSet 【查询结果集】

![image-20220506204212423](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506204212423.png)

## 2. DAO

### 2.1 什么是DAO

![image-20220506210707068](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506210707068.png)

### 2.2 DAO作用

![image-20220506210831671](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506210831671.png)

### 2.3 组成部分

![image-20220506210903737](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506210903737.png)

### 2.4 示例

![image-20220506210953246](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220506210953246.png)

## 3.Dbutils
Dbutils：主要是封装了JDBC的代码，简化dao层的操作。
作用：帮助java程序员，开发Dao层代码的简单框架。
框架的作用：帮助程序员，提高程序的开发效率。

### 3.1 环境搭建

jar包下载

https://mvnrepository.com/artifact/commons-dbutils/commons-dbutils/1.7

![image-20230704123211576](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230704123211576.png)



### 3.2 官网地址

https://commons.apache.org/proper/commons-dbutils/examples.html

### 3.3 Dbutils三个核心类介绍
1：DbUtils：连接数据库对象----jdbc辅助方法的集合类，线程安全
构造方法：DbUtils()
作用：控制连接，控制事务，控制驱动加载额一个类。
     
2：QueryRunner：SQL语句的操作对象，可以设置查询结果集的封装策略，线程安全。
构造方法：
（1）QueryRunner()：创建一个与数据库无关的QueryRunner对象，后期再操作数据库的会后，需要手动给一个Connection对象，它可以手动控制事务。
Connection.setAutoCommit(false);     设置手动管理事务
Connection.commit();     提交事务
 
（2）QueryRunner(DataSource ds)：创建一个与数据库关联的queryRunner对象，后期再操作数据库的时候，不需要Connection对象，自动管理事务。
DataSource：数据库连接池对象。
```java
      //构造函数与增删改查方法的组合：
     QueryRunner()
           update(Connection conn, String sql, Object... params)
           query(Connection conn, String sql, ResultSetHandler<T> rsh, Object... params)
 
     QueryRunner(DataSource ds)     
           update(String sql, Object... params)
           query(String sql, ResultSetHandler<T> rsh, Object... params)
```
 
3：ResultSetHandle：封装数据的策略对象------将封装结果集中的数据，转换到另一个对象
策略：封装数据到对象的方式（示例：将数据库保存在User、保存到数组、保存到集合）
方法介绍：handle（ResultSet rs）

### 3.4 创建DBCP连接池
1、创建DBCP链接池配置文件：名称为 dbcp.properties      内容如下：
```
      driverClassName=com.mysql.jdbc.Driver
      username=root
      password=123456
      url=jdbc:mysql://127.0.0.1:3306/long1?characterEncoding=UTF8
      maxActive=2
```
2、读取配置文件，创建 DataSource 连接池实例
```java
      package com.test.dbcp;
      import java.sql.Connection;
      import java.util.Properties;
      import javax.sql.DataSource;
      import org.apache.commons.dbcp.BasicDataSourceFactory;
      public class DbcpDataSource {
     	/*
       * 重点：创建一个DataSource
       * 步骤为：1、用Properties类读取配置文件。
       * 2、通过工厂类，读取这个Properties类获取的配置文件，创建出DataSource
       * DataSource作用：
       * 创建DataSource可以返回多个连接。可以实现dbutil简化操作数据库的流程。
       */
     	//1、创建一个静态的datasource
     	private static DataSource ds;
     	//2、在静态代码块中，给ds赋值
     	static{
     		//读取资源文件
     		try{
     			Properties p = new Properties();
      			p.load(DbcpDataSource.class.getClassLoader().getResourceAsStream("dbcp.properties"));
     			//在dbcp中有一个工厂类，读取一个资源文件,创建一个datasource
      			ds = BasicDataSourceFactory.createDataSource(p);
      			System.out.println("创建DataSource为"+ds);
      		}catch(Exception e){
     			throw new RuntimeException(e);
      		}
      	}
     	//提供一个方法用于获取整个datasource对象
     	public static DataSource getDataSource(){
      		return ds;
      	}
     	//提供一个方法，获取connection连接
     	public static Connection getConnection(){
     		Connection con = null;
     		try{
      			con = ds.getConnection();
      			System.out.println("通过DataSource获取connection连接"+con);
      		}catch(Exception e){
      			e.printStackTrace();
      		}
     		return con;
      	}
      }
```

另外也可以不配置文件直接使用
```java
    static {
        //模拟初始化数据
        Properties prop = new Properties();
        prop.setProperty("driverClassName","com.mysql.jdbc.Driver");
        prop.setProperty("url","jdbc:mysql://localhost:3306/customer?useSSL=false&characterEncoding=utf8");
        prop.setProperty("password","root");
        prop.setProperty("username","root");
        try{
            dataSource = DruidDataSourceFactory.createDataSource(prop);
        }catch (Exception e) {
            e.printStackTrace();
        }
        //初始化 连接池
    }
```

### 3.5 DBUtil 增删改查
```java
      package com.test.ts;
      import java.sql.SQLException;
      import java.util.List;
      import javax.persistence.Version;
      import org.apache.commons.dbutils.QueryRunner;
      import org.apache.commons.dbutils.handlers.ArrayHandler;
      import org.apache.commons.dbutils.handlers.ArrayListHandler;
      import org.junit.Test;
      import com.test.dbcp.DbcpDataSource;
      public class DbutilTests {
     	/*
       * dbutil向数据库新增数据
       */
     	@Test
     	public void add() throws SQLException{
     		//1、申明QueryRunner对象执行SQL语句
     		QueryRunner run1 = new QueryRunner(DbcpDataSource.getDataSource()); //申明对象，获取DataSource链接池
     		//2、书写SQL字符串语句
     		String sq1 = "INSERT INTO student(id,namee,sex,birth,department,address)"
      				+ "VALUES(3,'王思','男',1995,'英文系','山东沧州')";
     		//3、执行sql语句
     		int result1 = run1.update(sq1);
      		System.out.println("新增数据库结果，更新了数据有："+result1+"条");
      	}
     	/*
       * dbutil 修改数据库数据
       *
       */
     	@Test
     	public void update() throws SQLException{
     		//1、申明QueryRunner对象执行SQL语句
     		QueryRunner run2 = new QueryRunner(DbcpDataSource.getDataSource());//申明对象，获取DataSource连接池
     		//2、书写SQL字符串语句
     		String sq2 = "UPDATE student SET sex=? WHERE id=?";
     		//3、执行SQL语句
     		int result2 = run2.update(sq2, "男",16);
      		System.out.println("更新数据库结果，更新了数据有："+result2+"条");
      	}
     	/*
       * dbutil 删除数据库数据
       */
     	@Test
     	public void delete() throws SQLException{
     		//1、申明QueryRunner对象执行SQL语句
     		QueryRunner run3 = new QueryRunner(DbcpDataSource.getDataSource());//申明对象，获取DataSource连接池
     		//2、书写SQL字符串语句（这个sql语句的作用是每次删除ID号最大的数据）
     		String sq3 = "DELETE FROM student WHERE id IN "
      				+ "(SELECT a.id FROM "
      				+ "(SELECT MAX(id) id FROM student a WHERE id IN"
      				+ "(SELECT id FROM student b WHERE a.id=b.id ORDER BY id DESC)) a)";
     		//3、执行SQL语句
     		int result3 = run3.update(sq3);
      		System.out.println("更新数据库结果，更新了数据有："+result3+"条");
      	}
     	/*
       * dbutil 查询，将结果封装成Object对象，返回第一行数据
       */
     	@Test
     	public void query() throws SQLException{
     		//1、申明QueryRunner对象执行SQL语句
     		QueryRunner run = new QueryRunner(DbcpDataSource.getDataSource());//申明对象，获取DataSource链接池
     		//2、查询
     		String sq4 = "Select * from student";
      		Object[] var = run.query(sq4, new ArrayHandler());
     		if(var !=null){
      			System.out.print("输出学生表查询的信息"+"\n");
     			for(Object o :var){
      				System.out.print(o+"\n");
      			}
      		}
      	}
     	/*
       * dbutil 查询 ，将结果封装成Object对象，返回所有的数据
       *
       */
     	@Test
     	public void query2() throws SQLException{
     		//1、申明QueryRunner对象执行SQL语句
     		QueryRunner run5 = new QueryRunner(DbcpDataSource.getDataSource());
     		//2、书写sql语句
     		String sq5 = "select * from student";
     		//3、查询sql
      		List<Object[]> list = run5.query(sq5, new ArrayListHandler());
     		//4、遍历结果
     		if(list!=null){
     			for(Object[] os:list){
     				for(Object o :os){
      					System.out.println("ArrayListHandler()显示查询到的所有数据："+o+"\t");
      				}
      				System.out.println("\n--------------------------------------------");
      			}
      		}
      	}
      }
```


  
