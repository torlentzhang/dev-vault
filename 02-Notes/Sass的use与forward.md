---
type: concept
tags: [CSS, Sass, 模块化, @use, @forward]
created: 2026-09-14
aliases: [@use, @forward, Sass模块系统]
source: Sass 模块系统文档 https://sass-lang.com/documentation/at-rules
---

# Sass 的 @use 与 @forward

## 一句话定义
Dart Sass 的**模块系统**：用 `@use` 引入并使用其他模块的成员，用 `@forward` 把某模块的导出"转发"出去（只透传不引入），二者共同取代已被废弃的 `@import`。

## 要点
- **为什么弃用 `@import`**：全局作用域（变量名易冲突）、同一文件被多次引入会重复编译、无命名空间、性能差。Dart Sass 已把 `@import` 标为 deprecated。
- **`@use` —— 使用**：
  - 引入模块，默认命名空间为文件名（如 `@use "math"` → 用 `math.$pi`）
  - 每个模块**只加载一次**，且成员需通过命名空间访问
  - 支持 `as` 改别名（`@use "a" as b`）、`with` 配置变量（`@use "a" with ($primary: red)`）
- **`@forward` —— 转发**：
  - 把另一个模块导出的成员"透传"出去，让当前文件成为一个聚合入口
  - **本身不把这些成员引入当前文件的作用域**（不能在本文件直接用，只能继续往外转）
- **组合用法**：典型库结构用 `@forward` 把多个子模块聚成一个 `index`，用户只需 `@use "index"`。

`@use` 与 `@forward` 的核心区别：
| | 引入到当前作用域 | 透传给下游 |
|---|---|---|
| `@use` | ✅ 能直接用 | ❌ |
| `@forward` | ❌ 不能用 | ✅ |

## 我的理解 / 类比
`@use` 像 `import`（拿来用），`@forward` 像 `re-export`（转手给别人）。建一个工具库时，子模块各自实现细节，顶层 `index.scss` 用 `@forward` 把接口汇总，调用方只 `@use "index"` 即可。

## 关联
- [[为什么需要CSS预编译语言]] —— 模块化是预编译语言的核心价值之一
- [[Sass变量与原生CSS变量的区别]] —— @use 的 `with` 能在编译期为 Sass 变量传参
