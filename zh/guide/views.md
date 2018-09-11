---
title: 视图
description: 视图章节的内容阐述了如何在 Nuxt.js 应用中为指定的路由配置数据和视图，包括页面、布局和HTML头部等内容。
---

> 本章节的内容阐述了如何在 Nuxt.js 应用中为指定的路由配置数据和视图，包括应用模板、页面、布局和HTML头部等内容。

![nuxt-views-schema](/nuxt-views-schema.png)

## 模板

> 你可以定制化 Nuxt.js 默认的应用模板。

定制化默认的 html 模板，只需要在应用根目录下创建一个 `app.html` 的文件。

默认模板为：

```html
<!DOCTYPE html>
<html {{ HTML_ATTRS }}>
  <head>
    {{ HEAD }}
  </head>
  <body {{ BODY_ATTRS }}>
    {{ APP }}
  </body>
</html>
```

举个例子，你可以修改模板添加 IE 的条件表达式：

```html
<!DOCTYPE html>
<!--[if IE 9]><html lang="en-US" class="lt-ie9 ie9" {{ HTML_ATTRS }}><![endif]-->
<!--[if (gt IE 9)|!(IE)]><!--><html {{ HTML_ATTRS }}><!--<![endif]-->
  <head>
    {{ HEAD }}
  </head>
  <body {{ BODY_ATTRS }}>
    {{ APP }}
  </body>
</html>
```

## 布局

Nuxt.js 允许你扩展默认的布局，或在 `layouts` 目录下创建自定义的布局。

### 默认布局

可通过添加 `layouts/default.vue` 文件来扩展应用的默认布局。

*别忘了在布局文件中添加 `<nuxt/>` 组件用于显示页面的主体内容。*

默认布局的源码如下：

```html
<template>
  <nuxt/>
</template>
```

### 错误页面

> 你可以通过编辑 `layouts/error.vue` 文件来定制化错误页面.

这个布局文件不需要包含 `<nuxt/>` 标签。你可以把这个布局文件当成是显示应用错误（`404`，`500`等）的组件。

默认的错误页面源码在[这里](https://github.com/nuxt/nuxt.js/blob/master/lib/app/components/nuxt-error.vue)。

举一个个性化错误页面的例子 `layouts/error.vue`：

```html
<template>
  <div class="container">
    <h1 v-if="error.statusCode === 404">页面不存在</h1>
    <h1 v-else>出现了一个错误</h1>
    <nuxt-link to="/">首页</nuxt-link>
  </div>
</template>

<script>
export default {
  props: ['error'],
  layout: 'blog' // 你可以为错误页面指定自定义的布局
}
</script>
```

### 个性化布局

> `layouts` *根*目录下的所有文件都属于个性化布局文件，可以在页面组件中利用 `layout` 属性来引用。

*请确保在布局文件里面增加 `<nuxt/>` 组件用于显示页面非布局内容。*

举个例子 `layouts/blog.vue`：

```html
<template>
  <div>
    <div>这里是博客导航</div>
    <nuxt/>
  </div>
</template>
```

在 `pages/posts.vue` 里，可以指定页面组件使用 blog 布局：

```html
<script>
export default {
  layout: 'blog'
}
</script>
```

关于 `layout` 属性的更多信息，请参考[页面 `layout` API 文档](/api/pages-layout)。

看一下[示例视频](https://www.youtube.com/watch?v=YOKnSTp7d38)感受实际效果。

> 译者注：视频链接来自 YouTube，所以你可能需要特殊的方式才能打开。

## 页面

页面组件实际上是 Vue 组件，只不过 Nuxt.js 为这些组件添加了一些特殊的配置项（对应 Nuxt.js 提供的功能特性）以便你能快速开发通用应用。

```html
<template>
  <h1 class="red">你好 {{ name }}!</h1>
</template>

<script>
export default {
  asyncData (context) {
    // 在每一次加载组件之前执行
    return { name: 'World' }
  },
  fetch () {
    // `fetch` 方法用于在渲染页面前将所需数据写入 store
  },
  head () {
    // 为此页面设置 meta 标签
  },
  // 还有更多功能等你探索
  ...
}
</script>

<style>
.red {
  color: red;
}
</style>
```

Nuxt.js 为页面提供的特殊配置项：

| 属性 | 描述 |
|-----------|-------------|
| `asyncData` | 最重要的属性。它支持异步逻辑，并以上下文对象为参数。阅读[异步数据章节](/guide/async-data)学习如何使用它。 |
| `fetch` | 用于在渲染页面之前获取数据填充 store。它有点像 `asyncData` 方法，不同的是它不会改动组件数据。参考[页面 `fetch` API 文档](/api/pages-fetch)。 |
| `head` | 为当前页面设置 `<meta>` 标签。参考[页面 `head` API 文档](/api/pages-head)。 |
| `layout` | 指定一个布局（即 `layouts` 目录下的文件）。参考[页面 `layout` API 文档](/api/pages-layout)。 |
| `loading` | 设置为 `false` 会阻止页面自动在进入时调用 `this.$nuxt.$loading.finish()` 和离开时调用 `this.$nuxt.$loading.start()`，这允许你**手动**控制这些行为，例如[这个示例](https://nuxtjs.org/examples/custom-page-loading)。此属性只在 `nuxt.config.js` 中设置了 `loading` 选项时有效。参考[配置 `loading` API 文档](/api/configuration-loading)。
| `transition` | 指定页面切换的过渡动效。参考[页面 `transition` API 文档](/api/pages-transition)。 |
| `scrollToTop` | 布尔值（默认为 `false`）。用于控制渲染页面前是否滚动窗口到顶部的行为。此属性用于[嵌套路由](/guide/routing#嵌套路由)的应用场景。 |
| `validate` | 用于[动态路由](/guide/routing#动态路由)的校验函数。 |
| `middleware` | 设置用于当前页面的中间件。中间件会在页面渲染之前被调用。参考[中间件](/guide/routing#中间件)。 |

关于页面配置项的更多信息请参考：[API 文档](/api)

## HTML 头部

Nuxt.js 使用了 [vue-meta](https://github.com/declandewet/vue-meta) 更新应用的 `头部标签 (head)` and `html 属性`。

Nuxt.js 使用以下参数配置 `vue-meta`:

```js
{
  keyName: 'head', // 设置 meta 信息的组件对象的字段，vue-meta 会根据这 key 值获取 meta 信息
  attribute: 'data-n-head', // vue-meta 在监听标签时所添加的属性名
  ssrAttribute: 'data-n-head-ssr', // 让 vue-meta 获知 meta 信息已完成服务端渲染的属性名
  tagIDKeyName: 'hid' // 让 vue-meta 用来决定是否覆盖还是追加 tag 的属性名
}
```

### 默认 Meta 标签

Nuxt.js 允许你在 `nuxt.config.js` 里定义应用所需的所有默认 `<meta>` 标签，在 `head` 字段里配置就可以了：

一个使用自定义 viewport 和谷歌字体的配置示例：

```js
head: {
  meta: [
    { charset: 'utf-8' },
    { name: 'viewport', content: 'width=device-width, initial-scale=1' }
  ],
  link: [
    { rel: 'stylesheet', href: 'https://fonts.googleapis.com/css?family=Roboto' }
  ]
}
```

想了解 `head` 变量的所有可选项的话，请查阅 [`vue-meta` 使用文档](https://github.com/declandewet/vue-meta#recognized-metainfo-properties)。

关于 `head` 配置项的更多信息，请参考 [`head` 配置文档](/api/configuration-head)。

### 个性化特定页面的 Meta 标签

关于特定页面的 `head` 方法的更多信息，请参考[页面 `head` API 文档](/api/pages-head)。

<p class="Alert">为了避免使用子组件时 meta 标签重复生成的问题，建议使用 `hid` 属性为 meta 标签设定一个唯一标识。[了解更多](https://github.com/declandewet/vue-meta#lists-of-tags)。</p>
