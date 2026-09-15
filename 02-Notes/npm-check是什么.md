---
type: concept
tags: [node, npm, 包管理器, 依赖更新, 工具]
created: 2026-09-15
aliases: [npm-check, 依赖检查, 过期依赖检查]
source: https://github.com/dylang/npm-check
---
# npm-check 是什么

## 一句话定义
`npm-check` 是一个第三方命令行工具，交互式地列出项目中**过期、错误、未使用**的依赖，可勾选后直接升级；本质是对 `npm outdated` + `depcheck`（未使用检测）的封装，并加了交互界面。

## 要点
- 干三件事：① 告诉你哪些包过期（附文档链接帮你判断要不要升）；② 告诉你哪些包代码里没用到；③ 交互式勾选升级。
- 跑法（免全局装，呼应 [[npm全局安装路径]] 的坑——偶尔用一次别占全局目录）：
  ```bash
  npx npm-check          # 检查当前项目
  npx npm-check -g       # 检查全局装的包
  ```
- 常用选项：

  | 选项 | 作用 |
  | --- | --- |
  | `-u` / `--update` | 交互界面勾选后升级；**底层用 `npm install` 而非 `npm update`**（按 npm 团队建议） |
  | `-y` / `--update-all` | 非交互，全部更新不询问（适合自动化 / CI） |
  | `-g` / `--global` | 检查 / 更新全局包 |
  | `-s` / `--skip-unused` | 跳过"未使用依赖"检测（更快、避免误报） |
  | `-p` / `--production` | 只看 `dependencies`，跳过 `devDependencies` |
  | `-d` / `--dev-only` | 只看 `devDependencies` |
  | `-i` / `--ignore` | 按 glob 忽略某些依赖（如 `-i babel-*`） |
  | `-E` / `--save-exact` | 升级时写精确版本（去掉 `^` `~`） |
  | `--no-emoji` | 关掉 emoji（CI / 日志友好） |

- 与近邻工具的区别（定位与对比见 [[npm依赖更新方法]]）：
  - `npm outdated`：npm 自带，只列过期、不动手，零依赖成本
  - `npm-check-updates`（`ncu`）：把 `package.json` 范围直接拉到最新（含跨主版本），偏"暴力拉满"
  - `npm-check`：偏"交互式挑升"，还额外管未使用依赖

## 常见坑
- **"未使用依赖"检测易误报**：底层用 `depcheck` 扫 `require` / `import`，动态 require、条件加载的包会被误判为没用，所以常配 `-s` 跳过。
- **工具偏老**：README 仍停留在 npm@2 / npm@3 时代（连 `ied` 都当"新锐安装器"提），但核心功能至今可用；它升级时用你全局装的 npm 版本来执行 `install`。

## 实战建议
- 只想"看一眼哪些过期" → 先用 `npm outdated`（零成本），要挑着升再上 `npm-check -u`。
- 全局包也想交互式安全升级：`npm-check -u -g`（官方 tip，含 npm 自身）。
- 和 `ncu` 定位不同：要"一键把范围拉到含大版本的最新"用 `ncu`；要"挨个看 changelog 再勾升"用 `npm-check`。

## 关联
- [[npm依赖更新方法]] —— 它是路径二"借助 npm 工具"里的交互式检查 / 升级工具之一；本文只收具体 flags 与定位，更新路径的全景对比在那张卡
