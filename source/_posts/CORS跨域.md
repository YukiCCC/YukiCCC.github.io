---
title: CORS跨域
date: 2023-10-22 12:51:57
tags: CORS
---
## 为什么会有跨域的问题？
为了保证用户信息的安全，所有的浏览器都遵循同源策略，那什么情况下算同源呢？同源策略又是什么呢？ 记住：协议、域名、端口号完全相同时，才是同源

可以参考 Web安全 - 浏览器的同源策略

在同源策略下，会有以下限制：

无法获取非同源的 Cookie、LocalStorage、SessionStorage 等
无法获取非同源的 dom
无法向非同源的服务器发送 ajax 请求
但是我们又经常会遇到前后端分离，不在同一个域名下，需要ajax请求数据的情况。那我们就要规避这种限制。 可以在网上搜到很多解决跨域的方法，有些方法比较古老了，现在项目中用的比较多的是 jsonp 和 CORS（跨域资源共享），这篇主要讲 CORS 的原理和具体实践。
## CORS 跨域原理
CORS 跨域的原理实际上是浏览器与服务器通过一些 HTTP 协议头来做一些约定和限制。可以查看 HTTP-访问控制（CORS）
### 与跨域相关的协议头
| 请求头      | 说明 |
| ----------- | ----------- |
| Origin      | 表明预检请求或实际请求的源站URL，不管是否跨域ORIGIN字段总是被发送       |
| Access-Control-Request-Method   | 将实际请求所使用的HTTP方法告诉服务器       |
| Access-Control-Request-Headers   | 将实际请求所携带的首部字段告诉服务器       |
| 响应头      | 说明 |
| Access-Control-Allow-Origin | 指定允许访问该资源的外域URI，对于携带身份凭证的请求不可使用通配符 |
| Access-Control-Expose-Headers | 指定XMLHttpRequest的getResponseHeader可以访问的响应头 |
| Access-Control-Max-Age | 指定preflight请求的结果能够被缓存多久 |
| Access-Control-Allow-Credentials | 是否允许浏览器读取response的内容；当 |
| Access-Control-Allow-Methods | 指定实际请求所允许使用的HTTP方法 |
| Access-Control-Allow-Headers | 指定实际请求中允许携带的首部字段 |

## 代码实例
这里写了个 demo，一步步来分析。目录如下：
```
.
├── README.md
├── client
│   ├── index.html
│   └── request.js
└── server
    ├── pom.xml
    ├── server-web
    │   ├── pom.xml
    │   ├── server-web.iml
    │   └── src
    │       └── main
    │           ├── java
    │           │   └── com
    │           │       └── example
    │           │           └── cors
    │           │               ├── constant
    │           │               │   └── Constants.java
    │           │               ├── controller
    │           │               │   └── CorsController.java
    │           │               └── filter
    │           │                   └── CrossDomainFilter.java
    │           ├── resources
    │           │   └── config
    │           │       └── applicationContext-core.xml
    │           └── webapp
    │               ├── WEB-INF
    │               │   ├── dispatcher-servlet.xml
    │               │   └── web.xml
    │               └── index.jsp
    └── server.iml
```
### Client：前端，简单的ajax请求
在client文件夹下，启动静态服务器，前端页面通过http://localhost:8000/index.html访问：
```
anywhere -h localhost -p 8000
```
### Server： java项目，SpringMVC
在 IntelliJ IDEA 中本地启动 tomcat，设置host: http://localhost:8080/，服务端数据通过http://localhost:8080/server/cors请求。

这里前端和后端因为**端口号不同**，存在跨域限制，下面通过 CORS 来解决因为跨域无法通过ajax请求数据的问题。

### 没有允许跨域的情况

这种情况就是前端什么都不做，服务端也什么都不做。

Client: 请求成功后，将数据显示在页面上

```js
new Request().send('http://localhost:8080/server/cors',{
    success: function(data){
        document.write(data)
    }
});
```

Server:

```java
@Controller
@RequestMapping("/server")
public class CorsController {
    @RequestMapping(value="/cors", method= RequestMethod.GET)
    @ResponseBody
    public String ajaxCors(HttpServletRequest request) throws Exception{
        return "SUCCESS";
    }
}
```

在浏览器地址栏输入`http://localhost:8080/server/cors`直接请求服务端，可以看到返回结果： ‘SUCCESS’

![img](https://pic4.zhimg.com/80/v2-fcb8bd47b373e10e570eaa1f67f4857f_1440w.webp)

在浏览器地址栏输入`http://localhost:8000/index.html`，从不同域的网页中向 Server 发送 ajax 请求。可以看到几个方面：

从 network 可以看到，请求返回正常。

![img](https://pic3.zhimg.com/80/v2-ff3aa966990c73866f8b005240fc821a_1440w.webp)



但 Response 中没有内容，显示 `Failed to load response data`。

![img](https://pic4.zhimg.com/80/v2-2b037a038afb4a4bb35756bad1432a2b_1440w.webp)

并且控制台报错：

![img](https://pic3.zhimg.com/80/v2-a35e89c0b079750c11e70a93c59ec1a2_1440w.webp)



**总结：**

1. 浏览器请求是发出去了的，服务端也会正确返回，但是我们拿不到response的内容
2. 浏览器控制台会报错提示可以怎么做，而且提示的很明白： xhr不能请求`http://localhost:8080/server/cors`，请求资源的响应头中没有设置Access-Control-Allow-Origin，Origin：`http://localhost:8000`是不允许跨域请求的。

那下一步，我们要在服务端响应跨域请求时，设置响应头: Access-Control-Allow-Origin

### 设置 Access-Control-Allow-Origin 允许跨域

先说明为什么要设置 Access-Control-Allow-Origin，可以把 Access-Control-Allow-Origin 当作一个指令，服务端设置 Access-Control-Allow-Origin 就是告诉浏览器允许向服务端请求资源的域名，浏览器通过 Response 中的 Access-Control-Allow-Origin 就可以知道能不能把数据吐出来。

官方解释是这样的： Access-Control-Allow-Origin 响应头指定了该响应的资源是否被允许与给定的 origin 共享。

Access-Control-Allow-Origin可以设置的值有：

```text
Access-Control-Allow-Origin: *
Access-Control-Allow-Origin:
```

那在java服务端给响应头设置 Access-Control-Allow-Origin 可以这么做：

1、添加一个过滤器

```java
public class CrossDomainFilter implements Filter{
    public void init(FilterConfig filterConfig) throws ServletException {}
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse resp = (HttpServletResponse)servletResponse;
        resp.setHeader("Access-Control-Allow-Origin", "http://localhost:8000");
        filterChain.doFilter(servletRequest,servletResponse);
    }
    public void destroy() {}
}
```

2、然后在web.xml文件中添加过滤器配置：

```xml
<filter>
     <filter-name>crossDomainFilter</filter-name>
     <filter-class>com.example.cors.filter.CrossDomainFilter</filter-class>
</filter>
<filter-mapping>
      <filter-name>crossDomainFilter</filter-name>
      <url-pattern>/*</url-pattern>
</filter-mapping>
```

3、然后重新启动tomcat，client重新发送请求`http://localhost:8000/index.html`

![img](https://pic3.zhimg.com/80/v2-54a9185e158b88abb17403933fdda4ee_1440w.webp)

![img](https://pic4.zhimg.com/80/v2-5383d4b88c843f9111b9aa115a8b038b_1440w.webp)

可以看到我们能够拿到返回结果了，响应头中有我们在服务端设置的Access-Control-Allow-Origin: `http://localhost:8000`，这个应该跟请求头中的origin一致，或者设置Access-Control-Allow-Origin:*也是可以的，这就允许任何网站来访问资源了（前提是不带凭证信息，这个后面讲）

**以上就是允许一个简单的跨域请求的做法，只需要服务端设置响应头Access-Control-Allow-Origin。**

### 简单请求与预检请求

上面讲述了一个简单请求通过在服务端设置响应头 Access-Control-Allow-Origin 就可以完成跨域请求。

那怎样的请求算是一个简单请求？与简单请求相对应的是什么样的请求呢？解决跨域的方式又有什么不一样呢？

符合以下条件的可视为简单请求：

1、使用下列 HTTP 方法之一

```text
- GET
- HEAD
- POST，并且Content-Type的值在下列之一：
  - text/plain
  - multipart/form-data
  - application/x-www-form-urlencoded
```

2、并且请求头中只有下面这些

```text
- Accept
- Accept-Language
- Content-Language
- Content-Type （需要注意额外的限制）
- DPR
- Downlink
- Save-Data
- Viewport-Width
- Width
```

不满足上述要求的在发送正式请求前都要先发送一个预检请求，预检请求以 OPTIONS 方法发送，浏览器通过请求方法和请求头能够判断是否发送预检请求。

比如 Client 发送如下请求：

```js
new Request().send('http://localhost:8080/server/options',{
    method: 'POST',
    header: {
        'Content-Type': 'application/json'  //告诉服务器实际发送的数据类型
    },
    success: function(data){
        document.write(data)
    }
});
```

Server 端处理请求的 controller:

```java
@Controller
@RequestMapping("/server")
public class CorsController {
    @RequestMapping(value="/options", method= RequestMethod.POST)
    @ResponseBody
    public String options(HttpServletRequest request) throws Exception{
        return "SUCCESS";
    }
}
```

因为请求时，请求头中塞入了 header，'Content-Type': 'application/json'。根据前面讲述的可以知道，浏览器会以 OPTIONS 方法发出一个预检请求，浏览器会在请求头中加入：

```text
Access-Control-Request-Headers:content-type
Access-Control-Request-Method:POST
```

这个预检请求的作用在这里就是告诉服务器：我会在后面请求的请求头中以 POST 方法发送数据类型是application/json 的请求，询问服务器是否允许。

![img](https://pic3.zhimg.com/80/v2-722e7d35c49b45e7043db9cf4cce1a56_1440w.webp)



在这里服务器还没有做任何允许这种请求的设置，所以浏览器控制台报错：

![img](https://pic1.zhimg.com/80/v2-8d53d75c41bda244e2a67f5a833e3514_1440w.webp)

也清楚的说明了出错的原因： **服务端在预检请求的响应中没有告诉浏览器允许协议头 Content-Type**，即服务端需要设置响应头 Access-Control-Allow-Headers，允许浏览器发送带 Content-Type 的请求。

Server端过滤器中添加Access-Control-Allow-Headers：

```java
public class CrossDomainFilter implements Filter{
    public void init(FilterConfig filterConfig) throws ServletException {}
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse resp = (HttpServletResponse)servletResponse;
        resp.setHeader("Access-Control-Allow-Origin", "http://localhost:8000");
        resp.setHeader("Access-Control-Allow-Headers", "Content-Type");
        filterChain.doFilter(servletRequest,servletResponse);
    }
    public void destroy() {}
}
```

可以看到请求成功

![img](https://pic1.zhimg.com/80/v2-3bc3393966990f9a0d7ababc283c6dd0_1440w.webp)

再来看请求的具体信息，第一次以 OPTIONS 方法发送预检请求，浏览器设置请求头：

```text
Access-Control-Request-Headers:content-type //请求中加入的请求头
Access-Control-Request-Method:POST  //跨域请求的方法
```

服务端设置响应头：

```text
Access-Control-Allow-Headers:Content-Type   //允许的header
Access-Control-Allow-Origin:http://localhost:8000 //允许跨域的源
```

![img](https://pic3.zhimg.com/80/v2-552bf447a5db176b5aa3289366ca17ba_1440w.webp)

也可以设置 Access-Control-Allow-Methods 来限制客户端的的请求方法。

这样预检请求成功了，浏览器会发出第二个请求，这是真正请求数据的请求：

![img](https://pic2.zhimg.com/80/v2-f7c0f8009d15db3faaca6ed390405ef5_1440w.webp)

可以看到 POST 请求成功了，第二次请求头中没有设置 Access-Control-Request-Headers 和 Access-Control-Request-Method。

**但是这里有个问题**，需要预检请求时，浏览器会发出两次请求，一次 OPTIONS，一次 POST。两次都返回了数据。这样服务端如果逻辑复杂一些，比如去数据库查找数据，从 web 层、 service 到数据库这段逻辑就会走两遍，浏览器会两次拿到相同的数据，所以服务端的 filter 可以改一下，如果是 OPTIONS 请求，在设置完跨域请求响应头后就不走后面的逻辑直接返回。

```java
public class CrossDomainFilter implements Filter{
    public void init(FilterConfig filterConfig) throws ServletException {}
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse resp = (HttpServletResponse)servletResponse;
        resp.setHeader("Access-Control-Allow-Origin", "http://localhost:8000");
        resp.setHeader("Access-Control-Allow-Headers", "Content-Type");   
        //OPTION请求就直接返回
        HttpServletRequest req = (HttpServletRequest) servletRequest;
        if (req.getMethod().equals("OPTIONS")) {
            resp.setStatus(200);
            resp.flushBuffer();
        }else {
            filterChain.doFilter(servletRequest,servletResponse);
        }
    }
    public void destroy() {}
}
```

**总结：** **

1. 对于POST请求设置响应头Content-Type为某些值、自定义请求头等情况，浏览器会先以OPTIONS方法发送一个预检请求，并设置相应的请求头。
2. 服务端还是正常返回，但如果预检请求响应头中不设置相应的响应头，预检请求不通过，不会再发出第二次请求来获取数据。
3. 服务端设置相应的响应头，浏览器会发出第二个请求，并将服务端返回的数据吐出，我们可以获得response的内容

### 带凭证信息的请求

还有一种情况我们经常遇到。浏览器在发送请求时需要给服务端发送 cookie，服务端根据 cookie 中的信息做一些身份验证等。

默认情况下，浏览器向不同域的发送 ajax 请求，不会携带发送 cookie 信息。

Client:

```js
var containerElem = document.getElementById('container')
new Request().send('http://localhost:8080/server/testCookie',{
    success: function(data){
        containerElem.innerHTML = data
    }
});
```

Server:

```java
@RequestMapping(value="/testCookie", method= RequestMethod.GET)
@ResponseBody
public String testCookie(HttpServletRequest request,HttpServletResponse response) throws Exception{
    String str = "SUCCESS";
    Cookie[] cookies = request.getCookies();
    String school = getSchool(cookies);
    if(school == null || school.length() == 0){
        addCookie(response);
    }
    return str + buildText(cookies);
}
```

服务端收到请求，判断 cookie 中有没有 school，没有就添加 cookie.



![img](https://pic3.zhimg.com/80/v2-eec242cda522cb7ceecb3bbae0ea07f6_1440w.webp)



可以看到响应头中有 Set-Cookie，再次请求时，如果是同源请求，浏览器会将 Set-Cookie 中的值放在请求头中，但是**对于跨域请求，默认是不发送这个 Cookie 的。**

如果要让浏览器发送 cookie，需要在 Client 设置 XMLHttpRequest 的 withCredentials 属性为 true。

Client:

```js
var containerElem = document.getElementById('container')
new Request().send('http://localhost:8080/server/testCookie',{
    withCredentials: true,
    success: function(data){
        containerElem.innerHTML = data
    }
});
```

现在浏览器在请求头中加入了 cookie 信息



![img](https://pic2.zhimg.com/80/v2-7f7a6f4d29e065b4fc799c0023ea0ca5_1440w.webp)



但是服务端返回的数据没有在页面中展示，并且报错：



![img](https://pic2.zhimg.com/80/v2-d85b4ec50ae4e2d688706e70984cb3f5_1440w.webp)



报错信息很明白： **当请求中包含凭证信息时，需要设置响应头 Access-Control-Allow-Credentials，是否带凭证信息是由 XMLHttpRequest的withCredentials 属性控制的。** ** 所以我们在 Server 端 filter 中加入这个响应头：

```java
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse resp = (HttpServletResponse)servletResponse;
        resp.setHeader("Access-Control-Allow-Origin", "http://localhost:8000");
        resp.setHeader("Access-Control-Allow-Credentials","true");
        HttpServletRequest req = (HttpServletRequest) servletRequest;
        if (req.getMethod().equals("OPTIONS")) {
            resp.setStatus(200);
            resp.flushBuffer();
        }else {
            filterChain.doFilter(servletRequest,servletResponse);
        }
    }
```

现在浏览器知道响应头中 Access-Control-Allow-Credentials 为 true，就会把数据给吐出来了，我们能够从response 中拿到内容了。



![img](https://pic4.zhimg.com/80/v2-8cc119cc07473d426a1cfda43cec0c8b_1440w.webp)





![img](https://pic1.zhimg.com/80/v2-2039078544df47e4972ca16871d980ac_1440w.webp)



那如果附带凭证信息并且有预检请求呢？如果有预检请求，并附带凭证信息（ XMLHttpRequest 的withCredentials 设置为 true）， 服务端需要设置 Access-Control-Allow-Credentials: true，否则浏览器不会发出第二次请求，并报错。



![img](https://pic1.zhimg.com/80/v2-cce27b356ddbf9cdbab5d6ed3b9fa1e4_1440w.webp)



**总结：**

1. 跨域请求时，浏览器默认不会发送cookie，需要设置XMLHttpRequest的withCredentials属性为true
2. 浏览器设置XMLHttpRequest的withCredentials属性为true，表明要向服务端发送凭证信息(这里是cookie)。那么服务端就需要在响应头中添加Access-Control-Allow-Credentials为true。否则浏览器上有两种情况：
3. 如果是简单请求，服务端结果吐出了，浏览器拿到了但就是不给吐出来，并报错。
4. 如果是预检请求，同样我们拿不到返回结果，并报错提示预检请求不通过，不会再发第二次请求。

## 其他

### cookie 的同源策略

另外就是设置了 XMLHttpRequest 的 withCredentials 属性为 true，浏览器发出去了，服务端还是拿不到 cookie的问题。

cookie 也遵循同源策略的，在设置 cookie 的时候可以发现除了键值对，还可以设置 cookie 的这些值：

![img](https://pic3.zhimg.com/80/v2-dd83f5aed3d6428179edf871711f5fe6_1440w.webp)

如果获取不到 cookie，可以检查下 cookie 的 domain 和 path.

### IE 上跨域访问没有权限

在跨域发送 ajax 请求时提示没有权限。 因为IE浏览器默认对跨域访问有限制。需要在浏览器设置中去除限制。 **方法：** 设置 > Internet 选项 > 安全 > 自定义级别 > 在设置中找到其他 - 在【其他】中将【通过域访问数据源】启用。

## Demo 源码

[CORS Demo 源码](https://link.zhihu.com/?target=https%3A//github.com/chang20159/daily-example/tree/master/CORS)