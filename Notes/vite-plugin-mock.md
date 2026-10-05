---
type: howto
tags: [vite, mock, 前端工程化]
source: https://github.com/vbenjs/vite-plugin-mock · https://www.npmjs.com/package/vite-plugin-mock
created: 2026-10-04
---
# vite-plugin-mock

## 适用场景
前后端分离开发时，后端接口未就绪 / 不稳定，前端需要独立联调接口逻辑；或临时演示部署时无需真后端。

## 本质
vbenjs 出品、基于 mockjs 的 Vite 插件。提供「本地 + 生产」两套 mock 接口能力，dev 与 prod 走两条不同的机制：
- dev（`vite dev`）：用 **Connect 中间件**挂在 Vite dev server 上拦截请求 → 浏览器 Network 能看到真实请求与响应，支持热更
- prod（`vite build`）：用 **mockjs** 在客户端（XHR/fetch 层）拦截 → 打包进产物、无需后端也能跑，但 Network 看不到「真实」请求

这是它和「纯 mockjs」最大的不同：开发环境是服务端拦截（请求可见），生产环境是客户端拦截（请求被截胡）。

## 步骤

### 1. 安装
```bash
npm i mockjs -S           # 运行时依赖：prod 模式要用
npm i vite-plugin-mock -D  # 开发依赖
```

### 2. 配置（vite.config.ts）
```ts
import { defineConfig, ConfigEnv } from 'vite'
import { viteMockServe } from 'vite-plugin-mock'

export default ({ command }: ConfigEnv) => defineConfig({
  plugins: [
    viteMockServe({
      mockPath: 'mock',                  // mock 文件目录
      localEnabled: command === 'serve', // 开发开
      prodEnabled: false,                // 生产关
      watchFiles: true,                  // 热更
      logger: true,                      // 控制台日志
    }),
  ],
})
```

### 3. 写 mock 文件
根目录建 `mock/`，每个 `.ts` 导出一个 `MockMethod[]`：
```ts
import { MockMethod } from 'vite-plugin-mock'
export default [{
  url: '/api/user/login',
  method: 'post',
  response: ({ body, headers }) => ({ code: 200, data: { token: 'fake' } }),
}] as MockMethod[]
```
前端 axios 照常发请求，**零侵入**业务代码。`MockMethod` 关键字段：`url` / `method` / `response({ query, body, headers })` / `timeout` / `statusCode` / `rawResponse`。

## 关键选项与默认值
| 选项 | 默认 | 说明 |
| --- | --- | --- |
| `mockPath` | `'mock'` | mock 文件目录；`watchFiles` 开时监听其变化 |
| `localEnabled` | `command === 'serve'` | 开发环境是否启用（2.3.0+ 拆分自老 `enable`） |
| `prodEnabled` | `command !== 'serve'` | 生产环境是否启用（同上） |
| `supportTs` | `true` | 是否读 `.ts` mock 文件 |
| `watchFiles` | `true` | 监听 mock 文件变化热更 |
| `logger` | `true` | 控制台打印请求日志 |
| `configPath` | `vite.mock.config.ts` | 统一 mock 入口文件 |
| `injectFile` | `src/main.ts` | 生产注入代码的宿主文件 |
| `injectCode` | `''` | 生产注入代码；为空则不注入 |

## dev vs prod 双机制（核心）
| 环节 | dev（Connect 中间件） | prod（mockjs 注入） |
| --- | --- | --- |
| 拦截位置 | Vite **dev server（Node 侧）** | 浏览器**客户端（XHR/fetch 层）** |
| 请求是否真发出 | 是，被服务器返回 | 否，浏览器内被拦下 |
| Network 面板 | 能看到真实请求+响应 | 看不到「真实」请求 |
| 热更 | 支持 | — |

## 注意事项 / 踩坑
- **版本坑（已核实）**：2.3.0+ 已把老选项 `enable` 拆成 `localEnabled` / `prodEnabled`。很多老教程仍用 `enable`，新项目直接用两个开关。
- **生产动态注入（已核实）**：`prodEnabled: true` 时靠 `injectCode` 注入到 `injectFile`（默认 `src/main.ts`）。若不开 `prodEnabled`，注入不发生，`mock.js` 不会被打进生产包——这是设计成「动态注入」而非「直接写在 main.ts」的原因（后者无论开不开都会打包进去）。
- **版本号存疑**：GitHub 主页文件树出现 "release 3.0.2" 字样，但 npm `latest` 为 **2.9.1**。`3.x` 是否为 npm 正式发行版未核实，以 npm 的 2.9.x 为准。
- npm 主流稳定版：**2.9.x**（2.x 系列）。

## 我的理解 / 类比
和 `vite-plugin-pwa` 一样，它是在 `vite.config.ts` 的 `plugins` 数组里注册一项、传配置对象的**业务能力插件**——不是 `unplugin` 那种「抽象构建工具钩子」的适配层（见 [[unplugin]]）。区别只在接管哪条链路：pwa 接管**构建期**的 SW 生成，mock 接管**开发期**的请求拦截。但二者解决的问题完全不同（一个离线、一个接口模拟），只是同名前缀、同注册位置，不应据此互链。

## 关联
- [[unplugin]] —— 对比本质：本卡是业务能力插件（dev 拦截 / prod 注入），非 unplugin 那种钩子适配层
- [[MOC-Engineering-BuildPlugins]] —— 归位于「常用插件」小节导航
