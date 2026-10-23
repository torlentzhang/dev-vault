---
type: concept
tags: [node, npm, 包管理器, nvm, 全局安装]
created: 2026-09-14
aliases: [npm prefix, npm root -g, 全局包路径, npm全局安装, 全局安装落点]
source:
  - https://docs.npmjs.com/cli/commands/npm-prefix
  - https://raw.githubusercontent.com/npm/cli/latest/docs/lib/content/configuring-npm/folders.md
  - https://github.com/nvm-sh/nvm
---
# npm 全局安装的路径规则

## 一句话定义
`npm i -g` 装到哪，由 **prefix** 决定：prefix 默认就是 **node 自身的安装位置**，全局包落在 `{prefix}/lib/node_modules`（Windows 为 `{prefix}/node_modules`）。用 nvm 时每个 node 版本各装在 `~/.nvm/versions/node/<版本>/` 下，全局包便天然按版本隔离——**隔离是 prefix 规则的结果，不是 nvm 额外做的事**。

## 要点
- 通用规则（npm 官方）：`npm prefix -g` 打印全局 prefix；`npm root -g` 打印最终全局 `node_modules` 目录。不带 `-g` 时，prefix 指"最近的含 `package.json` 或 `node_modules` 的父目录"。
- **落点随平台不同**：Unix 是 `{prefix}/lib/node_modules`，可执行文件软链到 `{prefix}/bin`；**Windows 是 `{prefix}/node_modules`（没有 `lib` 这一层），可执行文件直接放进 `{prefix}`**。所以 `npm root -g` 的结果在 Windows 上不含 `lib`。
- 用 [[nvm]] 时：`~/.nvm/versions/node/<版本>/` 就是这个版本的 prefix（node 在 `bin/node`），全局包落在 `~/.nvm/versions/node/<版本>/lib/node_modules`，**每个版本一个 prefix、彼此不可见**。切版本后"全局命令不见了"是正常表现，不是丢了。
- 各安装方式的 prefix 与落点：

  | 安装方式 | prefix（≈ node 安装位置） | 全局包目录 |
  | --- | --- | --- |
  | nvm 的 node | `~/.nvm/versions/node/<版本>` | `<prefix>/lib/node_modules` |
  | Homebrew 的 node | `/opt/homebrew` | `/opt/homebrew/lib/node_modules` |
  | 官网 pkg 的 node | `/usr/local` | `/usr/local/lib/node_modules` |
  | Windows 官方安装包 | `%AppData%\npm` | `%AppData%\npm\node_modules` |

- 想知道"现在到底装到哪了"，随时跑 `npm prefix -g` 与 `npm root -g`，不用猜。

## 我的理解 / 类比
npm 装全局包时**不是自己挑目录**，而是先问一句"node 在哪"，然后装在 node 旁边。所以"全局包跟着 node 走"是默认行为。nvm 装了好几份 node，就等于有好几个"旁边"——隔离就是这么来的。

## 实战建议
- 偶尔用一次的 CLI（如 `npm-check`）别全局装，用 `npx npm-check -u` 跑完即走，不占任何目录。
- 真要全局装：先 `nvm use <版本>` 切到目标版本，再 `npm i -g`，避免装错版本。
- 写 `~/.nvm/default-packages` 文件，每行一个包名；之后每次 `nvm install <新版>` 会自动把这些包装上，免去逐个手动重装。
- 迁移旧版本全局包：`nvm install 22 --reinstall-packages-from=18`，一次性把 v18 的全局包搬进 v22。

## 关联
- [[nvm切换node导致pnpm不兼容]] —— 因果："切 node 后 pnpm 罢工"现象，根因就是本卡讲的落点隔离
- [[npx]] —— 互补：不想污染全局目录，就用 npx 临时跑，跑完就走
- [[nvm]]
- [[Corepack]] —— 互补：Node 25 起 corepack 不再随 node 分发，作为普通 npm 全局包安装，路径规则同源
