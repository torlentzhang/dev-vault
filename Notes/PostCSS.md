---
type: concept
tags: [CSS, 前端工程化, PostCSS]
created: 2026-10-12
aliases: [PostCSS, autoprefixer, postcss-preset-env, CSS Modules, CSS的Babel]
source: PostCSS 官方站 https://postcss.org/
---
# PostCSS

## 一句话定义
PostCSS 是一个「用 JavaScript 转换 CSS」的工具管道：把 CSS 解析成 AST，交给插件链做变换，再生成回 CSS——它不发明书写语法，也不生产样式，只负责在构建期**加工**已有的 CSS。

## 用于解决什么问题
没有这条管道，-webkit- 前缀要在源码里一个个手写，浏览器支持表一变就得回头翻遍样式；想用 oklch() 这类新语法，要么放弃旧浏览器，要么自己再写一份降级副本。

## 要点

### 先划边界：它不是这四类东西
| | 本质 | 和 CSS 的关系 |
| --- | --- | --- |
| PostCSS | 转换管道（加工器） | CSS 进、CSS 出，中间由 JS 插件变换 |
| Sass / Less | 预编译语言 | 发明新语法，编译成 CSS |
| Tailwind / UnoCSS | 生成器 | 从类名**生成** CSS |
| Element Plus / Ant Design | UI 组件库 | 不处理 CSS，只消费/输出样式 |

### 能力全部来自插件
PostCSS 本身只提供解析与管道，典型插件（官方站示例）：
- **Autoprefixer**：根据 Can I Use 数据自动补 `-webkit-` 等浏览器前缀——这是它历史上成名的原因
- **postcss-preset-env**：把 `oklch()` 等新特性按目标浏览器降级（基于 cssdb 数据）
- **CSS Modules**：`.name` → `._Logo__name__SVK0g`，解决全局类名冲突
- **cssnano**（压缩）、**stylelint**（检查）

插件按配置顺序依次处理同一棵 AST，前一个插件的输出是后一个的输入。

### 与 Tailwind 的历史交叉（曾是一家人）
- **Tailwind v3 本身就是一个 PostCSS 插件**——很多项目 `postcss.config.js` 里同时出现 `tailwindcss` 和 `autoprefixer`，因为两者当时都是这条管道上的插件（Tailwind 生成 CSS → autoprefixer 补前缀）
- **Tailwind v4 改用 Lightning CSS**（Rust 实现），不再依赖 PostCSS
- 现状（2026）：Vite 检测到 `postcss.config.js` 会自动把 CSS 接入该管道；新项目里装 PostCSS 的常见理由只剩 Autoprefixer / preset-env，Lightning CSS 正在部分取代它的角色，但其插件生态仍是 CSS 领域最大的存量工具生态

## 我的理解 / 类比
PostCSS 是 **CSS 界的 Babel**：Babel 不定义 JS 语法，只做「解析 → 转换 → 生成」；PostCSS 同样只提供这条流水线，输入输出都是合法 CSS，最终效果完全取决于挂了什么插件。也像加工厂流水线——原料（CSS）进来，换一套流水线配置（插件组合），出来的产品就不一样。

## 关联
- [[为什么需要CSS预编译语言]] —— 同属构建期 CSS 处理，但边界分明：Sass 发明新语法再编译，PostCSS 只转换合法 CSS
- [[UnoCSS与Tailwind的定位差异]] —— Tailwind v3 曾是 PostCSS 插件，v4 用 Lightning CSS 取代 PostCSS，这段历史是两卡的接缝
