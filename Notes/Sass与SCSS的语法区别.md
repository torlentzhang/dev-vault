---
type: concept
tags: [CSS, Sass, SCSS, 语法]
created: 2026-09-14
aliases: [Sass语法, SCSS语法, .sass与.scss]
source: Sass 语法文档 https://sass-lang.com/documentation/syntax
---

# Sass 与 SCSS 的语法区别

## 一句话定义
Sass 有两种语法：**缩进式 Sass**（`.sass`）靠缩进表达层级、无花括号无分号；**SCSS**（`.scss`）是 CSS 的超集，用花括号和分号、且完全兼容原生 CSS。

## 要点
- **扩展名不同**：`.sass` = 缩进式旧语法；`.scss` = 类 CSS 新语法
- **符号风格**：
  - Sass：靠缩进，属性结尾无分号，层级靠换行而非符号
  - SCSS：和 CSS 一样用 `{}` 包裹、`;` 结尾
- **与 CSS 的关系**：SCSS 是 CSS 的超集——**任何合法 CSS 文件都是合法 SCSS 文件**，迁移成本低；Sass 旧语法不兼容 CSS，纯靠缩进
- **现状**：**SCSS 是绝对主流**，Dart Sass 官方推荐；`.sass` 缩进式已很少用，但编译器仍支持

示例对比：
```scss
// SCSS
.nav {
  color: red;
  .link { font-weight: bold; }
}
```
```sass
// Sass（缩进式）
.nav
  color: red
  .link
    font-weight: bold
```

## 我的理解 / 类比
SCSS 像是"会写 CSS 的人直接上手"的方言，把 CSS 包了一层糖；Sass 缩进式更极简，但和团队既有的 CSS 习惯割裂，所以渐渐被 SCSS 取代。

## 关联
- [[为什么需要CSS预编译语言]] —— 预编译语言存在的意义
- [[Sass的use与forward]] —— 语法之外，现代 Sass 的模块化写法
