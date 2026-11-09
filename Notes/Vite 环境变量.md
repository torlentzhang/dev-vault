---
type: concept
tags: [vite, 环境变量, 构建工具, 前端工程化]
created: 2026-11-05
aliases: [.env 文件, env-and-mode, import.meta.env, Vite mode]
source: Vite 官方文档 https://vite.dev/guide/env-and-mode · https://vite.dev/config/ · https://vite.dev/config/shared-options · https://vite.dev/guide/api-javascript
---
# Vite 环境变量

## 一句话定义
Vite 在**启动时**读取 `.env` 系列文件，把带指定前缀的变量**在构建期静态替换**进源码，通过 `import.meta.env` 暴露给代码使用。

## 用于解决什么问题
不同部署环境（开发 / 测试 / 生产）需要不同的接口地址、开关、端口。若把这些写死在代码里，改一处要重新找遍全库；若在运行时读配置文件，则多一次网络请求且无法被 tree-shaking 优化。构建期替换把它们变成常量，既能按环境切换，又不产生运行时开销。

## 要点

### 本质是构建期替换，不是运行时读取
`import.meta.env.VITE_X` 在产物中会被替换成字面量，源码里读环境的表达式本身不会保留。这一步决定了后面所有设计：

- 值进了产物的变量等于公开，因此只有带 `VITE_` 前缀的才注入客户端
- 替换发生在源码层，无法推断类型，所以值永远是字符串
- 文件只在启动时读一次，改动需重启

### 四个 `.env` 文件与加载规则
```text
.env                # 所有情况加载
.env.local          # 所有情况加载，官方建议加入 .gitignore
.env.[mode]         # 仅指定 mode 加载
.env.[mode].local   # 仅指定 mode 加载，官方建议加入 .gitignore
```
`.local` 文件的定位是本地专有、可能含敏感变量；把 `*.local` 写进 `.gitignore` 由使用者自行配置，Vite 不代为忽略。

优先级：进程已有的环境变量（如 `VITE_X=1 vite build`）最高；带模式名的文件高于通用文件；通用文件的变量只要未被模式文件覆盖，仍然生效。

`.env` 与 `.env.local` 之间、`.env.[mode]` 与 `.env.[mode].local` 之间的覆盖次序，官方文档未作明文规定。

### mode 与 NODE_ENV 是两个独立维度
| 命令 | NODE_ENV | mode |
| --- | --- | --- |
| `vite dev` | `development` | `development` |
| `vite build` | `production` | `production` |
| `vite build --mode staging` | `production` | `staging` |
| `NODE_ENV=development vite build` | `development` | `production` |

`mode` 决定加载哪个 `.env.[mode]`，由 `--mode` 或 `config.mode` 控制；`NODE_ENV` 决定 `PROD` / `DEV` 的真假。`--mode` 不改变 `NODE_ENV`，两个轴可各自独立变化。

### import.meta.env 的五个内置常量
| 常量 | 类型 | 含义 |
| --- | --- | --- |
| `MODE` | string | 当前 mode |
| `BASE_URL` | string | 应用基路径，由 `base` 配置项决定 |
| `PROD` | boolean | 是否 production 的 `NODE_ENV` |
| `DEV` | boolean | 与 `PROD` 恒为相反值 |
| `SSR` | boolean | 是否服务端渲染 |

`BASE_URL` 是部署基路径，与接口地址无关，两者仅在前后端同域同前缀部署时数值相同。

### 值不做类型转换
`.env` 中的值全部作为字符串暴露，`VITE_X=false` 取到的是 `"false"`，判断布尔需显式比较（`=== 'true'`），数字需显式转换（`Number(...)`）。

### 配置文件里读 env 必须用 loadEnv
配置求值阶段能用的只有进程已有的环境变量，`.env*` 文件尚未加载。原因是要加载哪些文件取决于 `root`、`envDir` 和最终的 `mode`，而这些本身就是待解析的配置，形成顺序约束。

```ts
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  return {
    server: { port: env.APP_PORT ? Number(env.APP_PORT) : 5173 },
  }
})
```
签名是 `loadEnv(mode, envDir, prefixes = 'VITE_')`，第三参传 `''` 可加载全部前缀。`envDir` 默认 `root`，设为 `false` 关闭加载。

### 变量展开与 HTML 替换
`.env` 内可用 `$VAR` 或 `${VAR}` 引用其他变量（由 `dotenv-expand` 提供），`$` 本身需写成 `\$` 转义。

HTML 中使用 `%CONST_NAME%` 引用，例如 `%MODE%`、`%VITE_API_URL%`。未定义的名称会被原样保留、不替换，也不产生提示。

## 我的理解 / 类比
像**印刷前排版**：`.env` 是排版师傅手里的替换清单，付印（build）那一刻把所有占位符换成实际文字，印出来的书里只剩最终文字，读的人是看不到占位符的。也因此「印完之后再改清单」没有意义——只能重新排版。与此相对，运行时读环境变量像**翻书时现场填空**，灵活但每次都要动手。

## 关联
- [[编译时与运行时]] —— `import.meta.env` 的静态替换是「编译时确定」的典型例子，本卡是它在构建工具中的具体落地
- [[vite+Vue3工程的初始化与启动]] —— Vite 工程起点，`.env` 的加载发生在 `vite dev` / `vite build` 启动阶段
- [[axios 接口层封装]] —— `BASE_URL` 常被用作实例的 `baseURL`，但它是部署基路径，不是接口地址
