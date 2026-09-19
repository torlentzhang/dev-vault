---
type: concept
tags: [node, npm, 包管理器, nvm, 全局安装]
created: 2026-09-14
aliases: [npm prefix, npm root -g, 全局包路径, npm全局安装]
source:
  - https://docs.npmjs.com/cli/commands/npm-prefix
  - https://github.com/nvm-sh/nvm
---
# npm 全局安装的路径规则

## 一句话定义
用 nvm 时，npm 全局包按 node 版本**隔离存放**，装在 `~/.nvm/versions/node/<版本>/lib/node_modules`；切版本不共享、需重装。

## 要点
- `npm prefix -g` 显示当前 node 版本的全局根目录；`npm root -g` 显示全局包目录（即 prefix 下的 `lib/node_modules`）。随时跑这两个就能确认"现在装到哪了"。
- nvm 的原理是把 `~/.nvm/versions/node/<版本>/bin` **动态插到 PATH 最前面**来实现版本切换，连 `npm` 本身也随版本走——"谁家的 npm，就把包装进谁家"。
- 每个 node 版本有独立全局包目录，彼此不可见。这本身是优点（旧项目用的旧 CLI 和新项目的新 CLI 互不干扰），但切版本后"包不见了"是正常表现，不是丢了。
- 全局安装**永远不要 sudo**：sudo 会绕开版本隔离、写进系统目录（`/usr/local/lib/node_modules` 或 Homebrew 的 `/opt/homebrew/lib/node_modules`），后续报错极难查。
- 不同安装方式的全局包落点对比：

  | 安装方式 | 全局包位置 | 需 sudo？ |
  | --- | --- | --- |
  | nvm 的 node | `~/.nvm/versions/node/<版本>/lib/node_modules` | 否 |
  | Homebrew 的 node | `/opt/homebrew/lib/node_modules` | 通常否 |
  | 官网 pkg 的 node | `/usr/local/lib/node_modules` | 是 |

## 我的理解 / 类比
把每个 node 版本想成一台独立电脑，`npm -g` 是把软件装进这台电脑的"程序目录"；`nvm use` 切版本等于换了一台电脑，旧电脑装的软件新电脑上自然没有。nvm 只是帮你把"当前这台电脑"的开关拨到对应版本。

## 实战建议
- 偶尔用一次的 CLI（如 `npm-check`）别全局装，用 `npx npm-check -u` 跑完即走，不占任何目录。
- 真要全局装：先 `nvm use <版本>` 切到目标版本，再 `npm i -g`，避免装错版本。
- 写 `~/.nvm/default-packages` 文件，每行一个包名；之后每次 `nvm install <新版>` 会自动把这些包装上，免去逐个手动重装。
- 迁移旧版本全局包：`nvm install 22 --reinstall-packages-from=18`，一次性把 v18 的全局包搬进 v22。

## 关联
- [[nvm切换node导致pnpm不兼容]] —— 那篇记的"切 node 后 pnpm 罢工"现象，根因就是本卡片讲的版本隔离机制
- [[npx]] —— 互补：不想污染全局目录，就用 npx 临时跑，跑完就走
- [[Corepack]] —— 互补：corepack 与 npm 一样随 node 版本驻留在全局 bin 目录，路径规则同源
