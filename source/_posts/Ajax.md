---
title: Ajax学习
date: 2023-07-13 18:33:25
tags: Ajax
category: 前端
---

## 1.接口文档

**概念及说明**

我们使用的 url，数据接口，或者简称为接口。

接口是服务器提供的一个url地址，通过这个url地址，我们可以操作服务器上的资源。

通过Ajax技术向一个接口发送请求，也叫做调用接口。

- 接口是谁设计的呢
  - **后端**设计的（学java的、学php的同学、.....）
  - 后端设计完接口之后，会提供一个接口文档给前端开发工程师
- 一个好的接口文档，至少需要包含下面几项内容
  - 接口说明（通过接口说明，大致了解到接口是干什么用的）
  - 接口的url（发送ajax请求，必要的条件）
  - 接口请求方式（发送ajax请求，必要的条件）
  - 请求参数（参数名称、数据类型、是否必填、参数说明等）
  - 响应格式
  - 响应示例

**请求的根路径**

> http://localhost:8080/day19-02-ajax

**图书管理**

### 1.1 图书列表

+ 接口URL：  /api/getbooks
+ 调用方式： GET
+ 参数格式：

| 参数名称      | 参数类型   | 是否必选 | 参数说明 |
| --------- | ------ | ---- | ---- |
| id        | String | 否    | 图书Id |
| bookname  | String | 否    | 图书名称 |
| author    | String | 否    | 作者   |
| publisher | String | 否    | 出版社  |

+ 响应格式：

| 数据名称       | 数据类型   | 说明               |
| ---------- | ------ | ---------------- |
| status     | Number | 200 成功；500 失败；   |
| msg        | String | 对 status 字段的详细说明 |
| data       | Array  | 图书列表             |
| +id        | String | 图书Id             |
| +bookname  | String | 图书名称             |
| +author    | String | 作者               |
| +publisher | String | 出版社              |

+ 返回示例：

```json
{
  "status": 200,
  "msg": "获取图书列表成功",
  "data": [
    { "id": "B001", "bookname": "西游记", "author": "吴承恩", "publisher": "北京图书出版社" },
    { "id": "B002", "bookname": "红楼梦", "author": "曹雪芹", "publisher": "上海图书出版社" },
    { "id": "B003", "bookname": "三国演义", "author": "罗贯中", "publisher": "北京图书出版社" }
  ]
}
```



### 1.2 添加图书

+ 接口URL：  /api/addbook
+ 调用方式： POST
+ 参数格式：

| 参数名称      | 参数类型   | 是否必选 | 参数说明 |
| --------- | ------ | ---- | ---- |
| bookname  | String | 是    | 图书名称 |
| author    | String | 是    | 作者   |
| publisher | String | 是    | 出版社  |

+ 响应格式：

| 数据名称   | 数据类型   | 说明                 |
| ------ | ------ | ------------------ |
| status | Number | 201 添加成功；500 添加失败； |
| msg    | String | 对 status 字段的详细说明   |

+ 返回示例：

```json
{
    "status": 201,
    "msg": "添加图书成功"
}
```





### 1.3 删除图书

+ 接口URL：  /api/delbook
+ 调用方式： GET
+ 参数格式：

| 参数名称 | 参数类型   | 是否必选 | 参数说明 |
| ---- | ------ | ---- | ---- |
| id   | String | 是    | 图书Id |

+ 响应格式：

| 数据名称   | 数据类型   | 说明                                      |
| ------ | ------ | --------------------------------------- |
| status | Number | 200 删除成功；500 未指定要删除的图书Id；502 要删除的图书不存在； |
| msg    | String | 对 status 字段的详细说明                        |

+ 返回示例：

```json
{
    "status": 200,
    "msg": "删除图书成功！"
}
```

## 2.服务器端开发

### 2.1 vo封装返回结果

```java
/**
 * 封装 ajax 响应结果
 */
public class AjaxResult {
	/**响应状态码 */
	private Integer status ;
	/**响应消息*/
	private String msg ;
	/**响应数据   主要是查询接口返回的数据*/
	private Object data ;
	
	public AjaxResult(Integer status, String msg) {
		super();
		this.status = status;
		this.msg = msg;
	}

	public AjaxResult(Integer status, String msg, Object data) {
		super();
		this.status = status;
		this.msg = msg;
		this.data = data;
	}
}
```

### 2.2 pojo

```java
public class Book {
	private String id ;
	private String bookname ;
	private String author ;
	private String publisher ;
	public Book() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Book(String id, String bookname, String author, String publisher) {
		super();
		this.id = id;
		this.bookname = bookname;
		this.author = author;
		this.publisher = publisher;
	}
}  
```

### 2.3 Service

```java
public class BookService {
	private static Map<String,Book> bookMap = Collections.synchronizedMap(new LinkedHashMap<>()) ;
	static {
		bookMap.put("B001", new Book("B001","西游记","吴承恩","北京图书出版社")) ;
		bookMap.put("B002", new Book("B002","西红楼梦","曹雪芹","上海图书出版社")) ;
		bookMap.put("B003", new Book("B003","三国演义","罗贯中","北京图书出版社")) ;
	}
	public AjaxResult save(Book book) {
		//获得随机id
		String id = UUID.randomUUID().toString() ;
		//设置书籍id
		book.setId(id);
		bookMap.put(id, book);
		return new AjaxResult(201, "添加图书成功") ;
	}
	
	public AjaxResult remove(String id) {
		if(!bookMap.containsKey(id)) {
			return new AjaxResult(500, "502 要删除的图书不存在") ;
		}
		bookMap.remove(id) ;
		return new AjaxResult(200, "删除图书成功！") ;
	}
	
	public AjaxResult find(Book book) {
		//此次忽略 条件 查询
		return new AjaxResult(200, "获取图书列表成功",bookMap.values()) ;
	}
}
```

### 2.4 Controller

```java
@WebServlet({ "/api/getbooks", "/api/addbook", "/api/delbook" })
public class BookController extends HttpServlet {
	private static final long serialVersionUID = 1L;
	private BookService bookService = new BookService() ;

	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		request.setCharacterEncoding("UTF-8");
		response.setContentType("application/json;charset=UTF-8");
		String requestURI = request.getRequestURI() ;
		requestURI = requestURI.substring((request.getContextPath()+"/api/").length()) ;
		try {
			Method m = this.getClass().getDeclaredMethod(requestURI, HttpServletRequest.class,HttpServletResponse.class) ;
			AjaxResult ajaxResult = (AjaxResult)m.invoke(this, request,response) ;
			String json = JSON.toJSONString(ajaxResult) ;
			PrintWriter out = response.getWriter();
			out.print(json);
			out.flush();
			out.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	protected AjaxResult getbooks(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		if("POST".equals(request.getMethod())) {
			return new AjaxResult(405, "请求的方式不允许！") ;
		}
		return bookService.find(null) ;
	}
	
	protected AjaxResult addbook(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		if("GET".equals(request.getMethod())) {
			return new AjaxResult(405, "请求的方式不允许！") ;
		}
		String bookname = request.getParameter("bookname") ;
		String author = request.getParameter("author") ;
		String publisher = request.getParameter("publisher") ;
		Book book = new Book() ;
		book.setBookname(bookname);
		book.setAuthor(author);
		book.setPublisher(publisher);
		return bookService.save(book) ;	
	}
	
	protected AjaxResult delbook(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		if("POST".equals(request.getMethod())) {
			return new AjaxResult(405, "请求的方式不允许！") ;
		}
		String id = request.getParameter("id") ;
		if(id==null || id.equals("")) {
			return new AjaxResult(500, "未指定要删除的图书Id") ;
		}
		return bookService.remove(id) ;
	}
}
```

## 3. 接口测试

**准备工作：**

```
 1. tomcat先启动
 2. 安装 postman【默认安装】
```

### 3.1 图书列表

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/list.png)

### 3.2 添加图书

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/add.png)

### 3.3 删除图书

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/del.png)

## 4. Ajax

### 4.1 浏览器/服务器交互模型

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/bs.png)

### 4.2 Ajax基本概念

```
HTML是骨架
CSS是颜值
JS是行为
数据是灵魂
```

AJAX是异步的JavaScript和XML（**A**synchronous **J**avaScript **A**nd **X**ML）。简单点说，就是使用浏览器内置对象 `XMLHttpRequest` 与服务器通信。 

**Ajax是一种技术，通过浏览器内置对象和服务器进行数据交互的技术**。

它可以使用JSON，XML，HTML和text文本等格式发送和接收数据。

AJAX最吸引人的就是它的“异步”特性，也就是说它可以在不重新刷新页面的情况下与服务器通信，交换数据，或更新页面。

你可以使用AJAX最主要的两个特性做下列事：

- 在不重新加载页面的情况下发送请求给服务器。
- 接受并使用从服务器发来的数据。

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/ajax.png)

### 4.3 Ajax请求和浏览器请求

```
浏览器请求
    是浏览器本能的请求
    不需要执行任何JS代码
    地址栏输入地址后，按下回车即可发送请求
    接收到服务器的响应后，浏览器自动渲染响应结果到页面中
Ajax请求
    是通过浏览器内置对象（XHR）完成的
    通过执行一段JS代码，才能实现的请求
    响应结果由程序员自行处理，浏览器只负责接收响应结果。

无论是浏览器请求，还是Ajax请求，道理都一样，都符合浏览器 – 服务器交互模型
```

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/ajax-req-resp.png)

```
Ajax 优点：
	可以在不重新刷新页面的情况下与服务器通信，交换数据，或更新页面。
      只更新页面的部分数据，节省带宽
      用户体验好
	异步，不阻塞
      不阻塞，Ajax 代码之后的 其他 JS 代码无需等待 Ajax 代码执行完毕，即可执行。
      多个Ajax请求可以同时进行。
      
Ajax 缺点:
    1、增加了设计和开发时间
    2、比构建经典Web应用程序更复杂
    3、AJAX应用程序中的安全性较低，因为所有文件都是在客户端下载的。
    4、可能出现网络延迟问题
    5、禁用JavaScript的浏览器无法使用该应用程序。
    6、由于安全限制，只能使用它来访问服务于初始页面的主机的信息。如果需要显示来自其他服务器的信息，则无法在AJAX中显示      
```

### 4.4 JQuery封装的ajax方法

**$.get()**

```
jQuery 中 $.get() 函数的功能单一，专门用来发起 get 请求，从而将服务器上的资源请求到客户端来进行使用。 
$.get() 函数的语法如下：
   $.get(url,[data],[success],[dataType])
```

| 参数       | 是否必填 | 类型                    | 说明                      |
| -------- | ---- | --------------------- | ----------------------- |
| url      | 是    | string                | 请求资源的接口地址，一般由后端开发工程师提供  |
| data     | 否    | string\|array\|object | 请求参数，由后端提供              |
| success  | 否    | function              | 请求成功后触发的回调函数，函数的形参为响应结果 |
| dataType | 否    | string                | 预期服务器端响应的数据的类型，一般不用设置   |

```js
//定义 请求接口地址  及 请求参数
let url = 'http://localhost:8080/day19-02-Ajax/api/getbooks'
let obj = {
    id: 1,
    bookname: '西游记',
} ;
//发送ajax  get 请求
$.get(
  url,
  obj,
  function (res) {
    console.log(res)
  }
)
```



**$.post()**
```
jQuery 中 $.post() 函数的功能单一，专门用来发起 post 请求，用于向服务器提交资源。 
$.post() 函数的语法如下： 
   $.post(url,[data],[success],[dataType])
```

| 参数       | 是否必填 | 类型                    | 说明                      |
| -------- | ---- | --------------------- | ----------------------- |
| url      | 是    | string                | 请求资源的接口地址，一般由后端开发工程师提供  |
| data     | 否    | string\|array\|object | 请求参数，由后端提供              |
| success  | 否    | function              | 请求成功后触发的回调函数，函数的形参为响应结果 |
| dataType | 否    | string                | 预期服务器端响应的数据的类型，一般不用设置   |

```js
// 定义路径和参数
let url = 'http://localhost:8080/day19-02-Ajax/api/addbook';
let obj = {
    bookname: '三体之地球往', 
    author: '刘慈欣', 
    publisher: '中央人民出版社'
};
// 发送post类型的ajax
$.post(url, obj, function (res) {
    console.log(res);
});
```



**$.ajax()**
```
$.ajax() - 一个综合的发送Ajax请求的方法，即可以发送GET方式的请求，也可以发送POST方式的请求，还可以根据需求配置更加复杂的Ajax请求，使用频率最高。
$.ajax({
  method:'GET|POST', // 请求方式   必填
  url:'url接口地址',  //请求的接口地址 必填
  data:'object|array|string', // 请求参数 ，可选
  success:function(res){ //请求成功后的回调函数
    res 表示响应结果
  }
})
```

```js
// $.ajax(); 传递的参数是对象;
//   四个关键参数都是一个属性形式存在！
$.ajax({
    method: 'GET',
    url: 'http://localhost:8080/day19-02-Ajax/api/getbooks',
    data: {
        id: 1
    },
    success: function (res) {
        console.log(res);
    }
})
```

#### 4.5 network工具使用

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/network.png)

network工具使用

- All   --  查看所有请求
- XHR -- 查看Ajax请求
- JS -- 查看请求了哪些JS文件
- CSS -- 查看请求了哪些CSS文件
- Img  -- 查看请求了哪些图片
- Media -- 查看请求了哪些音频、视频等
- Font -- 查看请求了哪些字体文件
- Doc -- document，查看请求了哪些html文件

### 4.6 综合案例

#### 效果图

![image-20210806085831519](https://raw.githubusercontent.com/YukiCCC/figure/main/ex.png)

#### 素材

```html
<!-- 套一个 .container 内容就有版心了 -->
<div class="container">
  <!-- bootstrap3 => 组件 => 面板 => 带标题的面版 => 情境效果 -->
  <div class="panel panel-primary">
    <div class="panel-heading">
      <h3 class="panel-title">添加图书</h3>
    </div>
    <div class="panel-body">
      <!-- bootstrap3 => 全局CSS样式 => 表单 => 内联表单 -->
      <div class="form-inline">
        <div class="form-group">
          <label class="sr-only" for="exampleInputAmount">书名</label>
          <div class="input-group">
            <div class="input-group-addon">书名</div>
            <input
              type="text"
              class="form-control"
              id="inpBookname"
              placeholder="请输入书名"
            />
            <!-- <div class="input-group-addon">.00</div> -->
          </div>
        </div>
        <div class="form-group">
          <label class="sr-only" for="exampleInputAmount">作者</label>
          <div class="input-group">
            <div class="input-group-addon">作者</div>
            <input
              type="text"
              class="form-control"
              id="inpAuthor"
              placeholder="请输入作者"
            />
            <!-- <div class="input-group-addon">.00</div> -->
          </div>
        </div>
        <div class="form-group">
          <label class="sr-only" for="exampleInputAmount">出版社</label>
          <div class="input-group">
            <div class="input-group-addon">出版社</div>
            <input
              type="text"
              class="form-control"
              id="inpPublisher"
              placeholder="请输入出版社"
            />
            <!-- <div class="input-group-addon">.00</div> -->
          </div>
        </div>
        <button id="btnAdd" type="submit" class="btn btn-primary">
          添加
        </button>
      </div>
    </div>
  </div>

  <!-- 注意: 第二部分是table标签，要写道 container 里面，panel下面 -->
  <!-- bootstrap3 => 全局CSS样式 => 表格 => 带边框的表格 -->
  <table class="table table-bordered table-hover">
    <thead>
      <tr>
        <th>Id</th>
        <th>书名</th>
        <th>作者</th>
        <th>出版社</th>
        <th>操作</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1</td>
        <td>西游记</td>
        <td>吴承恩</td>
        <td>北京图书出版社</td>
        <td><a href="javascript:;">删除</a></td>
      </tr>
    </tbody>
  </table>
</div>
```

## 5. CORS 错误
跨源资源分享（CORS）是一种允许服务器放宽同源策略的标准。这用于明确允许一些跨源请求，同时拒绝其他请求。例如，如果站点提供外界嵌入的服务，则可能需要放宽同源策略。设置这样的 CORS 配置并不一定容易，并且可能存在一些挑战。在这些页面中，我们将研究一些常见的 CORS 错误消息以及如何解决它们。

如果未正确设置 CORS 配置，浏览器控制台将显示错误，例如 "Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at $somesite" 表示请求因违反 CORS 安全规则而被阻止。但这可能不一定是设置错误。因为用户的 Web 应用程序和远程外部服务可能故意禁止该请求。如果要使端点可用，则需要进行一些调试才能成功。

### 什么是跨域问题
前端调用的后端接口不属于同一个域（域名或端口不同），就会产生跨域问题，也就是说你的应用访问了该应用域名或端口之外的域名或端口。

服务器端 开启跨域

```java
/**
 * 功能描述：解决跨域过滤器 
 */
@WebFilter("/api/*")
public class CORSFilter implements Filter {
	private final int time = 20*24*60*60;
	
	public void init(FilterConfig fConfig) throws ServletException {
		
	}
	
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		HttpServletResponse resp = (HttpServletResponse) response;
        // 添加参数，允许任意domain访问
        resp.setHeader("Access-Control-Allow-Origin", "*");
        // 这个allow-headers要配为*，这样才能允许所有的请求头
        resp.setHeader("Access-Control-Allow-Headers", "*");
        resp.setHeader("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
        resp.setHeader("Access-Control-Max-Age", time+"");
        chain.doFilter(request, resp);
	}

	
	public void destroy() {
		
	}

}
```
