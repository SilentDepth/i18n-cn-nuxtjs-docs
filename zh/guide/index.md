---
title: 关于 Nuxt.js
description: “2016 年 10 月 25 日，zeit.co 背后的团队发布了 Next.js，一个面向 React 应用的服务端渲染框架。几个小时后，开发一个与 Next.js 异曲同工、面向 Vue.js 应用的服务端渲染框架的点子应运而生，我们称之为：Nuxt.js。”
---

> “2016 年 10 月 25 日，[zeit.co](https://zeit.co/) 背后的团队发布了 [Next.js](https://zeit.co/blog/next)，一个面向 React 应用的服务端渲染框架。几个小时后，开发一个与 Next.js 异曲同工、面向 [Vue.js](https://cn.vuejs.org/) 应用的服务端渲染框架的点子应运而生，我们称之为：**Nuxt.js**。”

## Nuxt.js 是什么？

Nuxt.js 是一个基于 Vue.js 的通用应用框架。

通过对客户端/服务端基础架构的抽象组织，Nuxt.js 主要关注的是应用的 **UI 渲染**。

我们的目标是创建一个灵活的应用框架，你可以基于它初始化新项目的基础结构代码，或者在已有 Node.js 项目中使用 Nuxt.js。

Nuxt.js 预设了利用 Vue.js 开发**服务端渲染**的应用所需要的各种配置。

除此之外，我们还提供了一种命令叫：*nuxt generate*，为基于 Vue.js 的应用提供生成对应的**静态**站点的功能。

我们相信这个命令所提供的功能，是向开发集成各种微服务（microservices）的 Web 应用迈开的新一步。

作为框架，Nuxt.js 为客户端/服务端这种典型的应用架构模式提供了许多有用的特性，例如异步数据加载、中间件支持、布局支持等。

## Nuxt.js 是如何运作的？

![基于 Vue、Webpack 和 Babel](https://i.imgur.com/avEUftE.png)

Nuxt.js 集成了以下组件/框架，用于开发完整而强大的 Web 应用：

- [Vue 2](https://cn.vuejs.org/)
- [Vue Router](https://router.vuejs.org/zh/)
- [Vuex](https://vuex.vuejs.org/zh/)（添加了 [Store 配置](/guide/vuex-store)时才会引入）
- [Vue Server Renderer](https://ssr.vuejs.org/zh/)（[`mode: 'spa'`](/api/configuration-mode) 的情况下不会引入）
- [vue-meta](https://github.com/declandewet/vue-meta)

总体积仅为：**57kB min+gzip**（如果包含 Vuex 则为 53kB）。

在底层我们使用 [Webpack](https://github.com/webpack/webpack) 和 [vue-loader](https://vue-loader.vuejs.org/zh/) 以及 [babel-loader](https://github.com/babel/babel-loader) 来打包、分割、压缩你的代码。

## 特性

- 支持 Vue 文件（`*.vue`）
- 自动代码分割
- 服务端渲染
- 强大的路由功能，支持异步数据
- 静态文件服务
- ES6/ES7 语法转译
- 打包和压缩 JS 和 CSS
- `<head>` 元素管理（`<title>`、`<meta>` 等）
- 开发环境模块热更新
- 预处理器：Sass、Less、Stylus 等
- HTTP/2 Push 支持
- 可扩展的模块化架构

## 流程图

下图阐述了 Nuxt.js 应用一个完整的服务器请求到渲染（或用户通过 `<nuxt-link>` 切换路由渲染页面）的流程：

![nuxt-schema](/nuxt-schema.png)

## 服务端渲染（通用 SSR）

你可以使用 Nuxt.js 作为你项目的 UI 渲染框架。

运行 `nuxt` 命令会启动一个已配置好热重载和 [Vue Server Renderer](https://ssr.vuejs.org/zh/) 的开发服务器，并自动由服务端渲染你的应用。

### 单页面应用 (SPA)

不论出于什么原因，如果你更倾向于不使用服务端渲染或需要静态托管你的应用，你可以简单地通过 `nuxt --spa` 命令使用 SPA 模式。结合*静态化*特性，它提供了一个强大的 SPA 开发能力而无需使用 Node.js 运行时或任何特殊的服务端处理。

阅读[命令](/guide/commands)章节以了解更多的信息。

如果你的项目有自己的 Web 服务器，你还可以将 Nuxt.js 当作中间件来使用。在开发通用的 Web 应用过程中，Nuxt.js 是可插拔的，没有太多的限制。阅读[在代码中使用 Nuxt.js](/api/nuxt) 章节了解更多信息。

## 静态化（预渲染）

Nuxt.js 的一个重大创新是 `nuxt generate` 命令。

当构建你的应用时，该命令会生成每一个路由的静态 HTML 文件。

例如，对于如下的文件目录结构：

```bash
-| pages/
----| about.vue
----| index.vue
```

会静态化为：

```
-| dist/
----| about/
------| index.html
----| index.html
```

凭借这个特性，你可以将你的应用的静态化版托管到任何一个静态托管服务上。

本站点就是一个绝佳的例子，它静态化后托管于 GitHub Pages：

- [源码](https://github.com/nuxt/nuxtjs.org)
- [静态化生成的文件](https://github.com/nuxt/nuxtjs.org/tree/gh-pages)

我们不想每次更新[文档代码](https://github.com/nuxt/docs)时手动生成站点的静态化版本，所以每一次推送都会调用如下的 AWS Lambda 函数：

1. 克隆 [nuxtjs.org 代码库](https://github.com/nuxt/nuxtjs.org)
2. 使用 `npm install` 命令安装依赖
3. 运行 `nuxt generate`
4. 将生成的 `dist` 目录推送至 `gh-pages` 分支

然后我们就有了一个**无服务端静态化 Web 应用** :)

我们进一步考虑下电商应用的场景，利用 `nuxt generate`，是不是可以将应用静态化后部署在 CDN 服务器，每当一个商品的库存发生变化时，就重新静态化下，更新下商品的库存。但是如果用户访问的时候恰巧更新了呢？我们可以通过调用电商的 API，保证用户访问的是最新的数据。这样相对于传统的电商应用来说，这种静态化的方案可以大大节省服务器的资源。
 
> 译者注：因为 CDN 节点目前只能缓存静态文件，像动态脚本 PHP 是 CDN 节点无法运行。也就是只能存 HTML、CSS、JS 这样的前端页面到 CDN 节点。所以通过 CDN 缓存静态页面，进行全球 CDN 节点布局。相对传统的动态网站，分散了对服务器的请求，降低了服务器压力。从而降低了运维成本。提升了用户体验。简单而言就是，页面静态文件 CDN，数据通过 API，前后端分离。

<div class="Alert">阅读[如何部署到 GitHub Pages](/faq/github-pages) 章节了解更多信息。</div>
