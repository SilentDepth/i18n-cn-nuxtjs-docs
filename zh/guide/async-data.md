---
title: 异步数据
description: 如果你想在服务端获取并渲染一些数据，Nuxt.js 对提供了一个 `asyncData` 方法，使你可以在设置组件数据之前进行异步操作。
---

> 如果你想在服务端获取并渲染一些数据，Nuxt.js 对提供了一个 `asyncData` 方法，使你可以在设置组件数据之前进行异步操作。

## asyncData 方法

有时你只是想在不使用 store 的情况下由服务端获取并预渲染一些数据。`asyncData` 方法会在组件（**限于页面组件**）每次加载之前被调用。它可以在服务端或路由导航之前被调用。这个方法接受[上下文对象](/api/context)作为第一个参数，你可以利用它来获取数据，而 Nuxt.js 会将其合并入组件数据。

<div class="Alert Alert--orange">你**不能**在 `asyncData` 方法中通过 `this` 访问组件实例，因为它是在**组件初始化之前**被调用的。</div>

Nuxt.js 提供了几种不同的方式来使用 `asyncData` 方法，你可以选择其中你最熟悉的一种：

1. 返回一个 `Promise`。Nuxt.js 会等这个 Promise 被 resolve 之后再渲染组件。
2. 使用 [async/await](https://github.com/lukehoban/ecmascript-asyncawait)（[了解更多](https://zeit.co/blog/async-and-await)）
3. 定义一个回调函数作为第二个参数。它会以这种形式被调用：`callback(err, data)`

<div class="Alert Alert--grey">我们使用 [axios](https://github.com/mzabriskie/axios) 构造同构 HTTP 请求，我们<strong>强烈建议</strong>在你的 Nuxt 项目中使用我们的 [axios 模块](https://axios.nuxtjs.org/)。</div>

### 返回 Promise

```js
export default {
  asyncData ({ params }) {
    return axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      return { title: res.data.title }
    })
  }
}
```

### 使用 async/await

```js
export default {
  async asyncData ({ params }) {
    let { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}
```

### 使用回调函数

```js
export default {
  asyncData ({ params }, callback) {
    axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      callback(null, { title: res.data.title })
    })
  }
}
```

### 数据的展示

`asyncData` 方法的返回值会被合并入 data，你可以像平常一样展示这些数据：

```html
<template>
  <h1>{{ title }}</h1>
</template>
```

## 上下文对象 context

若要了解 `context` 的全部可用属性，参阅 [API 基础 `context`](/api/context)。

### 访问动态路由数据

你可以使用注入到 `asyncData` 方法中的上下文对象来访问动态路由数据。例如，你可以使用配置好的文件或目录名称来访问动态路由参数，也就是说如果你新建了一个名为 `_slug.vue` 的文件，你可以通过 `context.params.slug` 来访问它。

### 监听 query 变化

默认情况下当 query 变化时 `asyncData` 方法**不会被调用**。如果你不想这样，比如说在编写一个分页组件时，你可以通过页面组件的 `watchQuery` 属性设置需要监听的 query 参数。更多内容请见 [API `watchQuery` 页](/api/pages-watchquery)。

## 错误处理

Nuxt.js 在 `context` 中提供了一个 `error(params)` 方法，你可以通过调用该方法来显示错误信息页面。`params.statusCode` 可用于渲染服务端返回的请求状态码。

以返回 `Promise` 的方式举个例子：

```js
export default {
  asyncData ({ params, error }) {
    return axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      return { title: res.data.title }
    })
    .catch((e) => {
      error({ statusCode: 404, message: 'Post not found' })
    })
  }
}
```

如果你使用回调函数的方式, 你可以将错误信息对象直接传给该回调函数，Nuxt.js 内部会自动调用 `error` 方法：

```js
export default {
  asyncData ({ params }, callback) {
    axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      callback(null, { title: res.data.title })
    })
    .catch((e) => {
      callback({ statusCode: 404, message: 'Post not found' })
    })
  }
}
```

如果你想定制错误信息页面，参阅[页面布局](/guide/views#布局)。
