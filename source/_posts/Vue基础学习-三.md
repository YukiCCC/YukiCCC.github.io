---
title: Vue基础学习(三)
date: 2023-08-15 12:58:17
tags: Vue
category: 前端
---
## 一、组件化与模块化

### 1. 组件化

​	组件化，是从UI界面的角度进行划分的，将页面构成拆分为一个个组件，方便UI组件的重用

​	组件包含：页面构成（template）、样式（css）、功能（js）

### 2. 模块化

​	模块化，是从代码逻辑的角度进行划分的，将项目按照一定的规则拆分成一个个模块，进行组合使用

​	方便代码的分层开发，保证每个功能模块的职能单一

<img src="https://raw.githubusercontent.com/YukiCCC/figure/main/image-20190811182608043.png" width="600px">

## 二、构建模块化环境

### 1. 开发环境

模块化开发一般需要独立的开发环境，一般都基于 Node.js 和 webpack 进行环境搭建

+ Node.js 独立JS运行环境和模块系统支持
+ webpack 自动化的前端构建工作流工具，类似于gulp，但功能更强大

Vue模块化项目环境搭建，可以通过官方提供的环境构建工具 `Vue CLI` ，称为Vue脚手架

### 2. Vue脚手架

#### 2.1 简介

 `@vue/cli`是一个Vue脚手架，用来快速构建模块化项目结构

+ cli：command line interface 命令行接口
+ 通过命令行的方式快速构建Vue模块化项目运行环境

#### 2.2 安装脚手架

系统环境：

+ node环境：`node -v`
+ npm环境：`npm -v`
+ nrm环境：` nrm ls`

安装@vue/cli：

+ 全局安装：`npm install @vue/cli -g` （需要具有管理员权限）
+ 验证安装：`vue --version`
+ 查看帮助：`vue --help`

常用命令：

- 创建项目：`vue create 项目名`　或　`vue ui`
- 为项目增加扩展插件：`vue add 插件`

### 3. 使用脚手架创建项目

切换到项目的存放目录，执行 `vue create 项目名称`，进入交互模式

```bash
# 1.选择构建模式
Vue CLI v4.1.2
? Please pick a preset: (Use arrow keys)
	default (babel, eslint)  # 默认选项，只包含基础依赖 ——> 直接执行第6步
❯ Manually select features # 自定义环境，手动选择特性依赖
  
# 2.自定义环境的选择
Vue CLI v4.1.2
? Please pick a preset: Manually select features
? Check the features needed for your project: (Press <space> to select, <a> to toggle all, <i> to invert selection) # 选择需要的环境特性 
❯◉ Babel # ES6语法兼容转换器（Babel是一个广泛使用的转码器，可以将ES6代码转为ES5代码，避免运行环境不支持）
 ◯ TypeScript # TS语法支持
 ◯ Progressive Web App (PWA) Support # 构建渐进式Web应用，优化用的
 ◯ Router # 集成路由功能
 ◯ Vuex # 集成统一数据状态管理器
 ◉ CSS Pre-processors # CSS预编译，支持使用LESS、SASS等动态样式语言
 ◉ Linter / Formatter # 代码规范检查和格式化检测
 ◯ Unit Testing # 单元测试
 ◯ E2E Testing # 端到端测试
 
# 3.所选环境的进一步配置
# 如果选择了CSS Pre-processors，则需要选择 动态样式语言
? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): (Use arrow keys)
	Sass/SCSS (with dart-sass) 
  Sass/SCSS (with node-sass) 
❯ Less 
  Stylus 
# 如果选择Linter / Formatter，则需要选择 代码规范检查的规则
? Pick a linter / formatter config: (Use arrow keys)
❯ ESLint with error prevention only # 使用ESLint，仅检测错误（ESLint是用来统一代码规范和风格的工具）
  ESLint + Airbnb config # 使用 eslint+airbnb 前端规范
　ESLint + Standard config # 使用标准规范
  ESLint + Prettier # 使用严格规范
# 选择代码规范检查的时机
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◉ Lint on save # 文件保存时
 ◯ Lint and fix on commit # 代码整理和提交时
 
# 4.选择工具配置文件的存放位置
? Where do you prefer placing config for Babel, PostCSS, ESLint, etc.? (Use arrow keys)
❯ In dedicated config files  # 存放在独立的配置文件中
  In package.json # 集成在package.json文件中
  
# 5.是否将上述配置保存为一个预设选项，方便以后创建项目使用
? Save this as a preset for future projects? Yes
? Save preset as: mypreset

# 6.创建项目，安装所需依赖
Vue CLI v4.1.2
✨  Creating project in /Users/appleuser/Desktop/aa-aa.
🗃  Initializing git repository...
⚙  Installing CLI plugins. This might take a while...
.......
🎉  Successfully created project first-project.
👉  Get started with the following commands:
```

### 4. 启动项目

切换到项目的根目录，执行 `npm run serve` 启动开发服务器，然后根据提示url访问

+ `npm run 命令名称` 是npm内置的脚本执行命令，该命令会自动搜索执行目录下`package.json`文件中的`scripts`对应的命令并执行

## 三、项目结构

### 1. 文件结构

​	通过多个关联文件构成模块化开发环境，最终由webpack工具进行打包合并

```bash
|-项目名称
	|-node_modules 	# 项目运行所依赖的模块，内容较多，文件较大，拷贝项目时一般不拷贝此目录，可通过npm install自动下载安装
	|-public				# 项目的静态资源目录，如html、css、js、image等，服务器访问的根目录，对外公开的资源
		|-index.html 	# 默认访问的页面文件
	|-src  					# 源代码目录 
		|-assets 					# 静态资源目录
    	|-components  		# 自定义组件目录
    	|-App.vue  			  # 项目的主组件
    	|-main.js  			  # 项目的启动文件
    |-.browserslistrc  	# 项目的浏览器适配版本
    |-.eslintrc.js  		# eslint配置文件，定义语法校验规则
    |-.gitignore  			# git配置文件，定义忽略文件列表
    |-babel.config.js  	# babel配置文件，定义ES语法转换规则
    |-package-lock.json # 对项目的安装模块信息进行本地记录
    |-package.json  		# node项目的描述文件，定义项目的相关描述信息
    |-README.md  			  # 项目的说明文件，对项目进行说明介绍
```

### 2. 核心文件详解

​	Vue模块化项目启动时会自动查找两个核心文件： **index.html** 和 **main.js** 

#### 2.1 index.html

​	index.html是访问应用时默认显示的页面文件，也是Vue项目的容器定义文件

#### 2.2 main.js

​	main.js是项目的启动文件，也是Vue模块化项目的整合入口文件

```javascript
/**
 * 使用ES6提供的模块导入导出功能
 */
import 模块名 from 文件路径  // 导入模块，模块名可自定义

export default {}  // 导出模块，默认导出
```

#### 2.3 App.vue

​	App.vue是Vue中提供的一种特殊的组件定义文件，这类文件用于描述模块化项目中的组件

​	App.vue是项目的主组件，其他组件都是在该组件下加载的

### 3. 单文件组件

​	以`.vue`结尾的文件，是Vue.js自定义的一种文件格式，称为单文件组件

​	一个.vue文件就是一个单独的组件，在文件内封装了该组件相关的html、css和js，实现了对一个组件的封装

​	.vue文件由三部分组成：

```vue
<template>
    <!-- html，组件的页面结构 -->
</template>

<script>
    // js，组件的功能配置
</script>

<style>
    /* css，组件的样式 */
</style>
```

##  四、模块化开发

### 1. 组件样式

全局样式和局部样式：

- 在style标签中定义的样式，默认为全局样式，在所有组件中都有效
- 可以为style标签指定scoped属性，此时定义的样式为局部样式，只在当前组件中有效
- 用法：`<style scoped>`

动态样式支持：

- 可以为style标签指定lang属性，设置当前使用的样式语言，如：css、less、sass、stylus等
- 默认使用的是css语法，可以使用动态样式语言
- 用法：`<style lang="动态样式语言">`

### 2. 脚本架配置文件

`vue.config.js` 是Vue脚手架@vue/cli的全局配置文件，是一个可选的配置文件，如果不存在则使用默认配置，如果项目的根目录中存在这个文件，则项目运行时会自动加载

编辑vue.config.js文件

```js
module.exports = {
    // 项目的基础url
	publicPath:'/film/',
	// 开发服务器的配置
	devServer: {
		port: 8888
	}
｝
```

注：修改vue.config.js文件后，需要重启项目才生效

### 3. 项目打包

当项目要部署上线时，需要先对开发环境的项目进行打包构建操作，生成一个由纯静态文件构成的项目

切换到项目的根目录，执行 `npm run build` 对项目进行打包构建操作

- 默认会在项目的根目录下生成一个dist目录，该目录中存放打包后的项目静态文件，可直接部署到生产环境中
- 打包后可将dist目录中的静态文件拷贝到任意Web服务器中部署运行

生产环境和开发环境：

- 由于生产环境和开发环境可能会有所不同，有时会导致资源无法访问，此时需要对环境进行配置
- 编辑vue.config.js文件

```js
module.exports = {
  // 项目的基础url，所有静态资源在加载时的基础路径BASE_URL
  // publicPath:'/film/',
  // 判断环境，分别指定生成环境production和开发环境development的地址
  publicPath: process.env.NODE_ENV === 'production' ? '/film/' : '/', 
  // 开发服务器的配置
  devServer: {
    port: '8888' // 使用的端口
  },
  // 打包时是否生成map文件，主要用于生成环境下调试js源代码，建议改为false
  productionSourceMap: false,
}
```

## 五、全局功能定义

### 1. 简介

用于统一定义全局的功能，如：全局组件、全局插件、全局过滤器、全局指令等

- 使用Vue提供的全局方法`Vue.use()`，一般在 main.js 文件中使用
- `Vue.use(Object)` 参数Object是一个必须提供install方法的对象，Vue.use会自动调用执行install方法

### 2. 用法

​	全局组件、全局插件

## 六、数据交互

### 1. 简介

Vue模块化项目的数据交互，使用的就是AJAX技术，发送异步请求

+ 实现异步请求的模块有很多，官方推荐使用 `axios` 模块
+ axios是一个基于Promise的HTTP请求客户端，用来发送AJAX请求

### 2. 安装axios

切换到项目的根目录，执行`	npm install axios --save `安装模块

在需要使用axios的文件中，通过`import axios from 'axios'`导入模块

导入模块后，可以使用axios对象提供的方法：

- axios.get(url[, config])
- axios.delete(url[, config])
- axios.post(url[, data[, config]])
- axios.put(url[, data[, config]])

补充：Restful请求，根据HTTP请求方式来区分对资源的操作

- GET 获取资源
- POST 新建资源
- PUT 更新资源
- DELETE 删除资源

### 3. 用法

#### 3.1 基本用法

使用`axios.get(url[,config]) `，传参方式：

- 通过url传参，即查询字符串方式
- 通过params选项传参

使用`axios.post(url[,data,[config]])`，传参方式：

- 通过查询字符串方式
- 使用qs模块进行转换`import qs from 'qs'`             
- 注：对于post方式请求，axios发送数据时默认的数据格式为Request Payload，并非我们常用的Form Data格式，所以参数不能直接以json形式传递 

#### 3.2 全局引入

全局引入axios并添加到Vue原型中，避免在组件中重复引用axios，简化使用

- 在main.js中导入axios模块
- 为Vue函数添加一个原型属性`$http`，指向axios对象
- 在组件中无需再引入axios模块，直接使用`this.$http`即可

### 4. 跨域访问

发送请求时，如果协议、主机或端口，只要有任何一个不同，则为跨域访问

两种解决方式：

- 后端允许跨越访问，设置响应头为`Access-Control-Allow-Origin`

- 在前端构建代理服务器，进行请求的转发。配置代理服务器的步骤：

1. 编辑vue.config.js文件 ，添加代理服务器配置

   ```javascript
   // 配置代理服务器，解决开发阶段的跨域问题（只在开发阶段有效）
   proxy: {
     '/data': { // 所有以/data开头的请求都需要被代理转发
       	target: 'http://127.0.0.1:80', // 转发到的目标服务器
         logLevel: 'debug' // 开启调试模式
     } 
   }
   ```

2. 重启开发服务器，查看日志输出

## 七、路由

### 1. 简介

SPA应用：Single Page Application  单页应用程序

- 就是只有一个Web页面的应用，所有的操作都在这个页面上完成（容器页面）
- 浏览器一开始会加载相应的HTML、CSS和JavaScript，然后将所有的活动都局限在该Web页面中
- 当用户与应用程序交互时通过JavaScript动态更新页面中的内容

在开发SPA单页应用时，需要在不同的组件间切换，从而实现在一个容器页面中显示不同的内容

我们可以使用Vue Router来开发单页应用，根据不同的url地址，路由跳转到不同的组件

- 路由是一种组件动态分发机制，通过url路径进行组件的切换
- 根据url路径查找对应的组件，然后将组件显示到指定的页面位置

### 2. 基本用法

创建项目时选择Router路由功能

```bash
|-项目名称
  |-node_modules
		|-vue-router  			# 路由模块
  |-src
    |-router
    	|-index.js				# 路由配置文件
    |-views							# 视图组件目录
    |-App.vue  			
    |-main.js  				
```

核心文件详解：main.js、index.js、App.vue

- 提供了两个路由组件

  `<router-link>`组件：用来定义导航，默认会创建`<a>`标签，根据指定的url路径跳转到组件

  `<router-view>`组件：用来渲染url路径对应的组件，将组件显示到当前定义的位置

### 3. 路由嵌套

当应用由多层嵌套的组件构成时，需要使用嵌套路由

关于url路径的定义：

+ 根路径

  根路径就是`/`，默认访问项目时显示的就是根路径对应的组件，即默认显示的组件

+ 嵌套路由的路径

  子级路由可以使用相对路径或绝对路径

+ 通配路径

  可以使用通配符 `*`来设置当url路径不存在时显示的组件

### 4. 路由传参

通过`<router-link>`的to属性传递参数

两种方式：

1. 查询字符串传参

   传递参数：

   - 字符串形式

     语法：`to="路径?参数名=参数值&参数名=参数值"`

   - 对象形式

     语法：`:to="{path:'组件的url路径'，query:{key:value,key:value}}"`   或

     ​		  `:to="{name:'组件的路由名称'，query:{key:value,key:value}}"`

   获取参数：

   - 使用路由对象`vm.$route.query`获取参数信息

2. Rest风格传参

   传递参数：

   - 字符串形式

     语法：`to="/路径/参数/参数"`   将参数伪装成请求地址的一部分

     在定义路由时需要在url路径中通过`:参数名`指定参数

   - 对象形式

     语法：`:to="{name:'组件的路由名称'，params:{key:value,key:value}}"`

     对于Rest风格的对象形式传参，只能使用路由名称的形式

   获取参数：

   - 使用路由对象`vm.$route.params`获取参数信息

### 5. 路由对象

路由对象`$route`，表示当前激活的路由的状态信息，称为**激活路由信息对象**

- 该对象为所有组件共享的实例属性，在所有组件中都可以通过`this.$route`访问，且访问的都是同一个对象

- 该对象中存储着当前激活路由的相关信息

  ```js
  $route.path 		// 当前激活路由的路径
  $route.fullPath 	// 当前激活路由的完整路径
  $route.name  		// 当前激活路由的名称，即命名路由的name
  $route.query  		// 当前激活路由的字符串查询传递的参数
  $route.params  		// 当前激活路由的rest方式传递的参数
  $route.meta  		// 当前激活路由的元信息
  ```

路由元信息：

- 在定义路由时，可以通过meta属性为当前路由指定一些自定义配置项，称为元信息
- 在组件中可以通过$route.meta获取到这些元信息

### 6. 路由守卫

也称为路由导航守卫，用来对路由导航进行守卫，控制导航的跳转，可以进行权限的控制

全局前置守卫：使用 `router.beforeEach`

```javascript
const router = new VueRouter({ ... })

// 当一个导航触发时，全局前置守卫将被调用　
router.beforeEach((to, from, next) => {
  // to 即将要访问的目标路由对象
	// from 来源于哪个路由对象
	// next 是一个函数，表示下一步怎么做，如：next()表示放行，next('/login')表示跳转到/login路径　
})
```

全局解析守卫：使用`router.beforeResolve`

全局后置钩子：使用`router.afterEach`

### 7. 路由模式

通过mode选项可以修改路由的模式

两种模式：

+ hash模式（默认值）

  原理：使用HTML的锚点技术，实现路由匹配和跳转

  ​	   通过`location.hash`获取url中的锚点值，完成 url 跳转，且不会重新加载页面，即不刷新页面

  缺点：地址中会出现#号，比较丑陋

+ history模式

  原理：使用HTML5的History技术

  ​	  通过`history.pushState` 完成 url 跳转，且不会重新加载页面

  缺点：在生产环境下，如果用户在浏览器中**直接访问**url路径，则会报错 Not Found 

  ​	   所以在生产环境下，需要后台服务器的配置支持才行（在开发环境下是没问题的）

### 8. 编程式导航

除了使用`<router-link>`组件来定义导航链接，还可以通过编写js代码，调用路由实例`$router`的方法来实现路由跳转，称为编程式导航

路由实例`$router`，表示构建路由时的实例对象，主要用来对路由进行控制，称为 **路由控制对象**

- 该对象为所有组件共享的实例属性，在所有组件中都可以通过`this.$router`访问，且访问的都是同一个对象

- 该对象中包含控制路由的相关方法

  ```js
  $router.push(location) 	// 跳转到指定页面，参数location可以为字符串或对象
  $router.forward()		// 向前一步
  $router.back()  		// 后退一步
  $router.go(n)  			// 向前或后退多少步
  ```

## 八、Vuex

### 1. 简介

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**

采用集中式存储管理应用中所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化

简单来说，就是用来集中管理数据的，统一维护项目中的数据状态，相当于是一个全局的数据存储对象store，在所有组件中都可以访问

应用场景：适合于大型的单页应用程序

![vuex](https://raw.githubusercontent.com/YukiCCC/figure/main/vuex.png)

### 2. 基本用法

创建项目时选择Vuex功能

```bash
|-项目名称
	|-node_modules
		|-vuex  			# vuex模块
  	|-src
    	|-store
    		|-index.js		# vuex配置文件		
    	|-main.js  				
```

Vuex的核心是store（存储），相当于是一个容器，核心概念：

- State 	定义属性，用来存储状态数据
- Getter    定义属性，用来获取从state派生出的状态数据，类似于计算属性
- Mutation   定义变化，用来改变状态数据（突变），只能是同步操作，需要通过commit来提交变化mutations
- Action     定义方法，用来改变状态数据，可以包含异步操作，类似于mutations，但mutations只能是同步操作
- Module    用来分模块组织Vuex

### 3. 使用Vuex

#### 3.1 在Vuex中定义状态

配置store对象：

- 定义state
- 定义getters
- 定义mutations
- 定义actions
- 定义modules

#### 3.2 在组件中访问状态

在组件中访问store对象，两种方式：

- 方式1：通过存储对象`$store`访问

  存储对象`$store`，表示Vuex中定义的数据存储对象store，包含Vuex中的所有数据信息

  该对象为所有组件共享的实例属性，在所有组件中都可以通过`this.$store`访问，且访问的都是同一个对象

- 方式2：通过辅助函数访问

  Vuex中提供了一些辅助函数，用于获取Vuex中的数据信息，如：

  mapState()    		  获取state

  mapGetters()  		获取getters

  mapMutations() 	获取mutations

  mapActions()  		获取actions

  注：需要先导入辅助函数`import {mapState,mapGetters,mapActions} from 'vuex'`

### 4. 模块化

当应用变得非常复杂时，store对象会变得相当臃肿，store/index.js文件内容会非常多

此时可以将store分成多个模块，每个模块拥有自己的state、getters、mutations、actions

+ 可以根据业务分模块，如：用户数据、产品数据、购物车数据、订单数据等
+ 可以根据页面分模块，如：主页数据、搜索页数据、详情页数据等

Vuex模块化的项目结构：

```js
|-src
    |-store
        |-index.js
        |-state.js 				// 根级别的state
        |-getters.js 			// 根级别的getters
        |-mutations.js			// 根级别的mutations
        |-actions.js			// 根级别的actions
        |-modules 				// 分为多个模块，每个模块都可以拥有自己的state、getters、actions、mutations
            |-user.js			// 用户模块
            |-product.js		// 产品模块
            |-cart.js			// 购物车模块
            |-order.js			// 订单模块
            |-....
```

## 九、Element UI

### 1. 简介

​	Element是饿了么团队提供的一套基于Vue的桌面端组件库，可以快速的搭建网站，提高开发效率

​    官方文档 https://element.eleme.cn

### 2. 用法

执行 `vue add element`为项目安装Element插件

```bash
? How do you want to import Element? Fully import  # 完整引入
? Do you wish to overwrite Element's SCSS variables? No 
? Choose the locale you want to load zh-CN    
```

​    

