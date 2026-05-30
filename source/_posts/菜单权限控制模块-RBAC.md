---
title: 菜单权限控制模块(RBAC)
date: 2025-07-03 22:06:11
tags: Spring
category: Java
---

### 一、权限系统框架

![图片](https://static001.geekbang.org/infoq/5d/5dfa3fd59bcd9676e8655b3c637d475b.png)

如图所示，权限系统主要解决两个问题：

1. 前端渲染：接入系统用户登录后，获取自己有权限的菜单，也就是前端sdk请求权限系统获取有权限的菜单并进行自动渲染。
2. 后端鉴权：用户请求接入系统后端，拒绝没有权限的接口访问，防止无权限用户获取后端接口地址后直接访问无权限的接口。
为了解决这两个问题，必然需要引入一些配套内容，其中重要的功能点如下：

1. 用户管理：统一登录系统，支撑权限系统识别登录用户。
2. 权限管理：系统管理、角色管理、菜单管理、数据管理、角色人员绑定等功能，方便为用户绑定相应的权限。
3. 后端鉴权：访问接入系统接口调用时使用。
后面会一一介绍用户管理、权限管理、后端鉴权sdk的实现。

首先让我们思考一个问题，为什么首先前端渲染，另外既然前端已经获得了用户可点击的菜单按钮，为什么还要经过后端校验？

#### 1.1 安全性考虑
前端权限控制不可信任 ：

- 前端代码运行在用户浏览器中，用户可以通过开发者工具、浏览器插件等方式修改前端代码
- 恶意用户可以绕过前端的权限检查，直接调用后端API接口
- 前端权限控制只是为了提升用户体验，不能作为安全防护的最后一道防线
API接口直接访问风险 ：

- 即使前端隐藏了某些菜单，用户仍可能通过抓包、查看网络请求等方式获取API地址
- 恶意用户可以使用Postman、curl等工具直接调用后端接口
- 没有后端鉴权的话，系统存在严重的安全漏洞
#### 1.2 双重防护机制
权限系统需要解决两个核心问题：
1. 前端渲染 ：提供良好的用户体验，只显示用户有权限的菜单
2. 后端鉴权 ：确保安全性，拒绝无权限的接口访问
这是一个"前端控制体验，后端保证安全"的设计原则。前端权限控制让用户界面更友好，后端鉴权确保数据和业务逻辑的安全性。

### 二、用户管理

本文通过转转权限控制举例：

权限系统支持转转、转转精神、发条爱客、自注册四种来源的用户数据，支持用户在一个系统登录在其他系统保持登录状态功能（统一登录），支持员工入职自动加入权限系统（员工同步），这里重点介绍统一登录。

#### 2.1、统一登录
先从接入权限的域名都是*.zhuanspirit.com说起，统一登录利用浏览器携带Cookie特点：不同二级域名可以携带一级域名的Cookie。统一登录系统为接入系统种植一级域名Cookie，例如OA系统oa.zhuanspirit.com，权限系统自身id.zhuanspirit.com统一种植域名为*.zhuanspirit.com的Cookie，这样OA、权限系统等接入统一登录系统的系统就可以做到一处登录处处访问。现在抛出两个问题：

1. 接入系统并未引入统一登录系统的任何代码，未登录用户是怎样跳转到统一登录页的？

2. 如何校验Cookie是否合法的？

  ![图片](https://static001.geekbang.org/infoq/8d/8ddec8d96bbed1c7693ca7ba82b00ff6.png)从以OA系统为例的图中可以看出问题答案：

3. ngix会对*.zhuanspirit.com域名的请求做Cookie合法校验，校验不通过跳转到登录页，同时携带原url信息。

4. 校验合法性是通过Cookie<sso_uid,sso_code>的值是否和Redis中一致，不一致就需要重新登录。
### 三、权限管理
转转权限管理系统是一个基于经典RBAC权限管理模型的实现，上图为转转权限系统的主要功能，本身实现复杂度并不高。但是由于历史原因，转转权限系统在实现的时候需要考虑兼容原有权限系统的数据，这是一个比较琐碎复杂的事情，在这里不做过多介绍。对于简单的管理系统来说，灵魂在数据。对应到转转权限系统，血肉就是管理UI，它的使命就是方便为数据表填充数据，具体一点就是为用户表、系统表、菜单表、数据表、角色表以及中间关系表填充数据。下面我就展现转转权限系统的灵魂--数据库表关系。

![图片](https://static001.geekbang.org/infoq/77/77c00651e3d8e8c265704f0d4e7b7022.png)从上图中可以看到转转权限系统和RBAC一些差异：用户直接可以与菜单或者数据绑定，增加灵活性。

**用户直接可以与菜单或者数据绑定，增加灵活性。**

#### 3.1 核心实体表
1. 用户表（User） ：存储用户基本信息
2. 角色表（Role） ：定义系统中的各种角色
3. 菜单表（Menu） ：存储系统菜单和功能点
4. 数据表（Data） ：存储数据权限相关信息
5. 系统表（System） ：管理接入的各个系统
#### 3.2 关系表（多对多关联）
1. 用户-角色关系表 ：用户可以拥有多个角色
2. 角色-菜单关系表 ：角色可以访问多个菜单
3. 角色-数据关系表 ：角色可以访问多种数据
4. 用户-菜单关系表 ：用户可以直接绑定菜单（增加灵活性）
5. 用户-数据关系表 ：用户可以直接绑定数据权限（增加灵活性）
#### 3.3 表关联逻辑
**标准RBAC模型流程：**

```
用户 → 角色 → 权限（菜单/数据）
```
**转转权限系统的增强模型：**

```
用户 → 角色 → 权限（菜单/数据）
用户 → 直接绑定 → 权限（菜单/数
据）  // 增加灵活性
```
#### 3.4 用户登录后获取权限信息的逻辑
1. 根据用户ID查询用户-角色关系表，获取用户的所有角色
2. 根据角色ID查询角色-菜单关系表，获取角色对应的菜单权限
3. 根据用户ID查询用户-菜单关系表，获取用户直接绑定的菜单权限
4. 合并角色权限和直接权限，去重后返回最终权限列表
5. 将权限信息缓存到Redis中，提升后续访问性能

### 四、后端鉴权
#### 4.1、工作原理
![图片](https://static001.geekbang.org/infoq/5a/5a9ea592ec7bf1710c0f27f67fdd5d68.png)

如上图所示，后端鉴权sdk通过切面编程的思想，对请求url或者code进行拦截鉴权，如果登录用户没有权限，则返回错误。

#### 4.2、核心方法
```
AuthResult res = authentication.check(new AppCodeAuthParmBuilder(APP_CODE, CODE, request));
```
上面是sdk面向用户的接口很容易可以看出来，进入非常简单，仅仅需要三个参数：

- APP_CODE：系统的系统编号
- CODE：权限编码（可为空，为空时根据url鉴权）
- request：HttpServletRequest（从中获取Cookie信息解析出登录用户，以及url信息判断用户是否有此url权限）
#### 4.3、使用demo
利用Spring Interceptor切面技术，下面是架构管理平台关于ZZLock后台操作的一个例子

```java
@Component
public class ZZLockInterceptor implements HandlerInterceptor {
    @Resource
    private Authentication authentication;
    private static final String APP_CODE = "arch_ipms";
    private static final String CODE = "ro_zzlock";
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {

        AuthResult res = authentication.check(new AppCodeAuthParmBuilder(APP_CODE, CODE, request));

        if (res.isSuccess() && res.getCode() == ResultCodeEnum.SUCCESS.getCode()) {
            return true;
        } else {
            UserDTO user = authentication.parseUser(request);
            logger.info("菜单 zzlock_get user {} 鉴权结果 code {} msg {}", user.getLoginName(), res.getCode(), res.getMsg());
            response.getWriter().write("user: " + user.getLoginName() + " no auth");
            response.getWriter().close();
        }

        return false;
    }
}
```

### 五、总结
从使用方的视角出发，也就是前端渲染和接口鉴权，引出转转权限系统如何识别用户（统一登录），如何存储权限数据（权限管理），如何实现后端鉴权。

简而言之，权限系统的主要功能：权限系统UI编辑权限数据，用户登录后，获取配置好的菜单和数据，并且校验用户访问的后端接口。