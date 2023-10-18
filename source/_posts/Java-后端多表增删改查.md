---
title: Java-后端多表增删改查
date: 2023-07-30 19:27:36
tags: Java
category: Java
---

## 1. Many2One

### 1.1 数据建模

#### 关系图

![image-20230202151532563](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230202151532563.png)

#### 表结构

```sql
drop table if exists t_clazz;

/*==============================================================*/
/* Table: t_clazz      父                                         */
/*==============================================================*/
create table t_clazz
(
   bno                  varchar(20) not null,
   name                 varchar(20),
   primary key (bno)
);

alter table t_clazz comment '班级表';

drop table if exists t_student;

/*==============================================================*/
/* Table: t_student       子                                      */
/*==============================================================*/
create table t_student
(
   sid                  varchar(20) not null comment '学号 主键',
   name                 varchar(20) comment '姓名',
   age                  int(3) comment '年龄',
   sex                  int(1) comment '性别 0 男 1 女',
   bno                  varchar(20),
   primary key (sid)
);

alter table t_student comment '学生信息表';

alter table t_student add constraint FK_stu_clazz_bno foreign key (bno)
      references t_clazz (bno) on delete restrict on update restrict;
```

#### 表数据

```java
public abstract class BaseTest {
    protected JDBCTemplate jdbcTemplate = new JDBCTemplate() ;
    // 初始化 测试环境
    @Before
    public void before(){
        //清空表数据  truncate table 截断表  无法使用 关联关系  先删除儿子 在删除爹
        jdbcTemplate.update("delete from t_student ") ;
        jdbcTemplate.update("delete from t_clazz ") ;

        //插入班级数据
        jdbcTemplate.update("insert into t_clazz(bno,name) values ('B001','Java177')") ;
        jdbcTemplate.update("insert into t_clazz(bno,name) values ('B002','Java178')") ;
        //插入 68 条测试数据
        for (int i = 0; i < 68; i++) {
            jdbcTemplate.update("insert into t_student(sid,name,age,sex,bno) values ('S"+(i+1)+"','张三"+(i+1)+"',18,0,'B001')") ;
        }
    }

}
```

### 1.2 领域建模

```java
public class Clazz {
    private String bno ;
    private String name ;
}
```

```java
public class Student {
    private String sid ;
    private String name ;
    private Integer age ;
    private Integer sex ;
    private Clazz clazz = new Clazz()  ;
}
```

### 1.3 列表

#### DAO

```sql
-- 分页查询的SQL：
    select sid,s.name,age,sex,c.bno "clazz.bno",c.name "clazz.name" from t_student s left join t_clazz c on s.bno = c.bno where 1=1
```

```java
// t_student表 t_clazz表 同名列 二义性 条件查询需指定 哪个表的name
if (query!=null &&  StringUtils.isNotEmpty(query.getName())) {
            sb.append(" and s.name like ?") ;
            paramList.add("%"+query.getName()+"%");
 }
```



#### jsp

list-stu.jsp

```jsp
<table class="table table-striped table-bordered table-hover">
    <thead>
    <tr>
        <th><input type="checkbox" /></th>
        <th>学号</th>
        <th>姓名</th>
        <th>年龄</th>
        <th>性别</th>
        <th>班号</th>
        <th>班名</th>
        <th>操作</th>
    </tr>
    </thead>
    <tbody>
    <c:forEach items="${studentPage.list}" var="student">
    <tr>
        <td><input type="checkbox" name="sid" value="${student.sid}" /></td>
        <td>${student.sid}</td>
        <td>${student.name}</td>
        <td>${student.age}</td>
        <td>${student.sex eq 0 ?"男":"女"}</td>
        <td>${student.clazz.bno}</td>
        <td>${student.clazz.name}</td>
        <td>
            <a href="${pageContext.request.contextPath}/view-stu.do?sid=${student.sid}" class="btn btn-info btn-sm">修改</a>
            <a href="${pageContext.request.contextPath}/del-stu.do?sid=${student.sid}" class="btn btn-danger btn-sm">删除</a>
        </td>
    </tr>
    </c:forEach>
    </tbody>
</table>
```

### 1.4 新增before

#### DAO

```java
public interface ClazzDAO {
    List<Clazz> selectAll() ;
}
```

```java
public class ClazzDAOImpl implements ClazzDAO {
    private JDBCTemplate jdbcTemplate = new JDBCTemplate() ;

    @Override
    public List<Clazz> selectAll() {
        String DQL = "select bno,name from t_clazz" ;
        Object[] paramAy = {} ;
        return jdbcTemplate.queryList(DQL,Clazz.class,paramAy);
    }

}
```

#### Service

```java
public class ClazzService {
    private ClazzDAO clazzDAO = new ClazzDAOImpl() ;

    /**
     * 查询所有班级
     * @return
     */
    public Collection<Clazz> list(){
        return clazzDAO.selectAll();
    }

}
```

#### Controller

```java
@WebServlet(..."/toAddView-stu.do"})
public class StudentServlet extends HttpServlet {
    private ClazzService clazzService = new ClazzService() ;
    
    ... service(..){
        else if ("/toAddView-stu.do".equals(requestURI)){
            this.toAddView(request,response) ;
        }
    }
    
    //跳转到新增学生界面 
    protected void toAddView(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Collection<Clazz> clazzList = clazzService.list();
        request.setAttribute("clazzList",clazzList);
        request.getRequestDispatcher("/add-stu.jsp").forward(request,response);
    }
}
```

#### Jsp

list-stu.jsp

```jsp
<a href="${pageContext.request.contextPath}/toAddView-stu.do" class="btn btn-primary">添加学生</a>
```

add-stu.jsp

```jsp
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<div class="form-group">
    <label for="age">班级</label>
    <select name="clazz.bno">
        <option value="">--请选择班级--</option>
        <c:forEach items="${clazzList}" var="clazz">
            <option value="${clazz.bno}">${clazz.name}</option>
        </c:forEach>
    </select>
</div>
```

### 1.5 新增

#### DAO

```java
@Override
public boolean insert(Student student) {
    String DQL = "insert into t_student(sid,name,age,sex,bno) values (?,?,?,?,?)" ;
    Object[] paramAy = {student.getSid(),student.getName(),student.getAge(),student.getSex(),student.getClazz().getBno()} ;
    return jdbcTemplate.update(DQL,paramAy);
}
```

#### Controller

```java
 //重定向 进行查询
       response.sendRedirect(request.getContextPath()+"/page-stu.do");
```



### 1.6 查看

#### jsp

modify-stu.jsp

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<div class="form-group">
    <label for="age">班级</label>
    <select name="clazz.bno" class="form-control">
        <option value="">--请选择班级--</option>
        <c:forEach items="${clazzList}" var="clazz">
            <option value="${clazz.bno}">${clazz.name}</option>
        </c:forEach>
    </select>
</div>


//设置下拉框被选中
$("select").val("${student.clazz.bno}")
```



#### Controller

```java
  protected void view(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.获得查看的学生编号
        String sid = request.getParameter("sid");
        //2.调用业务方法
        Student student = studentService.get(sid);
        //3.request作用域中设置 学生信息
        request.setAttribute("student",student);
        Collection<Clazz> clazzList = clazzService.list();
        request.setAttribute("clazzList",clazzList);
        //4.跳转页面  服务器内部跳转
        request.getRequestDispatcher("/modify-stu.jsp").forward(request,response);
    }
```

#### DAO

```java
String DQL = "select sid,s.name,age,sex,c.bno \"clazz.bno\",c.name \"clazz.name\" from t_student s left join t_clazz c on s.bno = c.bno where s.sid=?" ;
```

### 1.7 修改

#### DAO

```java
@Override
public boolean update(Student student) {
    String DQL = "update t_student set name=?,age=?,sex=?,bno=? where sid=?" ;
    Object[] paramAy = {student.getName(),student.getAge(),student.getSex(),student.getClazz().getBno(),student.getSid()} ;
    return jdbcTemplate.update(DQL,paramAy);
}
```

## 2. One2Many

### 2.1 列表[单表]

#### controller

```java
@WebServlet(name = "ClazzServlet",urlPatterns = {"/list-clazz.do"})
public class ClazzServlet extends HttpServlet {
    /** 班级业务实例 */
    private ClazzService clazzService = new ClazzService() ;

    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.获得请求的url  包含项目名
        String requestURI = request.getRequestURI();
        //2. 去掉项目名 /list-stu.do  /add-stu.do
        requestURI = requestURI.replace(request.getContextPath(),"");
        //3.判断 url 与方法 调用
        if ("/list-clazz.do".equals(requestURI)){
            this.list(request,response) ;
        }
    }

    /**
     *
     * @param request
     * @param response
     * @throws ServletException
     * @throws IOException
     */
    protected void list(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 调用业务方法
        Collection<Clazz> clazzCollection = clazzService.list();
        //2. 把查询数据保存 四种(pageContext,request,session,application)属性作用域中  set  ==> list-stu.jsp  get
        request.setAttribute("clazzList",clazzCollection);
        //3. 跳转到 list-stu.jsp
        request.getRequestDispatcher("/list-clazz.jsp").forward(request,response);
    }
}
```



#### jsp

index.jsp

```jsp
<a href="${pageContext.request.contextPath}/list-clazz.do">班级列表</a> <br>
```

list-clazz.jsp

```jsp
 <c:forEach items="${clazzList}" var="clazz">
    <tr>
        <td><input type="checkbox" name="bno" value="${clazz.sid}" /></td>
        <td>${clazz.bno}</td>
        <td>${clazz.name}</td>
        <td>
            <a href="${pageContext.request.contextPath}/view-clazz.do?sid=${clazz.bno}" class="btn btn-info btn-sm">修改</a>
            <a href="${pageContext.request.contextPath}/del-clazz.do?sid=${clazz.bno}" class="btn btn-danger btn-sm">删除</a>
        </td>
    </tr>
</c:forEach>
```

### 2.2 新增准备【裸跳页面】

逻辑上： 直接怼到页面   比如  add-clazz.jsp

OCP        A. 代码复用   B.新增班级 【权限访问控制？当前登录用户是否有权限？】

#### jsp

list-clazz.jsp

```jsp
<a href="${pageContext.request.contextPath}/toAddView-clazz.do" class="btn btn-primary">添加班级</a>
```

add-clazz.jsp

```jsp
<h1>新增班级</h1>
<form action="${pageContext.request.contextPath}/add-clazz.do" method="post">
    <div class="form-group">
        <label for="sid">班号</label>
        <input
                type="text"
                class="form-control"
                id="sid"
                name="bno"
                value="${param.bno}"
                placeholder="请输入班号"
        />
    </div>
    <div class="form-group">
        <label for="sname">班名</label>
        <input
                type="text"
                class="form-control"
                id="sname"
                name="name"
                value="${param.name}"
                placeholder="请输入班名"
        />
    </div>
    <div class="form-group" style="text-align: center">
        <button type="submit" class="btn btn-primary">新增</button>
        <button type="submit" class="btn btn-default">重置</button>
    </div>
</form>
```

#### Controller

```java
//  ClazzServlet
@WebServlet(name = "ClazzServlet",urlPatterns = {"/list-clazz.do","/toAddView-clazz.do"})
....
    else if("/toAddView-clazz.do".equals(requestURI)){
            this.toAddView(request,response) ;
        }

//跳转到新增班级界面
protected void toAddView(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    request.getRequestDispatcher("/add-clazz.jsp").forward(request,response);
}
```



### 2.3 新增【单表】

#### controller

```java
//添加新的请求 url
@WebServlet(name = "ClazzServlet",urlPatterns = {"/list-clazz.do","/toAddView-clazz.do","/add-clazz.do"})

//添加判断  调用自定义的方法
else if("/add-clazz.do".equals(requestURI)){
            this.add(request,response) ;
}


protected void add(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.设置请求中文乱码
        request.setCharacterEncoding("UTF-8");
        //调用 自定义 工具类  实现：解析请求自动封装 pojo中
        Clazz clazz = RequestUtil.parseParameter(request, Clazz.class);
        //4. 调用业务方法
        boolean saveRs = clazzService.save(clazz);
        //5. 根据业务方法的返回值
        if (saveRs){
            //6.响应
            //重定向 进行查询
            response.sendRedirect(request.getContextPath()+"/list-clazz.do");
            return;
        }
        //新增失败  view实现显示效果
        request.setAttribute("errorMsg","班号已经存在！！！");
        //使用服务器内部跳转
        request.getRequestDispatcher("/toAddView-clazz.do").forward(request,response);
}
```

#### Service

```java
... ClazzService {
    public boolean save(Clazz clazz) {
        //获得新增的班号
        String bno = clazz.getBno() ;
        //1.判断是否 新增的班号 db 中已经存在
        Clazz dbClazz = clazzDAO.selectById(bno);
        if (dbClazz!=null){
            return false ;
        }
        //班号不存在  向数据库插入数据
        return clazzDAO.insert(clazz) ;
    }
}
```

#### DAO

```java
... ClazzDAO{
    Clazz selectById(String bno);

    boolean insert(Clazz clazz);
}
```

```java
... ClazzDAOImpl ... {
    @Override
    public Clazz selectById(String bno) {
        String DQL = "select bno,name from t_clazz where bno=?" ;
        Object[] paramAy = {bno} ;
        return jdbcTemplate.queryObject(DQL,Clazz.class,paramAy);
    }

    @Override
    public boolean insert(Clazz clazz) {
        String DQL = "insert into t_clazz(bno,name) values (?,?)" ;
        Object[] paramAy = {clazz.getBno(),clazz.getName()} ;
        return jdbcTemplate.update(DQL,paramAy);
    }
}    
```

### 2.4 删除【单表/子记录依赖】

#### jsp

list-clazz.jsp

```jsp
 <form action='${pageContext.request.contextPath}/del-clazz.do'>
      <td><input type="checkbox" name="bno" value="${clazz.bno}" /></td>
<a href="${pageContext.request.contextPath}/del-clazz.do?bno=${clazz.bno}" class="btn btn-danger btn-sm">删除</a>
```

#### controller

```java
@WebServlet(name = "ClazzServlet",urlPatterns = {"/list-clazz.do","/toAddView-clazz.do","/add-clazz.do","/del-clazz.do"})

else if("/del-clazz.do".equals(requestURI)){
            this.del(request,response) ;
}

protected void del(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.获得 删除的班号
        String[] idAy = request.getParameterValues("bno");
        //把数组 拼接成一个 字符串
        String id ="";
        for (String s : idAy) {
            id +=s+"," ;
        }
        id = id.substring(0,id.length()-1) ;
        //2. 调用业务方法
        boolean removeRs = clazzService.remove(id);
        String msg = removeRs?"":"del-fail" ;
        response.sendRedirect(request.getContextPath()+"/list-clazz.do?msg="+msg);
}
```

#### Service

```java
public class ClazzService {
    private StudentDAO studentDAO = new StudentDAOImpl() ;

    public boolean remove(String id) {
        //字符串 转成数组
        String[] sidAy = id.split(",");
        for (String bno : sidAy) {
            //根据FK 查询学生列表
            List<Student> studentList = studentDAO.selectByFk(bno);
            //删除班级编号 在学生表中出现了   不能删除
            if (!studentList.isEmpty()){
                return false ;
            }
        }
        //调用批处理删除
        return clazzDAO.delete(sidAy) ;
    }
}
```

#### DAO

##### StudentDAO

```java
List<Student> selectByFk(String bno) ;
```

```java
@Override
public List<Student> selectByFk(String bno) {
    String DQL = "select sid,name,age,sex from t_student where bno=?" ;
    Object[] paramAy = {bno} ;
    return jdbcTemplate.queryList(DQL,Student.class,paramAy);
}
```

##### ClazzDAO

```java
boolean delete(String ... idAy) ;
```

```java
public class ClazzDAOImpl implements ClazzDAO {
    @Override
    public boolean delete(String... sidAy) {
        String DML = "delete from t_clazz where bno=?" ;
        Object[][] paramAy = new Object[sidAy.length][1] ;
        for (int i = 0; i < sidAy.length; i++) {
            paramAy[i][0] = sidAy[i] ;
        }
        return jdbcTemplate.updateBatch(DML,paramAy);
    }

}
```



### 2.5 查看【N+1】

#### jsp

list-clazz.jsp

```jsp
<a href="${pageContext.request.contextPath}/view-clazz.do?bno=${clazz.bno}" class="btn btn-info btn-sm">修改</a>
```

modify-clazz.jsp

```jsp
<input
        type="text"
        class="form-control"
        id="sname"
        name="name"
        value="${clazz.name}"
        placeholder="请输入班名"
/>

<c:forEach items="${clazz.studentList}" var="student">
    <tr>
        <td>${student.sid}</td>
        <td>${student.name}</td>
        <td>${student.age}</td>
        <td>${student.sex eq 0 ?"男":"女"}</td>
    </tr>
</c:forEach>
```



#### Controller

```java
@WebServlet(name = "ClazzServlet",urlPatterns = {"/list-clazz.do","/toAddView-clazz.do","/add-clazz.do","/del-clazz.do","/view-clazz.do"})

else if("/view-clazz.do".equals(requestURI)){
      this.view(request,response) ;
}


protected void view(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //1.获得查看的学生编号
    String bno = request.getParameter("bno");
    //2.调用业务方法
    Clazz clazz = clazzService.get(bno) ;
    //3.request作用域中设置 学生信息
    request.setAttribute("clazz",clazz);
    //4.跳转页面  服务器内部跳转
    request.getRequestDispatcher("/modify-clazz.jsp").forward(request,response);
}
```



#### Service

```java
 public Clazz get(String bno) {
    //N+1查询
    //1. 根据班级id 查询 班级对象  单表查询
    Clazz clazz = clazzDAO.selectById(bno);
    //2. 根据班级id 查询当前班级的学生列表  单表查询
    List<Student> studentList = studentDAO.selectByFk(bno);
    //3.当前班级对象的 stuList属性进行赋值
    clazz.setStudentList(studentList);
    return clazz ;
}
```



### 2.6 修改【单表】

#### jsp

modify-clazz.jsp

```jsp
<form action="${pageContext.request.contextPath}/modify-clazz.do" method="post">
    ....
```

#### controller

```java
@WebServlet(name = "ClazzServlet",urlPatterns = {"/list-clazz.do","/toAddView-clazz.do","/add-clazz.do","/del-clazz.do","/view-clazz.do","/modify-clazz.do"})


else if("/modify-clazz.do".equals(requestURI)){
     this.modify(request,response) ;
}

protected void modify(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //1.设置请求中文乱码
    request.setCharacterEncoding("UTF-8");
    //调用 自定义 工具类  实现：解析请求自动封装 pojo中
    Clazz clazz = RequestUtil.parseParameter(request, Clazz.class);
    //4. 调用业务方法
    boolean modifyRs = clazzService.modify(clazz);
    //5. 根据业务方法的返回值
    if (modifyRs){
        //6.响应
        //重定向 进行查询
        response.sendRedirect(request.getContextPath()+"/list-clazz.do");
        return;
    }
    request.setAttribute("errorMsg","修改班级失败！！！");
    request.getRequestDispatcher("/view-clazz.do").forward(request,response);
}
```



#### Service

```java
public boolean modify(Clazz clazz) {
        return clazzDAO.update(clazz) ;
}
```

#### DAO

```java
boolean update(Clazz clazz);
```

```java
@Override
public boolean update(Clazz clazz) {
    String DQL = "update t_clazz set name=? where bno=?" ;
    Object[] paramAy = {clazz.getName(),clazz.getBno()} ;
    return jdbcTemplate.update(DQL,paramAy);
}
```



## 3. Many2Many

### 3.1 数据建模

#### 表关系

![image-20230203150523130](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230203150523130.png)

#### 表结构

```sql
drop table if exists t_clazz;

/*==============================================================*/
/* Table: t_clazz      父                                         */
/*==============================================================*/
create table t_clazz
(
   bno                  varchar(20) not null,
   name                 varchar(20),
   primary key (bno)
);

alter table t_clazz comment '班级表';

drop table if exists t_student;

/*==============================================================*/
/* Table: t_student       子                                      */
/*==============================================================*/
create table t_student
(
   sid                  varchar(20) not null comment '学号 主键',
   name                 varchar(20) comment '姓名',
   age                  int(3) comment '年龄',
   sex                  int(1) comment '性别 0 男 1 女',
   bno                  varchar(20),
   primary key (sid)
);

alter table t_student comment '学生信息表';



drop table if exists t_teacher;

/*==============================================================*/
/* Table: t_teacher                                             */
/*==============================================================*/
create table t_teacher
(
   tno                  varchar(20) not null,
   tname                varchar(20),
   primary key (tno)
);

alter table t_teacher comment '教师表';

drop table if exists t_student_teacher;

/*==============================================================*/
/* Table: t_student_teacher                                     */
/*==============================================================*/
create table t_student_teacher
(
   tno                  varchar(20) not null,
   sid                  varchar(20) not null comment '学号 主键',
   primary key (tno, sid)
);

alter table t_student_teacher comment '教师学生关系表';

alter table t_student_teacher add constraint FK_stu_teacher_teacher_tno foreign key (tno)
      references t_teacher (tno) on delete restrict on update restrict;

alter table t_student_teacher add constraint FK_stu_teacher_stu_sid foreign key (sid)
      references t_student (sid) on delete restrict on update restrict;

alter table t_student add constraint FK_stu_clazz_bno foreign key (bno)
      references t_clazz (bno) on delete restrict on update restrict;
```

#### 表数据

```java
@Before
public void before(){
    //清空表数据  truncate table 截断表  无法使用 关联关系  先删除儿子 在删除爹
    jdbcTemplate.update("delete from t_student_teacher ") ;
    jdbcTemplate.update("delete from t_teacher ") ;
    jdbcTemplate.update("delete from t_student ") ;
    jdbcTemplate.update("delete from t_clazz ") ;

    //插入班级数据
    jdbcTemplate.update("insert into t_clazz(bno,name) values ('B001','Java177')") ;
    jdbcTemplate.update("insert into t_clazz(bno,name) values ('B002','Java178')") ;
    //插入学生 68 条测试数据
    for (int i = 0; i < 68; i++) {
        jdbcTemplate.update("insert into t_student(sid,name,age,sex,bno) values ('S"+(i+1)+"','张三"+(i+1)+"',18,0,'B001')") ;
    }
    //插入教师数据
    jdbcTemplate.update("insert into t_teacher(tno,tname) values ('T001','赵贺贺')") ;
    jdbcTemplate.update("insert into t_teacher(tno,tname) values ('T002','赵健')") ;
    jdbcTemplate.update("insert into t_teacher(tno,tname) values ('T003','马美平')") ;

    //插入学生与教师关系表 68*2 条测试数据
    for (int i = 0; i < 68; i++) {
        jdbcTemplate.update("insert into t_student_teacher(sid,tno) values ('S"+(i+1)+"','T001'),('S"+(i+1)+"','T002')") ;
    }
}
```

### 3.2 列表

#### pojo

```java
public class Teacher {
    private String tno ;
    private String tname ;
}
```

```java
public class Student {
    ...
    /**Many2Many 一个学生拥有多个老师 */
    private List<Teacher> teacherList ;
}
```

#### DAO

```java
public interface TeacherDAO {
    List<Teacher> selectBySid(String sid) ;
}
```

```java
public class TeacherDAOImpl implements TeacherDAO {
    private JDBCTemplate jdbcTemplate = new JDBCTemplate() ;
    @Override
    public List<Teacher> selectBySid(String sid) {
        String DQL = "select tno,tname from t_teacher where tno in (select tno from t_student_teacher where sid = ?)" ;
        Object[] paramAy = {sid} ;
        return jdbcTemplate.queryList(DQL, Teacher.class,paramAy);
    }
}
```

#### service

```java
public class StudentService {
    private TeacherDAO teacherDAO = new TeacherDAOImpl() ;

    public void selectPage(Page<Student, StudentQuery> page){
        //1.统计总记录数
        long count = studentDAO.count(page.getCond());
        //2.查询当前页面的数据
        List<Student> studentList = studentDAO.selectPage(page);
        //循环所有的学生列表
        for (Student student : studentList) {
            //根据学号查询老师信息
            List<Teacher> teacherList = teacherDAO.selectBySid(student.getSid());
            //设置到 当前学生的 关联老师集合属性
            student.setTeacherList(teacherList);
        }
        //3. 设置总记录数
        page.setTotalRecord(count);
        //4. 设置当前页的数据
        page.setList(studentList);
    }
}
```

### jsp

list-stu.jsp

```jsp
<td>
    <c:forEach items="${student.teacherList}" var="teacher" varStatus="vs">
        ${teacher.tname}
        <c:if test="${not vs.last}">,</c:if>
    </c:forEach>
</td>
```

### 3.3 显示关联教师

#### DAO

```java
public interface TeacherDAO {
    List<Teacher> selectAll();
}
```

```java
public class TeacherDAOImpl implements TeacherDAO {
    @Override
    public List<Teacher> selectAll() {
        String DQL = "select tno,tname from t_teacher" ;
        Object[] paramAy = {} ;
        return jdbcTemplate.queryList(DQL, Teacher.class,paramAy);
    }
}
```

#### Service

```java
public class StudentService {
    private TeacherDAO teacherDAO = new TeacherDAOImpl() ;

    /**
     * 根据id 查询学生信息
     * @param sid
     * @return
     */
    public Student get(String sid){
        //学生信息  班级信息
        Student student =  studentDAO.selectById(sid) ;
        // N+1 根据学生id 查询关联的教师
        List<Teacher> teacherList = teacherDAO.selectBySid(sid);
        // 设置当前学生关联的教师信息
        student.setTeacherList(teacherList);
        return student ;
    }
}
```

#### Controller

```java
@WebServlet(.... "/toSelectView-teacher.do"})


else if ("/toSelectView-teacher.do".equals(requestURI)){
     this.toSelectTeacherView(request,response) ;
}


protected void toSelectTeacherView(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //1.获得学生的id
    String sid = request.getParameter("sid");
    //2.查询当前学生信息
    Student student = studentService.get(sid);
    //3.查询所有的老师信息
    List<Teacher> teacherList = teacherService.list() ;
    //4. 数据存放在作用域中
    request.setAttribute("student",student);
    request.setAttribute("teacherList",teacherList);
    request.getRequestDispatcher("/select-teacher.jsp").forward(request,response);
}
```

#### Jsp

list-stu.jsp

```jsp
<a href="${pageContext.request.contextPath}/toSelectView-teacher.do?sid=${student.sid}" class="btn btn-info btn-sm">关联教师</a>
```

select-teacher.jsp

```jsp
<div class="container">
    <h1>学生选择教师</h1>
    <div class="widget-content padded clearfix">
        <!-- 学生基本信息 -->
        <table class="table detail-view">
            <tbody>
            <tr>
                <th>学号</th>
                <td>${student.sid}</td>
            </tr>
            <tr>
                <th>姓名</th>
                <td>${student.name}</td>
            </tr>
            <tr>
                <th>年龄</th>
                <td>${student.age}</td>
            </tr>
            <tr>
                <th>性别</th>
                <td>${student.sex eq 0 ?"男":"女"}</td>
            </tr>
            <tr>
                <th>班级</th>
                <td>${student.clazz.name}</td>
            </tr>

            </tbody>
        </table>
        <!-- 教师信息 -->
        <form action="">
        <table class="table table-striped table-bordered table-hover">
            <thead>
            <tr>
                <th><input type="checkbox" /></th>
                <th>工号</th>
                <th>姓名</th>
            </tr>
            </thead>
            <tbody>
            <c:forEach items="${teacherList}" var="teacher">
                <tr>
                    <td><input type="checkbox" name="tno" value="${teacher.tno}"/></td>
                    <td>${teacher.tno}</td>
                    <td>${teacher.tname}</td>
                </tr>
            </c:forEach>
            </tbody>
        </table>
        <div class="form-group" style="text-align: center">
            <button type="submit" class="btn btn-primary">关联教师</button>
            <button type="submit" class="btn btn-default">重置</button>
        </div>
        </form>
    </div>
</div>
<script>
    // 循环当前学生的关联老师信息
    <c:forEach items="${student.teacherList}" var="teacher">
        $(":checkbox[value='${teacher.tno}']").prop('checked',true)
    </c:forEach>
</script>
```



### 3.4 设置关联教师

#### jsp

select-teacher.jsp

```jsp
 <!-- 教师信息 -->
<form action="${pageContext.request.contextPath}/set-teacher.do">
    <input type="hidden" name="sid" value="${student.sid}">
....
        <tr>
            <td><input type="checkbox" name="tno" value="${teacher.tno}"/></td>
```

#### DAO

```java
public interface StudentDAO {
    boolean deleteRefBySid(String sid) ;

    boolean insertRef(Student student) ;
}
```

```java
public class StudentDAOImpl implements StudentDAO {
    @Override
    public boolean deleteRefBySid(String sid) {
        String DQL = "delete from t_student_teacher where sid = ?" ;
        Object[] paramAy = {sid} ;
        return jdbcTemplate.update(DQL,paramAy);
    }

    @Override
    public boolean insertRef(Student student) {
        //获得当前学生的学号
        String sid = student.getSid();
        //获得当前学生关联的教师信息
        List<Teacher> teacherList = student.getTeacherList();
        String DML = "insert into t_student_teacher(sid,tno) values(?,?)" ;
        Object[][] paramAy = new Object[teacherList.size()][2] ;
        for (int i = 0; i < teacherList.size(); i++) {
            //教师信息
            Teacher teacher = teacherList.get(i);
            paramAy[i][0] = sid ;
            paramAy[i][1] = teacher.getTno() ;
        }
        return jdbcTemplate.updateBatch(DML,paramAy);
    }
}
```

#### Service

```java
public class StudentService {

    public boolean setTeacher(Student student){
        //1.中间表删除 当前学号 关联所有信息
        boolean delRs = studentDAO.deleteRefBySid(student.getSid()) ;
        //2.批量新增 中间表 关联信息
        boolean insertRs = studentDAO.insertRef(student) ;
        return delRs && insertRs ;
    }
}
```

#### controller

```java
@WebServlet(..."/set-teacher.do"})

else if ("/set-teacher.do".equals(requestURI)){
     this.setTeacher(request,response) ;
}


protected void setTeacher(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // 获得学生的id
    String sid = request.getParameter("sid");
    // 关联的教师工号
    String[] tnoAy = request.getParameterValues("tno");
    //封装学生对象
    Student student = new Student();
    student.setSid(sid);
    List<Teacher> teacherList = new ArrayList<>() ;
    for (String tno : tnoAy) {
        Teacher teacher = new Teacher();
        teacher.setTno(tno);
        teacherList.add(teacher);
    }
    //设置关联关系
    student.setTeacherList(teacherList);
    boolean rs = studentService.setTeacher(student) ;
    //重新查询
    response.sendRedirect(request.getContextPath()+"/page-stu.do");
}
```