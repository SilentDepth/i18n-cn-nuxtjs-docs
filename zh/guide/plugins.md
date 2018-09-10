---
title: 插件
description: 你可以配置需要在根 Vue.js 应用实例化之前运行的 JavaScript 插件，可以是你自己写的库或第三方模块。
---

> 你可以配置需要在根 Vue.js 应用实例化之前运行的 JavaScript 插件，可以是你自己写的库或第三方模块。

<div class="Alert">需要注意的是，在任何 Vue 组件的[生命周期](https://vuejs.org/v2/guide/instance.html#Lifecycle-Diagram)内， 只有 `beforeCreate` 和 `created` 这两个钩子方法会在**客户端和服务端均被调用**。其他钩子方法仅在客户端被调用。</div>

## 第三方模块

我们可以在应用中使用第三方模块，一个典型的例子是在客户端和服务端使用 [axios](https://github.com/mzabriskie/axios) 做 HTTP 请求。

首先我们需要安装 npm 包：

```bash
npm install --save axios
```

然后在页面内可以这样使用：

```html
<template>
  <h1>{{ title }}</h1>
</template>

<script>
import axios from 'axios'

export default {
  async asyncData ({ params }) {
    let { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}
</script>
```

有一个**值得注意的问题**是，如果我们在另外一个页面内也引用了 `axios`，那么在应用打包发布的时候 `axios` 会被打包两次，而实际上我们只需要打包一次。这个问题可以通过在 `nuxt.config.js` 里面配置 `build.vendor` 来解决：

```js
module.exports = {
  build: {
    vendor: ['axios']
  }
}
```

经过上面的配置后，我们可以在任何页面里面引入 `axios` 而不用担心它会被重复打包。

## Vue 插件

假如我们想使用 [vue-notifications](https://github.com/se-panfilov/vue-notifications) 显示应用的通知信息，我们需要在程序运行前配置好这个插件。

首先增加文件 `plugins/vue-notifications.js`：

```js
import Vue from 'vue'
import VueNotifications from 'vue-notifications'

Vue.use(VueNotifications)
```

然后, 在 `nuxt.config.js` 内配置 `plugins` 如下：

```js
module.exports = {
  plugins: ['~/plugins/vue-notifications']
}
```

想了解更多关于 `plugins` 的配置方法，请参考[插件 API 文档](/api/configuration-plugins)。

实际上， `vue-notifications` 会被打包至应用的脚本代码里， 但是它属于第三方库，我们理应将它打包至库文件里以获得更好的缓存效果。（译者注：应用代码比库文件修改频繁，应尽量将第三方库打包至单独的文件中去）。

我们可以更新 `nuxt.config.js` 文件，在 `vendor` 构建配置项里添加 `vue-notifications`：

```js
module.exports = {
  build: {
    vendor: ['vue-notifications']
  },
  plugins: ['~/plugins/vue-notifications']
}
```

## 注入到 $root 和上下文

有些插件需要被注入到应用根组件，例如 [vue-i18n](https://github.com/kazupon/vue-i18n)。在 Nuxt.js 中，你可以通过导出一个方法并使用 `context` 中的 `app` 属性来做到这点：

`plugins/i18n.js`:

```js
import Vue from 'vue'
import VueI18n from 'vue-i18n'

Vue.use(VueI18n)

export default ({ app }, inject) => {
  // 将 `i18n` 实例注入到 `app`
  // 这样我们就可以在中间件和页面组件的 `asyncData`/`fetch` 中用到它
  app.i18n = new VueI18n({
    /* `VueI18n` 选项... */
  })
}
```

`nuxt.config.js`:

```js
module.exports = {
  build: {
    vendor: ['vue-i18n']
  },
  plugins: ['~/plugins/i18n.js']
}
```

请参考 [i18n 示例](/examples/i18n)来了解如何使用它。

## 只在浏览器里使用的插件

有些插件可能**只在浏览器里使用**。你可以在 `plugins` 中配置 `ssr: false` 选项来限制代码只在浏览器中运行。

例：

`nuxt.config.js`:

```js
module.exports = {
  plugins: [
    { src: '~/plugins/vue-notifications', ssr: false }
  ]
}
```

`plugins/vue-notifications.js`:

```js
import Vue from 'vue'
import VueNotifications from 'vue-notifications'

Vue.use(VueNotifications)
```

而如果你只想在服务端引入一些库，你可以利用 `process.server` 变量。在 Webpack 打包 `server.bundle.js` 文件时它的值会是 `true`。

同样的，如果你需要了解你的代码是否位于一个静态化应用（通过 `nuxt generate` 命令生成）中，你可以检查 `process.static` 变量，它的值会在静态化构建过程中及之后保持为 `true`。如果需要知道一个页面在服务端渲染的状态，你可以使用 `process.static && process.server`。
