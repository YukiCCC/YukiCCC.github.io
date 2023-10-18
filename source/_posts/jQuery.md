---
title: jQuery
date: 2023-07-13 10:52:36
tags: jQuery
category: 前端
---
# 1. JQ

## 1.1 jQ概述


jQuery 是一个快速、简洁的 JavaScript 库，其设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。
JavaScript库：是一个封装好的特定的集合包大量的方法。从封装函数的角度理解库，就是在JavaScrip库中，封装了很多预先定义好的函数在里面
JQuery就是这样的一个js文件: 本质上就是对我们之前原生js代码封装后的一个js文件
j 代表js  Query 代表查询

优点：
  轻量级。核心文件才几十kb，不会影响页面加载速度
  跨浏览器兼容。基本兼容了现在主流的浏览器
  链式编程、隐式迭代
  对事件、样式、动画支持，大大简化了DOM操作
  支持插件扩展开发。有着丰富的第三方的插件，例如：树形菜单、日期控件、轮播图等
  免费、开源


### 初体验

```html
<!-- 使用 原生js  与 JQ PK 点击事件 -->
 <ul>
      <li>data1</li>
      <li>data2</li>
      <li>data3</li>
      <li>data4</li>
      <li>data5</li>
      <li>data6</li>
      <li>data7</li>
      <li>data8</li>
    </ul>
    <script src="./jslib/jQuery.js"></script>
    <script>
      // 1. 隐式循环  2. 链式操作  3. $(dom对象) 转成 jq对象  调用 jq 的方法
      $("li").mouseover(e=>$(e.target).css('backgroundColor','pink')).mouseout(e=>$(e.target).css('backgroundColor',''))
    </script>

 <script>
      //原生
      let liCompAy = document.querySelectorAll('li')
      liCompAy.forEach((liComp) => {
        liComp.addEventListener('mouseover', (e) => {
          e.target.style.backgroundColor = `pink`
          console.log(`鼠标覆盖`)
        })

        liComp.addEventListener('mouseout', (e) => {
          e.target.style.backgroundColor = ``
          console.log(`鼠标离开`)
        })
      })
    </script>
```





### 入口函数

```
无需关心JS代码书写顺序
	DOM中如果在HTML结构之前写JS代码,需要设置加载事件,获取HTML元素 【加载事件 window.onload】
```

```html
<script>
    //原生
    window.addEventListener("load",()=>{
      let inputComp = document.querySelector("#name")
      console.log(`姓名 原生 :${inputComp.value}`);
    })
  </script>
  <script src="./jslib/jQuery.js"></script>
  <script>
     //JQ
    $(()=>console.log(`姓名JQ: ${$("#name").val()}`))
  </script>
</head>
<body>
  <input type="text" id="name" value="张三疯">
```



### DOM JQ转换

```
DOM对象: 
	通过 document 方式获取到的元素都叫DOM对象
jQuery对象
	通过 $ 方式获取到的元素都叫 JQ对象
将JQ对象转为DOM对象方式:
	JQ对象[索引]
	JQ对象.get(索引)
将DOM对象转化为JQ对象:
	$(dom对象)
注意： 只能由DOM对象 调用DOM的属性及方法，只能使用JQ对象调用 JQ的属性及方法
```

```html
<input type="text" id="name" value="张三疯">
  <script src="./jslib/jQuery.js"></script>
  <script>
    //dom对象
    // let inputComp = document.querySelector("#name")
    // .val() jq的方法 必须 jq对象调用   .value dom的属性  必须 dom对象调用
    // console.log(`姓名:${inputComp.value}`);

    // jq对象 使用$开头   dom对象 
    // let $input = $("#name")
    // console.log(`姓名: ${$input.val()}`);

    //dom 转 jq   let jq = $(dom)
    // let $input = $(inputComp)
    // console.log(`姓名: ${$input.val()}`);

    // jq 转 dom  [0]  .get(0)
    let $input = $("#name")
    // let input = $input[0]
    let input = $input.get(0)
    console.log(`姓名:${input.value}`);
  </script>
```



## 1.2 JQ 选择器

### 基本选择器

```
1. id选择器              #id值                $("#name")
2. class选择器           .class值             $(".a")
3. 标签选择器            标签名                $("li")
4. 并集 OR              ,					   $("h1,#name,.a")                
5. 交集 AND             直接写                $("li.c1")
6. 适配                 *                    $("*")
```

```html
<ul>
  <li>data1</li>
  <li class="red">data2</li>
  <li>data3</li>
  <li>data4</li>
  <li id="d5">data5</li>
  <li>data6</li>
  <li class="blue">data7</li>
  <li class="blue">data8</li>
</ul>
<script src="./jslib/jQuery.js"></script>
<script>
  console.log(`id选择器:${$('#d5').html()}`)
  console.log(`class选择器:${$('.red').html()}`)
  console.log(`标签选择器:${$('ul').html()}`)
  console.log(`并集选择器===========`)
  $('.red,#d5').css('backgroundColor', 'red')
  $('li.blue').css('backgroundColor', 'blue')

  $('*').css('fontSize', '80px')
</script>
```

### 层次选择器

```
1. 儿子                     >                             $("tr>td")
						  $('元素').children([选择器]);            
2. 后代                     空格               $("table td")
                          $('元素').find(选择器);	
3. 下一个弟弟                +           $("#data2+li") 
							next()         $("#data2").next("li")
4. 所有的弟弟                ~               $("#data2~li") 
							nextAll()       $("#data2").nextAll("li")
5. 上一个哥哥                prev()            $("#data2").prev("li")
6. 所有的哥哥                prevAll()         $("#data2").prevAll("li")
7. 同胞【哥哥弟弟们】         siblings()        $("#data2").siblings("li")
```

```html
<ul>
  <li>data1</li>
  <li>data2</li>
  <li>data3</li>
  <li>data4</li>
  <li id="d5">data5</li>
  <li>data6</li>
  <li>data7</li>
  <li>data8</li>
</ul>
<script src="./jslib/jQuery.js"></script>
<script>
  $('ul>li').css('color', 'red')
  $("html li").css('color', 'blue')
  $("#d5+li").css('color', 'blue')
  $('#d5').next('li').css('color', 'blue')
  $("#d5~li").css('color', 'blue')
  $('#d5').nextAll('li').css('color', 'blue')
  $("#d5").prev("li").css('color', 'blue')
  $("#d5").prevAll("li").css('color', 'blue')
  $('#d5').siblings('li').css('color', 'blue')
</script>
```

### 属性选择器

```
1.拥有该属性         [attr]              $("[name]")
2.等于属性值         [attr=value]        $("[name='sid']")
3.使用属性值开头      [attr^=value]       $("[name^='s']")
4.使用属性值结尾      [attr$=value]       $("[name$='s']")
5.包含该属性值        [attr*=value]       $("[name*='s']")
```

```html
<ul>
<li><a href="abc.html">data1</a></li>
<li><a href="aaa.html">data2</a></li>
<li><a href="ccc.html">data3</a></li>
<li><a href="abcd.html">data4</a></li>
<li><a>data5</a></li>
</ul>
<script src="./jslib/jQuery.js"></script>
<script>
  $("[href]").css("fontSize","36px")
  $("[href='aaa.html']").css("fontSize","36px")
  $("[href^='a']").css("fontSize","36px")
  $("[href$='c.html']").css("fontSize","36px")
  $("[href*='c']").css("fontSize","36px")
</script>
```

### 过滤选择器

#### 基本过滤选择器

```
1. 第一个             :first            $("li:first")
2. 最后一个           :last             $("li:last")
3. 偶数              :even             $("li:even")
4. 奇数              :odd              $("li:odd")
5. 等于索引           :eq(idx) 或 .eq(idx)   $("li:eq(1)") 或 $("li").eq(1)  从0开始
6. 小于索引           :lt(idx)          $("li:lt(3)")
7. 大于索引           :gt(idx)          $("li:gt(3)")
```

```html
 <ul>
  <li>data1</li>
  <li>data2</li>
  <li>data3</li>
  <li>data4</li>
  <li>data5</li>
</ul>
<script src="./jslib/jQuery.js"></script>
<script>
  $('li:first').css('color', 'red')
  $('li:last').css('color', 'red')
  $('li:even').css('color', 'red')
  $('li:odd').css('color', 'red')
  $('li:eq(2)').css('color', 'red')
  $('li').eq(2).css('color', 'red')
  $('li:lt(2)').css('color', 'red') 
  $('li:gt(2)').css('color', 'red')
</script>
```

#### 可见性过滤选择器

```
1. 隐藏   :hidden
2. 可见   :visible
```

```html
<!-- 
1. 隐藏   :hidden
2. 可见   :visible
-->
<button>隐藏</button>
<button>显示</button>
<img src="./images/1.jpg" alt="" />
<script src="./jslib/jQuery.js"></script>
<script>
  $('button:eq(0)').click(() => {
    $('img:visible').hide(1000)
  })

  $('button:eq(1)').click(() => {
    $('img:hidden').show(1000)
  })
</script>
```

#### 表单过滤选择器

```
1. input标签       :input      $("input")                <input />
2. type='text'    :text       $("input[type='text']")   <input type='text'/>
3. type='password':password  $("input[type='password']") <input type='password'/>
4. type='radio'   :radio     $("input[type='radio']")  <input type='radio'/>
5. type='checkbox':checkbox  $("input[type='checkbox']") <input type='checkbox'/>
6. type='submit'  :submit  $("input[type='submit']") <input type='submit'/>
7. type='image'  :image  $("input[type='image']") <input type='image'/>
8. type='reset'  :reset  $("input[type='reset']") <input type='reset'/>
9. type='button'  :button  $("input[type='button']") <input type='button'/>
10.type='file'   :file  $("input[type='file']") <input type='file'/>
===========================布尔值属性==================
11.被选中的单选/复选按钮  :checked    $(":radio:checked")
12.下拉框被选中          :selected   $(":selected")
13.被禁用               :disabled   $(":disabled")
```

```html
<form action="xxController" method="get">
  <input type="text" value="admin1" /> <br />
  <input type="text" value="admin2" /> <br />
  <input type="hidden" value="S1001" />
  <input type="text" value="S1001" disabled /> <br />
  <input type="password" value="123" /> <br />
  <input type="radio" name="sex" value="0" />男
  <input type="radio" name="sex" value="1" />女 <br />
  <input type="checkbox" name="likes" value="CS" />CS
  <input type="checkbox" name="likes" value="LOL" checked />LOL
  <input type="checkbox" name="likes" value="DOTA" checked />DOTA
  <br />
  <input type="file" name="pic" /> <br />
  <select name="">
    <option value="1">data1</option>
    <option value="2" selected>data2</option>
    <option value="3">data3</option>
  </select>
  <input type="submit" value="提交" />
  <input type="image" src="./images/1.jpg" alt="" /> <br />
  <input type="reset" value="重置" />
</form>
<script src="./jslib/jQuery.js"></script>
<script>
  console.log(`第一文本输入框:${$("input").val()}`);
  console.log(`第一文本输入框:${$(":input").val()}`);
  console.log(`第二文本输入框:${$("input[type='text']:eq(1)").val()}`);
  console.log(`第二文本输入框:${$(":input:text:eq(1)").val()}`);
  console.log(`密码输入框:${$(":password").val()}`);
  console.log(`单选按钮:${$(":radio").val()}`);
  console.log(`单选按钮:${$(":radio:checked").val()}`);
  $(":radio[value='1']")[0].checked = true

  $(":checkbox:checked").each((idx,e)=>{
    // console.log(`索引:${idx}========value:${e.value}`);
    console.log(`索引:${idx}========value:${$(e).val()}`);
  })

  $.each($(":checkbox:checked"),(idx,e)=>{
    console.log(`索引:${idx}========value:${e.value}`);
  })

  $(':submit').val('我是提交按钮')
  $(":image").attr("src",'./images/html-css-js.png')
  $(":reset").val(`回到解放前`)
  console.log(`file的name属性值:${$(':file').attr('name')}`)
  console.log(`select被选中的值：${$("select :selected").val()}`);
  console.log(`select被选中的值：${$("select :selected").text()}`);

  console.log(`获得禁用的值:${$(':disabled').val()}`)
  console.log(`获得隐藏的值:${$('input:hidden').val()}`)
</script>
```



## 1.3 JQ注册事件

### 鼠标事件

```
1. 鼠标进入  mouseover = mouseenter
2. 鼠标离开  mouseout = mouseleave
```

```html
<ul>
  <li>data1</li>
  <li>data2</li>
  <li>data3</li>
  <li>data4</li>
  <li>data5</li>
  <li>data6</li>
  <li>data7</li>
  <li>data8</li>
</ul>
<script src="./jslib/jQuery.js"></script>
<script>
  /* $('li')
    .mouseover(function () {
      $(this).css('backgroundColor', 'red')
    })
    .mouseout((e) => $(e.target).css('backgroundColor', 'pink')) */

  $('li')
    .mouseenter((e) => $(e.target).css('backgroundColor', 'red'))
    .mouseleave((e) => $(e.target).css('backgroundColor', 'pink'))
</script>
```


```html
<div class="nav">
    <div class="w">
        <ul class="ul1">
            <li><a href="javascript:;">首页</a></li>
            <li><a href="javascript:;">关于我们</a></li>
            <li>
                <a href="javascript:;">联系我们</a>
                <ul class="u2">
                    <li><a href="javascript:;">邮件联系</a></li>
                    <li><a href="javascript:;">电话联系</a></li>
                    <li><a href="javascript:;">微信联系</a></li>
                </ul>
            </li>
        </ul>
    </div>
 </div>

 <script src="./jQuery.js"></script>
 <script>
     $(".nav .ul1>li:last").mouseenter(()=>$(".u2").show()).mouseleave(()=>$(".u2").hide())
 </script>
```



### 键盘事件

```
1. 键盘按下  keydown
2. 键盘释放  keyup
```

```html
<input type="text" />
<script src="./jslib/jQuery.js"></script>
<script>
  $('input')
    .keydown((e) => console.log(e.keyCode))
    .keyup(() => console.log(`释放`))
</script>
```

### 表单事件

```
1.单击   click
2.双击   dblclick
3.值改变 change
4.失去焦点  blur
5.获得焦点  focus
6.表单提交  submit
```

```html
<form action="logcontroller" method="get">
  <input type="text" /> <span id="accountError"></span><br />
  <input type="password" /> <br />
  <button>提交</button>
</form>
<script src="./jslib/jQuery.js"></script>
<script>
  $(':text').focus((e) => $(e.target).css('borderColor', 'red'))

  $(':text').blur(() => {
    if ($(':text').val().length === 0) {
      // alert(`用户名不能为空`)
      $('#accountError').html(`<font color='red'>用户名不能为空<font>`)
      $(':text').focus()
      return
    }
    console.log(`======用户名=====`)
    $('#accountError').html(``)
  })

  $("form").submit((e)=>{
    if ($(':text').val().length === 0) {
      // alert(`用户名不能为空`)
      $('#accountError').html(`<font color='red'>用户名不能为空<font>`)
      $(':text').focus()
      return e.preventDefault()
    }
  })
</script>
```



### 事件绑定

```
1.绑定       bind("事件名",事件处理函数)   =  on ("事件名",事件处理函数)  【结构固定,内容多变】
	$("button:eq(0)").click(()=>{...})    $("button:eq(0)").on("click",()=>{...})
	$("button:eq(0)").dblclick(()=>{...}) $("button:eq(0)").on("dblclick",()=>{...})
	$("button:eq(0)").blur(()=>{...})     $("button:eq(0)").on("blur",()=>{...})
2.解绑       unbind("事件名")           =  off("事件名")
3.只执行一次  one("事件名",事件处理函数)
```

```html
<button>绑定 click 按钮1</button>
<button>解除 click 按钮2</button>
<button>只绑定一次 click 按钮3</button>
<script src="./jslib/jQuery.js"></script>
<script>
  $('button:eq(0)').on('click', () => console.log(`按钮1 被点击了..11.`))
  $('button:eq(1)').on('click', () => $('button:eq(0)').off('click'))
  $('button:eq(0)').bind('click', () => console.log(`按钮1 被点击了..11.`))
  $('button:eq(1)').bind('click', () => $('button:eq(0)').unbind('click'))
  $('button:eq(2)').one('click', () => console.log(`按钮3 被点击了..11.`))
</script>
```

### 复合事件

```
该事件由2个或2个以上的函数
hover(fnOver,fnOut)
```

```html
<ul>
  <li>data1</li>
  <li>data2</li>
  <li>data3</li>
  <li>data4</li>
  <li>data5</li>
  <li>data6</li>
  <li>data7</li>
  <li>data8</li>
</ul>
<script src="./jslib/jQuery.js"></script>
<script>
  $('li').hover(
    (e) => $(e.target).css('backgroundColor', 'pink'),
    (e) => $(e.target).css('backgroundColor', 'red')
  )
</script>
```

```html
 <div class="box">
  <div class="nav">
    <ul>
      <li><a href="javascript:;">商品介绍</a></li>
      <li><a href="javascript:;">价格与包装</a></li>
      <li><a href="javascript:;">售后保障</a></li>
      <li><a href="javascript:;">商品评价</a></li>
    </ul>
  </div>
</div>

<script src="./jQuery.js"></script>
<script>
  $('.box li').hover(
    (e) => (e.target.className = 'active'),
    (e) => (e.target.className = '')
  )
</script>
```



### 事件冒泡

```
触发子元素事件  级联 触发 父元素的事件
【阻止事件冒泡:】
1. event.stopPropagation()
2. event.cancelBubble = true
```

```html
<style>
  #d1 {
    width: 200px;
    height: 200px;
    background-color: pink;
  }

  #d2 {
    width: 100px;
    height: 100px;
    background-color: yellowgreen;
  }

  #d3 {
    width: 40px;
    height: 40px;
    background-color: red;
  }
</style>
</head>
<body>
<div id="d1">
  <div id="d2">
    <div id="d3"></div>
  </div>
</div>

<script src="./jslib/jQuery.js"></script>
<script>
  $('#d1').bind('click', () => console.log(`爷爷被点击了`))
  $('#d2').bind('click', function () {
    console.log(`爸爸被点击了`)
    // event 内置对象  浏览器中 内置 对象
    event.cancelBubble = true
  })
  $('#d3').bind('click', (e) => {
    console.log(`你小子被点击了`)
    //e.stopPropagation()
    event.stopPropagation()
  })
</script>
```

### 事件默认行为

```
默认事件行为： 比如 <a href="addStu()">...</a> 默认链接到目标地址
【阻止事件默认行为】
1. javaScript:
2. event.preventDefault()
```

```html
<a href="http://www.baidu.com">去百度 1 </a> <br />
<a href="javaScript:">去百度 2 </a> <br />
<a href="javaScript:void(0)">去百度 3 </a> <br />
<a href="">去百度 4 </a> <br />
<script src="./jslib/jQuery.js"></script>
<script>
  $('a').bind('click', () => event.preventDefault())
</script>
```

### 表单序列化

```
序列化: 一次获取到表单中所有元素内容
$('form').on('submit', function () {
    // 通过 jQuery 对象的 serialize 方法，获取所有表单元素的数据
    $(this).serialize();
	 // 阻止表单默认提交
      return false;
})
```

```html
 <form action="02-login-rs.html" method="get">
  <input type="text" name="account" placeholder="请输入账号" /> <br />
  <input type="text" name="pswd" placeholder="请输入密码" /> <br />
  <input type="text" name="age" placeholder="请输入年龄" /> <br />
  <button>提交</button>
</form>
<script src="./jslib/jQuery.js"></script>
<script>
  $('form').bind('submit', () => {
    location.href = `xxController?${$('form').serialize()}`
    //阻止默认行为
    event.preventDefault()
  })

  /*  $('form').bind('submit', () => {
    let account = $("input[name='account']").val()
    let pswd = $("input[name='pswd']").val()
    location.href = `xxController?account=${account}&pswd=${pswd}`
    //阻止默认行为
    event.preventDefault()
  }) */
</script>
```



## 1.4 JQ操作样式

```
1.$('元素').css('属性', 值);  设置单个属性样式
  $('元素').css({'属性': '值','属性': '值'})  设置多个属性样式
2.$('元素').addClass('类名  类名'); 
3.$('元素').hasClass('类名');
4.$('元素').removeClass('类名');
5.$('元素').toggleClass('类名');
```

```html
<style>
  .pic {
    width: 400px;
    height: 400px;
    border: 10px solid #000;
  }
</style>
</head>
<body>
<img src="./images/1.jpg" alt=""/>
<button>添加样式1</button>
<button>添加样式2</button>
<button>添加样式3</button>
<button>删除样式4</button>
<button>切换样式5</button>
<script src="./jslib/jQuery.js"></script>
<script>
  $('button:eq(0)').bind('click', () => $('img').css('width', '80px'))
  $('button:eq(1)').bind('click', () => $('img').css({ width: '80px' }))
  $('button:eq(2)').bind('click', () => $('img').addClass('pic'))
  $('button:eq(3)').bind('click', () => $('img').removeClass('pic'))
  $('button:eq(4)').bind('click', () => $('img').toggleClass('pic'))
  console.log(`是否拥有pic样式: ${$('img').hasClass('pic')}`)
</script>
```



## 1.5 jQ操作属性

```
1. 操作内置属性
	获取: $(对象).prop('属性名');
	设置: $(对象).prop('属性名', 值);
2.操作自定义属性
	获取: $(对象).attr(自定义属性名);
	设置: $(对象).attr(自定义属性名, 值);
3. 获取表单控件中的值
    $(对象).val()
    $(对象).val(值);
4.  操作普通标签中的值
    $(对象).text(值);    $(对象).html(值); 
```

```html
<input type="text" value="admin" name="username" data-user-id="S1001" />
<span>111111</span>
<div><h1>2222</h1></div>

<script src="./jslib/jQuery.js"></script>
<script>
  console.log(`获得内置属性: ${$('input').prop('name')}`)
  $('input').prop('name', 'uname')

  console.log(`获得自定义属性: ${$('input').attr('data-user-id')}`)
  $('input').attr('data-user-id', 'S6666')

  console.log(`获得value属性: ${$('input').val()}`)
  $('input').val('zhang3')

  console.log(`获得innerText: ${$('span').text()}`)
  $('span').text('span')

  console.log(`获得innerHTML: ${$('div').html()}`)
  $('div').html(`<ul>
  <li>data1</li>
  <li>data2</li>
  <li>data3</li>
</ul>`)
</script>
```



## 1.6 jQ操作元素

### 删除元素

```
1. $对象.remove();  从页面中将当前标签删除
2. $对象.empty();   将标签中的所有内容清空
3. $对象.html('');  将标签中的所有内容清空
```



### 创建元素

```
1. $对象.html('html标签名');  直接在标签中添加新标签
2. let res = $('html标签');   创建标签,  返回: JQ标签对象
```

### 添加元素

```
1. $父元素.append(元素);   将创建元素添加到父元素末尾
2. $父元素.prepend(元素);  将创建元素添加到父元素开始
1.尾部添加
	父.append(子)
	子.appendTo(父)
	
2.开始添加
    父.prepend(子)
    子.prependTo(父)
```

```html
<ul>
  <li>data1</li>
  <li>data2</li>
  <li>data3</li>
  <li>data4</li>
  <li>data5</li>
  <li>data6</li>
  <li>data7</li>
  <li>data8</li>
</ul>
<div>
  <p>我的div中p</p>
</div>
<script src="./jslib/jQuery.js"></script>
<script>
  // 1.删除元素
  $('ul li:first').remove()
  $('ul li:last').empty()
  $('ul li:last').html(``)

  //2.创建元素
  $("ul li:first").html(`<p>aaaa</p>`)
  let btn = $(`<button>按钮</button>`)

  //3.添加元素
  $('div').append(btn)
  btn.appendTo($('div'))

  $('div').prepend(btn)
  btn.prependTo($('div'))
</script>
```

### 例：

根据素材： 03-信息发布.html   实现元素添加/删除功能

```html
<style type="text/css">
  * {
    margin: 0;
    padding: 0;
  }
  .msg {
    width: 980px;
    padding-bottom: 10px;
    border: 1px solid #ccc;
    margin: 50px auto;
  }

  textarea {
    width: 880px;
    height: 100px;
    border: 0 none;
    border: 1px solid orange;
    resize: none;
    outline-style: none;
    border-radius: 10px;
    display: block;
    margin: 50px auto 0 auto;
    padding-left: 20px;
    padding-top: 20px;
    box-sizing: border-box;
  }

  .btn {
    width: 80px;
    height: 40px;
    display: block;
    float: right;
    margin-right: 50px;
    margin-top: 20px;
    background-color: blue;
    color: #fff;
    text-align: center;
    text-decoration: none;
    line-height: 40px;
    border-radius: 10px;
    clear: both;
  }

  .content {
    width: 880px;
    margin: 80px auto 0 auto;
  }
  .item {
    height: 50px;
    line-height: 50px;
    border-bottom: 1px dashed #ccc;
    padding-left: 20px;
  }

  .item p {
    float: left;
  }

  .del {
    float: right;
    text-decoration: none;
    color: #999;
  }
  .del:hover {
    color: orange;
  }
</style>
</head>
<body>
<div class="msg">
  <textarea></textarea>
  <a href="javascript:;" class="btn">发布</a>

  <div class="content">
    <div class="item">
      <p>三天没吃肉啦</p>
      <a href="javascript:;" class="del">删除</a>
    </div>
  </div>
</div>
<script src="./jslib/jquery.js"></script>
<script>
  //发布
  $(`a.btn`).bind(`click`, () => {
    //创建 元素
    let divItemElt = $(`<div class="item">
      <p>${$('textarea').val()}</p>
      <a href="javascript:;" class="del">删除</a>
            </div>`)
    //添加元素
    $(`div.content`).prepend(divItemElt)
    //清空输入框
    $('textarea').val(``)
    console.log(`===发布=======`)
  })

  //删除  使用 事件委托  代理
  $(`div.content`).on(`click`, `.del`, () => {
    if (confirm(`确认删除吗?`)) {
      $(event.target).parent().remove()
    }
    console.log(`===删除=========`)
  })
</script>
```