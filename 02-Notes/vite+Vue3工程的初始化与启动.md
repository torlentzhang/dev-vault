---
type: howto
tags: [vue, vite, pnpm, 前端工程化]
created: 2026-08-14
---
# 用 Vite 脚手架初始化 Vue3 工程

## 适用场景
从零起一个 Vue3 + Vite 前端工程（TS 模板）。

## 步骤
1. `npm init vite@latest` 按提示选 vue + TS 模板，生成最新工程
2. `cd <项目目录>` 装依赖：`pnpm install`
3. `pnpm dev` 启动开发服务器

## 关于包管理器（可沉淀的要点）
- Vite 脚手架本身不绑定包管理器，npm / pnpm / yarn 都能用
- 用 pnpm 时推荐通过 **corepack** 启用（见 [[corepack是什么]]），而非全局安装——可避免全局 pnpm 与 node 版本冲突的老问题（见 [[nvm切换node导致pnpm不兼容]]）

## 关于样式初始化（可沉淀的要点）
- 前端样式初始化常见两种方案：`normalize.css` 与 `reset.css`
- 区别：normalize 保留浏览器中一些有用的默认样式并做跨浏览器统一；reset 则把默认样式完全清零。选型见 [[normalize与reset的区别]]

## 关联
- [[pnpm为什么拦截自执行脚本]]
- [[corepack是什么]]
- [[nvm切换node导致pnpm不兼容]]
