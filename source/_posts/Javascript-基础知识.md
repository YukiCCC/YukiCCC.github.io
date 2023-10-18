---
title: Javascript 基础知识
date: 2023-06-12 13:11:08
tags: JS
category: 前端
---
JavaScript 是互联网上最流行的脚本语言，这门语言可用于 HTML 和 web，更可广泛用于服务器、PC、笔记本电脑、平板电脑和智能手机等设备。


# JavaScrip简介

JavaScript 是脚本语言
JavaScript 是一种轻量级的编程语言。
JavaScript 是可插入 HTML 页面的编程代码。
JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行。
JavaScript 很容易学习。


## ECMAScript  语法标准(es)
### JavaScript 输出

JavaScript 可以通过不同的方式来输出数据：
使用 window.alert() 弹出警告框。
使用 document.write() 方法将内容写到 HTML 文档中。
使用 innerHTML 写入到 HTML 元素。
使用 console.log() 写入到浏览器的控制台。


外部的 JavaScript
也可以把脚本保存到外部文件中。外部文件通常包含被多个网页使用的代码。

外部 JavaScript 文件的文件扩展名是 .js。
```javascript
<script src="./01.js"></script>
```

### 变量
```javascript
var a = 20;
```

var 申明变量名
a 变量名 用来存储变量
20 变量
= 赋值符号 从右向左赋值

变量命名规则：
1.可以是字母或者下划线_或者$开头
2.长度不可超过255个字符
3.名中不能含有空格，首字头不能是数字
4.严格区分大小
5.不能使用关键字或保留字
6.汉字可使用不推荐    
当使用空的变量名时，得到的结果就是未定义 undefined
使用不存在的变量名时，报错，d is not defined
变量的声明提升:先使用变量再创建变量 得到的结果会是 undefined
       只能提升变量名，不提升变量


```javascript
console.log(e);
var e = 30; 
```
相当于

```javascript
var e;
console.log(e);
var e = 30;
```
### 数据类型与转换 

基本数据类型:
number  数字
string  字符串 只要有引号包裹就是字符串
boolean 布尔 true/false
将数据类型转化为number类型
强制类型转换:
Number（） 将数据类型转换成number
转换字母等非数字内容的话会显示NaN (No a Number)       
parseFloat()   浮点型 将数据保留小数 并且转换为数字类型
转换带数字的字符串时，必须开头为数字才能识别
parseInt() 整型 将数据保留整数，并转换成数字类型
isNaN()：判断内容是否是 非数字 如果是非数字 返回 true 反之为 false
只查看数据内容，不查看数据类型
数据类型转换为字符串
强制类型转换
String()  将数据类型转换为字符串
xxx.toString()  将数据类型转换为字符串  需要转化的变量名写在前面
将数据类型转换为布尔
Boolean()
0为false，其他为true
值类型: 
number  数字
string  字符串
boolean 布尔
null    空
undefined未定义
symbol  独一无二的
引用数据类型：
object  对象
function函数
array   数组
tpyeof(undefined)= undefined;
tpyeof(null)= object;
tpyeof(error)= object;

### 算数运算符

<span>+</span> 加法
当符号两边都是数字的时候，会自动相加求和
当符号两边有字符串时，会起到拼接字符串的作用（结果时字符串类型--隐式类型转换）
当符号两边有布尔类型时，true为1，false为0 参与求和计算
++ 自增加1
++在后面 表示后加，处于正在加1的过程，还没加上，当再次使用变量时，才算自增结束
++在前面 表示先加，直接自增结束，得到就是自增+1的结果
<span>-</span> 减法
当符号两边都是数字的时候，会自动相减求差
当符号两边有字符串时，会先转换成数字再计算（非数字内容为NaN）
当符号两边有布尔类型时，true为1，false为0 参与求和计算
-- 自减1，同自增
<span>*</span> 乘法
当符号两边都是数字的时候，会自动相乘求积
当符号两边有字符串时，会先转换成数字再计算（非数字内容为NaN）
当符号两边有布尔类型时，true为1，false为0 参与求和计算
**/** 除法
当符号两边都是数字的时候，会自动相除求商
当符号两边有字符串时，会先转换成数字再计算（非数字内容为NaN）
当符号两边有布尔类型时，true为1，false为0 参与求和计算
除数如果是0 得到的是 Infinity 无穷/无限
% 除余
()优先运算符

### 比较运算符

**注意：= == === 的不同**
==    等号(只比较值，不比较数据类型)
===    全等号（值和数据类型都相同）
<span>></span>    大于号
<span>>=</span>      大于等于
<       小于
<=      小于等于
!=      不等于       
!==     不全等于

### 逻辑运算符

&& 与   符号两边的表达都为true整个表达式的结果就是true
                有一方为false，整个表达式的结果就是false       
|| 或   有一方为true，整个表达式的结果就是true   
！ 非

### 对象与事件
实例化对象：var obj=new Object();
键值对（属性：属性值）obj.name="zs";//键值对就是XX=XX
字面量的方式创建对象:

```javascript
var obj2 = {
    name:"1s",
    age:20,
    height:172
}
```
日期对象var= new Date();
date.getFullYear();获取当前年份
getMonth 获取当前月份
getDate日期
getDay星期
getHours小时
getMinutes分钟
getSeconds当前秒
getMilliseconds 当前毫秒
getTime从1970.1.1至今的毫秒数
Math对象
Math.max 返回最大数
Math.min 返回最小数
Math.ceil 天花板函数，有小数部分向上取整
Math.floor 地板函数  舍掉小数部分
Math.round(b) 四舍五入
Math.random() 随机数字0-1 取不到1
事件
on绑定事件的关键字 click 点击事件
box.onclick=function(){}
onblur 失去焦点

### if else
略

### switch
switch 用来监视链路，捕获数据某种情况下需要执行的代码块
switch 具有数据穿透性 需要break中断余下代码的执行
default 相当于else

```javascript
switch(n){
	case 10:
		XXXX;
		break;
	case 20:
		XXXX;
		break;

	default:
		XXXX;
		break;
}
```
第二种情况：switch(true)
```javascript
switch(true){
    case n<10&&n>=0:
    console.log("switch范围监测")
    break;
}
```

### 三目运算符
表达式？ 结果1(true):结果2(false)

```javascript
a>b ? console.log("a大"):console.log("b大");
```

### for循环
for循环执行顺序：
for 首次执行 先创造变量 再判断条件 不走增量 直接执行代码块
剩下的执行次数都是 先增量再判断 最后走代码块
如果 变量不满足判断条件 循环结束
for(初始化变量;判断条件；增量){重复执行代码块}
break： 终止循环 终止整个循环体 余下代码不执行
continue： 终止循环 终止当前次数的循环，余下代码不执行

### while循环与do while循环
```javascript
while(i<10){
            console.log(i);
            i++;
}
```

```javascript
do {
            i++;
            console.log(i);
        } while (i < 10)
```

while 和 do while的区别
while循环是 先判断再执行
do while 循环 是先执行再判断

### 数组和数组api
```javascript
var arr = new Array();//实例化数组
// 数组内数据的序号 我们叫做下标（索引号）
arr[0]="李白";
arr[1]="白居易";
console.log(arr);
// 字面量的方式创建数组
var arr2 = ["ls","zs",123,false];
console.log(arr2);
// 数组的使用方式  数组名[下标]
console.log(arr2[2]);
// 数组名.length 数组内成员的个数
console.log(arr2.length);
//二维数组
var arr4 = [1,3,4,[1,2,3]];
console.log(arr4[3][1]);
//多维数组
var arr5 =[
{
    name:"zs",
    age:18,
    friends:['ls','wr']
},
{
    name:"ls",
    age:20,
    friends:['zs','wr']
},
{
    name:"wr",
    age:22,
    friends:['ls','zs']
}
]
```
数组的操作方法
1.数组.includes()  查看数组内是否包含指定成员，如果是就返回true,如果不包含就返回false
 2.Array.isArray()  判断是否是一个数组格式 返回 true 或 false
 3.数组.indexOf()  查看数组内的成员，如果存在就返回首次出现的下标，如果不存在就返回-1
4.数组.lastIndexOf()  查看数组内的成员，如果存在就返回最后一次出现的下标，如果不存在就返回-1
5.数组.join()  在数组各元素之间插入相同的字符串拼接，将数组转换成字符串
6.数组.push()  在原数组末尾添加新成员，返回新数组长度，原数组被改变
7.数组.unshift()  在数组开头添加新成员，返回新数组长度，原数组被改变
8.数组.pop()  删除数组最后一名成员，返回被删除内容，原数组被改变
9.数组.shift()  删除数组开头的成员，返回被删除内容，原数组被改变
 10.数组.reverse()  反转数据
11.数组.splice()  当括号内有一个参数的时候，表示从当前下标开始一直截取到末尾，返回被截取的内容，原数组改变
当括号内有两个参数的时候，表示从当前下标开始一直截取到几个，返回被截取的内容，原数组改变
当括号内有三个参数的时候，第三个参数表示在截取位置添加的新内容，返回被截取的内容，原数组改变
12.数组.slice()    当括号内有一个参数的时候，表示从当前下标开始一直截取到末尾，返回被截取的内容，原数组不改变
当括号内有两个参数的时候，表示从当前下标开始到第二个参数下标结束，返回被截取的内容，原数组不改变
var res =arr.includes("奥斯");

### 字符串API
字符串.length    获取字符串的长度
字符串.split()   字符串转换成数组，根据括号的内容进行字符分割
字符串.charAt()  返回指定下标处的字符
字符串.indexOf() 返回查找首次出现字符的下标，如果不存在返回-1
字符串.lastIndexOf() 返回查找最后一次出现字符的下标，如果不存在返回-1
字符串.substr()  截取字符串 如果有一个参数 表示从当前下标开始截取到末尾 返回截取的内容
如果有两个参数，表示从当前下标开始截取几个 返回截取的内容

### 函数
匿名函数：  自调用  通过事件绑定在一起触发
具名函数： function . 函数名()
函数特性：不调用，不执行
函数的使用叫做调用
函数具有 预加载(函数的位置在定义先后不影响执行)
具名函数的调用  函数名()
return 返回值（余下代码不执行）
function 函数名（形式参数）{}
var a = 函数名（实际参数）；
封装函数

### 全局变量与局部变量
变量是存在作用域的 分为 全局变量和局部变量
局部变量： 在函数内部用var声明的变量就是局部变量（只在函数内部生效）
全局变量：在函数外部用var声明的变量就是全局变量，可以在整个JS中生效
不用var声明的，变量也是全局变量（不推荐使用）
闭包：
闭包的形式：多个函数互相嵌套
闭包的目的：将内部函数的局部变量提到全局中去使用
闭包的实现方式：不断地设置return 返回值
闭包的缺点：会消耗电脑内存 影响性能

## DOM
document object model 文档对象模型

### 定时器
```javascript
//    setInterval(callback) 多次执行的定时器 callback参数 表示 回调函数
start.onclick = function () {
    timer = setInterval(
        function () {
            console.log(1);
        }, 1000)
    }
    stop.onclick = function () {
// clearInterval(定时器的名字)  清除定时器
    clearInterval(timer);
}
```

### this
this 表示 这个
this 在函数中  代指函数的调用者
this在函数外  指向的是window对象，最大的对象
```javascript
var lis = document.getElementsByTagName("li")
console.log(this);
for (var i = 0; i < lis.length; i++){
    //属性绑定  写在循环的内部，事件的外部
    // index 一般表示为下标
    lis[i].index = i;
    lis[i].onclick = function() {
        //先让所有颜色都变成蓝色 再让当前的这个变成红色
        //这就是排他思想
        for (var j = 0; j < lis.length; j++){
            lis[j].style.background = "skyblue";
        }
        this.style.background = "red";
        console.log(this.index);//获取到下标
    }
}
```

### 轮播图
参考https://swiper.com.cn/

### Node节点操作
1.xx.parentNode   当前节点的父节点
2.xx.childNodes   当前节点的所有子节点,包含文本节点(本次返回的text为回车造成的空格)  返回nodeList 伪数组
3.xx.children     当前节点的所有子元素节点   返回HTMLCollection 伪数组
4.xx.firstChild   当前节点的第一个子节点,包含文本节点
5.xx.firstElementChild    当前节点的第一个子元素节点
6.xx.lastChild   当前节点的最后一个子节点,包含文本节点
7.xx.lastElementChild    当前节点的最后一个子元素节点
8.xx.previousSibling   当前节点的前一个兄弟节点,包含文本节点
9.xx.previousElementSibling  当前节点的前一个兄弟元素节点
10.xx.nextSibling   当前节点的前一个兄弟节点,包含文本节点
11.xx.nextElementSibling  当前节点的前一个兄弟元素节点
节点的操作
1.document.createElement()        创建节点
2.xx.innerHTML                    往节点内添加或替换内容(文本或标签)
2.xx.innerText                    往节点内添加或替换内容(文本)
4.xx.appendChild()                往父节点的末尾添加新节点
5.xx.insertBefore(新节点,目标节点)  将新节点添加到目标节点之前
6.xx.cloneNode()                  克隆节点,true的时候,将该节点及其子节点全部复制；flase的时候只复制节点本身。
7.父节点.removeChild()             删除父节点的子节点

### 偏移量
offsetWidth   元素自身的宽度  width+border+padding
offsetHeight  元素自身的高度  同上
offsetLeft    元素自身的位置
offsetTop     元素自身的位置    
onscroll      滚动事件
scroll卷曲的距离
clientWidth可视区域的宽 width padding
clientWidth可视区域的宽 height padding

document.body;//获取body 标签
document.documentElement//获取html标签

### 事件对象
event 事件对象 通过事件触发的时候调用函数内的参数
该对象内包含了事件触发时的信息
事件对象的兼容写法：
```javascript
var e = event || window.event;//兼容低版本ie浏览器
```
pageX 光标相对于网页的水平位置（ie无）*
pageY 光标相对于网页的垂直位置（ie无）*
screenX 光标相对于屏幕的水平位置
screenY 光标相对于屏幕的垂直位置
clientX 光标相对于可视区域的水平位置*
clientY 光标相对于可视区域的垂直位置*（*重要且相同）
xx.onkeypress   键盘按键 按下并且弹起
xx.keyCode  表示键盘上对应按键的编码
xx.onmouseenter   鼠标进入
xx.onmouseleave   鼠标离开

### 事件冒泡和事件句柄
事件冒泡：
当一个元素的事件被触发的时候,比如鼠标点击了一个元素，同样的事件就会在
这个元素的所有祖先元素上被触发
这个过程就叫做事件冒泡,这个事件是从原始事件一直冒泡到dom树的最上层.
不支持事件冒泡的事件: focus,blur,mouseenter,mouseleave,load,resize
```javascript
//阻止事件冒泡的兼容写法
e.stopPropagation ? e.stopPropagation() :e.cancelBubble = true;
```
事件句柄:
addEventLister  添加事件句柄
第一个参数   事件的名称
第二个参数   callback  回调函数
第三个参数   false  触发的顺序是由内到外，叫做冒泡的顺序（默认）； true  触发的顺序是由外到内，叫做捕获的顺序
同一个元素可以绑定多个相同事件，不会覆盖，挨个执行
```javascript
box.addEventListener("click",function(){
            alert("222");
        },true)
```

### 缓存
注意：cookie localStorage sessionStorage的区别
cookie     默认浏览器关闭时消失，存在于web服务器中，存储大小为4KB
localStorage  本地储存   永久有效，除非手动删除，存储大小为5MB
sessionStorage  会话存储  关闭浏览器或窗口事就消失，存储大小为5MB
cookie缓存：
```javascript
document.cookie="username = 李白"//创建cookie缓存
document.cookie="age = 30; expires = Tue, 20 Jun 2023 12:00:00 GMT"//设置过期事件
var res =document.cookie;
var n = res.indexOf("=") +1 ;
res = res.substr(n);
console.log(res); 
```
localStorage缓存：
```javascript
localStorage.setItem("NAME","TOM");//创建缓存 localStorage.setItem（key,value）      
localStorage.setItem("AGE","18");
localStorage.removeItem("AGE");//删除指定缓存  localStorage.removeItem(key)
var res = localStorage.getItem("NAME");//获取缓存   localStorage.getItem(key)      
var age = localStorage.getItem("AGE");
localStorage.clear();//清空缓存  localStorage.clear()
```

## BOM
browser object model  浏览器对象模型

### Window对象
window.open("https://www.baidu.com");//打开新窗口
location.href = "https://www.baidu.com"//在当前窗口跳转
window.close();//关闭窗口

### location
location.hostname    web主机域名
location.pathname    当前页面的路径
location.port        端口号(0-65535)
location.herf        整个URL
location.protocol    web协议
常见的协议:
http 和 https 的区别*
http  免费  相对不安全，端口号默认80   
https 收费  比较安全，端口号默认443   secure
ftp   
file

### history
两种后退功能：
history.back();            history.go(-1);
两种前进功能：
history.go();            history.go(1);

### navigator
navigator  设备信息对象
navigator.appCodeName   浏览器代号
navigator.appName       浏览器名称
navigator.appVersion    浏览器版本
navigator.vender        浏览器供应商
navigator.cookieEnabled 浏览器是否启用了缓存
navigator.platform      硬件平台
navigator.userAgent    用户代理语言
navigator.language  用户代理语言

### 正则表达式
 实例化创建正则对象
```javascript
var reg = new RegExp();
```
//字面量方式创造正则对象
reg = /@/;    含有@
reg = /\d/    含有数字
reg = /\D/    含有非数字
reg = /\s/    含有不可见字符(空格，回车等)
reg = /\S/    含有可见字符
reg = /\w/    含有单词字符 字母 数字
reg = /\W/    含有非单词字符

简单类
reg = /[23]/    含有2或者3
负向类
reg = /[^23]/    含有非2或者3
范围类
reg.onblur = /[0-9]/    含有0-9中的任意一个
组合类
reg = /[0-9a-z]/   含有数字或者字母中的一个
边界
reg = /^12/ 必须以12开头
reg = /23$/ 必须以23结束
reg = /^123$/  必须是123
//量词
reg= /^123*$/  '3'的重复次数>=0次
reg= /^123+$/  '3'的重复次数>=1次
reg= /^123?$/  '3'的重复次数只能是0或1次
reg= /^12{4}3$/ '2'的重复次数4次
reg= /^12{4,}3$/ '2'的重复次数>=4次
reg= /^12{4,10}3$/ '2'的重复次数4到10次
            
正则.test(需要校验的内容)  返回布尔类型
```javascript
var res = reg.test(inp.value);
```

## 高级JS(面向对象)
### 值类型与引导类型
值类型  ：number string boolean null undef symbol
引用数据类型： array function object
        
值类型与引用数据类型的区别
值类型 ：  
存储在栈中，内存空间固定
当数据复制的时候，可以直接复制互不影响
typeof 判断数据类型
引用数据类型：
存储在堆中，内存空间不固定
浅拷贝：当数据复制的时候，只能复制数据的引用地址
深拷贝：将数据复制并在堆中重新申请一片空间进行存储
深拷贝实现的两种方式：
```javascript
var obj2 = JSON.stringify(obj);//对象 => string  将对象转换成json字符串
var obj2 = JSON.parse(obj2);//string => 对象  将json字符串转换成对象     
```

通过instanceof()   判断数据是哪一种引用类型（返回true/false）
```javascript
  var res = arr instanceof(Arry);  
```

### 工厂模式和构造函数
面向对象编程的基本特征
封装：  将客观事物封装成抽象的类
继承：  子类具有父类的公有属性
多态：  对象的多功能，多方法，一个方法可以有多种表现形式
字面量创建对象  创建量如果比较多就比较繁琐，并且对象之间没有关系
```javascript
var personOne = {
    name:"zs",
    age:20
}
var personTwo = {
    name:"ls",
    age:24
}
```
通过封装的方式创建对象，解决代码重复的问题
```javascript
function createPerson(name,age){
    return{
        name:name,
            age:age
        }
    }
```
工厂模式
```javascript
function createPerson(name,age){
    var obj =new Object();//准备工厂环境
    obj.name = name;//将属性进行加工
    obj.age = age;//将属性进行加工
    return obj;//将加工好的对象进行输出
}
personOne = createPerson("王二",10);
personTwo = createPerson("zy",30);
```
构造函数
```javascript
function CreatePerson(name,age){
    this.name = name;
    this.age = age;
}
var personOne = new CreatePerson("张三丰","108")
var personTwo = new CreatePerson("孙悟空","正无穷")
console.log(personOne);
console.log(personTwo);
```
构造函数需要注意的事情：
1.CreatePerson 称之为 构造函数 也叫做 类，构造函数就是类
2.personOne 就是 CreatePerson 的实例对象
3.构造函数中的this指向的是通过new实例化出来的
4.必须使用new关键字 将函数实例化
5.构造函数的开头必须大写
6.构造函数会自动创造出来一个 constructor(构造器)属性，这是属性就是指向CreatePerson

### prototype原型
属性，存在于每个构造函数之中
通过prototype原型创建的方法可以在构造函数生成的实例中公用，有利于提升效率
prototype 的顶端 是 object
```javascript
 function CreatePerson(name,age){
    this.name = name;
    this.age = age;
}
CreatePerson.prototype.say = function(){
    console.log("yyy");
    return 0;
}
var one = new CreatePerson("zs",20);
```

### 对象继承
```javascript
function Person(name,age){
    this.name=name;
    this.age=age;
}
Person.prototype.run=function(){
    console.log("啊啊啊啊啊");
}
Person.prototype.findWork=function(){
    console.log("working");
}
function Man(){
    console.log("nnnnn");
}
//看似赋值的过程 => 实际上实现的是浅拷贝
// Man.prototype = Person.prototype;
 //for in 循环（用来遍历对象）
for(var k in Person.prototype){
    console.log(k);//k是对象的属性
   Man.prototype[k] = Person.prototype[k]
}
Man.prototype.jump = function(){
    console.log("hhhhhhhhhhh");
    }
console.log(Man.prototype);
console.log(Person.prototype);
```

### 多态
多态：同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。
多态的实现方式：覆盖指子类重新定义父类方法，基于prototype继承就是。

### call和apply和bind
```javascript
 //this
window.color = "red";
document.color = "green";
// console.log(window);
var obj = {
    color:"white"
}
function changeColor(a,b){
    console.log(a+"和"+b+"喜欢"+this.color)
}
//函数.call(this需要指向的对象，参数必须使用逗号分隔)
changeColor("小王","小明");
changeColor.call(document,"小王","小明");
changeColor.call(obj,"小王","小明");
//函数.apply(this需要指向的对象，参数必须使用数组)
changeColor.apply(document,["小王","小明"]);
changeColor.apply(obj,["小王","小明"]);
//函数名.bind(this需要指向的对象，参数可以是任意形式)()   返回的是函数需要再次调用
changeColor.bind(obj,["小李"],["zs"])()
```









