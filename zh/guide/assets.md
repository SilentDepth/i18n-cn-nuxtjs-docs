---
title: 资源文件
description: 默认情况下 Nuxt 使用 vue-loader、file-loader 和 url-loader 等 Webpack 加载器来处理资源。你也可以使用静态资源目录来存放静态资源。
---

> 默认情况下 Nuxt 使用 vue-loader、file-loader 和 url-loader 等 Webpack 加载器来处理资源。你也可以使用静态资源目录来存放静态资源。

## Webpack 构建资源

默认情况下, [vue-loader](http://vue-loader.vuejs.org/zh/) 自动使用 css-loader 和 Vue 模板编译器来处理你的样式和模板文件。在此编译过程中，所有的资源 URL 例如 `<img src="...">`、 `background: url(...)` 和 CSS 中的 `@import` 均会被解析成模块依赖。

举个例子, 假设我们有以下文件目录结构：

```bash
-| assets/
----| image.png
-| pages/
----| index.vue
```

如果我们在 CSS 代码中使用 `url('~/assets/image.png')`，那么编译后它将被转换成 `require('~/assets/image.png')`。

> 注意 `~assets`（没有斜杠）不能发挥作用，因为 `~` 在 `css-loader` 中是一个特殊符号，并且会被解析到 [modules](https://github.com/css-modules/css-modules)。

又或者如果我们在 `pages/index.vue` 中编写：

```html
<template>
  <img src="~/assets/image.png">
</template>
```

这会被编译成：

```js
createElement('img', { attrs: { src: require('~/assets/image.png') }})
```

由于 `.png` 并非 JavaScript 文件，因此 Nuxt.js 通过配置 Webpack 使用 [file-loader](https://github.com/webpack/file-loader) 和 [url-loader](https://github.com/webpack/url-loader) 来处理此类引用。

使用这两个加载器的好处有：

- file-loader 能让你指定从什么地方拷贝资源文件、发布后放到哪个目录去，以及如何使用版本哈希码命名文件以实现更好的缓存策略。
- url-loader 允许你在文件体积小于指定阈值时使其转换成内联 base-64 编码。这能减少小文件产生的 HTTP 请求数量。如果文件体积超过了阈值，将自动降级交给 file-loader 来处理。

Nuxt.js 实际使用的默认加载器配置如下：

```js
[
  {
    test: /\.(png|jpe?g|gif|svg)$/,
    loader: 'url-loader',
    query: {
      limit: 1000, // 1kB
      name: 'img/[name].[hash:7].[ext]'
    }
  },
  {
    test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
    loader: 'url-loader',
    query: {
      limit: 1000, // 1kB
      name: 'fonts/[name].[hash:7].[ext]'
    }
  }
]
```

这表示，任何小于 1 KB 的文件会被转换成 base-64 编码形式的内联引用。否则图片、字体资源会被拷贝至相应的子目录（位于 `.nuxt` 目录中），并重命名为包含版本哈希码的文件名以增强缓存性能。

当用 `nuxt` 命令启动我们的应用时，`pages/index.vue` 中的模板代码：

```html
<template>
  <img src="~/assets/image.png">
</template>
```

将被编译成：

```html
<img src="/_nuxt/img/image.0c61159.png">
```

如果你想更新这些加载器的配置或者禁用他们，请参考 [build.extend](/api/configuration-build#extend)。

## 静态文件

如果你不想你的资源经 Webpack 处理，你可以在项目根目录创建 `static` 目录并存放你的资源文件。

这些文件会自动被 Nuxt 映射到你的项目根路径。

这个特性在处理诸如 `robots.txt`、`sitemap.xml` 或 `CNAME`（用于 GitHub Pages）等文件时非常有用。

你可以在代码中使用以根路径 `/` 开头的绝对地址来引用这些文件：

```html
<!-- 引用 static 目录下的图片 -->
<img src="/my-image.png"/>

<!-- 引用 assets 目录下经 webpack 处理后的图片 -->
<img src="~/assets/my-image-2.png"/>
```
