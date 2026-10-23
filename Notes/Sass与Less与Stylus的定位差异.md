---
type: concept
tags: [CSS, Sass, Less, Stylus, 技术选型]
created: 2026-09-14
aliases: [预编译语言选型, CSS预处理器对比]
source: 各项目官方文档（Sass / Less / Stylus）
---

# Sass 与 Less 与 Stylus 的定位差异

## 一句话定义
三者都是 CSS 预编译语言，但定位不同：**Sass（Dart Sass）是工业标准首选**，**Less 常见于老项目 / Bootstrap 生态**，**Stylus 语法最极简但社区偏小**。

## 用于解决什么问题
只看语法爽不爽选型，容易把新项目押在社区已放缓的 Stylus 上，招人、找插件、搜报错都先卡一层；接手 Bootstrap 3 老项目时硬推 Sass，也只是多养一条编译链。

## 要点
- **Sass（Dart Sass）**
  - 最成熟、生态最大，社区事实标准
  - SCSS 语法对 CSS 开发者友好，功能全面（mixin / 函数 / 控制指令 / `@use` 模块系统）
  - 由 Dart 实现，维护活跃；旧的 LibSass / Node Sass 已废弃
- **Less**
  - 类 CSS 语法，上手快
  - JS 实现（可在浏览器端编译），早期因 Bootstrap 采用而流行（Bootstrap 3 用 Less，v4 起转 Sass）
  - 模块系统较弱，功能整体比 Sass 少一截
- **Stylus**
  - 语法最极简——可省略花括号、分号、冒号，表达力强
  - 偏冷门，社区和维护活跃度明显低于前两者
  - 多在历史项目 / 极简偏好者中使用

定位速记：
| | 定位 | 语法风格 | 现状 |
|---|---|---|---|
| Sass | 工业标准首选 | SCSS（类 CSS） | 主流，活跃 |
| Less | 老项目 / Bootstrap 生态 | 类 CSS | 存量大，增量少 |
| Stylus | 极简偏好 | 可省略符号 | 小众，放缓 |

## 我的理解 / 类比
Sass 是"行业标准普通话"——大家都会、生态全；Less 是"方言"，简单场景够用且一堆老项目在说；Stylus 是"极简黑话"，写得爽但听得懂的人少。新项目无特殊理由，闭眼选 Sass。

## 关联
- [[为什么需要CSS预编译语言]] —— 三者共同解决的问题
- [[UnoCSS与Tailwind的定位差异]] —— 同属 CSS 工具链选型对比卡，可对照阅读（预编译语言 vs 原子 CSS 框架）
