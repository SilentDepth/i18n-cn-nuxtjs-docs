---
title: 命令和部署
description: Nuxt.js 提供了一系列常用的命令，用于开发或发布部署。
---

> Nuxt.js 提供了一系列常用的命令，用于开发或发布部署。

## 命令列表

| 命令 | 描述 |
|---|---|
| nuxt | 在 localhost:3000 启动一个带热加载特性的开发服务器。 |
| nuxt build | 使用 Webpack 编译应用并压缩 JS 和 CSS 资源（发布用）。 |
| nuxt start | 以生产模式启动一个 Web 服务器（需要先执行 `nuxt build`）。 |
| nuxt generate | 编译应用，并依据路由配置生成对应的 HTML 文件（用于静态站点的部署）。 |

#### 参数

你可以通过在任何命令后面添加 `--help` 参数来获得详细用法说明。常见的参数有：

- **`--config-file` 或 `-c`：**指定 `nuxt.config.js` 文件的路径。
- **`--spa` 或 `-s`：**以 SPA 模式执行命令（即禁用服务端渲染）。

#### 在 package.json 中使用

你应当将这些命令添加至 `package.json`：

```json
"scripts": {
  "dev": "nuxt",
  "build": "nuxt build",
  "start": "nuxt start",
  "generate": "nuxt generate"
}
```

这样你便可以通过 `npm run <命令>` 来执行相应的命令，如：`npm run dev`。

<p class="Alert Alert--nuxt-green"><b>高级技巧：</b>要给 npm 命令传递其他参数，你需要在命令名称后面添加额外的 <code>--</code>（例如：<code>npm run dev -- --spa</code>）。</p>

## 开发环境

可通过以下命令以开发模式启动带热加载特性的 Nuxt 服务：

```bash
nuxt
// 或
npm run dev
```

## 生产部署

Nuxt.js 提供了三种模式供你部署应用：服务端渲染、SPA 和静态应用。

### 服务端渲染（通用应用）

部署 Nuxt.js 服务端渲染的应用不能直接使用 `nuxt` 命令，而需要先进行预编译构建。因此，构建和启动应用分为如下两个命令：

```bash
nuxt build
nuxt start
```

推荐的 `package.json` 配置如下：

```json
{
  "name": "my-app",
  "dependencies": {
    "nuxt": "latest"
  },
  "scripts": {
    "dev": "nuxt",
    "build": "nuxt build",
    "start": "nuxt start"
  }
}
```

注意：我们建议将 `.nuxt` 加入到 `.npmignore` 和 `.gitignore` 文件中。

### 静态应用（预渲染）

Nuxt.js 可使你能将应用部署至任何一个静态站点服务上。

可使用下面的命令生成应用的静态文件：

```bash
npm run generate
```

这个命令会创建一个 `dist` 文件夹，所有静态化后的资源文件均在其中。

如果你的项目需要用到[动态路由](/guide/routing#动态路由)，请参阅 [generate 配置](/api/configuration-generate) 了解如何让 Nuxt.js 生成此类动态路由的静态文件。

<div class="Alert">当使用 `nuxt generate` 命令构建你的 Web 应用时，传递给 [asyncData()](/guide/async-data#asyncdata-方法) 方法和 [fetch()](/guide/vuex-store#fetch-方法) 方法的[上下文对象](/api#上下文对象)不会包含 `req` 和 `res` 两个属性。</div>

### 单页面应用 (SPA)

尽管拥有预渲染页面的优势和更好的 SEO 及页面加载性能，在构建/生成应用时 `nuxt generate` 命令仍然需要服务端渲染引擎，因为页面内容是在*构建时*生成的。例如，我们不能在内容依赖用户验证或实时 API 时使用它（至少在首次加载时）。

SPA 的想法很简单！当 SPA 模式被启用（通过 `mode: 'spa'` 或 `--spa` 标记）时，我们运行的 build 命令会在构建完成后自动运行 generate 过程。此时 generate 生成的内容包含通用的元信息和资源链接，但不包含页面内容。

于是，对于 SPA 部署，你需要做如下动作：

- 修改 `nuxt.config.js` 中的 `mode` 为 `spa`。
- 运行 `npm run build`。
- 将生成的 `dist/` 文件夹部署到你的静态站点托管服务上，例如 Surge、GitHub Pages 或 nginx。

另一个可能的部署方法是在 `spa` 模式下将 Nuxt 作为框架中间件使用。这有助于减少服务器负担，同时通过 Nuxt 为项目添加 SSR 能力。

<div class="Alert">关于常见主机的部署示例请见[如何部署到 Heroku](/faq/heroku-deployment)。</div>

<div class="Alert">关于部署到 GitHub Pages 的更多详细信息请见[如何部署到 GitHub Pages](/faq/github-pages)。</div>