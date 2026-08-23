---
type: concept
tags: [node, package-manager, corepack]
created: 2026-08-11
aliases: [corepack]
source:
---
# corepack 是什么

## 一句话定义
corepack 是 Node.js 内置的"包管理器管理器"，让你不用全局装 pnpm / yarn，而是按项目的 `packageManager` 字段自动启用对应版本。

## 要点
- Node 16.9+ 已内置，零额外安装
- 读取项目 `package.json` 里的 `"packageManager"` 字段，自动下载并锁定对应的 pnpm / yarn 版本
- 解决"全局 pnpm 版本和项目要求不匹配"的老大难问题
- 每个项目用自己声明的版本，互不污染

## 我的理解 / 类比
就像一个"包管理器版本调度器"：你只需要在项目里写一句"我要 pnpm@10"，corepack 自动把这个版本准备好并接管 `pnpm` 命令，再也不用全局装一个固定的 pnpm。

## 关联
- [[nvm切换node导致pnpm不兼容]]
