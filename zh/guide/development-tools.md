---
title: 开发工具
description: Nuxt.js 可让你的 Web 开发过程更愉悦。
---

> 测试是 Web 应用开发过程中不可获缺的工作。Nuxt.js 尽量帮助你简化这部分工作。

## 端对端测试

[AVA](https://github.com/avajs/ava) 是一个很强大的 JavaScript 测试框架，结合 [jsdom](https://github.com/tmpvar/jsdom)，我们就可以轻松地给 `nuxt` 应用进行端对端测试。

首先，我们需要添加 AVA 和 jsdom 作为项目的开发依赖：

```bash
npm install --save-dev ava jsdom
```

然后在 `package.json` 中添加测试脚本，并配置 AVA 如果编译待测试的文件：

```javascript
"scripts": {
  "test": "ava",
},
"ava": {
  "require": [
    "babel-register"
  ]
},
"babel": {
  "presets": [
    "env"
  ]
}
```

接下来我们可以在 `test` 目录下编写单元测试的逻辑代码：

```bash
mkdir test
```

假设我们有这样一个页面 `pages/index.vue`：

```html
<template>
  <h1 class="red">Hello {{ name }}!</h1>
</template>

<script>
export default {
  data () {
    return { name: 'world' }
  }
}
</script>

<style>
.red {
  color: red;
}
</style>
```

当我们利用 `npm run dev` 启动开发服务器的时候，用浏览器打开 http://localhost:3000，我们能看到红色的 `Hello world` 标题。

添加一个单元测试文件 `test/index.test.js`：

```js
import test from 'ava'
import { Nuxt, Builder } from 'nuxt'
import { resolve } from 'path'

// 我们保留一个对 Nuxt 的引用
// 这样我们可以在单元测试结束之后关掉服务进程
let nuxt = null

// 初始化 Nuxt.js 并创建一个监听 localhost:4000 的服务器
test.before('Init Nuxt.js', async t => {
  const rootDir = resolve(__dirname, '..')
  let config = {}
  try { config = require(resolve(rootDir, 'nuxt.config.js')) } catch (e) {}
  config.rootDir = rootDir // 项目目录
  config.dev = false // 生产构建模式
  config.mode = 'universal' // 同构应用
  nuxt = new Nuxt(config)
  await new Builder(nuxt).build()
  nuxt.listen(4000, 'localhost')
})

// 测试生成的 html 文件
test('路由 / 有效且能渲染 HTML', async t => {
  let context = {}
  const { html } = await nuxt.renderRoute('/', context)
  t.true(html.includes('<h1 class="red">Hello world!</h1>'))
})

// 测试 DOM 结果
test('路由 / 有效且渲染的 HTML 有特定的 CSS 样式', async t => {
  const window = await nuxt.renderAndGetWindow('http://localhost:4000/')
  const element = window.document.querySelector('.red')
  t.not(element, null)
  t.is(element.textContent, 'Hello world!')
  t.is(element.className, 'red')
  t.is(window.getComputedStyle(element).color, 'red')
})

// 关闭 Nuxt 服务进程
test.after('关闭服务器', t => {
  nuxt.close()
})
```

运行上面的单元测试：

```bash
npm test
```

实际上 `jsdom` 有一定的限制，因为它背后并没有使用任何的浏览器引擎，但是也能涵盖大部分测试场景了。如果想使用真实的浏览器引擎来测试你的应用，推荐看一看 [Nightwatch.js](http://nightwatchjs.org)。

## ESLint 和 Prettier

> [ESLint](http://eslint.org) 是一个很棒的工具，帮助我们提升代码的规范和质量。

> [Prettier](https://prettier.io) 是一个非常流行的代码格式化工具。

在 Nuxt.js 中集成 ESLint 和 Prettier 是非常容易的，首先我们需要安装一系列 npm 依赖：

```bash
npm install --save-dev babel-eslint eslint eslint-config-prettier eslint-loader eslint-plugin-vue eslint-plugin-prettier prettier
```

然后, 在项目根目录下创建 `.eslintrc.js` 文件来配置 ESLint：

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true
  },
  parserOptions: {
    parser: 'babel-eslint'
  },
  extends: [
    "eslint:recommended",
    // https://github.com/vuejs/eslint-plugin-vue#priority-a-essential-error-prevention
    // 如果需要更严格的规则，可以考虑换成 `plugin:vue/strongly-recommended`。
    "plugin:vue/recommended",
    "plugin:prettier/recommended"
  ],
  // 检查 *.vue 文件的必要配置
  plugins: [
    'vue'
  ],
  // 自定义规则
  rules: {
    "semi": [2, "never"],
    "no-console": "off",
    "vue/max-attributes-per-line": "off",
    "prettier/prettier": ["error", { "semi": false }]
  }
}
```

最后，我们在 `package.json` 文件中添加 `lint` 和 `lintfix` 命令：

```js
"scripts": {
  "lint": "eslint --ext .js,.vue --ignore-path .gitignore .",
  "lintfix": "eslint --fix --ext .js,.vue --ignore-path .gitignore ."
}
```

你现在可以运行 `lint` 命令来检查错误了：

```bash
npm run lint
```

或 `lintfix` 命令来自动修复可修复的错误：

```bash
npm run lintfix
```

ESLint 会检查所有的 JavaScript 和 Vue 文件，除了你在 `.gitignore` 中忽略了的文件。

同时推荐在 Webpack 中启用 ESLint 热重载模式。这样运行 `npm run dev` 后 ESLint 会在每次保存文件时检查代码。这只需要在 `nuxt.config.js` 文件中添加如下代码：

```
...
  /*
   ** 构建配置
   */
  build: {
   /*
    ** 你可以在这里添加 Webpack 配置
    */
   extend(config, ctx) {
      // 保存文件时运行 ESLint
      if (ctx.isDev && ctx.isClient) {
        config.module.rules.push({
          enforce: "pre",
          test: /\.(js|vue)$/,
          loader: "eslint-loader",
          exclude: /(node_modules)/
        })
      }
    }
  }
```

<p class="Alert Alert--info">有个最佳实践是在 package.json 中增加 `"precommit": "npm run lint"` ，这样可以实现每次提交代码之前自动进行代码检查。</p>
