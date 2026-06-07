---
title: MVVM设计模式
date: 2026-06-07 21:01:52
tags: 前端
category: 前端
---

### 一、MVVM 简介

#### 1. MVVM 是什么

MVVM（Model-View-ViewModel）是一种**架构设计模式**，将应用程序分为三层：

| 层级 | 英文 | 职责 |
| --- | --- | --- |
| 模型 | Model | 业务数据与逻辑 |
| 视图 | View | 用户界面展示 |
| 视图模型 | ViewModel | 连接 View 与 Model，处理交互与状态 |

核心思想：**View 不直接操作 Model**，而是通过 ViewModel 进行数据绑定和命令转发，从而降低界面与业务逻辑的耦合。

#### 2. 为什么需要 MVVM

传统页面开发中，DOM 操作与业务逻辑混在一起，导致：

- 代码难以维护，改一处牵动多处
- 界面状态与数据状态容易不一致
- 单元测试困难

MVVM 通过**数据驱动视图**，开发者主要关注数据变化，框架自动更新界面，大幅提升开发效率与可维护性。

### 二、MVC、MVP 与 MVVM 对比

#### 1. 三种模式演进

```
MVC：  View ↔ Controller ↔ Model
MVP：  View ↔ Presenter ↔ Model
MVVM： View ↔ ViewModel ↔ Model（数据绑定）
```

#### 2. 对比表

| 模式 | 核心角色 | View 与业务关系 | 典型框架 |
| --- | --- | --- | --- |
| MVC | Controller | View 通过 Controller 间接访问 Model | Spring MVC、ASP.NET MVC |
| MVP | Presenter | View 被动，Presenter 驱动 | Android 早期、WinForms MVP |
| MVVM | ViewModel | 双向/单向数据绑定，View 无业务逻辑 | Vue、WPF、Android Jetpack |

#### 3. 关键区别

- **MVC**：Controller 较「重」，常同时处理请求路由和业务
- **MVP**：Presenter 手动更新 View，接口回调多，代码量偏大
- **MVVM**：ViewModel 暴露可绑定数据，View 自动同步，Presenter 式胶水代码最少

### 三、MVVM 三层详解

#### 1. Model（模型）

Model 负责**数据和业务规则**，不依赖 UI。

典型内容：

- 实体类（User、Order、Product）
- 数据访问（API 请求、数据库）
- 业务校验与计算

```javascript
// 示例：用户模型
const userModel = {
  name: 'YukiCCC',
  age: 25,
  isValid() {
    return this.name.length > 0 && this.age > 0;
  }
};
```

#### 2. View（视图）

View 只负责**展示和交互**，应尽量「傻」——不包含业务逻辑。

- HTML 模板
- 按钮、表单、列表
- 样式与布局

在 Vue 中，View 就是模板 + 样式；在 WPF 中，View 是 XAML 界面。

#### 3. ViewModel（视图模型）

ViewModel 是 MVVM 的**核心**，职责包括：

- 从 Model 获取数据并转换为 View 可直接绑定的格式
- 暴露 View 需要的状态（如 `isLoading`、`errorMsg`）
- 提供命令/方法响应用户操作（如 `submit`、`delete`）
- 不直接操作 DOM，只维护状态

```javascript
// Vue 中 ViewModel 体现为组件实例的 data + methods
export default {
  data() {
    return {
      username: '',
      loading: false
    };
  },
  methods: {
    async submit() {
      this.loading = true;
      await api.saveUser({ name: this.username });
      this.loading = false;
    }
  }
};
```

### 四、数据绑定

#### 1. 单向绑定

数据从 ViewModel → View，View 变化不自动回写。

```html
<!-- Vue 插值表达式 -->
<p>{{ message }}</p>
```

适合：展示型数据，如标题、列表、统计数字。

#### 2. 双向绑定

ViewModel ↔ View 同步变化。

```html
<!-- Vue v-model -->
<input v-model="username" />
```

适合：表单输入、开关、选择器。双向绑定是 MVVM 最具代表性的特性之一。

#### 3. 绑定带来的好处

- 减少手动 `document.getElementById` 操作
- 状态单一来源（Single Source of Truth）
- 界面与数据自动保持一致

### 五、MVVM 在前端中的实践（Vue）

#### 1. Vue 与 MVVM 的对应关系

| MVVM | Vue |
| --- | --- |
| Model | 后端 API、Pinia/Vuex 状态、业务模块 |
| View | Template 模板 |
| ViewModel | 组件实例（data、computed、methods） |

Vue 官方描述就是「渐进式 MVVM 框架」，通过响应式系统自动追踪依赖、更新 DOM。

#### 2. 简单示例

```html
<div id="app">
  <input v-model="name" placeholder="请输入姓名" />
  <p>你好，{{ name || '陌生人' }}</p>
  <button @click="greet">打招呼</button>
</div>
```

```javascript
const { createApp } = Vue;
createApp({
  data() {
    return { name: '' };
  },
  methods: {
    greet() {
      alert(`Hello, ${this.name}`);
    }
  }
}).mount('#app');
```

- `data` 中的 `name` 属于 ViewModel 状态
- 模板是 View
- 若提交到服务器，API 层属于 Model

#### 3. 组件化进一步分离

实际项目中，MVVM 常结合组件化：

- 页面级组件：组织 ViewModel
- 业务模块：Model 层
- UI 组件：纯 View 展示

### 六、MVVM 在 Csharp / WPF 中的实践

MVVM 不仅用于前端，**WPF、UWP、MAUI** 等桌面开发也广泛采用 MVVM。

#### 1. 典型技术栈

| 技术 | 作用 |
| --- | --- |
| XAML | View |
| ViewModel 类 | 实现 INotifyPropertyChanged |
| Model 类 | 业务实体与服务 |
| DataBinding | 绑定 View 与 ViewModel 属性 |
| Command（ICommand） | 按钮命令绑定 |

#### 2. Csharp ViewModel 示例

```csharp
public class UserViewModel : INotifyPropertyChanged
{
    private string _username = string.Empty;
    public string Username
    {
        get => _username;
        set
        {
            _username = value;
            OnPropertyChanged(nameof(Username));
        }
    }

    public ICommand SaveCommand { get; }

    public UserViewModel()
    {
        SaveCommand = new RelayCommand(Save);
    }

    private void Save()
    {
        // 调用 Model 层保存
    }

    public event PropertyChangedEventHandler? PropertyChanged;
    protected void OnPropertyChanged(string name) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

本博客《Csharp基础知识》中的语言特性，可与 WPF MVVM 桌面开发结合学习。

### 七、MVVM 的优势与局限

#### 1. 优势

| 优势 | 说明 |
| --- | --- |
| 低耦合 | View 与 Model 互不直接依赖 |
| 易测试 | ViewModel 可脱离 UI 单独单元测试 |
| 可维护 | 职责清晰，利于团队协作 |
| 复用性 | 同一 ViewModel 可适配不同 View |
| 开发效率 | 数据绑定减少样板代码 |

#### 2. 局限

| 局限 | 说明 |
| --- | --- |
| 学习成本 | 需理解绑定、命令、响应式等概念 |
| 简单页面略重 | 小页面用 MVVM 可能过度设计 |
| 调试绑定问题 | 数据流不直观时排查较费劲 |
| 框架依赖 | 通常需 Vue、WPF 等框架支持 |

### 八、MVVM 最佳实践

#### 1. View 保持简洁

- 不在模板中写复杂业务逻辑
- 避免在 View 中直接请求 API
- 复杂计算放到 `computed` 或 ViewModel 属性

#### 2. ViewModel 不引用 View

- 不操作 DOM
- 不依赖具体 UI 控件类型
- 通过数据状态驱动界面（如 `isDialogVisible`）

#### 3. Model 独立于 UI

- API、数据库、算法不感知 View
- 便于前后端分离与接口复用

#### 4. 状态集中管理

复杂应用可引入：

- Vue：Pinia / Vuex
- Csharp：依赖注入 + Service 层
- 统一处理 loading、error、缓存

### 九、常见问题

| 问题 | 原因 | 建议 |
| --- | --- | --- |
| 数据更新了界面不变 | 未响应式、直接改数组索引 | 用 Vue 响应式 API 或替换对象 |
| ViewModel 过于臃肿 | 所有逻辑堆在一个类 | 拆分模块、抽取 Service |
| 双向绑定混乱 | 多处修改同一状态 | 明确单一数据源 |
| 测试困难 | View 与逻辑耦合 | 将逻辑下沉到 ViewModel |

### 十、小结

MVVM 通过 **Model - View - ViewModel** 分层，用**数据绑定**替代大量手动 UI 更新，是现代界面开发的主流模式之一。

学习建议：

1. 先理解 MVC、MVP、MVVM 的差异
2. 用 Vue 体验前端 MVVM 与双向绑定
3. 了解 WPF/MAUI 中 Csharp MVVM 实现
4. 实践中坚持「View 轻、ViewModel 管状态、Model 管业务」

结合本博客《Vue基础学习》《Csharp基础知识》，可以从**前端组件化**和**桌面上位机**两个方向深入掌握 MVVM。
