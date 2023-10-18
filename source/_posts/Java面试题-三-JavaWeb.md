---
title: Java面试题-三(JavaWeb)
date: 2023-09-18 13:17:52
tags: 面试技巧
category: Java
---
## 1. 如何解决POST请求中文乱码问题，GET的又是如何处理的

- 解决post请求乱码问题：

在web.xml中配置一个CharacterEncodingFilter过滤器，设置成UTF-8；

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

- get请求中文参数出现乱码的解决方法有两个：

  - 修改tomcat配置文件添加编码与工程编码一致，如下：

  ```xml
  <ConnectorURIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
  ```

  - 另一种方法对参数进行重新编码

  ```java
  String userName = new String(request.getParamter("userName").getBytes("ISO8859-1"),"utf-8")
  ```

   ISO8859-1是tomcat默认编码，需要将tomcat编码后的内容按utf-8编码 

## 2. Servlet接口中有哪些方法?

Servlet接口定义了5个方法，其中前三个方法与Servlet生命周期相关： 

- void init(ServletConfig config) throws ServietException

- void service(ServletRequest req，ServietResponse resp) throws ServietException,java.io.IOException 

- void destory() 

- java.lang.String getServletlnfo() 

- ServletConfig getServletConfig() 

  Web容器加载Servlet并将其实例化后，Servlet生命周期开始容器运行其init()方法进行Servlet的初始化；请求到达时调用Servlet的service(）方法，service()方法会根据需要调用与请求对应的doGet或doPost等方法；当服务器关闭或项目被卸载时服务器会将Servlet实例销毁，此时会调用 Servlet的destroy()方法。 


## 3. 转发（forward）和重定向（redirect）的区别？

- 相同点：

  - 都能跳转到其它页面

- 不同点：
  - 转发请求只发出一次请求，而重定向发出多次请求
  - 转发request对象是同一个,而重定向request对象是多个
  - 转发地址栏不变,而重定向地址栏改变
  - 转发的"/"是相对于项目,而重定向的"/"相对于WEB容器
  - 转发只能转发到项目资源,而重定向可以是项目资源也可以是外部资源 

- 请求转发有两种写法

```java
//forward 一去不回头，在转发之后向控制台(System.out.println)仍然会输出，但out.println("xxx")不会在页面上输出
request.getRequestDispatcher("/xx").forward(request,response);
//include 一去还回头，在转发之后向控制台(System.out.println)仍然会输出，但out.println("xxx")也会在页面上输出
request.getRequestDispatcher("/xx").include(request,response);
```

## 4. web项目"/"的相对路径 的理解

- web.xml里"/"相当于项目
- index.html里"/"相当于web容器
- 请求转发里"/"相当于项目
- 请求重定向"/"相当于web容器
- 自己处理的相当于项目
- 服务器处理相当于web容器

## 5. jsp有哪些内置对象？作用分别是什么？

- Jsp有9个内置对象： 
  - request：封装客户端的请求，其中包合来自GET或Post请求的参数； 
  - response：封装服务器对客户端的响应； 
  - pageContext：通过该对象可以获取其他对象； 
  - session：封装用户会话的对象； 
  - application：封装服务器运行环境的对象； 
  - out：输出服务器响应的输出流对象； 
  - config: Web应用的配置对象； 
  - page: JsP页面本身（相当于Java程序中的this ）
  - exception：封装页面抛出异常的对象。

## 6. GET与POST请求比较

1. 使用GET方式传递参数:

   - 在浏览器地址栏中输入某个地址或单击网页上一个超链接 ，浏览器发出的HTTP请求方式为GET

   - 如果form表单中method属性设置成了"GET，浏览器提交这个FORM表单的HTTP请求方式也为GET

   - 使用GET请求给WEB服务器传递参数的格式 （Query String Paramaters）

     ```uri
     http://localhost:8080/02_JavaWeb_Servlet_HTTP/loginServlet2?uname=%E5%BC%A0%E4%B8%89
     ```

   - 缺点:

     - 地址栏大小有限制，最多255个字符 
     - 不安全，如果是密码则在地址栏中会以明码显示 

   - 优点:

     - 书签  http://localhost:8080/books/java100day?page=10; 

2. 使用POST方式传递参数 【推荐】

   - 如果form表单中method属性设置成了"POST"，浏览器提交这个FORM表单的HTTP请求方式就为POST
   - 在址址栏中看不到传递的数据，因为它是使用表单数据传递的(Form Datas)
   - 优点：
     - 大小没有限制
     - 安全

## 7. 常用的Web服务器有哪些？

- Unix和Linux平台下使用最广泛的免费HTTP服务器，Apache服务器，而Windows平台的服务器通常使用IIS作为Web服务器。 
- 选择Web服务器应考虑的因素有：性能、安全性、日志统计、虚拟主机、代理服务器、缓冲服务和集成应用程序等。
- 下面是对常见服务器的简介：
  - IIS : Microsoft的Web服务器产品全称是Internet Information Services。IIS是允许在公共Intranet或Internet上发布信息的Web服务器。IIS是目前最流行的Web服务产品之一，很多著名的网站都是建立在IIS的平台上。IIS提供了一个图形界面的管理工具称为Internet服务管理器，可用于监视配置和控制Internet服务。IIS是一种Web服务软件，其中包括Web服务器、FTP服务器、NNTF服务器、 SMTP服务器分别用于网页浏览、文件传输、新闻服务和邮件发送等方面它使得在网络（包括互联网和局域网上发布信息成了一件很容易的事。它提供ISAPI(lntranet Server API）作为扩展Web服务器功能的编程接口；同时它还提供一个Internet数据库连接器可以实现对数据库的查询和更新。
  - Kangle:Kangle Web服务器是一款跨平台、功能强大、安全稳定、易操作的高性能Web服务器和反向代理服务 软件。此外Kangle也是一款专为做虚拟主机研发的WEB 服务器。实现虚拟主机独立进程、独立身份运行。用户 之间安全隔离，一个用户出问题不影响其他用户。支持PHP、ASP、ASP.NET、Java、Ruby等多种动态开发语言。 
  - WebSphere:WebSphere Application Server是功能完善、开放的Web应用程序服务器，是IMB电子商务计划的核心部分它是基于Java的应用环境用于建立、部署和 处理Internet和Intranet Web应用程序适应各种Web应用程序服务器的需要。 
  - WebLogic : WebLogic Server是一款多功能、基于标准 的Web应用服务器，为企业构建企业应用提供了坚实的 基础。针对各种应用开发、关键性任务的部署，各种系统性数据库的集成、跨Internet协作等Weblogic都提供了相应的支持。由于它具有全面的功能、对开放标准的遵从性、多层架构、支持基于组件的开发等优势，很多公司的企业的应用都选择它来作为开发和部署的环境。WebLogic Server在使应用服务器成为企业应用架构的基础方面一直处于领先地位为构建集成化的企业级应用提供了稳固的基础。
  - Apache：目前Apache仍然是世界上用得最多的Web服 器，其市场占有率很长时间都保持在60％以上（目前的 场份额约40％左右）。世界上很多著名的网站都是Apache 的产物，它的成功之处主要在于它的源代码开放、有一支强大的开发团队、支持跨平台的应用（可以运行在几乎所有的Unix, Windows, Linux系统平台上）以及它的可移动性等方面。
  - Tomcat:Tomcat是一个开放源代码、运行Servlet和JSP的WEB容器。Tomcat实现了Servlet和JSP规范。此外Tomc at还实现了Apache-Jakarta规范而且比绝大多数商业应用软件服务器要好，因此目前也有不少的Web服务器都选择了Tomcat。
  - Nginx：读作 'engine x'，是一个高性能的HTTP和反向代理服务器也是一个IMAP/POP3/SMTP代理服务器。Nginx是由Igor Sysoev 俄罗斯访问量第二的Rambler站点开发的，第一个公开版本0.1.0发布于2004年10月4日。其将源代码以类BSD许可的形式发布，因它的稳定性、丰富的功能集、示例配置组件和低系统资源的消耗而闻名。在2014年下半年Nginx: 市场份额达到了14%。 

## 8 . 过滤器的理解

- JavaWEB有三大组件 ： Servlet、Filter和Listener。Filter就是其中之一。

- 过滤器可以动态地拦截请求和响应，以变换或使用包含在请求或响应中的信息。

- 可以将一个或多个过滤器附加到一个 Servlet 或一组 Servlet。过滤器也可以附加到 JavaServer Pages (JSP) 文

  件和 HTML 页面。调用 Servlet 或页面前都会先执行过滤器。

- Servlet 过滤器是可用于 Servlet 编程的 Java 类，可以实现以下目的：

  - 在客户端的请求访问后端资源之前，拦截这些请求。
  - 在服务器的响应发送回客户端之前，处理这些响应。

## 10. Ajax简介

- AJAX = Asynchronous JavaScript and XML（异步 JavaScript 和 XML）。

- Ajax 的原理简单来说通过 XmlHttpRequest 对象来向服务器发异步请求，从服务器获得数据，然后用 Javascript 来操作 DOM 而更新页面。这其中最关键的一步就是从服务器获得请求数据。

- XmlHttpRequest 是 ajax 的核心机制，它是在 IE5 中首先引入的，是一种支持异步请求的技术。简单的说，也就是 Javascript 可以及时向服务器提出请求和处理响应，而不阻塞用户。达到无刷新的效果。

- 通过在后台与服务器进行少量数据交换，Ajax 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

- 传统的网页（不使用 Ajax）如果需要更新内容，必须重载整个网页页面。

## 11. http响应码301和302代表的是什么？有什么区别？

- 官方的比较简洁的说明：
  - 301 代表永久性转移(Permanently Moved)
  - 302 代表暂时性转移(Temporarily Moved )

- 共同点：
  - 301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）
- 不同点
  - 301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），**[搜索引擎](http://lib.csdn.net/base/searchengine)在抓取新内容的同时也将旧的网址交换为重定向之后的网址**；302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，**搜索引擎会抓取新的内容而保存旧的网址。**

## 12. web.xml文件中可以配置哪些内容？

web.xml 用于配置 Web 应用的相关信息，如：监听器（listener）、过滤器（filter）、 Servlet、相关参数、会话超时时间、安全验证方式、错误页面等。

## 13. 怎么防止表单重复提交？

- 禁掉提交按钮。表单提交后使用 Javascript 使提交按钮 disable。

- Post/Redirect/Get 模式。在提交后执行页面重定向，这就是所谓的 Post-Redirect-Get (PRG) 模式。简言之，当用户提交了表单后，你去执行一个客户端的重定向，转到提交成功信息页面。

- 在 session 中存放一个特殊标志。当表单页面被请求时，生成一个特殊的字符标志串，存在 session 中，同时放在表单的隐藏域里。接受处理表单数据时，检查标识字串是否存在，并立即从 session 中删除它，然后正常处理数据。

## 14. cookie简述

- 在 web 程序中是使用 HTTP 协议来传输数据的，因为 http 是无状态协议，一旦数据交换完毕，客户端和服务器端的连接就会关闭，再次交换数据需要建立新的连接，所以无法实现会话跟踪，cookie 技术则弥补了这一缺陷。
- cookie 实际上一段的文本信息，客户端请求服务器。如果服务器需要记录该用户的状态，就使用 response 向客户端浏览器颁发一个 cookie。客户端浏览器会把 cookie 保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该 cookie 一同提交给服务器。服务器检查该 cookie，以此来辨认用户的状态。服务器还可以根据需要修改 cookie 的内容。
- cookie 生命周期:
  cookie 的 maxAge 决定 cookie 的生命周期，单位为秒（second）。cookie 通过 getMaxAge() 方法和 setMaxAge()方法来获得 maxAge 属性，如果 maxAhe 属性为正，则表示 cookie 会在 maxAge 秒之后自动失效。如果 maxAge 属性为负，则说明 cookie 仅在本浏览器窗口和本窗口打开的子窗口下有效，关闭窗口 cookie 则失效。maxAge 的默认值是-1 当 maxAge 的值为 0 时，表示删除 cookie。

## 15. cookie和session的区别

- session:
  1. session机制采用的是在服务器端保持状态的方案
  2. session会在一定时间内保存在服务器上。当访问增多,会比较占用你服务器的性能
  3. 考虑到减轻服务器性能方面,应当使用COOKIE

- cookie:
  1. cookie机制采用的是在客户端保持状态的方案
  2. cookie不是很安全,别人可以分析存放在本地的COOKIE并进行COOKIE欺骗
  3. 考虑到安全应当使用session

## 16. 监听器有哪些方法和作用

- JavaWEB有三大组件 ： Servlet、Filter和Listener。Listener就是其中之一。

- 三个分类

  - 监听对象自身创建和销毁的监听器 （不常用 ）
    - ServletContextListener       Initialized:容器开始创建， Destroyed： 容器关闭时消亡
    - HttpSessionListener           Initialized:session创建时创建， Destroyed： session失效时消亡
    - ServletRequestListener      Initialized:request创建时创建， Destroyed： 页面访问结束时消亡
  - 监听对象中属性的增加和删除的监听器
    - ServletContextAttributeListener 	 attributeReplaced  attributeAdded   attributeRemoved
    - HttpSessionAttributeListener
    - ServletRequestAttributeListener   
  - 监听Httpsession状态的事件监听器  ,不需要在xml中配置，要在JavaBean上实现此接口。（不常用 ）
    - HttpSessionActivationListener    sessionDidActivate，sessionWillPassivate ，需要在META-INF中配置
    - HttpSessionBindingListener   valueBound    valueUnbound

- 总8个监听接口

  | Listener接口                    | Event类                      |
  | ------------------------------- | ---------------------------- |
  | ServletContextListener          | ServletContextEvent          |
  | ServletContextAttributeListener | ServletContextAttributeEvent |
  | HttpSessionListener             | HttpSessionEvent             |
  | HttpSessionActivationListener   |                              |
  | HttpSessionAttributeListener    | HttpSessionBindingEvent      |
  | HttpSessionBindingListener      |                              |
  | ServletRequestListener          | ServletRequestEvent          |
  | ServletRequestAttributeListener | ServletRequestAttributeEvent |


## 17. 什么是跨域，如何实现跨域

- 跨域说明：当浏览器执行脚本时会检查是否同源，只有同源的脚本才会执行，如果不同源即为跨域。
  - 这里的同源指访问的协议、域名、端口都相同。
  - 同源策略是由 Netscape 提出的著名安全策略，是浏览器最核心、基本的安全功能，它限制了一个源中加载脚本与来自其他源中资源的交互方式。
  - Ajax 发起的跨域 HTTP 请求，结果被浏览器拦截，同时 Ajax 请求不能携带与本网站不同源的 Cookie。
  - `<script> <img> <iframe> <link> <video> <audio>` 等带有 src 属性的标签可以从不同的域加载和执行资源。
- 实现跨域方式如下
  - 1、jsonp
    利用了 script 不受同源策略的限制
    缺点：只能 get 方式，易受到 XSS攻击
  - 2、CORS（Cross-Origin Resource Sharing）,跨域资源共享
    当使用XMLHttpRequest发送请求时，如果浏览器发现违反了同源策略就会自动加上一个请求头 origin；
    后端在接受到请求后确定响应后会在后端在接受到请求后确定响应后会在 Response Headers 中加入一个属性 Access-Control-Allow-Origin；
    浏览器判断响应中的 Access-Control-Allow-Origin 值是否和当前的地址相同，匹配成功后才继续响应处理，否则报错
    缺点：忽略 cookie，浏览器版本有一定要求
  - 3、代理跨域请求
    前端向发送请求，经过代理，请求需要的服务器资源
    缺点：需要额外的代理服务器
  - 4、Html5 postMessage 方法
    允许来自不同源的脚本采用异步方式进行有限的通信，可以实现跨文本、多窗口、跨域消息传递
    缺点：浏览器版本要求，部分浏览器要配置放开跨域限制
  - 5、6、基于 Html5 websocket 协议
    websocket 是 Html5 一种新的协议，基于该协议可以做到浏览器与服务器全双工通信，允许跨域请求
    缺点：浏览器一定版本要求，服务器需要支持 websocket 协议

## 18. Jquery简介

- JQuery 是一个 JavaScript 库。功能包括 HTML 元素选取和操作、CSS 操作、HTML 事件函数、 JavaScript 特效和动画、HTML DOM 遍历和修改、AJAX 和 Utilities。除此之外，JQuery 还提供了大量插件。
- 重点三个方向
  - 基础语法： $(selector).action()。
  - 选择器：主要分四大选择器，分别是基本选择器、层次选择器、过滤选择器、属性过滤选择器。
  - 事件：例如 click()、dblclick()、mouseenter()、mouseleave()、mousedown()等。

## 19. session简介

- session 也是一种记录客户状态的机制，不同的是 cookie 保存在客户端浏览器中，而 session 保存在服务器上。客户端浏览器访问服务器是时候把客户端信息以某种形式记录在服务器上，这就是 session 中查找该客户的状态。

- session 生命周期：
  - session 保存在服务器端，为了获得更高的存取速度，服务器一般把 session 放在内存。每个用户都会有一个独立的 session,如果 session 内容过于复杂，当大量客户访问服务器时可能会导致内存溢出。
  - session 在用户第一次访问服务器的时候自动创建，需要注意只有访问 JSP，Servlet 等程序时才会创建 session；只要访问 HTML、IMAGE 等静态资源不会创建 session。如果尚未生成session,可以使用 request.getSession(true)强制生成 session。
  - session 生成后，只要用户访问，服务器就会更新 session 的最后访问时间，并维护该 session。用户每访问服务器一次，无论是否续写 session 服务器都认为该用户的 session 活跃（active）了一次。
  - Session 对应的类是 javax.servlet.http.HttpSession，每一个访问者都对应一个 session 对象，并将其状态信息保存在这个 session 对象中，session 对象的创建是在用户第一次访问服务器时产生的。

## 20. JSP和Servlet是什么关系？

- Servlet是一个特殊的Java程序，它运行于服务器的JVM中，能够依靠服务器的支持向浏览器提供显示内容。
  JSP本质上是Servlet的一种简易形式，JSP会被服务器处理成一个类似于Servlet的Java程序，可以简化页面内容的生成。
- Servlet和JSP最主要的不同点在于，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML分离开来。而JSP的情况是Java和HTML可以组合成一个扩展名为.jsp的文件。有人说，Servlet就是在Java中写HTML，而JSP就是在HTML中写Java代码，当然这个说法是很片面且不够准确的。
  JSP侧重于视图，Servlet更侧重于控制逻辑，在MVC架构模式中，JSP适合充当视图（view）而Servlet适合充当控制器（controller）。

## 21. JSP和Servlet是什么关系？

- Servlet是一个特殊的Java程序，它运行于服务器的JVM中，能够依靠服务器的支持向浏览器提供显示内容。
  JSP本质上是Servlet的一种简易形式，JSP会被服务器处理成一个类似于Servlet的Java程序，可以简化页面内容的生成。
- Servlet和JSP最主要的不同点在于，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML分离开来。而JSP的情况是Java和HTML可以组合成一个扩展名为.jsp的文件。
- 给人感觉，Servlet就是在Java中写HTML，而JSP就是在HTML中写Java代码，当然这个说法是很片面且不够准确的。JSP侧重于视图，Servlet更侧重于控制逻辑，在MVC架构模式中，JSP适合充当视图（view）而Servlet适合充当控制器（controller）。

## 22. 过滤器的生命周期

- 创建，随着WEB容器的启动而启动，这一点与Servlet有区别

  void init(FilterConfig)

- 过滤 ： 每次请求时都会执行

  doFilter（ServletRequest request, ServletResponse response, FilterChain chain）

- 消亡：随着WEB容器的结束而结束 

  destroy( )