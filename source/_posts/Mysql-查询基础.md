---
title: Mysql-查询基础
date: 2023-07-03 18:27:29
tags: Mysql
category:  数据库
---

# 1. 查询基础

## 1.0 基本概念

```
DDL:[数据定义语言]
  create , alter , drop , truncate  语句 【事务自动提交】
DML:[数据操作语言]
  insert ,update, delete 语句
DQL:[数据查询语言]
  select
DCL:[数据控制语言]
  grant[授权] ，revoke[撤销] ,commit , rollback , savepoint
```

### 表结构

![image-20220214134412121](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220214134412121.png)

![image-20220327091956399](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220327091956399.png)

### 初始化脚本

scott.sql

```sql
drop database if exists scott;
create database scott;
use scott;

DROP TABLE IF EXISTS BONUS;
CREATE TABLE BONUS (
ENAME VARCHAR(10) NULL ,
JOB VARCHAR(9) NULL ,
SAL DOUBLE NULL ,
COMM DOUBLE NULL 
) ;

-- ----------------------------
-- Records for BONUS
-- ----------------------------


-- ----------------------------
-- Table structure for DEPT
-- ----------------------------
DROP TABLE IF EXISTS DEPT;
CREATE TABLE DEPT (
DEPTNO INT(2) PRIMARY KEY,
DNAME VARCHAR(14) NULL ,
LOC VARCHAR(13) NULL 
);

-- ----------------------------
-- Records for DEPT
-- ----------------------------
INSERT INTO DEPT VALUES ('10', 'ACCOUNTING', 'NEW YORK');
INSERT INTO DEPT VALUES ('20', 'RESEARCH', 'DALLAS');
INSERT INTO DEPT VALUES ('30', 'SALES', 'CHICAGO');
INSERT INTO DEPT VALUES ('40', 'OPERATIONS', 'BOSTON');

-- ----------------------------
-- Table structure for EMP
-- ----------------------------
DROP TABLE IF EXISTS EMP;
CREATE TABLE EMP (
EMPNO INT(4) PRIMARY KEY ,
ENAME VARCHAR(10) NULL ,
JOB VARCHAR(9) NULL ,
MGR INT(4) NULL ,
HIREDATE DATETIME NULL ,
SAL DOUBLE(7,2) NULL ,
COMM DOUBLE(7,2) NULL ,
DEPTNO INT(2) NULL 
);

-- ----------------------------
-- Records for EMP
-- ----------------------------
INSERT INTO EMP VALUES ('7369', 'SMITH', 'CLERK', '7902', '1980-12-17 00:00:00', '800', null, '20');
INSERT INTO EMP VALUES ('7499', 'ALLEN', 'SALESMAN', '7698', '1981-02-20 00:00:00', '1600', '300', '30');
INSERT INTO EMP VALUES ('7521', 'WARD', 'SALESMAN', '7698', '1981-02-22 00:00:00', '1250', '500', '30');
INSERT INTO EMP VALUES ('7566', 'JONES', 'MANAGER', '7839', '1981-04-02 00:00:00', '2975', null, '20');
INSERT INTO EMP VALUES ('7654', 'MARTIN', 'SALESMAN', '7698', '1981-09-28 00:00:00', '1250', '1400', '30');
INSERT INTO EMP VALUES ('7698', 'BLAKE', 'MANAGER', '7839', '1981-05-01 00:00:00', '2850', null, '30');
INSERT INTO EMP VALUES ('7782', 'CLARK', 'MANAGER', '7839', '1981-06-09 00:00:00', '2450', null, '10');
INSERT INTO EMP VALUES ('7788', 'SCOTT', 'ANALYST', '7566', '1987-04-19 00:00:00', '3000', null, '20');
INSERT INTO EMP VALUES ('7839', 'KING', 'PRESIDENT', null, '1981-11-17 00:00:00', '5000', null, '10');
INSERT INTO EMP VALUES ('7844', 'TURNER', 'SALESMAN', '7698', '1981-09-08 00:00:00', '1500', '0', '30');
INSERT INTO EMP VALUES ('7876', 'ADAMS', 'CLERK', '7788', '1987-05-23 00:00:00', '1100', null, '20');
INSERT INTO EMP VALUES ('7900', 'JAMES', 'CLERK', '7698', '1981-12-03 00:00:00', '950', null, '30');
INSERT INTO EMP VALUES ('7902', 'FORD', 'ANALYST', '7566', '1981-12-03 00:00:00', '3000', null, '20');
INSERT INTO EMP VALUES ('7934', 'MILLER', 'CLERK', '7782', '1982-01-23 00:00:00', '1300', null, '10');

-- ----------------------------
-- Table structure for "SALGRADE"
-- ----------------------------
DROP TABLE IF EXISTS SALGRADE;
CREATE TABLE SALGRADE (
GRADE INT(2) NULL ,
LOSAL DOUBLE NULL ,
HISAL DOUBLE NULL 
) ;

-- ----------------------------
-- Records for SALGRADE
-- ----------------------------
INSERT INTO SALGRADE VALUES ('1', '700', '1200');
INSERT INTO SALGRADE VALUES ('2', '1201', '1400');
INSERT INTO SALGRADE VALUES ('3', '1401', '2000');
INSERT INTO SALGRADE VALUES ('4', '2001', '3000');
INSERT INTO SALGRADE VALUES ('5', '3001', '9999');



-- ----------------------------
-- Foreign Key structure for table EMP
-- ----------------------------
ALTER TABLE EMP ADD FOREIGN KEY (DEPTNO) REFERENCES DEPT (DEPTNO);

```



## 1.1 简单select

### 所有行所有列

```sql
SELECT * FROM emp ;
```

### 限制列

```sql
SELECT empno,ename,sal FROM emp ;
```

### 限制行

```sql
SELECT * FROM emp WHERE deptno = 10 ;
```

## 1.2 算术运算符 + - * /

注意：  null 值不参与计算  【null 未知值  不确定值  x？】

```sql
SELECT sal+comm FROM emp ;  --  comm  null 值
```

### + 【运算】

```sql
SELECT 3+4 ;
SELECT '3'+4 ;
SELECT '3'+'4' ;
SELECT '3'+'ABC' ;  -- 3 [不报错]
```

### /  【除法】

```sql
SELECT 5/2 ;   -- 2.5000
```

## 1.3  别名 AS

标准：  AS  "dept.dname"

```sql
SELECT ename AS "from" ,sal AS "员工 工资" FROM emp AS e
```

## 1.4 null 空值

安全 等于/全等于 ：<=>   与  =  区别：可以判断null值 

### is null

```sql
SELECT * FROM emp WHERE comm IS NULL ;

SELECT * FROM emp WHERE comm <=> NULL ;
```

### is not null

```sql
SELECT * FROM emp WHERE comm IS NOT NULL ;

SELECT * FROM emp WHERE NOT comm IS NULL ;
```

## 1.5 去重复行 distinct

```sql
SELECT DISTINCT deptno ,job FROM emp ;
```

## 1.6 排序 order by

堆表：快速插入数据  【搬家公司 --> 家具】

默认： 升序  ASC   降序  DESC

### 基本排序

```sql
SELECT  * FROM emp ORDER BY sal ASC;
SELECT  * FROM emp ORDER BY sal DESC;
```

### 结果集列

```sql
SELECT  empno,ename,sal FROM emp ORDER BY 3 DESC;
```

### 排序多列

```sql
SELECT  empno,ename,sal FROM emp ORDER BY 3 DESC ,1 ASC ;
```

### null

Oracle 数据库有专用关键字，mysql没有

```sql
-- null first
SELECT  empno,ename,sal,comm FROM emp ORDER BY IF(ISNULL(comm),99999,comm) DESC ;

-- null last
SELECT  empno,ename,sal,comm FROM emp ORDER BY IF(ISNULL(comm),-1,comm) DESC ;
```

## 1.7 比较运算符

   <=> 安全等于【比较null值】  = 等于【不能比较null值】

  !=    <>

<   <=    >   >=     between   and 

```sql
SELECT * FROM emp WHERE deptno != 10 ;
SELECT * FROM emp WHERE deptno <> 10 ;
SELECT * FROM emp WHERE sal BETWEEN 800 AND 3000 ;
SELECT * FROM emp WHERE sal>=800 AND  sal<=3000 ;
```

## 1.8 in  not in

### in

```sql
SELECT * FROM emp WHERE deptno IN (10,20) ;
SELECT * FROM emp WHERE deptno =10 OR deptno =20 ;
```

### not in

```sql
--  注意： not in (去null值)
SELECT * FROM emp WHERE deptno NOT IN (10,20,NULL) ;
```

## 1.9 模糊查询

_     一个字符

%   N个字符

```sql
SELECT * FROM emp WHERE ename LIKE '__A%';
```

```sql
--  zhang_san   _进行转义  _不是 like 匹配字符  而是数据
SELECT * FROM emp WHERE ename LIKE '%\_%';

SELECT * FROM emp WHERE ename LIKE '%\%%';
```

## 1.10 正则 REGEXP

```
^  匹配开头
$  匹配结尾
.  任何一个字符
[abc] 范围匹配一个  [a-z]   [0-9]
*  匹配任意次
```

```sql
-- s开头的姓名
SELECT * FROM emp WHERE ename REGEXP '^S' ;
-- T结尾的姓名
SELECT * FROM emp WHERE ename REGEXP 'T$' ;
-- 第二个字母 C
SELECT * FROM emp WHERE ename REGEXP '.C' ;
-- 包含字母 O
SELECT * FROM emp WHERE ename REGEXP '.*O.*' ;
```

## 1.11 逻辑运算符

not  !     and  &&   or  ||

```sql
SELECT NOT 1=1 , ! (1=1) ;
```

## 1.12 limit

TOPN

```sql
-- limit 位置偏移量, 行数 
-- 第一页
SELECT * FROM emp LIMIT 0,5 ;
SELECT * FROM emp LIMIT 5 ;

-- 第二页
SELECT * FROM emp LIMIT 5,5 ;
```

## 例1：

```sql
-- 1.选择在部门 30 中员工的所有信息 
select * from emp where deptno = '30' ;
-- 2 列出职位为（MANAGER）的员工的编号，姓名 
select empno,ename from emp where job = 'MANAGER' ;
-- 3 找出奖金高于工资的员工 
select * from emp where comm>sal ;
-- 4 找出每个员工奖金和工资的总和 
select sal+if(isnull(comm),0,comm) month_sal ,ename from emp order by month_sal desc ;
-- 5 找出部门 10 中的经理(MANAGER)和部门 20 中的普通员工(CLERK)
select * from emp where (deptno,job) in ((10,'MANAGER'),(20,'CLERK')) ;
-- 6 找出部门 10 中既不是经理MANAGER也不是普通员工CLERK，而且工资大于等于 2000 的员工
select * from emp where deptno = 10 and job not in ('MANAGER','CLERK') and sal>=2000 ;
-- 7 找出有奖金的员工的不同工作
select distinct  job from emp where comm is not null ;
-- 8 找出没有奖金或者奖金低于 500 的员工
select * from emp where comm is null or comm<500 ;
-- 9 显示雇员姓名，根据其服务年限，将最老的雇员排在最前面
select ename from emp order by hiredate asc ;
```

# 2. 单行函数

## 2.1 数值函数

绝对值

```sql
SELECT ABS(-11.5) ;
```

平方根

```sql
SELECT SQRT(100) ;
```

求余

```sql
SELECT MOD(5,2) ;
```

向上取整

```sql
SELECT CEIL(3.001) ,CEIL(3.000) ;
```

向下取整

```sql
SELECT FLOOR(3.999) ,CEIL(3.000) ;
```

随机数

```sql
SELECT RAND() ;
```

四舍五入

```sql
SELECT ROUND(11.5),ROUND(-11.5) ;
```

## 2.2 字符函数

字符个数

```sql
SELECT * FROM emp WHERE CHAR_LENGTH(ename) = 5 ;

SELECT * FROM emp WHERE ename LIKE '_____' ;
```

字符串连接

```sql
-- concat()  null 值 返回 null
SELECT CONCAT('hello','word','java167'),CONCAT('hello','word','java167',NULL) ;

-- concat_ws()  null值不参与  【第一个参数 连接符 】
SELECT CONCAT_WS(',','hello','word','java167'),CONCAT_WS('*','hello',NULL,'word','java167') 
```

字符串替换

```sql
-- db 索引从 1 开始
SELECT INSERT('hellojava167',3,2,'**') ;
```

大小写转换

```sql
SELECT UPPER('hello java'),LOWER('JAVA hello') ;
```

左右截取字符串

```sql
SELECT LEFT('helloworld',5),RIGHT('helloworld',5) ;
```

左右填充字符串

```sql
SELECT LPAD('helloworld',15,'-'),RPAD('helloworld',15,'*') ;
```

首尾去空格

```sql
SELECT TRIM(' abc ddd ') ;
SELECT TRIM('abc' FROM 'abcxxxyyyabc') ;
```

重复生成字符串

```sql
SELECT REPEAT('hello',5) ;
```

字符串比较

```sql
-- 1  0  -1
SELECT STRCMP('abc','def') ,STRCMP('abc','abc'),STRCMP('zzz','abc');
```

字符串截取

```sql
SELECT SUBSTRING('helloworld',1,5) ,SUBSTRING('helloworld',5),
SUBSTRING('helloworld',-5);

SELECT MID('helloworld',1,5) ,MID('helloworld',5),MID('helloworld',-5);
```

查找索引

```sql
SELECT LOCATE('o','helloworld') ,POSITION('o' IN 'helloworld'),INSTR('helloworld','o') ;
```



## 2.3 日期函数

当前日期

```sql
-- 年月日
SELECT CURRENT_DATE , CURRENT_DATE() , CURDATE();
-- 时分秒
SELECT CURRENT_TIME ,CURRENT_TIME(), CURTIME();
-- 年月日 时分秒
SELECT CURRENT_TIMESTAMP,CURRENT_TIMESTAMP(),LOCALTIME(),NOW(),SYSDATE() ;
```

日期之间相差天数

```sql
SELECT DATEDIFF(NOW(),hiredate) FROM emp ;
```

当前日期该月最后一天

```sql
SELECT LAST_DAY(NOW()) ;
```

日期加减

```sql
SELECT DATE_ADD(NOW(),INTERVAL 1 DAY) ;

SELECT DATE_SUB(NOW(),INTERVAL 1 DAY) ;
```

周 年 月 小时 分

```sql
SELECT WEEK(NOW()),YEAR(NOW()),MONTH(NOW()),HOUR(NOW()),MINUTE(NOW()) ;
SELECT DAYOFMONTH(NOW()),DAY(NOW()),DAYOFYEAR(NOW()) ;

-- 提取日期中 部分字段
SELECT EXTRACT(YEAR FROM NOW()),EXTRACT(MONTH FROM NOW()) ;
```

日期格式化

```sql
SELECT DATE_FORMAT(NOW(),'%Y/%m/%d %h:%i');
SELECT * FROM emp WHERE DATE_FORMAT(hiredate,'%m') = '02' ;
```



## 2.4 条件判断函数

if

```sql
SELECT IF(3<2,'aaa','bbb');
SELECT IF(STRCMP('xyz','abc'),'yes','no');
```

ifnull

```sql
SELECT sal+IFNULL(comm,0) FROM emp ;
```

case

```sql
-- 根据部门 10 dev 20 test 30 public  others 
SELECT 
  empno,
  ename,
  CASE
    deptno 
    WHEN 10 THEN 'dev' 
    WHEN 20 THEN 'test' 
    WHEN 30 THEN 'public' 
    ELSE 'others' 
   END dname
FROM
  emp ;
```

## 例2:

```sql
-- 1 找出每个月倒数第三天受雇的员工（如：2009-5-29） 
-- date_add  last_day
select * from emp where date_add(hiredate,INTERVAL 2 day) = last_day(hiredate);

-- 2 找出 25 年前雇的员工 
-- datediff
select * from emp where datediff(now(),hiredate)/365>25 ;

-- 3 所有员工名字前加上 Dear ,并且名字首字母大写 
-- concat concat_ws upper  substring  mid
select concat('Dear',upper(mid(ename,1,1)),mid(ename,2)) from emp ;

-- 4 找出姓名为 5 个字母的员工 
-- char_length   like _
select * from emp where char_length(ename) = 5 ;
-- 5 找出姓名中不带 R 这个字母的员工 
-- not like
select * from emp where ename not like '%R%' ;
-- 6 显示所有员工的姓名的第一个字 
-- substring
select substring(ename,1,1) from emp ;
-- 7 显示所有员工，按名字降序排列，若相同，则按工资升序排序 
select ename,sal from emp order by ename desc , sal asc ;
-- 8 假设一个月为 30 天，找出所有员工的日薪，不计小数 
-- floor  isnull if   ifnull
select floor((sal+ifnull(comm,0))/30) from emp ;
-- 9 找到 2 月份受雇的员工 
-- month  date_format
select * from emp where month(hiredate) = 2 ;
-- 10 列出员工加入公司的天数(四舍五入） 
select datediff(now(),hiredate) from emp ;

-- 11 分别用 case 列出员工所在的部门，
--   deptno=10 显示'部门 10', 
--   deptno=20 显示'部门 20'    
--   deptno=30 显示'部门 30'    
--  deptno=40 显示'部门 40'  
--   否则为'其他部门' 

select empno,ename ,
case 
 when deptno between 10 and 30 then '重要部门'
    when deptno>40 then '辅助部门'
    else '其他部门'
   end dname
   from emp ;
```

# 3. 分组函数

统计函数  组函数 聚合函数  count  sum  min  max avg

## 3.1 注意：

null值不参与统计

```sql
SELECT COUNT(comm) FROM emp ;  -- 4  表中数据 16行

SELECT COUNT(0) FROM emp ;
```

分组函数不能出现where子句

```sql
-- 查询工资大于平均工资的员工
SELECT * FROM emp WHERE sal > AVG(sal) ;
```

## 3.2 group by 

注意： 只要有group  by 子句：

​            select 子句要求：只能写 group by 出现的列名  + 5 个分组函数

```sql
SELECT 
  deptno,
  COUNT(0) dept_count,
  SUM(sal) dept_sum_sal,
  MIN(sal) dept_min_sal,
  MAX(sal) dept_max_sal,
  AVG(sal) dept_avg_sal 
FROM
  emp 
GROUP BY deptno ;
```

多个分组条件：

```sql
SELECT 
  deptno,job,
  COUNT(0) dept_count,
  SUM(sal) dept_sum_sal,
  MIN(sal) dept_min_sal,
  MAX(sal) dept_max_sal,
  AVG(sal) dept_avg_sal 
FROM
  emp 
GROUP BY deptno,job ;
```

## 3.3 having

```sql
-- 根据部门分组  查询部门人数大于 2人的部门编号 人数
-- 根据部门分组  查询部门人数大于 2人的部门编号 人数
SELECT
  deptno,
  COUNT(0) dept_count
FROM
  emp
GROUP BY deptno
HAVING  COUNT(0)>2 ;
```

## 3.4 完整的SQL

```sql
select
   列名1 ,列名2...列名N
from  
   表1,表2 .... 表N
where 
   限制行【分组前过滤】
group by
   分组列
having
   分组后过滤
order by
   排序
limit 
   偏移量,行数
```

```sql
-- 查询工资大于500 按照部门分组 如果部门相同按照工种分组  人数大于等于1  按照人数排降序  第2~5条
SELECT 
  deptno,
  job,
  COUNT(0) dept_count 
FROM
  emp 
WHERE sal > 500 
GROUP BY deptno,
  job 
HAVING COUNT(0) >= 1 
ORDER BY 3 DESC 
LIMIT 2, 3 ;
```

## 3.5 行列转换

```sql
CREATE TABLE stu(
  sname VARCHAR(20),
  sub  VARCHAR(20),
  score VARCHAR(20)
);

INSERT INTO stu (sname,sub,score) VALUES ('zs','chinese','100'),('zs','math','99'),('zs','english','98');
INSERT INTO stu (sname,sub,score) VALUES ('li','chinese','80'),('li','math','89'),('li','english','88');
INSERT INTO stu (sname,sub,score) VALUES ('ww','chinese','70'),('ww','math','79'),('ww','english','78');
COMMIT ;
```

![image-20220214171935462](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220214171935462.png)

```sql
SELECT
  sname,
  CASE sub WHEN 'chinese' THEN score END chinese ,
  CASE sub WHEN 'math' THEN score END math ,
  CASE sub WHEN 'english' THEN score END english 
FROM 
  stu 
```

![image-20220214172136135](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220214172136135.png)

```sql
SELECT
  sname,
  SUM(CASE sub WHEN 'chinese' THEN score END) chinese ,
  SUM(CASE sub WHEN 'math' THEN score END) math ,
  SUM(CASE sub WHEN 'english' THEN score END) english 
FROM 
  stu 
GROUP BY sname 
```

![image-20220214172516098](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220214172516098.png)

## 例3:

```sql
-- 1 分组统计各部门下工资>500 的员工的平均工资
-- group by deptno 分组统计各部门
-- where sal > 500  工资>500 的员工
-- select avg(sal)  平均工资
-- from emp 员工

select deptno,avg(sal) avg_sal from emp where sal>500 group by deptno ;

-- 2 统计各部门下平均工资大于 500 的部门 
-- group by deptno 统计各部门
-- select avg(sal)
-- having avg(sal)>500 工资大于 500 
select deptno,avg(sal) from emp group by deptno having avg(sal)>500 ;

-- 3 算出部门 30 中得到最多奖金的员工奖金 
-- where deptno = 30 部门 30
-- select max(comm) 最多奖金
select max(comm) from emp where deptno = 30 ;

-- 4 算出部门 30 中得到最多奖金的员工姓名 
-- where deptno = 30 部门 30 得到最多奖金
-- select ename 员工姓名
-- select ename,comm from emp where deptno=30 order by comm desc limit 0,1;

-- select ename,comm from emp where deptno=30 and comm=(select max(comm) from emp where deptno = 30 ) ;

select ename,comm from emp where (deptno,comm)=(select deptno,max(comm) from emp where deptno = 30 ) ;

-- 5 算出每个职位的员工数和最低工资 
-- group by job 每个职位
-- select count(0) , min(sal) 员工数和最低工资
select job,count(0) , min(sal) from emp group by job ;

-- 6 算出每个部门,每个职位的平均工资和平均奖金(平均值包括没有奖金)，
-- 如果平均奖金大于 300，显示“奖金不错”，
-- 如果平均奖金 100 到 300，显示“奖金一般”，
-- 如果平均奖金小于 100，显示“基本没有奖金”，
-- 按部门编号降序，平均工资降序排列
-- group by deptno,job  每个部门,每个职位
-- select avg(sal) ,avg(ifnull(comm,0)) 平均工资和平均奖金
-- case when then end ..  显示转换
-- order by deptno desc ,avg(sal) desc   按部门编号降序，平均工资降序排列

select deptno,job,avg(sal) ,
case 
	when avg(ifnull(comm,0))>300 then '奖金不错'
    when avg(ifnull(comm,0))>=100 and avg(ifnull(comm,0))<=300 then '奖金一般'
    when avg(ifnull(comm,0))<100 then '基本没有奖金'
 end  comm_msg  
 from emp group by deptno,job
 order by deptno desc ,avg(sal) desc;


-- 7 列出员工表中每个部门的员工数，和部门 no 
-- group by deptno 每个部门
-- select count(0) , deptno 员工数，和部门 no
select count(0) , deptno  from emp group by deptno ;

-- 8 得到工资大于自己部门平均工资的员工信息 
-- 注意： 别名
select * from emp e where sal > (select avg(sal) from emp where deptno=e.deptno) ;

-- 9 分组统计每个部门下，每种职位的平均奖金（也要算没奖金的人）和总工资(包括奖金) 
-- group by  deptno,job ; 每个部门下，每种职位
-- select  avg(ifnull(comm,0)), sum(sal+ifnull(comm,0))  平均奖金（也要算没奖金的人）和总工资(包括奖金) 
select deptno,job,avg(ifnull(comm,0)), sum(sal+ifnull(comm,0)) from emp group by deptno,job;

```

# 4. 多表查询

![image-20220327151340651](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220327151340651.png)



## 4.1 笛卡尔集【积】 Cross join

行相乘  列相加  【大结果集】  bug 【避免该查询】忘记写 where 条件 ，或条件无效

```sql
-- 忘记写 where 条件
SELECT * FROM emp,dept,salgrade ;
-- 条件无效 
SELECT * FROM emp,dept,salgrade WHERE emp.deptno=emp.deptno;
```

## 4.2 等值连接 Equi join/Natural join

两张表的数据 必须相关 【外键值  = 另一张表的主键值】

等值连接

```sql
-- 查询姓名 部门名称
SELECT e.empno,e.ename,d.deptno,d.dname FROM emp e , dept d WHERE e.deptno = d.deptno 
```

自然连接【两表中同名的列】

```sql
--  EMP  deptno [emp 的 FK]    Dept  deptno [dept 的 PK]
SELECT e.empno,e.ename,d.deptno,d.dname FROM emp e NATURAL JOIN dept d ;
```



## 4.3 非等值连接 Non-Equijoin

参考值 【emp表 sal  3000  salgrade表 3000 ？1201~4000】

```sql
-- 查询工号，姓名 ，工资，工资等级
SELECT 
  e.empno,
  e.ename,
  e.sal,
  s.grade 
FROM
  emp e,
  salgrade s 
WHERE e.sal BETWEEN s.losal 
  AND s.hisal ;
```



## 4.4 自连接 Self join

树型表   无限级分类表  【表的FK  指向自己表的主键】  必须使用别名

```sql
SELECT COUNT(0) FROM emp e,emp m ; -- 笛卡尔集 16*16
SELECT COUNT(0) FROM emp e,emp m WHERE e.empno = e.empno;  -- 笛卡尔集 16*16
SELECT COUNT(0) FROM emp e,emp m WHERE e.mgr = m.empno;  -- 自连接

-- 查询员工姓名 工资  直接领导姓名 工资 
SELECT e.ename emp_name,e.sal emp_sal,m.ename mgr_name,m.sal mgr_sal FROM emp e, emp m WHERE e.mgr = m.empno ;
```

## 4.5 左外连接 Left Outer Join

```sql
-- 查询所有员工姓名 部门名称 包括没有部门的员工
SELECT 
  e.ename,
  d.deptno,
  d.dname 
FROM
  emp e 
  LEFT OUTER JOIN dept d 
    ON e.deptno = d.deptno ;
```

## 4.6 右外连接 Right Outer Join

```sql
-- 查询所有部门名称 包括没有员工的 部门
select 
  e.ename,
  d.deptno,
  d.dname 
from
  emp e 
  right outer join dept d 
    on e.deptno = d.deptno ;
```

## 4.7 满外连接 Full Outer Join

mysql 不支持 Full join 【通过集合操作 union 合集】

```sql
-- 查询所有部门名称 包括没有员工的 部门 
-- + 所有员工姓名 部门名称 包括没有部门的员工
(SELECT 
  e.ename,
  d.deptno,
  d.dname 
FROM
  emp e 
  LEFT OUTER JOIN dept d 
    ON e.deptno = d.deptno) 
UNION
(SELECT 
  e.ename,
  d.deptno,
  d.dname 
FROM
  emp e 
  RIGHT OUTER JOIN dept d 
    ON e.deptno = d.deptno)
```



## 4.8 内连接 Inner  Join

```sql
-- 查询 员工姓名  部门名
SELECT e.ename,d.deptno,dname FROM emp e INNER JOIN dept d ON  e.deptno = d.deptno ;
```



## 4.9 集合操作

并集【UNION ，UNION ALL】

UNION   自动去重复   速度慢

UNION ALL 保留重复  速度快

注意： 列的个数相同   列的数据类型一致   无序

```sql
SELECT
* FROM
((SELECT * FROM emp WHERE deptno  = 10 ORDER  BY sal DESC )
UNION
(SELECT * FROM emp WHERE deptno  = 20 ORDER  BY sal DESC ))t 
ORDER BY t.sal DESC ;
```



# 5. 子查询

子查询：where 子句 ，from子句，select 子句 , having 子句

```sql
-- select 子句  [标量子查询]
SELECT ename,(SELECT dname FROM dept WHERE deptno=e.deptno) dname FROM emp e;
-- having 子句
SELECT 
  MIN(sal),
  deptno 
FROM
  emp 
GROUP BY deptno 
HAVING MIN(sal) = 
  (SELECT 
    MIN(min_sal) 
  FROM
    (SELECT 
      MIN(sal) min_sal 
    FROM
      emp 
    GROUP BY deptno) t) ;
```



子查询不返回  主查询不返回

```sql
-- 查询工资比 工号8888 的员工还高员工姓名
SELECT * FROM emp WHERE sal>(SELECT sal FROM emp WHERE empno = 8888) ;
```

单列 对 多列 

```sql
SELECT * FROM emp WHERE sal>(SELECT ename,sal FROM emp WHERE empno = 7788) ;
```

单行 对 多行

```sql
-- Subquery returns more than 1 row
SELECT * FROM emp WHERE sal = (SELECT MIN(sal) FROM emp GROUP BY deptno) ;
```



## 5.1 单行子查询

子查询结果【单行】

比较运算符  =   !=  >  >=  <  <=

```sql
-- 查询大于平均工资的员工
SELECT 
  * 
FROM
  emp 
WHERE sal > 
  (SELECT 
    AVG(sal) 
  FROM
    emp) ;
    
-- 查询 7788 相同部门 工种的员工
SELECT * FROM emp WHERE (deptno,job)=(SELECT deptno,job FROM emp WHERE empno = 7788) 
```

注意：返回多行  【一行 对多行/多值   ，一列对多列 ，NULL/空值】

## 5.2 多行子查询

返回 多行 【包含单行 --> NULL值】

in 【 = any ，= some】 ，not  in ， > all   >= all  <  <= all   ,>any

in ，not in 【无法使用索引查询，查询全表扫描】

```sql
-- 查询各部门中最低工资的员工姓名
SELECT e.ename,e.sal FROM emp e ,
(SELECT deptno,MIN(sal) dept_min_sal FROM emp GROUP BY deptno) t
WHERE e.deptno = t.deptno AND e.sal = t.dept_min_sal ;

-- 子查询
SELECT e.ename,e.sal FROM emp e WHERE (deptno,sal) IN
(SELECT deptno,MIN(sal) dept_min_sal FROM emp GROUP BY deptno)

SELECT e.ename,e.sal FROM emp e WHERE (deptno,sal) =ANY
(SELECT deptno,MIN(sal) dept_min_sal FROM emp GROUP BY deptno)

SELECT e.ename,e.sal FROM emp e WHERE (deptno,sal) =SOME
(SELECT deptno,MIN(sal) dept_min_sal FROM emp GROUP BY deptno)
```

exists ，not exists

```sql
-- 查询各部门中最低工资的员工姓名
SELECT 
  e.ename,
  e.sal 
FROM
  emp e 
WHERE EXISTS 
  (SELECT 
    NULL 
  FROM
    emp 
  GROUP BY deptno 
  HAVING deptno = e.deptno 
    AND MIN(sal) = e.sal) ;
```

in   VS  exists 

```sql
in : 无法使用索引，全表扫描   not in 【注意去 null值】
    主查询 3KW   子查询 1K
exists : 使用索引，不使用全表扫描 not exists
    主查询 1k    子查询 3kw
```


