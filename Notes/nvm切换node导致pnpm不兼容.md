---
type: howto
tags: [node, pnpm, nvm, 踩坑]
created: 2026-08-11
---
# nvm 切换 node 版本导致 pnpm 不兼容的解决

## 适用场景
用 nvm 切换 node 版本后，发现全局 pnpm 报错 / 版本冲突 / 命令找不到，本质是全局安装的那份 pnpm 和新 node 的 ABI、路径对不上。

## 步骤
1. 卸载全局 pnpm（如 Homebrew 装的）：`brew uninstall pnpm`
2. 清理残留：`rm -rf ~/Library/pnpm`
3. 在目标 node 版本下启用 corepack：`corepack enable`
4. 在项目 `package.json` 声明版本：`"packageManager": "pnpm@10"`（node 18/20 项目）或 `"pnpm@11"`（node 22+ 项目，要求 node 22.13+）
5. 之后用 `corepack use pnpm@x` 在项目目录内绑定版本（会在当前目录写 packageManager 字段）

## 注意事项 / 踩坑
- nvm 切到旧 node 时，全局 pnpm 因 ABI / 路径冲突往往直接罢工
- `corepack use` 必须在**具体项目目录内**执行，绝不能在 home / 根目录跑，否则会把家目录当项目污染
- `corepack use` 绑定的是**目录**而非 node 版本；缓存位于 `~/.cache/node/corepack`，跨目录共享无害

## 关联
- [[Corepack]]
- [[pnpm配置该写在哪]] —— 同系列 pnpm 版本 / 配置踩坑；本文涉及 `pnpm@11` 的配置位置变更
- [[npm全局安装路径]] —— 本文"切 node 后 pnpm 罢工"的根因：全局包按 node 版本隔离存放
