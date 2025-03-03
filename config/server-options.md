# 开发服务器选项 {#server-options}

## server.host {#server-host}

- **类型：** `string | boolean`
- **默认：** `'localhost'`

指定服务器应该监听哪个 IP 地址。
如果将此设置为 `0.0.0.0` 或者 `true` 将监听所有地址，包括局域网和公网地址。

也可以通过 CLI 使用 `--host 0.0.0.0` 或 `--host` 来设置。

::: tip NOTE

在某些情况下，可能响应的是其他服务器而不是 Vite。

第一种情况是 `localhost` 被使用了。Node.js 在 v17 以下版本中默认会对 DNS 解析地址的结果进行重新排序。当访问 `localhost` 时，浏览器使用 DNS 来解析地址，这个地址可能与 Vite 正在监听的地址不同。当地址不一致时，Vite 会打印出来。

你可以设置 [`dns.setDefaultResultOrder('verbatim')`](https://nodejs.org/api/dns.html#dns_dns_setdefaultresultorder_order) 来禁用这个重新排序的行为。Vite 将会将改地址打印为 `localhost`。

```js
// vite.config.js
import { defineConfig } from 'vite'
import dns from 'dns'

dns.setDefaultResultOrder('verbatim')

export default defineConfig({
  // omit
})
```

第二种情况是使用了通配主机地址（例如 `0.0.0.0`）。这是因为侦听非通配符主机的服务器优先于侦听通配符主机的服务器。

:::

## server.port {#server-port}

- **类型：** `number`
- **默认值：** `5173`

指定开发服务器端口。注意：如果端口已经被使用，Vite 会自动尝试下一个可用的端口，所以这可能不是开发服务器最终监听的实际端口。

## server.strictPort {#server-strictport}

- **类型：** `boolean`

设为 `true` 时若端口已被占用则会直接退出，而不是尝试下一个可用端口。

## server.https {#server-https}

- **类型：** `boolean | https.ServerOptions`

启用 TLS + HTTP/2。注意：当 [`server.proxy` 选项](#server-proxy) 也被使用时，将会仅使用 TLS。

这个值也可以是一个传递给 `https.createServer()` 的 [选项对象](https://nodejs.org/api/https.html#https_https_createserver_options_requestlistener)。

需要一个合法可用的证书。对基本使用的配置需求来说，你可以添加 [@vitejs/plugin-basic-ssl](https://github.com/vitejs/vite-plugin-basic-ssl) 到项目插件中，它会自动创建和缓存一个自签名的证书。但我们推荐你创建和使用你自己的证书。

## server.open {#server-open}

- **类型：** `boolean | string`

在开发服务器启动时自动在浏览器中打开应用程序。当此值为字符串时，会被用作 URL 的路径名。若你想指定喜欢的浏览器打开服务器，你可以设置环境变量 `process.env.BROWSER`（例如：`firefox`）。查看 [这个 `open` 包](https://github.com/sindresorhus/open#app) 获取更多细节。

**示例：**

```js
export default defineConfig({
  server: {
    open: '/docs/index.html'
  }
})
```

## server.proxy {#server-proxy}

- **类型：** `Record<string, string | ProxyOptions>`

为开发服务器配置自定义代理规则。期望接收一个 `{ key: options }` 对象。如果 key 值以 `^` 开头，将会被解释为 `RegExp`。`configure` 可用于访问 proxy 实例。

使用 [`http-proxy`](https://github.com/http-party/node-http-proxy)。完整选项详见 [此处](https://github.com/http-party/node-http-proxy#options).

在某些情况下，你可能也想要配置底层的开发服务器。（例如添加自定义的中间件到内部的 [connect](https://github.com/senchalabs/connect) 应用中）为了实现这一点，你需要编写你自己的 [插件](/guide/using-plugins.html) 并使用 [configureServer](/guide/api-plugin.html#configureserver) 函数。

**示例：**

```js
export default defineConfig({
  server: {
    proxy: {
      // 字符串简写写法
      '/foo': 'http://localhost:4567',
      // 选项写法
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      },
      // 正则表达式写法
      '^/fallback/.*': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, '')
      },
      // 使用 proxy 实例
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        configure: (proxy, options) => {
          // proxy 是 'http-proxy' 的实例
        }
      },
      // Proxying websockets or socket.io
      '/socket.io': {
        target: 'ws://localhost:3000',
        ws: true
      }
    }
  }
})
```

## server.cors {#server-cors}

- **类型：** `boolean | CorsOptions`

为开发服务器配置 CORS。默认启用并允许任何源，传递一个 [选项对象](https://github.com/expressjs/cors) 来调整行为或设为 `false` 表示禁用。

## server.headers {#server-headers}

- **类型：** `OutgoingHttpHeaders`

指定服务器响应的 header。

## server.hmr {#server-hmr}

- **类型：** `boolean | { protocol?: string, host?: string, port?: number, path?: string, timeout?: number, overlay?: boolean, clientPort?: number, server?: Server }`

禁用或配置 HMR 连接（用于 HMR websocket 必须使用不同的 http 服务器地址的情况）。

设置 `server.hmr.overlay` 为 `false` 可以禁用开发服务器错误的屏蔽。

`clientPort` 是一个高级选项，只在客户端的情况下覆盖端口，这允许你为 websocket 提供不同的端口，而并非在客户端代码中查找。如果需要在 dev-server 情况下使用 SSL 代理，这非常有用。

当 `server.hmr.server` 被定义后，Vite 将会通过所提供的的服务器来处理 HMR 连接。如果不是在中间件模式下，Vite 将尝试通过已有服务器处理 HMR 连接。这在使用自签证书或想通过网络在某端口暴露 Vite 的情况下，非常有用。

::: tip NOTE

在默认配置下, 在 Vite 之前的反向代理应该支持代理 WebSocket。如果 Vite HMR 客户端连接 WebSocket 失败，该客户端将兜底为绕过反向代理、直接连接 WebSocket 到 Vite HMR 服务器：

```
Direct websocket connection fallback. Check out https://vitejs.dev/config/server-options.html#server-hmr to remove the previous connection error.
```

当该兜底策略偶然地可以被忽略时，这条报错将会出现在浏览器中。若要通过直接绕过反向代理来避免此错误，你可以:

- 将反向代理配置为代理 WebSocket
- 设置 [`server.strictPort = true`](#server-strictport) 并设置 `server.hmr.clientPort` 的值与 `server.port` 相同
- 设置 `server.hmr.port` 为一个与 [`server.port`](#server-port) 不同的值

:::

## server.watch {#server-watch}

- **类型：** `object`

传递给 [chokidar](https://github.com/paulmillr/chokidar#api) 的文件系统监听器选项。

当需要再 Windows Subsystem for Linux (WSL) 2 上运行 Vite 时，如果项目文件夹位于 Windows 文件系统中，你需要将此选项设置为 `{ usePolling: true }`。这是由于 Windows 文件系统的 [WSL2 限制](https://github.com/microsoft/WSL/issues/4739) 造成的。

Vite 服务器默认会忽略对 `.git/` 和 `node_modules/` 目录的监听。如果你需要对 `node_modules/` 内的包进行监听，你可以为 `server.watch.ignored` 赋值一个取反的 glob 模式，例如：

```js
export default defineConfig({
  server: {
    watch: {
      ignored: ['!**/node_modules/your-package-name/**']
    }
  },
  // 被监听的包必须被排除在优化之外，
  // 以便它能出现在依赖关系图中并触发热更新。
  optimizeDeps: {
    exclude: ['your-package-name']
  }
})
```

## server.middlewareMode {#server-middlewaremode}

- **类型：** `'ssr' | 'html'`

以中间件模式创建 Vite 服务器。（不含 HTTP 服务器）

- `'ssr'` 将禁用 Vite 自身的 HTML 服务逻辑，因此你应该手动为 `index.html` 提供服务。
- `'html'` 将启用 Vite 自身的 HTML 服务逻辑。

- **相关：** [SSR - 设置开发服务器](/guide/ssr#setting-up-the-dev-server)

- **示例：**

```js
import express from 'express'
import { createServer as createViteServer } from 'vite'

async function createServer() {
  const app = express()

  // 以中间件模式创建 Vite 服务器
  const vite = await createViteServer({
    server: { middlewareMode: 'ssr' }
  })
  // 将 vite 的 connect 实例作中间件使用
  app.use(vite.middlewares)

  app.use('*', async (req, res) => {
    // 如果 `middlewareMode` 是 `'ssr'`，应在此为 `index.html` 提供服务.
    // 如果 `middlewareMode` 是 `'html'`，则此处无需手动服务 `index.html`
    // 因为 Vite 自会接管
  })
}

createServer()
```

## server.base {#server-base}

- **类型：** `string | undefined`

在 HTTP 请求中预留此文件夹，用于代理 Vite 作为子文件夹时使用。应该以 `/` 字符开始和结束。

## server.fs.strict {#server-fs-strict}

- **类型：** `boolean`
- **默认：** `true` (自 Vite 2.7 起默认启用)

限制为工作区 root 路径以外的文件的访问。

## server.fs.allow {#server-fs-allow}

- **类型：** `string[]`

限制哪些文件可以通过 `/@fs/` 路径提供服务。当 `server.fs.strict` 设置为 true 时，访问这个目录列表外的文件将会返回 403 结果。

Vite 将会搜索此根目录下潜在工作空间并作默认使用。一个有效的工作空间应符合以下几个条件，否则会默认以 [项目 root 目录](/guide/#index-html-and-project-root) 作备选方案。

- 在 `package.json` 中包含 `workspaces` 字段
- 包含以下几种文件之一
  - `lerna.json`
  - `pnpm-workspace.yaml`

接受一个路径作为自定义工作区的 root 目录。可以是绝对路径或是相对于 [项目 root 目录](/guide/#index-html-and-project-root) 的相对路径。示例如下：

```js
export default defineConfig({
  server: {
    fs: {
      // 可以为项目根目录的上一级提供服务
      allow: ['..']
    }
  }
})
```

当 `server.fs.allow` 被设置时，工作区根目录的自动检索将被禁用。当需要扩展默认的行为时，你可以使用暴露出来的工具函数 `searchForWorkspaceRoot`：

```js
import { defineConfig, searchForWorkspaceRoot } from 'vite'

export default defineConfig({
  server: {
    fs: {
      allow: [
        // 搜索工作区的根目录
        searchForWorkspaceRoot(process.cwd()),
        // 自定义规则
        '/path/to/custom/allow'
      ]
    }
  }
})
```

## server.fs.deny {#server-fs-deny}

- **类型：** `string[]`

用于限制 Vite 开发服务器提供敏感文件的黑名单。

默认为 `['.env', '.env.*', '*.{pem,crt}']`。

## server.origin {#server-origin}

- **类型：** `string`

用于定义开发调试阶段生成资产的 origin。

```js
export default defineConfig({
  server: {
    origin: 'http://127.0.0.1:8080'
  }
})
```
