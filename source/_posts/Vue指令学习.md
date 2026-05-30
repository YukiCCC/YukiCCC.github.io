---
title: Vue指令学习
date: 2023-07-24 14:34:03
tags: Vue
category: 前端
---
## Vue构成
vue 是一个完全支持组件化开发的框架。vue 中规定组件的后缀名是 .vue。之前接触到的 App.vue 文件本质上就是一个 vue 的组件。每个 .vue 组件都由 3 部分构成，分别是：
| 模板结构 | 功能                   |
| -------- | ---------------------- |
| template | 组件的模板结构         |
| script   | 组件的 JavaScript 行为 |
| style    | 组件的样式             |

其中，每个组件中必须包含 template 模板结构，而 script 行为和 style 样式是可选的组成部分。vue 规定：每个组件对应的模板结构，需要定义到 template 节点中。

### 在 template 中使用指令
在组件的 template 节点中，支持使用前面所学的指令语法，来辅助开发者渲染当前组件的 DOM 结构。代码示例如下：
```jsx
<template>
<h1>这是App根组件</h1>
<!-- 使用{{ }}插值表达式 -->
<p>生成一个随机数字: {{ (Math. random() * 10). toFixed(2) }}</p>
<!-- 使用v-bind属性绑定 -->
<p :title="new Date().tolocaleTimeString()">我在学习vue.js</p>
<!-- 属性v-on事件绑定 -->
<button @click=”showInfo">按钮</button>
</template>
```
### 在 template 中定义根节点
在 vue 2.x 的版本中，template 节点内的 DOM 结构仅支持单个根节点：

```jsx
<template> 
<!-- vue 2.x中，template节点内的所有元素，最外层“必须有”唯一的根节点进行包裹，否则报错 -->
<div>
<h1>这是App根组件</h1>
<h2>这是副标题</h2>
</div>
</template>
```
但是，在 vue 3.x 的版本中，< template> 中支持定义多个根节点：

```jsx
<template>
<!-- 这是包含多个根节点的template结构，因为h1标签和h2标签外层没有包裹性质的根元素 -->
<h1>这是App根组件</h1>
<h2>这是副标题</h2>
</template>
```

### 组件的 script 节点
vue 规定：组件内的 < script> 节点是可选的，开发者可以在 < script> 节点中封装组件的 JavaScript 业务逻辑。< script > 节点的基本结构如下：

```jsx
<script>
//今后，组件相关的data数据、methods方法等，都需要定义到export default所导出的对象中。
export default {}
</script>
```

### script 中的 name 节点
可以通过 name 节点为当前组件定义一个名称，代码如下：

```jsx
<script>
export default {
// name属性指向的是当前组件的名称（建议：每个单词的首字母大写）
name: 'MyApp'
}
</script>
```
在使用 vue-devtools 进行项目调试的时候，自定义的组件名称可以清晰的区分每个组件。

### script 中的 data 节点
vue 组件渲染期间需要用到的数据，可以定义在 data 节点中：

```jsx
<script>
export default {
// 组件的名称
  name: 'MyApp',
// 组件的数据（data方法中return出去的对象，就是当前组件渲染期间需要用到的数据对象）
  data() {
    return {
        username: '哇哈哈',
    }
  },
}
</script>
```
其中组件中的 data 必须是函数，vue 规定：组件中的 data 必须是一个函数，不能直接指向一个数据对象。因此在组件中定义 data 数据节点时，下面的方式是错误的：
```jsx
data: { // 组件中，不能直接让 data 指向一个数据对象（会报错）
    count: 0
}
```
### script 中的 methods 节点
组件中的事件处理函数，必须定义到 methods 节点中，示例代码如下：
```jsx
export default {
  name: 'MyApp', // 组件的名称
  data() { // 组件的数据
    return {
      count: 0,
    }
  },
  methods: { //处理函数
    addCount() { 
      this.count++
    },
  },
}
```
### 组件的 style 节点
vue 规定：组件内的 < style > 节点是可选的，开发者可以在 < style > 节点中编写样式美化当前组件的 UI 结构。< script > 节点的基本结构如下：
```jsx
<style>
h1 {
font-weight: normal;
}
</style>
```
其中 < style > <="" font=""> 标签上的 lang="css" 属性是可选的，它表示所使用的样式语言。默认只支持普通的 css 语法，可选值还有 less、scss 等。
多学一招：让 style 中支持 less 语法：
如果希望使用 less 语法编写组件的 style 样式，可以按照如下两个步骤进行配置：
① 运行 npm install less -D 命令安装依赖包，从而提供 less 语法的编译支持；
② 在 < style > <="" font=""> 标签上添加 lang="less" 属性，即可使用 less 语法编写组件的样式。
```jsx
<style>
h1 {
font-weight: normal;
}
i {
color: red;
font-style: normal;
}
</style>
```

## Vue指令

### 插值表达式

> 目的: 在dom标签中, 直接插入内容

又叫: 声明式渲染/文本插值

语法: {{ 表达式 }}

```jsx
<template>
  <div>
    <h1>{{ msg }}</h1>
    <h2>{{ person.name }}</h2>
    <h2>{{ person.age > 100 ? `老头` : `小伙` }}</h2>
  </div>
</template>

<script>
export default {
  // data 格式固定, 定义vue数据之处
  data() { 
    // key相当于变量名
    return {  
      msg: `hello vue!!!`,
      person: {
        name: `张三丰`,
        age: 108,
      },
    }
  },
}
</script>

<style></style>
```

> 总结: dom中插值表达式赋值, vue的变量必须在data里声明
注意点：
1.在插值表达式中使用的数据 必须在data中进行了提供
2.支持的是表达式，而非语句，比如：if for ...
3.不能在标签属性中使用插值（插值表达式只能在标签中间使用），比如：```<p title="{{username}}">我是P标签</p>```




### v-bind

> 目标: 给标签属性设置vue变量的值

**vue指令, 实质上就是特殊的 html 标签属性, 特点:  v- 开头**

每个指令, 都有独立的作用

- 语法：`v-bind:属性名="vue变量"`

- 简写：`:属性名="vue变量"`




```html
<!-- vue指令-v-bind属性动态赋值 -->
<a v-bind:href="url">我是a标签</a>
<img :src="imgSrc">

图片服务器:

https://demo-video-oss.oss-cn-hangzhou.aliyuncs.com/images/mm.jpg
或 指定尺寸 
https://demo-video-oss.oss-cn-hangzhou.aliyuncs.com/images/mm.jpg?x-oss-process=image/resize,w_60,h_60
```

> 总结: 把vue变量的值, 赋予给dom属性上, 影响标签显示效果

### v-on

> 目标: 给标签绑定事件

* 语法
  * v-on:事件名="要执行的 少量代码 "
  * v-on:事件名="methods中的函数"
  * v-on:事件名="methods中的函数(实参)" 
  
* 简写: @事件名="methods中的函数"


```html
<!-- vue指令:   v-on事件绑定-->
<p>你要买商品的数量: {{count}}</p>
<button v-on:click="count = count + 1">增加1</button>
<button v-on:click="addFn">增加1个</button>
<button v-on:click="addCountFn(5)">一次加5件</button>

<button @click="subFn">减少</button>

<script>
    export default {
        // ...其他省略
        methods: {
            addFn(){ // this代表export default后面的组件对象(下属有data里return出来的属性)
                this.count++
            },
            addCountFn(num){
                this.count += num
            },
            subFn(){
                this.count--
            }
        }
    }
</script>
```

> 总结: 常用@事件名, 给dom标签绑定事件, 以及=右侧事件处理函数

### v-on事件对象

> 目标: vue事件处理函数中, 拿到事件对象

* 语法:
  * 无传参, 通过形参直接接收
  * 传参, 通过$event指代事件对象传给事件处理函数

```vue
<template>
  <div>
    <a @click="one" href="http://www.baidu.com">阻止百度</a>
    <hr>
    <a @click="two(10, $event)" href="http://www.baidu.com">阻止去百度</a>
  </div>
</template>

<script>
export default {
  methods: {
    one(e){
      e.preventDefault()
    },
    two(num, e){
      e.preventDefault()
    }
  }
}
</script>
```

### v-on修饰符

> 目的: 在事件后面.修饰符名 - 给事件带来更强大的功能

* 语法:
  * @事件名.修饰符="methods里函数"
    * .stop - 阻止事件冒泡
    * .prevent - 阻止默认行为
    * .once - 程序运行期间, 只触发一次事件处理函数

```html
<template>
  <div @click="fatherFn">
    <!-- vue对事件进行了修饰符设置, 在事件后面.修饰符名即可使用更多的功能 -->
    <button @click.stop="btn">.stop阻止事件冒泡</button>
    <a href="http://www.baidu.com" @click.prevent="btn">.prevent阻止默认行为</a>
    <button @click.once="btn">.once程序运行期间, 只触发一次事件处理函数</button>
  </div>
</template>

<script>
export default {
  methods: {
    fatherFn(){
      console.log("father被触发");
    },
    btn(){
      console.log(1);
    }
  }
}
</script>
```

> 总结: 修饰符给事件扩展额外功能

### v-on按键修饰符

> 目标: 给键盘事件, 添加修饰符, 增强能力

* 语法:
  * @keyup.enter  -  监测回车按键
  * @keyup.esc     -   监测返回按键

[更多修饰符](https://v2.cn.vuejs.org/v2/guide/events.html#%E6%8C%89%E9%94%AE%E7%A0%81) 

```html
<template>
  <div>
    <input type="text" @keydown.enter="enterFn">
    <hr>
    <input type="text" @keydown.esc="escFn">
  </div>
</template>

<script>
export default {
 methods: {
   enterFn(){
     console.log("enter回车按键了");
   },
   escFn(){
     console.log("esc按键了");
   }
 }
}
</script>
```

> 总结: 多使用事件修饰符, 可以提高开发效率, 少去自己判断过程



### 例1：翻转字符串

> 目标: 点击按钮 - 把文字取反显示 - 再点击取反显示(回来了)

> 提示: 把字符串取反赋予回去



正确代码:

```html
<template>
  <div>
    <h1>反转字符串</h1>
    <p>{{ msg }}</p>
    <button @click="reverseStr">反转字符串</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msg: `我是我爹的儿`,
    }
  },

  methods: {
    reverseStr() {
      this.msg = this.msg.split('').reverse().join('')
    },
  },
}
</script>

<style scoped></style>
```

> 总结: 记住方法特点, 多做需求, vue是数据变化视图自动更新, 减少操作DOM时间, 提高开发效率

### v-model

> 目标: 把value属性和vue数据变量, 双向绑定到一起

* 语法: v-model="vue数据变量"
* 双向数据绑定
  * 数据变化 -> 视图自动同步
  * 视图变化 -> 数据自动同步
* 演示: 用户名绑定 - vue内部是MVVM设计模式

```vue
<template>
  <div>
    <!-- 
    	v-model:是实现vuejs变量和表单标签value属性, 双向绑定的指令
    -->
    <div>
      <span>用户名:</span>
      <input type="text" v-model="username" />
    </div>
    <div>
      <span>密码:</span>
      <input type="password" v-model="pass" />
    </div>
    <div>
      <span>来自于: </span>
      <!-- 下拉菜单要绑定在select上 -->
      <select v-model="from">
        <option value="北京市">北京</option>
        <option value="南京市">南京</option>
        <option value="天津市">天津</option>
      </select>
    </div>
    <div>
      <!-- (重要)
      遇到复选框, v-model的变量值
      非数组 - 关联的是复选框的checked属性
      数组   - 关联的是复选框的value属性
       -->
      <span>爱好: </span>
      <input type="checkbox" v-model="hobby" value="抽烟">抽烟
      <input type="checkbox" v-model="hobby" value="喝酒">喝酒
      <input type="checkbox" v-model="hobby" value="写代码">写代码
    </div>
    <div>
      <span>性别: </span>
      <input type="radio" value="男" name="sex" v-model="gender">男
      <input type="radio" value="女" name="sex" v-model="gender">女
    </div>
    <div>
      <span>自我介绍</span>
      <textarea v-model="intro"></textarea>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: "",
      pass: "",
      from: "",
      hobby: [], 
      sex: "",
      intro: "",
    };
    // 总结:
    // 特别注意: v-model, 在input[checkbox]的多选框状态
    // 变量为非数组, 则绑定的是checked的属性(true/false) - 常用于: 单个绑定使用
    // 变量为数组, 则绑定的是他们的value属性里的值 - 常用于: 收集勾选了哪些值
  }
};
</script>
```

> 总结: 本阶段v-model只能用在表单元素上, 以后学组件后讲v-model高级用法

### v-model修饰符

> 目标: 让v-model拥有更强大的功能

* 语法:
  * v-model.修饰符="vue数据变量"
    * .number   以parseFloat转成数字类型
    * .trim          去除首尾空白字符  （记下：去除字符串中间 空格 不是vue ）
    * .lazy           在change事件时触发而非inupt触发  时

```vue
<template>
  <div>
    <div>
      <span>年龄:</span>
      <input type="text" v-model.number="age">
    </div>
    <div>
      <span>人生格言:</span>
      <input type="text" v-model.trim="motto">
    </div>
    <div>
      <span>自我介绍:</span>
      <textarea v-model.lazy="intro"></textarea>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      age: "",
      motto: "",
      intro: ""
    }
  }
}
</script>
```

> 总结: v-model修饰符, 可以对值进行预处理, 非常高效好用

### v-text和v-html

> 目的: 更新DOM对象的innerText/innerHTML

* 语法:
  * v-text="vue数据变量"    
  * v-html="vue数据变量"
* 注意: 会覆盖插值表达式

```vue
<template>
  <div>
    <p v-text="str"></p>
    <p v-html="str"></p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      str: "<span>我是一个span标签</span>"
    }
  }
}
</script>
```

> 总结: v-text把值当成普通字符串显示, v-html把值当做html解析

### v-show和v-if

> 目标: 控制标签的隐藏或出现

* 语法:
  * v-show="vue变量"            
  * v-if="vue变量" 
* 原理
  * v-show 用的display:none隐藏   (频繁切换使用)
  * v-if  直接从DOM树上移除
* 高级
  * v-else使用

```html
<template>
  <div>
    <h1 v-show="isOk">v-show的盒子</h1>
    <h1 v-if="isOk">v-if的盒子</h1>

    <div>
      <p v-if="age > 18">我成年了</p>
      <p v-else>还得多吃饭</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isOk: true,
      age: 15
    }
  }
}
</script>
```

> 总结: 使用v-show和v-if以及v-else指令, 方便通过变量控制一套标签出现/隐藏

### 例2：折叠面板

> 目标: 点击展开或收起时，把内容区域显示或者隐藏

此案例使用了less语法, 项目中下载模块

```bash
yarn add less@3.0.4 less-loader@5.0.0 -D
```

只有标签和样式

```vue
<template>
  <div id="app">
    <h3>案例：折叠面板</h3>
    <div>
      <div class="title">
        <h4>芙蓉楼送辛渐</h4>
        <span class="btn" >
          收起
        </span>
      </div>
      <div class="container">
        <p>寒雨连江夜入吴,</p>
        <p>平明送客楚山孤。</p>
        <p>洛阳亲友如相问，</p>
        <p>一片冰心在玉壶。</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      
    }
  }
}
</script>

<style lang="less">
body {
  background-color: #ccc;
  #app {
    width: 400px;
    margin: 20px auto;
    background-color: #fff;
    border: 4px solid blueviolet;
    border-radius: 1em;
    box-shadow: 3px 3px 3px rgba(0, 0, 0, 0.5);
    padding: 1em 2em 2em;
    h3 {
      text-align: center;
    }
    .title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      border: 1px solid #ccc;
      padding: 0 1em;
    }
    .title h4 {
      line-height: 2;
      margin: 0;
    }
    .container {
      border: 1px solid #ccc;
      padding: 0 1em;
    }
    .btn {
      /* 鼠标改成手的形状 */
      cursor: pointer;
    }
  }
}
</style>
```

参考答案:

```vue
<template>
  <div id="app">
    <h3>案例：折叠面板</h3>
    <div>
      <div class="title">
        <h4>芙蓉楼送辛渐</h4>
        <span class="btn" @click="isShow = !isShow">
          {{ isShow ? '收起' : '展开' }}
        </span>
      </div>
      <div class="container" v-show="isShow">
        <p>寒雨连江夜入吴, </p>
        <p>平明送客楚山孤。</p>
        <p>洛阳亲友如相问，</p>
        <p>一片冰心在玉壶。</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isShow: false
    }
  }
}
</script>

```



### v-for

> 目标: 列表渲染, 所在标签结构, 按照数据数量, 循环生成

* 语法
  * v-for="(值, 索引) in 目标结构"
  * v-for="值 in 目标结构"
  
* 目标结构:
  
  * 可以遍历数组 / 对象 / 数字 / 字符串 (可遍历结构)
  
* 注意:

  v-for的临时变量名不能用到v-for范围外

```vue
<template>
  <div id="app">
      <!-- v-for 把一组数据, 渲染成一组DOM -->
      <!-- 口诀: 让谁循环生成, v-for就写谁身上 -->
      <p>学生姓名</p>
      <ul>
        <li v-for="(item, index) in arr" :key="item">
          {{ index }} - {{ item }}
        </li>
      </ul>

      <p>学生详细信息</p>
      <ul>
        <li v-for="obj in stuArr" :key="obj.id">
          <span>{{ obj.name }}</span>
          <span>{{ obj.sex }}</span>
          <span>{{ obj.hobby }}</span>
        </li>
      </ul>

      <!-- v-for遍历对象(了解) -->
      <p>老师信息</p>
      <div v-for="(value, key) in tObj" :key="value">
        {{ key }} -- {{ value }}
      </div>

      <!-- v-for遍历整数(了解) - 从1开始 -->
      <p>序号</p>
      <div v-for="i in count" :key="i">{{ i }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      arr: ["小明", "小欢欢", "大黄"],
      stuArr: [
        {
          id: 1001,
          name: "孙悟空",
          sex: "男",
          hobby: "吃桃子",
        },
        {
          id: 1002,
          name: "猪八戒",
          sex: "男",
          hobby: "背媳妇",
        },
      ],
      tObj: {
        name: "蜘蛛精",
        age: 18,
        class: "昆虫",
      },
      count: 10,
    };
  },
};
</script>
```

> 总结: vue最常用指令, 铺设页面利器, 快速把数据赋予到相同的dom结构上循环生成


