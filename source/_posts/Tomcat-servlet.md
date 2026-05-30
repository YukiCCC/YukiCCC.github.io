---
title: Tomcat-servlet
date: 2023-07-11 09:34:13
tags: Tomcat
category: 数据库
---

Tomcat是web容器。在进行web项目开发的时候，经常需要http协议，也就是基于请求和响应，比如你在百度输入一行内容搜索，那么百度服务器如何处理这个请求呢？它需要创建servlet来处理，servlet其实就是java程序，只是在服务器端的java程序，servlet通过配置文件拦截你的请求，并进行相应处理，然后展示给你相应界面。那么servlet如何创建？这时候就要用到tomcat了。

## 1.网络编程

### 1.1 图解

![image-20220601202154425](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220601202154425.png)

### 1.2 Server

```java
public class ServerApp {
    public static void main(String[] args) throws IOException {
        //忠告 数组
        final String[] ADVICE_AY = {"每天吃钙，到老我也健康膝盖!","听人劝，吃饱饭！","代码写的好，要饭要到老！"};
        //随机数
        final Random RAND = new Random();
        //创建服务器端 套接字
        ServerSocket serverSocket = new ServerSocket(8080);
        //服务器端 一直接受 客户端的请求 死循环
        while (true){
            //获得 客户端套接字
            Socket socket = serverSocket.accept();
            new Thread(()->{
                try {
                    // 获得输出流
                    OutputStream os = socket.getOutputStream();
                    //输出 随机字符串  字节流数组
                    os.write(ADVICE_AY[RAND.nextInt(ADVICE_AY.length)].getBytes("UTF-8"));
                    //清空 缓存
                    os.flush();
                    InputStream is = socket.getInputStream();
                    byte[] ay = new byte[1024] ;
                    is.read(ay) ;
                    System.out.println("来着客户端的消息:"+new String(ay));
                    //释放流
                    os.close();
                    is.close();
                    //关闭套接字
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```



### 1.3 Client

```java
public class ClientApp {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8080);
        //获得输入流
        InputStream is = socket.getInputStream();
        byte[] ay = new byte[1024];
        is.read(ay) ;
        //打印 服务器端消息
        System.out.println("来着服务器端消息:"+new String(ay));

        //获得输出流
        OutputStream os = socket.getOutputStream();
        os.write(("hello Server !!!"+Math.random()).getBytes());
        os.flush();

        is.close();
        os.close();
        socket.close();
    }
}
```



## 2. CS/BS架构

### 2.1 图解

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/bs-cs.png)

### 2.2 C/S与B/S区别

```
1．硬件环境不同：

C/S 一般建立在专用的网络上，小范围里的网络环境，局域网之间再通过专门服务器提供连接和数据交换服务。B/S 建立在广域网之上的，不必是专门的网络硬件环境，例如电话上网，租用设备. 信息自己管理. 有比C/S更强的适应范围，一般只要有操作系统和浏览器就行。

2．对安全要求不同：

C/S 一般面向相对固定的用户群，对信息安全的控制能力很强。 一般高度机密的信息系统采用C/S 结构适宜。可以通过B/S发布部分可公开信息。B/S 建立在广域网之上， 对安全的控制能力相对弱， 可能面向不可知的用户。

3．对程序架构不同：

C/S 程序可以更加注重流程， 可以对权限多层次校验， 对系统运行速度可以较少考虑。B/S 对安全以及访问速度的多重的考虑，建立在需要更加优化的基础之上. 比C/S有更高的要求 B/S结构的程序架构是发展的趋势，从MS的.Net系列的BizTalk 2000 Exchange 2000等，全面支持网络的构件搭建的系统。SUN 和IBM推JavaBean 构件技术等，使 B/S更加成熟.。

4．软件重用不同：

C/S 程序可以不可避免的整体性考虑， 构件的重用性不如在B/S要求下的构件的重用性好。B/S 的多重结构，要求构件相对独立的功能， 能够相对较好的重用，就如买来的餐桌可以再利用，而不是做在墙上的石头桌子。

5．系统维护不同：

C/S 程序由于整体性，必须整体考察，处理出现的问题以及系统升级、升级难、 可能是再做一个全新的系统。B/S 构件组成，方便构件个别的更换，实现系统的无缝升级. 系统维护开销减到最小.用户从网上自己下载安装就可以实现升级。
```


## 3. 手动部署

### 3.0 创建web应用

![image-20220310103928098](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310103928098.png)

![image-20220310104500193](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310104500193.png)

### 3.1 静态html

![image-20220310104735744](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310104735744.png)

http://localhost/web01/hello.html

### 3.2 动态jsp

![image-20220310111454412](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310111454412.png)

http://localhost/web01/hello.jsp

![image-20220310111650338](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310111650338.png)



### 3.3 Servlet

```
Servlet  = Server  applet  运行在服务器 tomcat上的小程序
1.必须 规范：必须实现 Servlet 接口
 //直接实现  Servlet 接口
 A  implements Servlet{
 
 
 }
 
 HttpServlet implments Servlet{
 
 
 }
 
 //间接实现 Servlet 接口
 A  extends HttpServlet{
   ...
 }
```

#### 源码

参考：

![image-20220310113227634](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310113227634.png)

```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class HelloWorld extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException
    {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Hello World!</title>");
        out.println("</head>");
        out.println("<body>");
        out.println("<h1>Hello World Servlet ... !</h1>");
        out.println("</body>");
        out.println("</html>");
    }
}
```

#### 编译

![image-20220310113730274](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310113730274.png)

javac -cp D:\java168_2\server\apache-tomcat-8.5.73\lib\servlet-api.jar .\HelloWorld.java

#### 部署

![image-20220310134552680](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310134552680.png)

```
1. 桌面： aa文件下/HelloWorld.class 放在 classess文件夹下
2.  tomcat/lib/servlet-api.jar   lib 空着
```

#### 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1"
  metadata-complete="true">

    <servlet>
        <servlet-name>HelloWorldExample</servlet-name>
        <servlet-class>HelloWorld</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>HelloWorldExample</servlet-name>
        <url-pattern>/hello.do</url-pattern>
    </servlet-mapping>
 
</web-app>
```



![image-20220310135945200](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310135945200.png)

#### 访问

http://localhost/web01/hello.do

![image-20230309145910314](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20230309145910314.png)



## 4. IDE web开发

### 4.1 创建项目

![image-20220601195318245](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220601195318245.png)



### 4.2 创建Servlet

```java
public class HelloServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		//控制台输出  IDE console 
		System.out.println("=====HelloServlet===doGet 该方法被执行啦=======");
		//通过 流  浏览器 输出  hello ...
		resp.getWriter().println("<h1> hello ...</h1>");
	}
}
```



### 4.3 配置Servlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1"
  metadata-complete="true">
    <!--  注册 servlet 类 -->
    <servlet>
    	<servlet-name>hello</servlet-name>
    	<servlet-class>com.wanho.java171.servlet.HelloServlet</servlet-class>
    </servlet>
    <!--  映射 servlet 类访问路径 -->
    <servlet-mapping>
    	<servlet-name>hello</servlet-name>
    	<url-pattern>/hello.do</url-pattern>
    </servlet-mapping>
  </web-app>
```



### 4.4 访问Servlet

http://localhost/day10-01-servlet/hello.do

基于请求驱动   地址栏直接怼  超链接  location.href   form action=“”

基于事件驱动

![image-20220601195318245](https://raw.githubusercontent.com/YukiCCC/figure/main/servlet.png)



## 5.Servlet API

### 5.1 HttpServletRequest

```java
//获得请求参数的值
方法1： String name = request.getParameter("name");
"name" 参数名：
      注意：如果参数名 不存在   String name 的值？ null
           如果参数名 存在 没有值   String name 的值？ ""
    
//1.针对 post 请求有效  中文乱码
   req.setCharacterEncoding("UTF-8");    
```



![image-20220310170633498](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220310170633498.png)

```java
//兼容： get/post
public class TestRequestServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse resp) throws ServletException, IOException {
		 //获得用户请求参数  使用 HttpServletRequest req 对象
		String name = request.getParameter("name");
		String age = request.getParameter("age");
		System.out.println("姓名:"+name+",年龄:"+age);
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		this.doGet(req, resp);
	}
}
```
#### 注意：doPost与doGet的区别
GET调用用于获取服务器信息，并将其做为响应返回给客户端。当经由Web浏览器或通过HTML、JSP直接访问Servlet的URL时，一般用GET调用。

POST用于客户端把数据传送到服务器端，是可以隐藏传送给服务器的任何数据。Post适合发送大量的数据。

区别：

1、生成方式

get生成方式有四种：1）直接在URL地址栏中输入URL。2）网页中的超链接。3）form中method为get。4）form中method为空时，默认是get提交。

post生成方式：form中method属性为post。

2、数据传送方式

get方式：表单数据存放在URL地址后面。所有get方式提交时HTTP中没有消息体。

post方式：表单数据存放在HTTP协议的消息体中以实体的方式传送到服务器。

3、服务器获取数据方式

GET方式：服务器采用request.QueryString来获取变量的值。

POST方式：服务器采用request.Form来获取数据。

4、传送的数据量

GET方式：数据量长度有限制，一般不超过2kb。因为是参数传递，且在地址栏中，故数据量有限制。

POST方式：适合大规模的数据传送。因为是以实体的方式传送的。

5、安全性

GET方式：安全性差。因为是直接将数据显示在地址栏中，浏览器有缓冲，可记录用户信息。所以安全性低。

POST方式：安全性高。因为post方式提交数据时是采用的HTTP post机制，是将表单中的字段与值放置在HTTP HEADER内一起传送到ACTION所指的URL中，用户是看不见的。

6、在用户刷新时

GET方式：不会有任何提示、

POST方式：会弹出提示框，问用户是否重新提交



### 5.2 HttpServletResponse

#### 响应字符流

```java
// 1.响应 [html内容] 字符输出流
resp.setContentType("text/html;charset=UTF-8");
PrintWriter out = resp.getWriter();
//通过 流  浏览器 输出  hello ...    类似： js ： document.write("html代码")
out.println("<h1> hello . 该方法被执行啦..</h1>");
```

#### 响应URL

```java
//2. 响应URL  浏览器 根据响应URL 再次发送新的请求
resp.sendRedirect("./request.do") ;
```



