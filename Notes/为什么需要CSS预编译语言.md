---
type: concept
tags: [CSS, 预编译语言, Sass, 工程化]
created: 2026-09-14
aliases: [CSS预处理器, CSS预编译]
source: Sass 官方文档 https://sass-lang.com/documentation
---

# 为什么需要 CSS 预编译语言

## 一句话定义
CSS 预编译语言（Sass / Less / Stylus 等）在**构建期**把一套扩展语法编译成普通 CSS，用来弥补原生 CSS 长期缺失的变量、嵌套、复用、模块化等工程能力。

## 要点
- **原生 CSS 的历史短板**：在原生 CSS 变量（CSS Custom Properties）、原生嵌套（2023 年起主流浏览器陆续支持）出现之前，CSS 没有变量、没有嵌套、没有函数、没有模块化，大型项目里重复代码极多、维护痛苦。
- **预编译语言补齐的核心能力**：
  - 变量（Sass `$x` / Less `@x`）：统一管理颜色、间距、断点
  - 嵌套：按 DOM 结构组织选择器，层级清晰
  - mixin / 函数：把可复用样式片段和逻辑抽象出来
  - 继承 / placeholder：共享样式而不产生冗余
  - 模块化（`@use` / `@forward`）：拆分文件、命名空间隔离
  - 运算与颜色函数：如 `color.adjust()`、lighten / darken
- **现代边界**：原生 CSS 现在已有变量、嵌套、`color-mix()`、`@scope`、`@layer` 等，预编译语言的"必要性"在被侵蚀，但在**编译期优化、成熟的 mixin 生态、跨浏览器一致性处理**上仍明显强于原生。

## 我的理解 / 类比
原生 CSS 像手写汇编——能写，但重复、易错、难维护；预编译语言像加了编译器的语言——写的是带变量和函数的"高级语法"，编译后吐出浏览器能跑的 CSS。

## 关联
- [[Sass与SCSS的语法区别]] —— 预编译语言里 Sass 的两种语法长什么样
- [[Sass的use与forward]] —— 现代 Sass 怎么组织模块化
- [[Sass变量与原生CSS变量的区别]] —— 预编译变量 vs 原生 CSS 变量的本质差异
- [[Sass与Less与Stylus的定位差异]] —— 三大预编译语言怎么选型
