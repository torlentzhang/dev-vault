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

## 踩坑 / 注意事项
- **tsconfig 的 node 版本比本机高**：脚手架生成的 tsconfig 里 `node` 写的是 24，本机是 22。直接改成 22 无影响——tsconfig 的 node 版本只是类型/目标提示，向下兼容不会报错。
- **cypress 构建报错**：pnpm 会拦截依赖的自执行（postinstall）脚本，导致 cypress 没装上原生二进制而报错。解决：用 `pnpm approve-builds` 手动批准 cypress 的 build 脚本，再 `pnpm dev` 即可。

## 关联
- [[pnpm为什么拦截自执行脚本]]
- [[corepack是什么]]
- [[nvm切换node导致pnpm不兼容]]
- [[Node全栈学习]]
