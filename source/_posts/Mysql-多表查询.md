---
title: Mysql-多表查询
date: 2023-07-19 00:27:20
tags: Mysql
category:  数据库
---
# 1.DDL

create  创建   alter 修改  drop 删除   truncate清空/截断 【事务自动提交】

## 1.1 创建表

常用创建

```sql
表名 练习： t_emp  tb_emp  tbl_emp
    项目： sys_  oa_   order_  product_
create table 表名(
    字段名 数据类型 [列级约束],
    字段名 数据类型 [列级约束],
    [表级约束]
) ;
```

```sql
CREATE TABLE t_emp01(
  id INT,
  NAME VARCHAR(20)
) ;
```

查看表结构

```sql
DESC t_emp01;
```

查看表创建语句

```sql
SHOW CREATE TABLE t_emp01 ;
```

CTAS语法创建表【没约束】

```sql
CREATE TABLE emp10 AS SELECT * FROM emp WHERE deptno = 10;
CREATE TABLE emp20 AS SELECT empno id,ename NAME,sal sal FROM emp WHERE deptno = 20;
CREATE TABLE myemp AS SELECT * FROM emp WHERE 1=2 ;
```

## 1.2 约束

主键 PK

非空+唯一

```sql
CREATE TABLE t_emp02(
  id INT PRIMARY KEY,
  NAME VARCHAR(20)
) ;
或
CREATE TABLE t_emp03(
  id INT ,
  NAME VARCHAR(20),
  PRIMARY KEY (id)
) ;
INSERT INTO t_emp02 (id,NAME) VALUES ('1','a') ;
INSERT INTO t_emp02 (id,NAME) VALUES ('1','b') ;

查询：INSERT INTO t_emp02 (id,NAME) VALUES ('1','b') 错误代码： 1062
Duplicate entry '1' for key 'PRIMARY'
```



```sql
CREATE TABLE t_emp04(
  id INT ,
  NAME VARCHAR(20),
  PRIMARY KEY (id,NAME)  -- 联合主键
) ;
INSERT INTO t_emp04 (id,NAME) VALUES ('1','a') ;
INSERT INTO t_emp04 (id,NAME) VALUES ('1','b') ;
INSERT INTO t_emp04 (id,NAME) VALUES ('2','b') ;
INSERT INTO t_emp04 (id,NAME) VALUES ('2','a') ;
```

查看表约束

```sql
-- 查询数据字典：
SELECT * FROM `information_schema`.`TABLE_CONSTRAINTS` 
    WHERE TABLE_SCHEMA='scott' AND table_name='t_emp04';
```

外键 FK

参考完整性  可以为 NULL  给值 只能关联 另一个主键

```sql
CREATE TABLE mydept AS SELECT * FROM dept ;
CREATE TABLE myemp02 (
  id INT PRIMARY KEY,
  NAME VARCHAR(20),
  deptno INT REFERENCES mydept(deptno)
) ;
或
CREATE TABLE myemp03 (
  id INT PRIMARY KEY,
  NAME VARCHAR(20),
  deptno INT ,
  CONSTRAINT fk_myemp03 FOREIGN KEY (deptno) REFERENCES mydept(deptno) -- 注意：PK
) ;

INSERT INTO myemp03(id,NAME,deptno) VALUES (3,'aaa',NULL) ;
INSERT INTO myemp03(id,NAME,deptno) VALUES (4,'aaa',88) ;   -- 88 部门表中没有该数据

查询：INSERT INTO myemp03(id,NAME,deptno) VALUES (4,'aaa',88) 错误代码： 1452
Cannot add or update a child row: a foreign key constraint fails (`scott`.`myemp03`, CONSTRAINT `fk_myemp03` FOREIGN KEY (`deptno`) REFERENCES `mydept` (`DEPTNO`))


DELETE FROM `mydept` WHERE deptno = 10 ;   -- 10部门下 有关联的员工
查询：delete from `mydept` where deptno = 10 错误代码： 1451
Cannot delete or update a parent row: a foreign key constraint fails (`scott`.`myemp03`, CONSTRAINT `fk_myemp03` FOREIGN KEY (`deptno`) REFERENCES `mydept` (`DEPTNO`))
```

非空

```sql
CREATE TABLE t_emp05(
  id INT PRIMARY KEY,
  NAME VARCHAR(20) NOT NULL   -- 该列不能为空
) ;
```

唯一

```sql
CREATE TABLE t_emp05(
  id INT PRIMARY KEY,
  NAME VARCHAR(20) UNIQUE   -- 该列唯一  可以为 null 【只能一个null】
) ;
```

默认值

```sql
CREATE TABLE t_emp05(
  id INT PRIMARY KEY,
  NAME VARCHAR(20) ,
  age INT DEFAULT 18 
) ;
```

自增长

```sql
-- 历史遗留问题：ID 使用雪花算法 【通过程序控制】
CREATE TABLE t_emp08(
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20) 
) ;

INSERT INTO t_emp08 (id,NAME) VALUES (NULL,'a') ;
INSERT INTO t_emp08 (id,NAME) VALUES (200,'b') ;
INSERT INTO t_emp08 (id,NAME) VALUES (NULL,'c') ;
```

## 1.3 修改表

注意： 权限问题 ？

修改表名

```sql
ALTER TABLE t_emp08 RENAME TO tbl_emp08 ;
```

修改字段名

```sql
ALTER TABLE tbl_emp08 CHANGE NAME e_name VARCHAR(20);
```

修改字段类型

```sql
ALTER TABLE tbl_emp08 MODIFY e_name VARCHAR(80);
```

添加字段

```sql
ALTER TABLE tbl_emp08 ADD age INT ;
ALTER TABLE tbl_emp08 ADD sex INT AFTER id ;
```

删除字段

```sql
ALTER TABLE tbl_emp08 DROP COLUMN sex ;
```

## 1.4 删除表

```sql
-- 最新的备份  最新的简历
DROP TABLE tbl_emp08 ;
```

## 1.5 清空表

```sql
-- 1. 清空表中数据  表结构保留  DDL事务自动提交  不可回滚  速度快  自增长从1开始

-- 降低高水位
TRUNCATE TABLE myemp03 ;

TRUNCATE TABLE mydept;  -- FK 外键引用
查询：truncate table mydept错误代码： 1701
Cannot truncate a table referenced in a foreign key constraint (`scott`.`myemp03`, CONSTRAINT `fk_myemp03` FOREIGN KEY (`deptno`) REFERENCES `scott`.`mydept` (`DEPTNO`))

-- 2. delete  where过滤条件  DML语句  事务不提交 可以回滚  速度慢
delete from myemp03 where ....
```

## DDL练习

```sql
1.创建一张表 student 
 	id int  
    name varchar(10)  
    age  int(10)  
    tel varchar（10） 
 给 id 字段添加主键约束  自增长 
 给 name 字段添加非空约束  
 给 age 字段 默认值 0
 给 tel 添加唯一 非空 约束 


2.创建一张学员兴趣爱好表 hobby 
 	id int(10)  
	hobby_name varchar(10) 
 	sid int 
--学生 id 
 给 sid 字段添加外键约束
 
 
3.创建一个表
emp1
empno int(10)
ename varchar(50)


4.emp1 添加一个字段
 sal int(4)

5.emp1 修改字段 
 ename varchar(100)
 
6.emp1 删除字段 sal

7.把表 emp1 改成 emp2 

```



# SQL面试题

## 面试题一（厦门）

```sql
Table: （员工emp1）
  id name
  1 a 
  2 b 
  3 c
  4 d
Table:( 性别sext)
  id sex
  1  男
  4  女
  5  男
找出忘记填写性别的员工?
  create table emp1(id int,name varchar(20)) ;
  insert into emp1(id,name) values (1,'a'),(2,'b'),(3,'c'),(4,'d') ;
  create table sext(id int,sex varchar(20)) ;
  insert into sext(id,sex) values (1,'男'),(4,'女'),(5,'男') ;
```




```sql
-- 员工id 没有在 sext表出现过?  
-- in
select * from emp1 where id not in (select id from sext) ;
-- exists
select * from emp1 e where not exists (select null from sext where id=e.id) ;
```



## 面试题二（上海）

```sql
表一(AAA)
	商品名称 mc 商品总量 sl
	 A         100
	 B         120
表二(BBB)
	商品名称 mc 出库数量 sl
	 A 			10
	 A 			20
	 B 			10
	 B 			20
	 B 			30
用一条 SQL 语句算出商品 A,B 目前还剩多少？
    create table AAA(mc varchar(20),sl int) ;
    insert into AAA (mc,sl) values ('A',100),('B',120) ;
    create table BBB(mc varchar(20),sl int) ;
    insert into BBB (mc,sl) values ('A',10),('A',20),('B',10),('B',20),('B',30) ;
```

![image-20230705112843564](images\image-20230705112843564.png)



```sql
 -- 期望结果：  
 --  mc  sl
 --  A  70 
 --  B  60 
    
 -- 方法1: 子查询   
 select mc, sl-(select sum(sl) from BBB where mc=a.mc) sy from AAA a ;
 
 -- 方法2: 多表
 -- 先分组
 select mc,sum(sl) from BBB group by mc ;
 -- 看做一张表
 select a.mc, a.sl-t.sum_sl from AAA a join (select mc,sum(sl) sum_sl from BBB group by mc) t on a.mc = t.mc ;
```





## 面试题三（上海）

```sql
人员情况表（employee）中字段包括，员工号（ID），姓名（name），年龄（age），文化程度（wh）：
包括四种情况（本科以上，大专，高中，初中以下）,
现在我要根据年龄字段查询统计出：
表中文化程度为本科以上，大专，高中，初中以下，各有多少人，占总人数多少。
结果如下A：
学历     年龄  人数  百分比
本科以上  20    34     14
大专      20    33    13
高中      20    33    13
初中以下  20    100    40
本科以上  21    50     20
。。。。。。
SQL 查询语句如何写？

create table employee(id int primary key auto_increment,
                      name varchar(20),
                      age int(2),
                      wh varchar(20)
                     ) ;

insert into employee(id,name,age,wh) values (null,'a',20,'本科以上') ;
insert into employee(id,name,age,wh) values (null,'b',20,'本科以上') ;
insert into employee(id,name,age,wh) values (null,'c',21,'本科以上') ;
insert into employee(id,name,age,wh) values (null,'d',20,'本科以上') ;
insert into employee(id,name,age,wh) values (null,'e',20,'大专') ;
insert into employee(id,name,age,wh) values (null,'e',21,'大专') ;
insert into employee(id,name,age,wh) values (null,'e',21,'高中') ;
insert into employee(id,name,age,wh) values (null,'e',20,'高中') ;
insert into employee(id,name,age,wh) values (null,'e',20,'初中以下') ;
```



```sql
查询结果集B：-- [行列转换]
学历            20岁    21岁
本科以上          3	    1
大专             2      2
高中             4      1
SQL 查询语句如何写？
```



## 面试题四（福州）

```sql
四张表
-- 学生表 
create table student(sid varchar(20),sname varchar(20));
insert into student values(1,'小明');
insert into student values(2,'小花');

-- 教师表 
create table teacher(tid varchar(20),tname varchar(20)) ;
insert into teacher values(1,'陈红');
insert into teacher values(2,'陈白');

-- 课程表 
create table course(cid varchar(20),cname varchar(20)，ctype varchar(20)) ;
insert into course values(1,'语文','文科');
insert into course values(2,'数学','理科');

-- 选课表 
create table choose_course(ccid varchar(20),sid varchar(20),tid varchar(20),
                           cid varchar(20));
-- 小明选了陈红老师的语文
insert into choose_course values(1,1,1,1);
-- 小明选了陈红老师的数学
insert into choose_course values(2,1,1,2);
-- 小花选了陈红老师的数学
insert into choose_course values(3,2,1,2);
-- 小明选了陈白老师的语文
insert into choose_course values(1,1,2,1);
-- 小花选了陈红老师的语文
insert into choose_course values(4,2,1,1);

-- 1. 查找陈红老师教的学生是那些？

-- 2.找学生小明所有的文科老师？

-- 3.找出没有选修陈红老师的学生？

-- 4.教的学生最少的老师？

```

## 面试题五（厦门）

```sql
-- 8:00--12:00 为迟到, 12:00--18:00 为早退
-- 打卡表 card
 create table card(
   cid int(10),
   ctime timestamp ,
   cuser int(10)
 );
 
--  人员表 person
create table person(
	pid int(10),
	name varchar(10)
) ;

-- 插入人员表的数据
insert into person values(1,'a');
insert into person values(2,'b');

-- 插入打卡的数据
insert into card values(1,'2009-07-19 08:02:00',1);
insert into card values(2,'2009-07-19 18:02:00',1);
insert into card values(3,'2009-07-19 09:02:00',2);
insert into card values(4,'2009-07-19 17:02:00',2);
insert into card values(5,'2009-07-20 08:02:00',1);
insert into card values(6,'2009-07-20 16:02:00',1);
insert into card values(7,'2009-07-20 07:02:00',2);
insert into card values(8,'2009-07-20 20:02:00',2);

--  查询 迟到 早退的员工姓名？
查询结果如下:
工号    姓名  打卡日期      上班打卡    下班打卡      迟到    早退
1       a    2009-07-19   08:02:00    18:02:00   是      否
1       a    2009-07-20   08:02:00    16:02:00   是      是
2       b    2009-07-19   09:02:00    17:02:00   是      是
```

# 2. DML

事务不自动提交

## 2.1 插入数据

所有字段插入

```sql
INSERT INTO dept(deptno,dname,loc) VALUES (11,'dev','NJ') ;
-- 所有列插入 可以省略列名  【不建议】 考虑兼容问题？
INSERT INTO dept VALUES (12,'dev','NJ') ;
```

插入多条

```sql
INSERT INTO dept(deptno,dname,loc) VALUES (13,'dev','NJ'),(14,'test','BJ') ;
```

查询结果集插入

```sql
-- CTAS 语法创建表 【同时拷贝数据  没有约束】
CREATE TABLE myemp AS SELECT * FROM emp ;
-- 查询结果集 插入 myemp表中
INSERT INTO myemp SELECT * FROM emp ;
```

## 2.2 更新数据

```sql
UPDATE myemp SET sal=sal*1.5 ,comm=500 WHERE empno =7788 ;
```

## 2.3 删除数据

```sql
DELETE FROM myemp WHERE empno = 7788 ;
```

## DML练习：

```sql
-- 1.往 emp 表中插入 empno,ename,sal 数据（111,'1',1000)(222,'2',2000) 
insert into emp(empno,ename,sal) values (111,'1',1000),(222,'2',2000) ;
-- 2.把 empno=111 的员工 comm 改成 100 
update emp set comm = 100 where empno = 111 ;
-- 3.往 dept 表中插入编号50 ，dname，loc 与10部门相同 的数据 
insert into dept(deptno,dname,loc) 
select 55,dname,loc from dept where deptno=10 ;
-- 4.删除 empno=111 的数据 
delete from emp where empno=111 ;
```

# 3. 视图

view  = 虚表      table = 基表 【1.数据安全性   2.简化查询】

- 简单：使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，对用户来说已经是过滤好的复合条件的结果集。
- 安全：使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但是通过视图就可以简单的实现。
- 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响；源表修改列名，则可以通过修改视图来解决，不会造成对访问者的影响。



## 3.1 简单视图

增删改查  ==>  基表 增删改查

```sql
-- 创建或替换视图
CREATE OR REPLACE VIEW v_emp10
AS
SELECT empno,ename,sal FROM emp WHERE deptno = 10 ;

-- 查询 基表
SELECT * FROM v_emp10 ;
-- 插入视图数据   基表数据插入
INSERT INTO v_emp10 (empno,ename,sal) VALUES (333,'xxx',800) ;
```



## 3.2 高级视图

只能查询   无法新增  删除 修改 【统计函数 ，多表数据】

```sql
CREATE OR REPLACE VIEW v_emp10
AS
SELECT deptno,COUNT(0) cut FROM emp GROUP BY deptno ;

SELECT * FROM v_emp10 ;
-- 无法插入数据  基表没有该列【cut 统计出来的】
INSERT INTO v_emp10 (deptno,cut) VALUES (80,6) ;
```

## 视图练习

```sql
-- 1.创建一个包含所有雇员的雇员编号、雇员名称、部门名称和薪金的视图 

-- 2.创建一个包含各种工作的薪金总和的视图 

```

# 4.索引

![image-20210806085831519](images\index.png)

## 4.1 优缺点/分类

```
优势
1） 类似于书籍的目录索引，提高数据检索的效率，降低数据库的IO成本。
2） 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗。

劣势
1） 实际上索引也是一张表，该表中保存了主键与索引字段，并指向实体类的记录，所以索引列也是要占用空间的。
2） 虽然索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE。因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。

索引分类
1） 单值索引 ：即一个索引只包含单个列，一个表可以有多个单列索引
2） 唯一索引 ：索引列的值必须唯一，但允许有空值
3） 复合索引 ：即一个索引包含多个列
```

## 4.2 创建索引

```sql
-- 创建索引
CREATE INDEX idx_dept_dname ON dept(dname) ;
-- 查看SQL执行计划
EXPLAIN SELECT * FROM dept WHERE dname='SALES';
```

## 4.3 删除索引

```sql
-- 删除索引
DROP INDEX idx_dept_dname ON dept ;
```

## 4.4 查看索引

```sql
SHOW INDEX FROM dept ;
```

## 4.5 索引设计原则

索引的设计可以遵循一些已有的原则，创建索引的时候请尽量考虑符合这些原则，便于提升索引的使用效率，更高效的使用索引。

- 对查询频次较高，且数据量比较大的表建立索引。
- 索引字段的选择，最佳候选列应当从where子句的条件中提取，如果where子句中的组合比较多，那么应当挑选最常用、过滤效果最好的列的组合。
- 使用唯一索引，区分度越高，使用索引的效率越高。
- 索引可以有效的提升查询数据的效率，但索引数量不是多多益善，索引越多，维护索引的代价自然也就水涨船高。对于插入、更新、删除等DML操作比较频繁的表来说，索引过多，会引入相当高的维护代价，降低DML操作的效率，增加相应操作的时间消耗。另外索引过多的话，MySQL也会犯选择困难病，虽然最终仍然会找到一个可用的索引，但无疑提高了选择的代价。
- 使用短索引，索引创建之后也是使用硬盘来存储的，因此提升索引访问的I/O效率，也可以提升总体的访问效率。假如构成索引的字段总长度比较短，那么在给定大小的存储块内可以存储更多的索引值，相应的可以有效的提升MySQL访问索引的I/O效率。
- 利用最左前缀，N个列组合而成的组合索引，那么相当于是创建了N个索引，如果查询时where子句中使用了组成该索引的前几个字段，那么这条查询SQL可以利用组合索引来提升查询效率。

```sql
创建复合索引:

	CREATE INDEX idx_name_email_status ON tb_xx(NAME,email,STATUS);

就相当于
	对name 创建索引 ;
	对name , email 创建了索引 ;
	对name , email, status 创建了索引 ;
```



## 索引练习:

```sql
-- 1. 在 emp 表的 ename 上创建一个索引  并查看执行计划？
```



# 5. 存储过程

 事先经过编译并存储在数据库中的一段 SQL 语句的集合，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。
## 5.1 创建存储过程

```sql
CREATE PROCEDURE procedure_name ([proc_parameter[,...]])
begin
	-- SQL语句
end ;
```

示例 ：

```sql 
delimiter $

create procedure pro_test1()
begin
	select 'Hello Mysql' ;
end$

delimiter ;
```

DELIMITER

该关键字用来声明SQL语句的分隔符 , 告诉 MySQL 解释器，该段命令是否已经结束了，mysql是否可以执行了。默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束，那么回车后，mysql将会执行该命令。

## 5.2 调用存储过程

```sql
call procedure_name() ;	
```

## 5.3 查看存储过程

```sql
-- 查询db_name数据库中的所有的存储过程
select name from mysql.proc where db='db_name';


-- 查询存储过程的状态信息
show procedure status;


-- 查询某个存储过程的定义
show create procedure test.pro_test1 \G;
```

##  5.4 删除存储过程

```sql
DROP PROCEDURE  [IF EXISTS] sp_name ；
```

## 5.5 语法

存储过程是可以编程的，意味着可以使用变量，表达式，控制结构 ， 来完成比较复杂的功能。

#### 变量

-  DECLARE

  通过 DECLARE 可以定义一个局部变量，该变量的作用范围只能在 BEGIN…END 块中。

```sql
DECLARE var_name[,...] type [DEFAULT value]
```

示例 : 

```sql
 delimiter $

 create procedure pro_test2() 
 begin 
 	declare num int default 5;
 	select num+ 10; 
 end$

 delimiter ; 
```

- SET

直接赋值使用 SET，可以赋常量或者赋表达式，具体语法如下：

```
  SET var_name = expr [, var_name = expr] ...
```

示例 : 

```sql
  DELIMITER $
  
  CREATE  PROCEDURE pro_test3()
  BEGIN
  	DECLARE NAME VARCHAR(20);
  	SET NAME = 'MYSQL';
  	SELECT NAME ;
  END$
  
  DELIMITER ;
```

也可以通过select ... into 方式进行赋值操作 :

```SQL
DELIMITER $

CREATE  PROCEDURE pro_test5()
BEGIN
	declare  countnum int;
	select count(*) into countnum from city;
	select countnum;
END$

DELIMITER ;
```

#### if条件判断

语法结构 : 

```sql
if search_condition then statement_list

	[elseif search_condition then statement_list] ...
	
	[else statement_list]
	
end if;
```

需求： 

```
根据定义的身高变量，判定当前身高的所属的身材类型 

	180 及以上 ----------> 身材高挑

	170 - 180  ---------> 标准身材

	170 以下  ----------> 一般身材
```

示例 : 

```sql
delimiter $

create procedure pro_test6()
begin
  declare  height  int  default  175; 
  declare  description  varchar(50);
  
  if  height >= 180  then
    set description = '身材高挑';
  elseif height >= 170 and height < 180  then
    set description = '标准身材';
  else
    set description = '一般身材';
  end if;
  
  select description ;
end$

delimiter ;
```

调用结果为 : 


#### 传递参数

语法格式 : 

```
create procedure procedure_name([in/out/inout] 参数名   参数类型)
...


IN :   该参数可以作为输入，也就是需要调用方传入值 , 默认
OUT:   该参数作为输出，也就是该参数可以作为返回值
INOUT: 既可以作为输入参数，也可以作为输出参数
```

**IN - 输入**

需求 :

```
根据定义的身高变量，判定当前身高的所属的身材类型 
```

示例  : 

```sql
delimiter $

create procedure pro_test5(in height int)
begin
    declare description varchar(50) default '';
  if height >= 180 then
    set description='身材高挑';
  elseif height >= 170 and height < 180 then
    set description='标准身材';
  else
    set description='一般身材';
  end if;
  select concat('身高 ', height , '对应的身材类型为:',description);
end$

delimiter ;
```

**OUT-输出**

 需求 :

```
根据传入的身高变量，获取当前身高的所属的身材类型  
```

示例:

```SQL 
create procedure pro_test5(in height int , out description varchar(100))
begin
  if height >= 180 then
    set description='身材高挑';
  elseif height >= 170 and height < 180 then
    set description='标准身材';
  else
    set description='一般身材';
  end if;
end$	
```

调用:

```
call pro_test5(168, @description)$

select @description$
```

@description :  这种变量要在变量名称前面加上“@”符号，叫做用户会话变量，代表整个会话过程他都是有作用的，这个类似于全局变量一样。

#### case结构

语法结构 : 

```SQL
方式一 : 

CASE case_value

  WHEN when_value THEN statement_list
  
  [WHEN when_value THEN statement_list] ...
  
  [ELSE statement_list]
  
END CASE;


方式二 : 

CASE

  WHEN search_condition THEN statement_list
  
  [WHEN search_condition THEN statement_list] ...
  
  [ELSE statement_list]
  
END CASE;

```

需求:

```
给定一个月份, 然后计算出所在的季度
```

示例  :

```sql
delimiter $


create procedure pro_test9(month int)
begin
  declare result varchar(20);
  case 
    when month >= 1 and month <=3 then 
      set result = '第一季度';
    when month >= 4 and month <=6 then 
      set result = '第二季度';
    when month >= 7 and month <=9 then 
      set result = '第三季度';
    when month >= 10 and month <=12 then 
      set result = '第四季度';
  end case;
  
  select concat('您输入的月份为 :', month , ' , 该月份为 : ' , result) as content ;
  
end$


delimiter ;
```

#### while循环

语法结构: 

```sql
while search_condition do

	statement_list
	
end while;
```

需求:

```
计算从1加到n的值
```

示例  : 

```sql
delimiter $

create procedure pro_test8(n int)
begin
  declare total int default 0;
  declare num int default 1;
  while num<=n do
    set total = total + num;
	set num = num + 1;
  end while;
  select total;
end$

delimiter ;
```

#### repeat结构

有条件的循环控制语句, 当满足条件的时候退出循环 。while 是满足条件才执行，repeat 是满足条件就退出循环。

语法结构 : 

```SQL
REPEAT

  statement_list

  UNTIL search_condition

END REPEAT;
```

需求: 

```
计算从1加到n的值
```

示例  : 

```sql
delimiter $

create procedure pro_test10(n int)
begin
  declare total int default 0;
  
  repeat 
    set total = total + n;
    set n = n - 1;
    until n=0  
  end repeat;
  
  select total ;
  
end$


delimiter ;
```

#### loop语句

LOOP 实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 LEAVE 语句实现，具体语法如下：

```sql
[begin_label:] LOOP

  statement_list

END LOOP [end_label]
```

如果不在 statement_list 中增加退出循环的语句，那么 LOOP 语句可以用来实现简单的死循环。

#### eave语句

用来从标注的流程构造中退出，通常和 BEGIN ... END 或者循环一起使用。下面是一个使用 LOOP 和 LEAVE 的简单例子 , 退出循环：

```SQL
delimiter $

CREATE PROCEDURE pro_test11(n int)
BEGIN
  declare total int default 0;
  
  ins: LOOP
    
    IF n <= 0 then
      leave ins;
    END IF;
    
    set total = total + n;
    set n = n - 1;
  	
  END LOOP ins;
  
  select total;
END$

delimiter ;
```

#### 游标/光标

游标是用来存储查询结果集的数据类型 , 在存储过程和函数中可以使用光标对结果集进行循环的处理。光标的使用包括光标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

声明光标：

```sql
DECLARE cursor_name CURSOR FOR select_statement ;
```

OPEN 光标：

```sql
OPEN cursor_name ;
```

FETCH 光标：

```sql
FETCH cursor_name INTO var_name [, var_name] ...
```

CLOSE 光标：

```sql
CLOSE cursor_name ;
```

示例 : 

初始化脚本:

``` sql
create table emp(
  id int(11) not null auto_increment ,
  name varchar(50) not null comment '姓名',
  age int(11) comment '年龄',
  salary int(11) comment '薪水',
  primary key(`id`)
)engine=innodb default charset=utf8 ;

insert into emp(id,name,age,salary) values(null,'金毛狮王',55,3800),(null,'白眉鹰王',60,4000),(null,'青翼蝠王',38,2800),(null,'紫衫龙王',42,1800);

```

``` SQL
-- 查询emp表中数据, 并逐行获取进行展示
create procedure pro_test11()
begin
  declare e_id int(11);
  declare e_name varchar(50);
  declare e_age int(11);
  declare e_salary int(11);
  declare emp_result cursor for select * from emp;
  
  open emp_result;
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
  select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
  select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
  select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
  select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  fetch emp_result into e_id,e_name,e_age,e_salary;
  select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为: ',e_salary);
  
  close emp_result;
end$
```

通过循环结构 , 获取游标中的数据 : 

```sql
DELIMITER $

create procedure pro_test12()
begin
  DECLARE id int(11);
  DECLARE name varchar(50);
  DECLARE age int(11);
  DECLARE salary int(11);
  DECLARE has_data int default 1;
  
  DECLARE emp_result CURSOR FOR select * from emp;
  DECLARE EXIT HANDLER FOR NOT FOUND set has_data = 0;
  
  open emp_result;
  
  repeat
    fetch emp_result into id , name , age , salary;
    select concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ', salary);
    until has_data = 0
  end repeat;
  
  close emp_result;
end$

DELIMITER ; 
```

# 6.函数

语法结构:

``` 
CREATE FUNCTION function_name([param type ... ]) 
RETURNS type 
BEGIN
	...
END;
```

案例 : 

定义一个存储过程, 请求满足条件的总记录数 ;

```SQL

delimiter $

create function count_city(countryId int)
returns int
begin
  declare cnum int ;
  
  select count(*) into cnum from city where country_id = countryId;
  
  return cnum;
end$

delimiter ;
```

调用: 

```
select count_city(1);
select count_city(2);
```

# 7.触发器

### 介绍

触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作 。

使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

| 触发器类型       | NEW 和 OLD的使用                      |
| ----------- | --------------------------------- |
| INSERT 型触发器 | NEW 表示将要或者已经新增的数据                 |
| UPDATE 型触发器 | OLD 表示修改之前的数据 , NEW 表示将要或已经修改后的数据 |
| DELETE 型触发器 | OLD 表示将要或者已经删除的数据                 |

### 创建触发器

语法结构 : 

```sql
create trigger trigger_name 

before/after insert/update/delete

on tbl_name 

[ for each row ]  -- 行级触发器

begin

	trigger_stmt ;

end;
```

示例 

需求

```
通过触发器记录 emp 表的数据变更日志 , 包含增加, 修改 , 删除 ;
```

首先创建一张日志表 : 

```sql
create table emp_logs(
  id int(11) not null auto_increment,
  operation varchar(20) not null comment '操作类型, insert/update/delete',
  operate_time datetime not null comment '操作时间',
  operate_id int(11) not null comment '操作表的ID',
  operate_params varchar(500) comment '操作参数',
  primary key(`id`)
)engine=innodb default charset=utf8;
```

创建 insert 型触发器，完成插入数据时的日志记录 : 

```sql
DELIMITER $

create trigger emp_logs_insert_trigger
after insert 
on emp 
for each row 
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params) values(null,'insert',now(),new.id,concat('插入后(id:',new.id,', name:',new.name,', age:',new.age,', salary:',new.salary,')'));	
end $

DELIMITER ;
```

创建 update 型触发器，完成更新数据时的日志记录 : 

``` sql
DELIMITER $

create trigger emp_logs_update_trigger
after update 
on emp 
for each row 
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params) values(null,'update',now(),new.id,concat('修改前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,', age:',new.age,', salary:',new.salary,')'));                                                                      
end $

DELIMITER ;
```

创建delete 行的触发器 , 完成删除数据时的日志记录 : 

```sql
DELIMITER $

create trigger emp_logs_delete_trigger
after delete 
on emp 
for each row 
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params) values(null,'delete',now(),old.id,concat('删除前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,')'));                                                                      
end $

DELIMITER ;
```

测试：

```sql
insert into emp(id,name,age,salary) values(null, '光明左使',30,3500);
insert into emp(id,name,age,salary) values(null, '光明右使',33,3200);

update emp set age = 39 where id = 3;

delete from emp where id = 5;
```

### 删除触发器

语法结构 : 

```
drop trigger [schema_name.]trigger_name
```

如果没有指定 schema_name，默认为当前数据库 。

### 查看触发器

可以通过执行 SHOW TRIGGERS 命令查看触发器的状态、语法等信息。

语法结构 ： 

```
show triggers ；
```

