---
type: concept
tags: [node, npm, npx, 包管理器]
created: 2026-09-01
aliases: [npx, npm exec]
source: https://docs.npmjs.com/cli/commands/npm-exec
---
# npx

## 一句话定义
npx 是 npm 自带的包执行器，用来**执行一个 npm 包提供的命令，而不必先把这个包装进项目或全局**。

## 用于解决什么问题
只想跑一次脚手架或 CLI 时，装进项目会往 package.json 塞一条根本用不上的依赖，装全局又要处理升级与清理；npx 让这类一次性执行不必付出安装成本。

## 要点
- npx 不是独立工具 —— 它是 `npm exec` 的别名，就住在 npm 包里（`lib/node_modules/npm/bin/npx-cli.js`）
- 官方文档原话：Run a command from a local or remote npm package
- 三级查找：本地 `node_modules/.bin` → `~/.npm/_npx` 缓存 → registry 下载
- 下载的包只进缓存，**不写入项目的 package.json**
- pnpm 的对应物是 `pnpm dlx`，yarn 是 `yarn dlx`

## 常见误区
- ❌「npm 只能执行 scripts 里的命令」→ 错。npm 有三类能力：管依赖（`install` / `publish` / `view`）、跑脚本（`npm run`，读 scripts 字段）、执行包（`npm exec`）
- ❌「npx 是独立的第三方工具」→ 2017–2020 年是事实，现在已过时

## 身份演变
| 时间 | 情况 |
| --- | --- |
| npm 5.2（2017） | npx 作为**独立包**发布，随 npm 捆绑安装 |
| npm 7（2020） | npm 重写并内置 `npm exec`，npx 变成指向它的薄封装 |
| 现在 | 独立 npx 包已标记 deprecated：「This package is now part of the npm CLI」 |

副作用：npm 7 后找不到包会**交互式询问** `Need to install the following packages?`，老 npx 则直接下载。这正是现在大家习惯写 `npx -y` 的原因。

## 常用参数
- `-y` / `--yes`：跳过安装确认
- `-p <pkg>`：指定包，再执行另一个命令。如 `npx -p node@18 node app.js`
- `--no-install`：只在本地找，找不到就报错，绝不联网
- `-c '<cmd>'`：执行一串命令，类似 `sh -c`。如 `npx -c 'eslint && echo ok'`
- `--`：分隔参数，避免被 npx 自己吃掉。如 `npx -- eslint --version`

## 坑
- **typosquatting**：npx 会静默联网下载，包名打错可能装到恶意包 → 脚手架命令一律从官方文档复制，别手打
- **版本漂移**：`npx some-tool` 每次拉最新版，行为可能突变 → CI 中必须锁版本 `npx some-tool@1.2.3`
- **缓存只增不减**：`~/.npm/_npx` 长期累积，可安全删除，用时重新下载
- **它不是 shell**：`npx ls` 不会执行系统的 `ls`，而是去找叫 `ls` 的 npm 包

## 我的理解 / 类比
`npm run` 是正式员工 —— 写进 scripts，长期跟项目绑定；`npx` 是临时工 —— 需要时叫来干活，干完就走，不用给它办入职（装进项目）。

## 关联
- [[Corepack]] —— 互补：npx 只负责临时执行某个包的命令，corepack 负责让 pnpm / yarn 按项目声明的版本跑
- [[nvm切换node导致pnpm不兼容]] —— 对比：那条路靠全局安装（按 node 版本存放），npx 刻意不写全局、用完即走
- [[pnpm为什么拦截自执行脚本]] —— 对比：同是「能不能执行外部包的东西」，pnpm 拦的是依赖安装期的生命周期脚本
- [[npm全局安装路径]] —— 互补：不想占全局目录就用 npx 临时跑，装到全局的规则在这里
