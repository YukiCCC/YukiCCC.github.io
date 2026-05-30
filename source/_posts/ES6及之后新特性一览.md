---
title: ES6及之后新特性一览
date: 2023-06-25 12:40:11
tags: ES6
category: 前端
---

## let声明变量

let 声明变量
1.let 不允许重复声明变量  var 可以重复声明
2.let 不支持变量的声明提升，var可以
3.let声明的变量会被所有代码块（{}内的范围）限制作用范围 var只会受到函数影响
4.let 声明的变量不和顶层变量挂钩

## const 声明常量

const 声明常量
1.const 不可以重复声明
2.不支持声明提升
3.作用范围受{}影响
4.不和最上层对象window挂钩
const num = 10;
const num = 20;//Cannot redeclare block-scoped variable 'num'
if(true){
    const a = 20;
}
console.log(a);

**let和const区别**
1.let声明的变量可以被修改，const声明的是常量 不可以改变
2.let声明时可以不赋值，const声明时必须赋值

## 变量的解构赋值
解构赋值；就是快速的从对象或者数组中取出成员的一个语法方式
### 解构对象
```javascript
const obj = {
    name:"zs",
    age:18,
    genders:"男"
}
let name = obj.name;
let age = obj.age;
let genders = obj.genders;
```
```javascript
const obj2 = {
    name:"ls",
    age:18,
    genders:"女"
}
//解构对象
//前面的必须是{}表示要从obj2这个对象中获取对象成员
//name age genders 都是obj2的现有成员
//obj2必须是对象
let{name,age,genders} = obj2;
console.log(name,age,genders);
let{name:a,age:b,genders:c} = obj2;
console.log(a,b,c);
let{max,min,ceil,floor,random}=Math;
console.log(max(10,20,30));
```
### 解构数组
```javascript
//使用解构赋值的方式从数组中提取成员
const arr2 = ["一","二","三"];
//解构数组
//a b c 分别对应这个数组中的索引下标0 1 2
//arr2  必须是数组
//如果解构失败，返回结果就是undefined
let [a,b,c,d] = arr2;
console.log(a,b,c,d);
```

## 模板字符串
ES5中 表示字符串的时候使用''或者""
ES6中 还有一种可以表示字符串的方法  就是``
ES5 字符串  需要同行书写，换行后需要拼接字符串
```javascript
const str2 = 'hello'
                +'world'
```
ES6 可以直接换行使用   
```javascript
const str3 = `hello
                world`
```
``的拼接方式与""不同
```javascript
console.log("我要"+a+"块钱")
console.log(`我要${a}块钱`)
```

## 字符串与数值拓展
### 字符串
```javascript
let {log} = console;
let str = "Tom";
//判断字符串中是否存在指定字符 返回true或者false
let res = str.includes("opm");
//判断字符串中以指定字符开头 返回true或者false
res = str.startsWith("p")
//判断字符串中以指定字符结尾 返回true或者false
res = str.endsWith("m")
//repeat()   将字符串重复N次，返回一个新的字符串
res = str.repeat(3)//TomTomTom
res = str.repeat(2.5)//TomTom
res = str.repeat(0)//""
log(res);
```
### 数值
```javascript
let{log} = console;
//Number.isFinite()  判断被传入的内容是否为有限数值
let res = Number.isFinite(100);//true
res = Number.isFinite(100/0);//false
res = Number.isFinite(Infinity);//false
res = Number.isFinite(NaN);//false
res = Number.isFinite("100")//false

//Number.isInteger()  判断被传入的内容是否为整数
res = Number.isInteger(100);//true
res = Number.isInteger(100.0);//true
res = Number.isInteger(100.1);//false
res = Number.isInteger("Tom");//false
res = Number.isInteger("100")//false

//Math.trunc() 将括号内的参数转化为数字再去掉小数点
res = Math.trunc(1.2);//1
res = Math.trunc(1.8);//1
res = Math.trunc(-1.8);//-1
res = Math.trunc("Tom");//NaN
res = Math.trunc("10.2");//10

//Math.sign() 判断括号内的数是正数负数还是0
res = Math.sign(200);//1
res = Math.sign(-200);//-1
res = Math.sign(0);//0
res = Math.sign(-0);//-0
res = Math.sign("asld");//NaN
```

## 数组拓展
```javascript
//...扩展运算符:
 let arr = [1,2,3];
    let arr2 = [4,5,6];
    let res =[...arr,...arr2];
    console.log(res);//1，2，3，4，5，6
function test(){
    //arguments  是函数参数的集合  是个伪数组
    console.log(arguments);
    //Array.from()  将  伪数组 转换为 真实数组
    res = Array.from(arguments);
    console.log(res);
}
test(1,2,3)
```
document.querySelectorAll()  通过选择器获取所有相关元素 返回的是NodeList 伪数组
document.querySelector()  通过选择器获取首个相关元素 直接返回元素对象
find()  主要用于查找一个符合条件的数组元素
findIndex()  主要用于查找一个符合条件的数组元素的下标
它的参数是一个回调函数 在回调函数中可以制定寻找元素的条件
当条件成立为true时间。返回该元素，如果没有符合的条件，返回undefined
```javascript
let arr3 = [11,22,33,44,55]
res = arr3.find(function(item){//item 表示数组内的每一个成员
    return item>20;
    console.log(item);
})
console.log(lis);
```
fill()  使用括号内的参数，直接在数组内填充数据（替换和添加）
第一个参数 是替换的新value值
第二个参数 是替换的起始坐标
第三个参数 是替换的结束坐标
```javascript
let arr5 = [1,"纯爱","纯爱"];
arr5.fill("牛头人",1,2);
console.log(arr5);
```
## 对象拓展
```javascript
<script>
    let name = "张三"
    let obj = {
        name:name,
        fn:function(){}
    }
    console.log(obj)
    //如果对象的属性和value的变量相同，就可以只写一个属性名
    let obj2 = {
        name, //name:name
        getMessage(){},
        getList(){}
    }
    console.log(obj2)

    let msg = "class";
    //对象的属性名可以使用表达式 需要用到符号 [表达式]
    let obj3 = {
        [msg+"one"]:"Tom",
        [`${msg}xxx`](){}
    }
    console.log(obj3)

    let obj4 = {
        name:"jerry",
        age:14
    }
    //对象内可以使用拓展运算符
    let obj5 = {
        ...obj4//相当于通过for in遍历，属于深拷贝
    }
    console.log(obj5)
    //Object.assign()  将需要操作的元素复制到目标对象中
    //第一个参数  目标参数
    //第二个参数及后面所有额参数都是  需要操作的对象
    //如果复制的数据是 值类型数据 实现的是深拷贝
    //如果复制的数据是 引用类型数据 使用的任然是该数据的指针(引用地址)，实现的是浅拷贝(重点)
    let obj6 = {};
    let obj7 = {
        name:"ggBANG",//值类型深拷贝
        friends:["小A"]//引用类型浅拷贝
    }
    Object.assign(obj6,obj7);
    obj6.age = 20;
    obj6.friends = "小王";
    console.log(obj6)
    console.log(obj7)
</script>
```

## 函数拓展
```javascript
<script>
        function fn(){}
        let fn2 = function(){}
        
        // =>
        const fn3 = (a,b) =>{
            console.log(a+b)
        }
        fn3(1,2)

        // xxx.onclick = ()=>{}
        // setInterval(()=>{})

        //传入一个参数a, 并且a为函数的返回值
        //只有一个参数的情况下 小括号可以省略
        //函数内 只有一行需要执行的代码的时候 花括号可以省略
        const fn4 = a =>a;
        fn4(10)

        //箭头函数制定参数默认值
        const fn5 = (a=20,b=50) =>{
            console.log(a,b);
        }
        fn5()
        let div = document.querySelector('div');
        //函数区别
        //1.箭头函数的this指向 函数的声明处
        //2.箭头函数中无法使用arguments
        //3.箭头函数不可以作为构造函数 使用
        div.onclick= () =>{
            console.log(this);
        }
        // const fn6 = () =>{
        //     console.log(arguments);
        // }
        // fn6(1,2,3)
        const Tom = () =>{
            console.log("123");
        }
        new Tom()
    </script>
```

## Symbol
Symbol：表示独一无二的值
```javascript
        var names = Symbol();
        console.log(names);
        //使用symbol 当作对象名
        var obj = {
            [names]:"Tom"
        }
        console.log(obj);
        //Symbol 函数可以接收一个字符串作为字符串，表示堆Symbol实例的描述
        //主要是为了在控制台上显示，比较容易区分
        const ages = Symbol("age")
        console.log(ages)
        var obj2 = {
            [ages]:18,
            [names]:"Tom"
        }
        //获取带有Symbol格式的属性
        console.log(obj2[ages]);//不要加.了
```
Reflect 是一个内置对象的反射机制，用来提供方法 拦截js的操作
Reflect.ownKeys(obj)返回一个数组
forEach（）实现遍历的方法
```javascript
Reflect.ownKeys(obj).forEach(function(item){
    console.log(item);
})
```

## Iterator接口
Iterator接口的作用
为各种数据结构，提供一个统一的，简单的访问接口
使数组格式的成员能够按照某种次序排列
es6创造了一种新的遍历命令 for of 循环， Iterator接口主要供for...of循环
Iterator接口遍历的过程
创建一个指针对象，指向当前数据结构的起始位置,
第一次调用指针对象的next方法，可以将指针指向该数据结构的第一个成员
第二次调用指针对象的next方法，指针就指向该数据结构的第二个成员
不断调用指针对象的next方法，直到他指向数据结构的结束位置
```javascript
let item = arr[Symbol.iterator]();
console.log(item);
console.log(item.next());
console.log(item.next());
```
ES6规定，默认的Iterator接口部署在数据结构[Symbol.iterator]属性
或者说 只要有 数据结构 具备Symbol.iterator属性 就认为是可遍历的
Symbol.iterator属性本身就是一个函数，就是当前数据结构默认的遍历器生成函数
执行这个函数就会返回一个遍历器
原生默认具备 iterator 接口的数据结构如下:
Array Set Map String arguments NodeList

## Set数据结构
Set类似于数组，成员的值是唯一的，没有重复的值
Set.size()  返回Set实例的成员总数
Set.add()   添加Set成员
Set.delete()删除Set成员
Set.has()   查看括号内的成员是否在Set中存在 返回布尔类型
```javascript
let s1 = new Set([1,2,3,4]);
console.log(s1);
let s2 = new Set();
s2.add('hello');
s2.add('world');
s2.delete("world");
s2.clear();
```
Set遍历 
Set.keys()   返回键名的遍历器
Set.values()   返回键值的遍历器
Set.entries()    返回键值对的遍历器
```javascript
let result =s2.keys();
console.log(result); 
result =s2.values();
console.log(result); 
result =s2.entries();   
console.log(result);     
s1.forEach(function(item){
    console.log(item);
})
```

### 数组去重
方法一：
```javascript
let arr = [19,20,19,19,20,21,30,90];
let s1 = new Set(arr);
console.log(s1);//输出的是一个对象
//需要转换为数组的话
s1 = Array.from(s1);//使用Array.from() 转换成数组
console.log(s1);
```
方法二：
```javascript
let s2 = [...new Set(arr)];//...扩展运算符
console.log(s2);
```

## Map
```javascript
<script>
    //Map 类型对象  键值对的集合，但是Map中的key不限于字符串，可以是各种类型的值
    let M1 = new Map();
    M1.set('name','tom');
    M1.set({a:1},'tom');
    M1.set('big','small');
    //操作方法
    //Map.set(key.value);  在Map对象中添加key和对应的value
    //Map.get(key);        在Map对象中获取key对象的value
    //Map.delete(key);     删除指定的key
    //Map.has(key);        查看key是否在map中存在  返回布尔类型
    //Map.clear();        清空
    console.log(M1);
    console.log(M1.get("name"));
    M1.delete("name");
    console.log(M1);
    let res = M1.has('big');
    console.log(res);
    //Map遍历方法 与 Set相同
    res = M1.keys()
    res = M1.values()
    res = M1.entries()
    console.log(M1);
    console.log(res);
</script>
```

## Proxy
Object.defineProperty()  拦截并处理数据
第一个参数  需要拦截处理的对象
第二个对象  对象内的属性
第三个对象  {}配置项，格式是个对象
```javascript
let obj = {
            data: 111,
            name:"zs",
            age:20
        }
let box = document.getElementById("box")
Object.defineProperty(obj, "data", {
    get() {//  当使用对象内的指定属性时调用
        console.log("get函数调用");
    },
    set(value){
        console.log("set函数接收到了:",value);
        if(value>=1000){
            box.innerHTML =`数据较大,请重新输入`;
        }
        else{
            box.innerHTML = `数据合理放心使用`;
        }
    }
})
```
Proxy代理:
```javascript
let proxy = new Proxy(obj,{
    get(target,key){
        //target 表示 需要代理的对象
        //key  表示的是访问的属性
        console.log(target,key);
        return target[key];
    },
    set(target,key,value){
        //target 表示 需要代理的对象
        //key  表示的是设置的属性
        //value  设置的新值
        console.log("set:",target,key,value);
        target[key] = value//确认操作
    }
})
```
## Reflect
Reflect 主要用来获取目标对象的行为,它与Object类似，但更容易读

## Promise对象
### 回调地狱
当一个回调函数嵌套另一个回调函数的时候
就会出现嵌套结构
当嵌套结构多的时候，就会出现回调地狱的情况
回调地狱 其实就是由多个回调函数互相嵌套导致的，代码维护性非常差
### 同步异步
异步  当一行代码还没有执行结束，就可以去执行另一行代码的顺序 叫做异步
同步  当代码逐行执行过程就是同步的过程
异步的操作：定时器 callback
promise  是异步编程的一种统一的解决方案，比传统回调函数，更合理更强大
```javascript
const api = new Promise(function(resolve,reject){
    setTimeout(()=>{
        if(true){
            resolve()//resolve  表示成功的回调函数
        }
        else{
            reject()//reject 表示失败的回调函数
        }
    },1000)
})

api.then(()=>{
    console.log("yyyyy");//.then() 成功
})
.catch(()=>{
    console.log("nnnnn");//.catch() 失败
})
```
Promise 对象通过自身的状态，来控制异步操作。Promise实例具有三种状态
异步操作未完成(pending)
异步操作完成(fulfilled)
异步操作失败(rejected)

### 链式调用
为什么promise可以实现链式调用
因为当promise方法执行结束后仍然会返回一个promise对象
```javascript
const a = promise.then((res) => {
    console.log("成功赚取" + res + "元");
    return 2000//传参
}).then((res)=>{
    console.log("成功赚取" + res + "元");
    return 3000//传参
}).then((res)=>{
    console.log("成功赚取" + res + "元");
    return 5000//传参
}).then((res)=>{
    console.log("成功赚取" + res + "元");
}).catch((err)=>{
    console.log("失败啦",err);
})
console.log(a)
```

### all方法
Promise.all() 将多个Promise实例  包装成一个新的promise实例
要求P1,P2,P3的状态都是fulfilled pAll的状态才是 fulfilled
此时P1,P2,P3的返回值组成一个数组，传递给pAll中

### race方法
Promise.race() 将多个Promise实例  包装成一个新的promise实例
要求P1,P2,P3的状态只要有一个fulfilled pRace的状态才是 fulfilled
此时返回值是首次达到fulfilled状态的值
除非全部reject否则不会触发.catch

## Generator函数
ES6 提供的一种异步编程解决方案
```javascript
function *gen(){
    console.log(1);
    yield;//yield 表示暂停执行标记，通过next方法恢复执行
    console.log(2);
    yield;
    console.log(3);
}
let g = gen()//next() 驱动下一步的执行
g.next()
g.next()
g.next()
console.log(g);
```

## Class语法与继承
```javascript
class Person{//创建一个Person类 也叫做 构造函数
    //类中的属性需要使用constructor 构造器创建
    constructor(name,age,height){
        this.name = name;
        this.age = age;
        this.height = height;
    }
    // 在类中创造方法
    say(){
        console.log("这是Person类");
    }
}
let obj = new Person("zhangsan",19,"180cm");
console.log(obj);
obj.say()
```
class继承
```javascript
class Person{
    constructor(name,age){
        this.name = name;
        this.age = age;
    }
    say(){
        console.log("这是",this.name,this.age,"岁");
    }
}
const one = new Person("张飞",100);
one.say();
class Student extends Person{//extends 表示StudentPerson类中继承
    constructor(name,age,height){
        super(name,age);//super() 表示从父类中继承的属性内容 必须写在 construtor中
        this.height = height;
    }
    say(){
        super.say();
        console.log("是学生")
    }
}
let obj = new Student('xz',12,'120');
console.log(obj);
obj.say();
```
