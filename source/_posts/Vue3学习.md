---
title: Vue3学习
date: 2023-08-22 16:31:19
tags: vue
category: 前端
---

## 一、简介

### 1. 介绍

​	Vue 2 将于 2023 年 12 月 31 日停止维护，详见 [Vue 2 延长 LTS](https://v2.cn.vuejs.org/lts/)。

​	Vue 3是Vue.js的下一个主要版本，最初于2020年9月发布，Vue 3支持Vue 2的大多数特性。

### 2. 优点

- 更快更省

    Object.defineProperty ——> Proxy

    重构 Virtual DOM

    运行速度更快，打包体积更小

- 更好的TypeScript集成

    团队开发更轻松

    架构更灵活，阅读源码更轻松

    可以独立使用Vue内部模块

- 组合式API（Composition API）

    一组低侵入式的、函数式的 API

    更好的逻辑复用与代码组织

    更好的类型推导

## 二、创建Vue3项目

### 1. 简介

Vue.js官方提供了两种创建Vue 3项目的方式：vue-cli 或 vite

区别如下:

1. 运行方式不同

vue-cli使用Webpack进行项目打包，这意味着启动和打包速度相对较慢

vite使用ES模块系统，利用浏览器原生的模块解析器来运行，启动速度更快，打包速度也更快。

2. 默认配置有区别

vue-cli提供了更多的默认配置和插件，可以帮助开发者更快地创建和部署Vue.js项目

vite则采用更加精简的配置和插件，让开发者自由地选择第三方插件和配置模块。

### 2. 使用 vite

Vite是一种新型前端构建工具，能够显著提升前端开发体验。

特点：

- 快速的冷启动，不需要等待打包操作
- 即时的热模块更新，替换性能和模块数量的解耦让更新飞起
- 真正的按需编译，不再等待整个应用编译完成，这是一个巨大的改变

操作步骤：

```bash
# 创建项目
npm init vue@latest # 基于Vite

# 安装依赖
cd vue3-project
npm install

# 运行项目
npm run dev
```


## 三、API 风格

Vue 的组件可以按两种不同的风格书写：**选项式 API** 和**组合式 API**

### 1. 选项式 API (Options API)

使用选项式 API，我们可以用包含多个选项的对象来描述组件的逻辑，例如 `data`、`methods` 和 `mounted`。

选项所定义的属性都会暴露在函数内部的 `this` 上，它会指向当前的组件实例。

```vue
<script>
export default {
  // data() 返回的属性将会成为响应式的状态，并且暴露在 this 上
  data() {
    return {
      count: 0
    }
  },
  // methods 是一些用来更改状态与触发更新的函数，它们可以在模板中作为事件处理器绑定
  methods: {
    increment() {
      this.count++
    }
  },
  // 生命周期钩子会在组件生命周期的各个不同阶段被调用，例如这个函数就会在组件挂载完成后被调用
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

### 2. 组合式 API (Composition API)

通过组合式 API，我们可以使用导入的 API 函数来描述组件逻辑。

在单文件组件中，组合式 API 通常会与`<script setup>`搭配使用。这个 `setup` 属性是一个标识，告诉 Vue 需要在编译时进行一些处理，让我们可以更简洁地使用组合式 API。比如，`<script setup>` 中的导入和顶层变量/函数都能够在模板中直接使用。

下面是使用了组合式 API 与 `<script setup>` 改造后和上面的模板完全一样的组件：

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 用来修改状态、触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

## 四、响应式API

Vue 3 提供的一组具有响应式特性的函数式API，都是以函数形式提供的

### 1. reactive

`reactive()`函数接收一个`普通对象`，返回该普通对象的响应式代理对象

简单来说，就是用来创建响应式的数据对象

```js
<script setup>
    // 导入reactive函数
    import { reactive } from 'vue'

    // 创建响应式数据对象，是一个代理对象
    const user = reactive({
        name:'tom',
        age:18
    })
</script>
```

### 2. ref

`ref()`函数接收一个参数值，返回一个响应式的数据对象。该对象只包含一个指向内部值的 `.value` 属性

在模板中访问时，无需通过.value属性，它会自动展开 

### 3. toRef

`toRef()`函数用来将 reactive 对象的一个属性创建为一个 ref，并且这个 ref 具有响应性，可以被传递。

### 4. toRefs

`toRefs()`函数用来将 reactive 对象创建为一个普通对象，但该普通对象的每个属性都是一个 ref，并且这个 ref 具有响应性，可以被传递。

### 5. computed

`computed()` 函数用来创建计算属性，函数的返回值是一个 `ref` 的实例

### 6. watch

`watch()` 函数用来监视数据的变化，从而触发特定的操作

### 6. watchEffect

`watchEffect()`函数接收一个函数作为参数，并立即执行该函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。

## 五、组件间数据传递 

### 1. 父子组件

父向子传递数据：属性绑定+数据拦截

- 使用defineProps接收父组件传递的数据

子向父传递数据：事件监听+事件触发

- 使用defineEmits自定义事件，然后触发事件

注：`defineProps` 和 `defineEmits`是自动导入的，不需要手动导入

### 2. 依赖注入

依赖注入就是祖先组件向后代组件传递数据，使用`provide()` 和 `inject()` 函数来实现

- 在祖先组件中使用`provide()`函数向下传递数据
- 在后代组件中使用`inject()`函数获取上层传递过来的数据

## 六、生命周期钩子函数

Vue 3 中的生命周期函数和 Vue 2.x 相比做了一些调整和变化，对应关系如下：

- `beforeCreate` -> 使用 `setup()`
- `created` -> 使用 `setup()`
- `beforeMount` -> `onBeforeMount`
- `mounted` -> `onMounted`
- `beforeUpdate` -> `onBeforeUpdate`
- `updated` -> `onUpdated`
- `beforeDestroy` -> `onBeforeUnmount`
- `destroyed` -> `onUnmounted`
- `errorCaptured` -> `onErrorCaptured`

## 七、模板 Refs

通过 `ref()`函数还可以引用页面上的元素或组件，功能类似于 vue 2.x中的 `vm.$refs`

步骤：

1. 创建一个空的 ref 对象并返回它
2. 在页面上为元素添加 ref 属性，并设置属性值与创建的 ref 对象的名称相同
3. 当页面渲染完成后，可以通过该 ref 对象获取到页面中对应的DOM元素

## 八、Pinia状态管理

Pinia（小菠萝）是一个轻量级的、基于Vue 3的状态管理库，提供了更简单的 API，支持Typescript和组合式API风格。

与Vuex相比，Pinia抛弃了Mutation操作，保留state、getters、actions，且提供了更简便的用法。

官网 https://pinia.vuejs.org/zh/

基本用法，步骤：

1. 定义store

    ```js
    import { ref, computed } from 'vue'
    import { defineStore } from 'pinia'
    
    export const useUser = defineStore('user', () => {
        const msg = ref('welcome to pinia')
    
        const reverseMsg = computed(() => msg.value.toUpperCase().split(' ').reverse().join(' ')) 
    
        function setMsg(newValue) {
          msg.value = newValue
        }
      
        return { msg, reverseMsg, setMsg }
    })
    ```

2. 使用store

    ```vue
    <script setup>
        import { useUser } from '@/stores/user'
    
        const user = useUser()
    </script>
    
    <template>
        <p>{{ user.msg }}</p>
        <p>{{ user.reverseMsg }}</p>
        <button @click="user.setMsg('hello world')">按钮</button>
    </template>
    ```



## 附录：vite常用插件

### 1. unplugin-auto-import

unplugin-auto-import 自动导入插件，能够自动引入ref、reavtive等函数，不需要再自己引入

步骤：

1. 安装插件

    执行`npm install -D unplugin-auto-import ` 

2. 配置插件，编辑vite.config.js

    ```js
    import AutoImport from 'unplugin-auto-import/vite'
    
    export default defineConfig({
      plugins: [
        vue(),
        AutoImport({
          imports:['vue','vue-router','pinia'] // 自动导入相关函数
        })
      ],
      resolve: {
        alias: {
          '@': fileURLToPath(new URL('./src', import.meta.url))
        }
      }
    })
    ```

### 2. vite-plugin-pages 

vite-plugin-pages 自动读取指定目录下的文件，生成路由信息，不需要自己去逐个配置路由

步骤：

1. 安装插件

    执行`npm install -D vite-plugin-pages ` 

2. 配置插件，编辑vite.config.js

    ```js
    import AutoImport from 'unplugin-auto-import/vite'
    import Pages from  'vite-plugin-pages'
    
    export default defineConfig({
      plugins: [
        vue(),
        AutoImport({
          imports:['vue','vue-router','pinia'] // 自动导入相关函数
        }),
        Pages({
          dirs:[ { dir: "src/views", baseRoute: "" }], // 修改默认文件夹
          importMode: "async"
        })	
      ],
      resolve: {
        alias: {
          '@': fileURLToPath(new URL('./src', import.meta.url))
        }
      }
    })
    
    ```

3. 自动生成路由，编辑router/index.js

    ```js
    import routes from '~pages'
    
    const router = createRouter({
      history: createWebHistory(import.meta.env.BASE_URL),
      routes
    })
    
    export default router
    ```

注意：

- vite-plugin-pages默认读取的页面文件夹是 pages，默认读取的页面是 index.vue
- 所以最好先在pages文件夹下面创建一个 index.vue文件
