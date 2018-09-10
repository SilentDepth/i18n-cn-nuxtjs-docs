---
title: 路由
description: Nuxt.js 依据页面文件目录结构生成应用的路由配置。
---

> Nuxt.js 依据 `pages` 目录结构自动生成 [vue-router](https://github.com/vuejs/vue-router) 模块的路由配置。

<div class="Alert Alert--grey">我们推荐使用 [`<nuxt-link>`](/api/components-nuxt-link) 组件在页面间导航。</div>

例如：

```html
<template>
  <nuxt-link to="/">Home page</nuxt-link>
</template>
```

## 简单路由

对于如下的文件目录结构：

```bash
pages/
--| user/
-----| index.vue
-----| one.vue
--| index.vue
```

Nuxt.js 会自动生成如下的路由配置：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'user',
      path: '/user',
      component: 'pages/user/index.vue'
    },
    {
      name: 'user-one',
      path: '/user/one',
      component: 'pages/user/one.vue'
    }
  ]
}
```

## 动态路由

在 Nuxt.js 里面定义带参数的动态路由，需要创建对应的**以下划线作为前缀**的 Vue 文件或目录。

对于如下的文件目录结构：

```bash
pages/
--| _slug/
-----| comments.vue
-----| index.vue
--| users/
-----| _id.vue
--| index.vue
```

Nuxt.js 会自动生成如下的路由配置：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'users-id',
      path: '/users/:id?',
      component: 'pages/users/_id.vue'
    },
    {
      name: 'slug',
      path: '/:slug',
      component: 'pages/_slug/index.vue'
    },
    {
      name: 'slug-comments',
      path: '/:slug/comments',
      component: 'pages/_slug/comments.vue'
    }
  ]
}
```

你会发现名称为 `users-id` 的路由路径带有 `:id?` 参数，表示该路由是可选的。如果你想将它设置为必选的路由，需要在 `users/_id` 目录内创建一个 `index.vue` 文件。

<p class="Alert Alert--info"><b>注意：</b>动态路由会被 `generate` 命令忽略。参考：[静态化 API 文档路由部分](/api/configuration-generate#routes)</p>

### 路由参数校验

Nuxt.js 可以让你在动态路由组件中定义参数校验方法。

举个例子：`pages/users/_id.vue`

```js
export default {
  validate ({ params }) {
    // 必须是个数字
    return /^\d+$/.test(params.id)
  }
}
```

如果校验方法返回的值不为 `true`，Nuxt.js 将自动加载显示 404 错误页面。

想了解关于路由参数校验的信息，请参考[页面校验 API 文档](/api/pages-validate)。

## 嵌套路由

Nuxt.js 允许你通过 vue-router 的子路由创建嵌套路由。

要定义包含嵌套路由的父组件，你需要创建一个 Vue 文件，同时创建一个**与该文件同名**的目录存放子视图文件。

<p class="Alert Alert--info"><b>注意：</b>别忘了在父组件（`.vue` 文件）里使用 `<nuxt-child/>`。</p>

对于如下的文件目录结构：

```bash
pages/
--| users/
-----| _id.vue
-----| index.vue
--| users.vue
```

Nuxt.js 会自动生成如下的路由配置：

```js
router: {
  routes: [
    {
      path: '/users',
      component: 'pages/users.vue',
      children: [
        {
          path: '',
          component: 'pages/users/index.vue',
          name: 'users'
        },
        {
          path: ':id',
          component: 'pages/users/_id.vue',
          name: 'users-id'
        }
      ]
    }
  ]
}
```

## 动态嵌套路由

这个应用场景比较少见，但是 Nuxt.js 仍然支持：在动态路由下配置动态子路由。

对于如下的文件目录结构：

```bash
pages/
--| _category/
-----| _subCategory/
--------| _id.vue
--------| index.vue
-----| _subCategory.vue
-----| index.vue
--| _category.vue
--| index.vue
```

Nuxt.js 会自动生成如下的路由配置：

```js
router: {
  routes: [
    {
      path: '/',
      component: 'pages/index.vue',
      name: 'index'
    },
    {
      path: '/:category',
      component: 'pages/_category.vue',
      children: [
        {
          path: '',
          component: 'pages/_category/index.vue',
          name: 'category'
        },
        {
          path: ':subCategory',
          component: 'pages/_category/_subCategory.vue',
          children: [
            {
              path: '',
              component: 'pages/_category/_subCategory/index.vue',
              name: 'category-subCategory'
            },
            {
              path: ':id',
              component: 'pages/_category/_subCategory/_id.vue',
              name: 'category-subCategory-id'
            }
          ]
        }
      ]
    }
  ]
}
```

### SPA 回落方案

你还可以对动态路由启用 SPA 回落。Nuxt.js 会额外输出一个作用与 `index.html` 相同的文件，但可用于 `mode: 'spa'`。大部分静态托管服务可配置为当没有匹配文件时使用这个 SPA 文件。它不会包含 `<head>` 信息或任何 HTML，但它仍然可用来解析和从 API 加载数据。

我们需要在 `nuxt.config.js` 文件中这样启用这个功能：

``` js
module.exports = {
  generate: {
    fallback: true, // 如果你想使用 '404.html'
    fallback: 'my-fallback/file.html' // 如果你的托管服务需要指定一个文件
  }
}
```

#### 用于 Surge

Surge [可以处理](https://surge.sh/help/adding-a-custom-404-not-found-page) `200.html` 和 `404.html`。`generate.fallback` 默认为 `200.html`，所以什么都不需要改。

#### 用于 GitHub Pages 和 Netlify

GitHub Pages 和 Netlify 可以自动识别 `404.html`，所以设置 `generate.fallback` 为 `true` 就全部搞定了！

#### 用于 Firebase 托管服务

要用于 Firebase 托管服务，设置 `generate.fallback` 为 `true` 之外还需要使用如下配置（[更多信息](https://firebase.google.com/docs/hosting/url-redirects-rewrites#section-rewrites)）：

``` json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/404.html"
      }
    ]
  }
}
```

## 过渡动效

Nuxt.js 使用 [`<transition>`](https://cn.vuejs.org/v2/guide/transitions.html#%E5%8D%95%E5%85%83%E7%B4%A0-%E7%BB%84%E4%BB%B6%E7%9A%84%E8%BF%87%E6%B8%A1) 组件来实现路由切换时的过渡动效。

### 全局设置

<p class="Alert Alert--info"><b>提示：</b>Nuxt.js 默认使用的过渡效果名称为 `page`</p>

如果想让每一个页面的切换都有淡出 (fade) 效果，我们需要创建一个所有路由共用的 CSS 文件。所以我们可以在 `assets/` 目录下创建这个文件：

在全局样式文件 `assets/main.css` 里添加以下样式：

```css
.page-enter-active, .page-leave-active {
  transition: opacity .5s;
}
.page-enter, .page-leave-active {
  opacity: 0;
}
```

然后添加到 `nuxt.config.js` 文件中：

```js
module.exports = {
  css: [
    'assets/main.css'
  ]
}
```

### 页面设置

如果想给某个页面单独定义过渡特效的话，只要在该页面组件中配置 `transition` 字段即可。

在全局样式 `assets/main.css` 中添加一下内容：

```css
.test-enter-active, .test-leave-active {
  transition: opacity .5s;
}
.test-enter, .test-leave-active {
  opacity: 0;
}
```

然后我们将页面组件中的 `transition` 属性的值设置为 `test` 即可：

```js
export default {
  transition: 'test'
}
```

关于 transition 属性的更多信息请参考：[过渡动效 API 文档](/api/pages-transition)

## 中间件

> 中间件允许您定义一个自定义函数运行在一个页面或一组页面渲染之前。

**每一个中间件应放在 `middleware/` 目录中。**文件名将成为中间件名称（`middleware/auth.js` 将成为 `auth` 中间件）。

一个中间件接受[上下文对象](/api#上下文对象)作为第一个参数：

```javascript
export default function (context) {
  context.userAgent = context.isServer ? context.req.headers['user-agent'] : navigator.userAgent
}
```

中间件执行流程顺序：

1. `nuxt.config.js`
2. 匹配的布局
3. 匹配的页面

中间件可以异步执行，只需要返回一个 `Promise` 或使用第 2 个参数 `callback`：

`middleware/stats.js`

```javascript
import axios from 'axios'

export default function ({ route }) {
  return axios.post('http://my-stats-api.com', {
    url: route.fullPath
  })
}
```

然后在你的 `nuxt.config.js`、布局组件或者页面组件中使用 `middleware` 属性：

`nuxt.config.js`

```javascript
module.exports = {
  router: {
    middleware: 'stats'
  }
}
```

此时 `stats` 中间件将在每个路由改变时被调用。

如果你想看到一个使用中间件的真实例子，请看在 GitHub 上的 [example-auth0](https://github.com/nuxt/example-auth0)。
