---
title: 模块
description: 模块可以扩展 Nuxt.js 的核心功能，带来无限可能。
---

> 模块可以扩展 Nuxt.js 的核心功能，带来无限可能。

## 介绍

在用 Nuxt 开发生产应用时，你会很容易发现框架的核心功能是不够用的。Nuxt 可以通过配置选项和插件得到一定的扩展，但在多个项目间维护这些自定义内容令人感到枯燥和疲惫。但另一方面，将所有可能的功能都集成到 Nuxt 中会使其变得非常复杂并难以使用。

这就是 Nuxt 提供一个高层次模块系统的原因，它使扩展核心功能这件事变得容易。模块只是一个函数，它们会在 Nuxt 启动时依次被调用，而框架会等每个模块完成调用后再执行其他逻辑。这样，模块可以定制 Nuxt 的几乎方方面面。有赖于 Nuxt 的模块化设计（基于 Webpack 的 [Tapable](https://github.com/webpack/tapable)），模块可以很方便地注册钩子到特定入口（例如 builder 初始化<!-- builder initialization 该怎么翻译？ -->）。模块还可以覆盖模板内容，配置 Webpack Loader，添加 CSS 库，甚至执行任意任务。

最棒的是，Nuxt 模块可以来自 npm 包，这使得它很容易在项目间复用，或在 Nuxt 社区间分享，以建立高质量的 Nuxt 扩展生态。

模块尤其适用于这些情况：

- 你的团队追求**行动灵敏**，需要快速创建新项目。
- 你不想再**重复造轮子**，例如集成 Google Analytics。
- 你是一个**开源**狂热爱好者，希望方便地与社区**分享**你的工作成果。
- 你的**企业**注重**质量**和**复用性**。
- 你时间匆忙，经常来不及探究每个库或工具的细节。
- 你不想再应对低层次接口的破坏性更新，只想要 **just work™** 的方案。

## 编写一个简单的模块

如刚才所说，模块只是一个单纯的函数，它们可以打包为 npm 模块或直接写入到项目源码。

**modules/simple.js**

```js
module.exports = function SimpleModule (moduleOptions) {
  // 在这里写你的代码
}

// 需要发布为 npm 包时请启用下面这行代码
// module.exports.meta = require('./package.json')
```

**`moduleOptions`**

这是用户使用 `modules` 数组传递的对象，我们可以用它定制其行为。<!-- ??? -->

**`this.options`**

你可以通过此引用直接获取到 Nuxt 选项。这是 `nuxt.config.js` 与默认选项合并后的结果，同时也可以在模块间共用。

**`this.nuxt`**

这是对当前 Nuxt 实例的引用。参考 [Nuxt 类](/api/internals-nuxt)文档了解公开方法。

**`this`**

模块的上下文。参考 [ModuleContainer 类](/api/internals-module-container)文档了解公开方法。

**`module.exports.meta`**

如果你需要将模块发布为 npm 包，这一行代码是**必需的**的。Nuxt 内部需要此元信息以更好地与你的 npm 包交互。

**nuxt.config.js**

```js
module.exports = {
  modules: [
    // 简单用法
    '~/modules/simple'

    // 传递选项
    ['~/modules/simple', { token: '123' }]
  ]
}
```

我们接下来告诉 Nuxt 为项目加载指定的模块，其中选项参数是可选的。请参考[模块配置](/api/configuration-modules)了解更多信息。

## 异步模块

不是所有模块都同步地进行工作。例如你可能要开发一个需要请求一些 API 或者执行异步 IO 操作的模块。为了解决这个问题，Nuxt 支持返回一个 Promise 或调用回调函数的异步模块。

### 使用 async/await

<p class="Alert Alert--orange">需要注意，`aysnc`/`await` 仅被 Node.js > 7.2 支持。所以如果你是一个模块开发者，请至少在用户使用这个语法时提醒用户这件事。如果模块本身包含较多的异步逻辑，或需要更好的旧版本兼容性，你可以使用打包器转译代码为旧版 Node.js 支持的语法，或使用 Promise。</p>

```js
const fse = require('fs-extra')

module.exports = async function asyncModule() {
  // 你可以在这里使用 `async`/`await` 执行异步逻辑
  let pages = await fse.readJson('./pages.json')
}
```

### 返回 Promise

```js
const axios = require('axios')

module.exports = function asyncModule() {
  return axios.get('https://jsonplaceholder.typicode.com/users')
    .then(res => res.data.map(user => '/users/' + user.username))
    .then(routes => {
      // 扩展 Nuxt 路由，然后做点什么
    })
}
```

### 使用回调函数

```js
const axios = require('axios')

module.exports = function asyncModule(callback) {
  axios.get('https://jsonplaceholder.typicode.com/users')
    .then(res => res.data.map(user => '/users/' + user.username))
    .then(routes => {
      callback()
    })
}
```

## 常用代码

### 顶级选项

在 `nuxt.config.js` 中注册模块时，如果我们可以使用顶级选项，事情会更方便。这允许我们合并多个选项来源。

```js
module.exports = {
  modules: [
    '@nuxtjs/axios'
  ],

  // axios 模块可以通过 `this.options.axios` 获取到这个
  axios: {
    option1,
    option2
  }
}
```

**module.js**

```js
module.exports = function (moduleOptions) {
  const options = Object.assign({}, this.options.axios, moduleOptions)
  // ...
}
```

### 提供插件

模块附带一个或多个插件是一种常见情况。例如 [bootstrap-vue](https://bootstrap-vue.js.org) 模块会要求将其自身注册到 Vue 中。为了解决这个问题我们可以使用 `this.addPlugin` 辅助函数。

**plugin.js**

```js
import Vue from 'vue'
import BootstrapVue from 'bootstrap-vue/dist/bootstrap-vue.esm'

Vue.use(BootstrapVue)
```

**module.js**

```js
const path = require('path')

module.exports = function nuxtBootstrapVue (moduleOptions) {
  // 注册 `plugin.js`
  this.addPlugin(path.resolve(__dirname, 'plugin.js'))
}
```

### 模板插件

已注册的模板和插件可以通过 [Lodash 模板](https://lodash.com/docs/4.17.4#template)根据条件改变注册插件的内容。

**plugin.js**

```js
// 设置 Google Analytics UA
ga('create', '<%= options.ua %>', 'auto')

<% if (options.debug) { %>
// 开发环境专有的代码
<% } %>
```

**module.js**

```js
const path = require('path')

module.exports = function nuxtBootstrapVue (moduleOptions) {
  // 注册 `plugin.js` 模板
  this.addPlugin({
    src: path.resolve(__dirname, 'plugin.js'),
    options: {
      // Nuxt 会在复制插件到项目时将 `options.ua` 替换为 `123`
      ua: 123,

      // 开发环境专有的部分会在构建生产版本时从插件代码中被移除
      debug: this.options.dev
    }
  })
}
```

### 添加 CSS 库

假设一个需要检测某个 CSS 库是否存在以避免重复的场景，我们可以添加一个**开关选项**到模块中来控制 CSS 库的使用情况。参考下面这个例子：

```js
module.exports = function (moduleOptions) {
  if (moduleOptions.fontAwesome !== false) {
    // 添加 Font Awesome
    this.options.css.push('font-awesome/css/font-awesome.css')
  }
}
```

### 提交资源

我们可以通过注册一个 Webpack 插件在构建时提交资源。

**module.js**

```js
module.exports = function (moduleOptions) {
  const info = 'Built by awesome module - 1.3 alpha on ' + Date.now()

  this.options.build.plugins.push({
    apply (compiler) {
      compiler.plugin('emit', (compilation, cb) => {

        // 这会生成 `.nuxt/dist/info.txt' 文件并以变量 info 的值为内容.
        // source 也可以是一个 Buffer
        compilation.assets['info.txt'] = { source: () => info, size: () => info.length }

        cb()
      })
    }
  })
}
```

### 注册自定义 Loader

我们可以像 `nuxt.config.js` 中的 `build.extend` 一样使用 `this.extendBuild` 方法。

**module.js**

```js
module.exports = function (moduleOptions) {
    this.extendBuild((config, { isClient, isServer }) => {
      // `.foo` Loader
      config.module.rules.push({
        test: /\.foo$/,
        use: [...]
      })

      // 定制已有的 Loader
      // 相关的 Nuxt 源码：
      // https://github.com/nuxt/nuxt.js/blob/dev/lib/builder/webpack/base.js
      const barLoader = config.module.rules.find(rule => rule.loader === 'bar-loader')
  })
}
```

## 运行钩子函数

你的模块可能需要在指定情况做点什么，但不只是在 Nuxt 初始化时。我们可以使用强大的 [Tapable](https://github.com/webpack/tapable) 插件系统在指定事件下执行任务。如果钩子函数返回 Promise 或定义为 `async` 函数，Nuxt 会等待异步过程结束。

```js
module.exports = function () {
  // 添加 module 钩子
  this.nuxt.hook('module', moduleContainer => {
    // 此函数会在所有模块完成加载后被调用
  })

  // 添加 renderer 钩子
  this.nuxt.hook('renderer', renderer => {
    // 此函数会在 renderer 创建后被调用
  })

  // 添加 build 钩子
  this.nuxt.hook('build', async builder => {
    // 此函数会在 builder 创建后被调用

    // 我们还可以在这里注册内部钩子
    builder.hook('compile', ({compiler}) => {
        // 此函数会在 Webpack 编译器启动之前被调用
    })
  })

  // 添加 generate 钩子
  this.nuxt.hook('generate', async generator => {
    // 此函数会在 Nuxt generate 功能启动时被调用
  })
}
```

<p class="Alert">模块还有很多很多钩子和可能性。请参考 [Nuxt 内部细节](/api/internals)了解更多 Nuxt 内部 API。</p>