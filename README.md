vue_shop

## Project setup
```
npm install
```
![](/src/assets/程序学习交流群聊二维码.png)
<div>811481586 编程学习交流群 API文档,数据库,软件,PDF手册,chrome插件等等资料在群文件</div>


### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

* http 是无状态的
* 通过 cookie 在客户端记录状态
* 通过 session 在服务器端记录状态
* 通过 token 方式维持状态
* 前端与服务器之间存在跨域问题时使用
* 登录前输入帐号、密码 -》 服务器验证后返回token -》 客户端保存token -》 后续请求都携带token数据发送请求（用户的验证） -》 服务器验证token是否通过
# 1. 项目概述

### 1.1 电商后台管理系统的功能

电商后台管理系统用于管理用户账号、商品分类、商品信息、订单、数据统计等业务功能。



### 1.4 电商后台管理系统的技术选型

前端项目技术栈

- Vue
- Vue-router
- Element-UI
- Axios
- Echarts

#  2. 项目初始化

### 2.1 前端项目初始化步骤

1. 安装 Vue 脚手架
2. 通过 Vue 脚手架创建项目
3. 配置 Vue 路由
4. 配置 Element-UI 组件库
5. 配置 axios 库
6. 初始化 git 远程仓库
7. 将本地项目托管到 Github 或 码云 中

# 3. 登录/退出功能

## 3.1 登录概述

### 1. 登录业务流程

- 在登录页面输入用户名和密码
- 调用后台接口进行验证
-  通过验证之后，根据后台的响应状态跳转到项目主页

### 2. 登录业务的相关技术点

- http 是无状态的
- 通过 cookie 在客户端记录状态
- 通过 session 在服务器端记录状态
- 通过 token 方式维持状态

## 3.2 登录功能实现

### 1. 登录页面的布局

通过 Element-UI 组件实现布局

- el-form
- el-form-item
- el-input
- el-button
- 字体图标

![](https://s2.loli.net/2021/12/09/kLv2gqDGs3R8OEF.png)

# 4. 主页布局

## 4.1 整体布局

整体布局：先上下划分，再左右划分。

菜单分为二级，并且可以折叠。

## 4.2 通过接口获取菜单数据

通过axios请求拦截器添加token，保证拥有获取数据的权限

```
// axios请求拦截
axios.interceptors.request.use(config => {
// 为请求头对象，添加 Token 验证的 Authorization 字段
config.headers.Authorization = window.sessionStorage.getItem('token')
return config
}
```

## 4.3 动态渲染菜单数据并进行路由控制

- 通过 v-for 双层循环分别进行一级菜单和二级菜单的渲染
- 通过路由相关属性启用菜单的路由功能

![](https://s2.loli.net/2021/12/09/NGBrpTn548EARbi.png)

# 5. 用户管理

## 5.1 用户管理概述

通过后台管理用户的账号信息，具体包括用户信息的展示、添加、修改、删除、角色分配、账号启用/注销等功能。

## 5.2 用户管理-列表展示

![](https://s2.loli.net/2021/12/09/cPE8YVNvfOj2Bwb.png)

- 面包屑导航 el-breadcrumb
- Element-UI 栅格系统基本使用 el-row
- 表格布局 el-table、el-pagination

# 6. 权限管理

## 6.1 权限管理业务分析

通过权限管理模块控制不同的用户可以进行哪些操作，具体可以通过角色的方式进行控制，即每个用户分配
一个特定的角色，角色包括不同的功能权限。

## 6.2 权限列表展示

![](https://s2.loli.net/2021/12/09/UpY7cF8yHjOqbsZ.png)

## 6.3 角色列表展示

- 实现角色分配权限对话框布局
- 控制对话框的显示和隐藏
- 对话框显示时调用后台接口加载权限列表数据
- 完成树形权限菜单的展示
- 选中默认的权限
- 保存选中的权限，调用后台接口完成角色权限的分配

![](https://s2.loli.net/2021/12/09/KDMUYchk6HP9gIz.png)

表格行展开效果，通过 el-table-column 组件的 type =“expand” 方式实现表格行展开效果

![](https://s2.loli.net/2021/12/09/U4NVHsFM72bWYxd.png)

# 7. 分类管理

## 7.1 商品分类概述

商品分类用于在购物时，快速找到所要购买的商品，可以通过电商平台主页直观的看到。

## 7.2 树形表格

### 第三方树形表格的基本使用

- 安装依赖包（地址： https://github.com/MisterTaki/vue-table-with-tree-grid）

```
npm i vue-table-with-tree-grid -S
```

- 基本使用

```
import Vue from 'vue'
import ZkTable from 'vue-table-with-tree-grid'
Vue.use(ZkTable)
```

## 7.3实现分类树形列表

![](https://s2.loli.net/2021/12/09/GrJpqzgNBZTC8HI.png)

# 8. 参数管理

## 8.1参数管理概述

商品参数用于显示商品的固定的特征信息，可以通过电商平台商品详情页面直观的看到。

![](https://s2.loli.net/2021/12/09/L2CYz4phxowR7JW.png)

![](https://s2.loli.net/2021/12/09/42EMybDed3qzNKI.png)

- 动态参数与静态属性表单重用
- 添加动态参数与静态属性使用的是同一个接口，参数是一样的

# 9. 商品管理

## 9.1 商品管理概述

商品管理模块用于维护电商平台的商品信息，包括商品的类型、参数、图片、详情等信息。通过商品管理模块可以
实现商品的添加、修改、展示和删除等功能。

![](https://s2.loli.net/2021/12/09/ay1EhFKdef96XMw.png)

## 9.2 添加商品

- 商品分类数据加载

- 获取商品动态参数数据
- 获取商品静态属性数据
- 商品图片上传
- 商品详情

![](https://s2.loli.net/2021/12/09/qK1sPNQhf4lcn9p.png)

富文本编辑器基本使用

```
// 安装vue-quill-editor
npm install vue-quill-editor -S
```

```
import VueQuillEditor from 'vue-quill-editor‘
Vue.use(VueQuillEditor)
```

```
<quill-editor v-model="addForm.goods_introduce"></quill-editor>
```

![](https://s2.loli.net/2021/12/09/RlK9VjnTNPm4d7p.png)

# 10. 订单管理

## 10.1订单管理概述

订单管理模块用于维护商品的订单信息，可以查看订单的商品信息、物流信息，并且可以根据实际的运营情况对订
单做适当的调整。

![](https://s2.loli.net/2021/12/09/qYVR51BkQtyONuA.png)

## 10.2 查看订单物流信息

- 调用接口获取物流数据
- 实现物流信息列表效果


![](https://s2.loli.net/2021/12/09/MAbGnXc7DsaEg3v.png)

# 11. 数据统计

## 11.1 数据统计概述

数据统计模块主要用于统计电商平台运营过程的中的各种统计数据，并通过直观的可视化方式展示出来，方便相关
运营和管理人员查看。

Echarts 第三方可视化库的基本使用

```
// 安装echarts库
npm install echarts -S
```

```
// 导入echarts接口
import * as echarts from 'echarts'
```

![](https://s2.loli.net/2021/12/09/o7l1anCKPjZJWwq.png)

# 项目优化

* nprogress 加载时进度条
  * NProgress.done() 隐藏进度条
  * NProgress.start() 展示进度条
    * 其条件可以在 请求拦截器里面设置

* 打包是 console的处理:  babel-plugin-transform-remove-console

  * plugins: [ "transform-remove-console"  ]

* 生成打包报告

  1. vue-cli-service build --report
  2. UI面版

* 项目优化 配置webpack

  * 如果程序员有修改 webpack 默认配置的需求，可以在项目根目录中，按需创建 vue.config.js 这个配置文件，从

    而对项目的打包发布过程做自定义的配置（具体配置参考 https://cli.vuejs.org/zh/config/#vue-config-js）。

  * 在 vue.config.js 导出的配置对象中，新增 configureWebpack 或 chainWebpack 节点，来自定义 webpack 

    的打包配置。

    在这里， configureWebpack 和 chainWebpack 的作用相同，唯一的区别就是它们修改 webpack 配置的方

    式不同：

    ①  chainWebpack 通过链式编程的形式，来修改默认的 webpack 配置

    ②  configureWebpack 通过操作对象的形式，来修改默认的 webpack 配置

    两者具体的使用差异，可参考如下网址：

    https://cli.vuejs.org/zh/guide/webpack.html#webpack-%E7%9B%B8%E5%85%B3

* **通过 externals 加载外部 CDN 资源**

  * 默认情况下，通过 import 语法导入的第三方依赖包，最终会被打包合并到同一个文件中，从而导致打包成功后，单文件体积过大的问题。
  * 为了解决上述问题，可以通过 webpack 的 externals 节点，来配置并加载外部的 CDN 资源。凡是声明在externals 中的第三方依赖包，都不会被打包。
  * 开发时直接下载引瑞
    * 发布时把直接引入可以省的包 使用window全局的方式来查找  也就是说 CDN 挂载 通过CDN挂载的方式进行引用

* 路由懒加载

  ```
  // 分组名生成文件
  const Login = () => import(/* webpackChunkName: "login_home_welome" */ 'components/login/Login')
  const Home = () => import(/* webpackChunkName: "login_home_welome" */ 'components/home/Home')
  const Welcome = () => import(/* webpackChunkName: "login_home_welome" */ 'components/home/welcome/Welcome')
  
  const Users = () => import(/* webpackChunkName: "Users_Rights_Roles" */ 'components/home/users/Users')
  const Rights = () => import(/* webpackChunkName: "Users_Rights_Roles" */ 'components/home/power/rights/Rights')
  const Roles = () => import(/* webpackChunkName: "Users_Rights_Roles" */ 'components/home/power/roles/Roles')
  
  const Cate = () => import(/* webpackChunkName: "Cate_Params" */ 'components/home/goods/cate/Cate')
  const Params = () => import(/* webpackChunkName: "Cate_Params" */ 'components/home/goods/params/Params')
  
  const GoodsList = () => import(/* webpackChunkName: "GoodsList_Add" */ 'components/home/goods/list/List')
  const Add = () => import(/* webpackChunkName: "GoodsList_Add" */ 'components/home/goods/list/children/Add')
  
  const Order = () => import(/* webpackChunkName: "Order_Report" */ 'components/home/order/Order')
  const Report = () => import(/* webpackChunkName: "Order_Report" */ 'components/home/report/Report')
  // 独立生成一个文件
  const Login = () => import('components/login/Login')
  const Home = () => import('components/home/Home')
  const Welcome = () => import('components/home/welcome/Welcome')
  
  const Users = () => import('components/home/users/Users')
  const Rights = () => import('components/home/power/rights/Rights')
  const Roles = () => import('components/home/power/roles/Roles')
  
  const Cate = () => import('components/home/goods/cate/Cate')
  const Params = () => import('components/home/goods/params/Params')
  
  const GoodsList = () => import('components/home/goods/list/List')
  const Add = () => import('components/home/goods/list/children/Add')
  
  const Order = () => import('components/home/order/Order')
  const Report = () => import('components/home/report/Report')
  
  ```



* 开启gzip包  compression
* 开启 HTTPS 
  * 官网: https://freessl.org/
  * 
* pm2 关闭命令行窗口依旧执行
  * npm i pm2 -g
    * 启动项目: pm2 start script --name(自定义名字)
    * 查看运行项目：pm2 ls 
    * 重启项目：pm2 restart 自定义名称
    * 停止项目：pm2 stop 自定义名称
    * 删除项目：pm2 delete 自定义名称
