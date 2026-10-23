---
type: concept
tags: [node, package-manager, corepack]
created: 2026-08-11
aliases: [corepack]
source: https://github.com/nodejs/corepack
---
# corepack

## 一句话定义
corepack 是 Node.js 生态的"包管理器管理器"，让你不用全局装 pnpm / yarn，而是按项目的 `packageManager` 字段自动启用对应版本（Node 14.19–24 随 Node 分发；Node 25 起独立，需单独安装）。

## 用于解决什么问题
不锁版本时，全局 pnpm 只有一份，切项目就得反复 npm i -g pnpm@x，一个人升级还会连带改掉同机其它项目的 install 结果，lockfile 因此无谓抖动。

## 要点
- 14.19.0 ≤ Node < 25.0.0 随 Node.js 分发（实验性）；**Node 25 起不再内置**，需 `npm install -g corepack` 独立安装，用 `npm install -g corepack@latest` 升级
- Windows 踩坑：若之前用 `.msi` 装过 Node 自带的 corepack，改用 npm 安装前需先在 Node.js 安装器的 Modify → 功能选择里把 "corepack manager" 设为 "Entire feature will be unavailable" 移除，否则会有冲突
- 读取项目 `package.json` 里的 `"packageManager"` 字段，自动下载并锁定对应的 pnpm / yarn 版本
- 解决"全局 pnpm 版本和项目要求不匹配"的老大难问题
- 每个项目用自己声明的版本，互不污染

## 我的理解 / 类比
就像一个"包管理器版本调度器"：你只需要在项目里写一句"我要 pnpm@10"，corepack 自动把这个版本准备好并接管 `pnpm` 命令，再也不用全局装一个固定的 pnpm。

## 关联
- [[nvm切换node导致pnpm不兼容]] —— 全局 pnpm 装崩后的标准解法，就是改用 corepack
- [[npx]] —— 另一条"免全局安装"路线：corepack 管包管理器版本，npx 管包的执行
- [[pnpm add与pnpm install的区别]] —— 这两个命令具体跑哪个 pnpm 版本，由 corepack 按 `packageManager` 决定
- [[pnpm配置该写在哪]] —— corepack 决定用 pnpm@10 还是 @11，版本直接决定 `.npmrc` 是否还读 `save-exact`
- [[npm全局安装路径]] —— Node 25 起 corepack 作为普通 npm 全局包安装，落点同样遵循 nvm 版本隔离规则
