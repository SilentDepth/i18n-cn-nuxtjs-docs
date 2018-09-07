---
title: 目录结构
description: Nuxt.js 默认的应用目录结构提供了一个良好的开发起点，适用于任意规模的应用。
---

> Nuxt.js 默认的应用目录结构提供了一个良好的开发起点，适用于任意规模的应用。当然，你也可以根据自己的偏好组织应用代码。

## 目录

### 资源目录

`assets` 目录用于存放未经编译的资源，如 Less、Sass 或 JavaScript。

[关于 assets 目录的更多信息](/guide/assets)

### 组件目录

`components` 目录用于存放你的 Vue.js 组件。Nuxt.js 不会扩展这里的组件的数据能力。

### 布局目录

`layouts` 目录用于存放你的应用布局。

_此目录名称不可修改。_

[关于布局的更多信息](/guide/views#布局)

### 中间件目录

`middleware` 目录用于存放你的应用中间件。你可以通过中间件在渲染一个页面或一组页面（布局）之前运行自定义函数。

[关于中间件的更多信息](/guide/routing#中间件)

### 页面目录

`pages` 目录用于存放你的应用视图和路由。Nuxt.js 会读取此目录下所有的 `.vue` 文件并生成对应的路由配置。

_此目录名称不可修改。_

[关于页面的更多信息](/guide/views)

### 插件目录

`plugins` 目录用于存放需要在主 Vue.js 应用实例化之前运行的 JavaScript 插件。

[关于插件的更多信息](/guide/plugins)

### 静态文件目录

`static` 目录用于存放应用的静态文件。此目录下的文件会被映射至 `/` 路径下。

**例如：**`/static/robots.txt` 会被映射至 `/robots.txt`

_此目录名称不可修改。_

[关于静态文件的更多信息](/guide/assets#静态文件)

### Store 目录

`store` 目录用于存放应用的 [Vuex Store](https://vuex.vuejs.org/zh/) 文件。Nuxt.js 框架内部实现了 Vuex Store 的相关配置，在此目录下新建 `index.js` 文件可自动启用这些配置。

_此目录名称不可修改。_

[关于 store 的更多信息](/guide/vuex-store)

### nuxt.config.js 文件

`nuxt.config.js` 文件用于储存 Nuxt.js 个性化配置。

_此文件名称不可修改。_

[关于 `nuxt.config.js` 的更多信息](/guide/configuration)

### package.json 文件

`package.json` 文件用于储存应用的依赖和命令信息。

_此文件名称不可修改。_

## 别名

| 别名 | 目录 |
|-----|------|
| `~` 或 `@` | [srcDir](/api/configuration-scrdir) |
| `~~` 或 `@@` | [rootDir](/api/configuration-rootdir) |

默认情况下，`scrDir` 与 `rootDir` 相同。

<p class="Alert Alert--nuxt-green"><b>提示：</b>在 `vue` 模板中，如果你需要引用 `assets` 或 `static` 目录下的文件，请使用 `~/assets/your_image.png` 和 `~/static/your_image.png` 的形式。</p>