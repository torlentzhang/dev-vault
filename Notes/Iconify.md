---
type: concept
tags: [Iconify, 图标, UnoCSS, 前端资源, 前端工程化]
created: 2026-09-03
aliases: [Iconify 为什么是图标基础设施, 图标基础设施]
source: https://iconify.design/docs/ · https://github.com/iconify/icon-sets
---
# Iconify

## 一句话定义
Iconify **不是"一个图标库"**，而是图标的**数据层**——它把 200+ 个图标集用统一的 `IconifyJSON` 格式描述，让上层工具能用一套 API 访问 35 万+ 图标。

## 用于解决什么问题
图标集各有各的取用方式，同时用 Lucide 和 Carbon 就得记两套；Lucide 1.0 移除品牌图标后，想补回 logo 还得再接一家来源，项目里的图标从好几个渠道各引一份。

## 要点

### 关键区别：数据层 ≠ 图标集
| | 是什么 | 例子 |
| --- | --- | --- |
| 图标集 | 画了哪些图标 | Lucide、Tabler、Carbon、Material Symbols |
| **数据层** | 怎么统一描述和访问它们 | **Iconify** |

Lucide 决定"首页图标长什么样"，Iconify 决定"你怎么用一行代码把它拿出来"。两者不在同一层，不存在竞争关系。

### 为什么它不可替代
- **统一数据格式** —— 任何图标集只要转成 `IconifyJSON`，就能被整个生态消费
- **一套 API 覆盖 200+ 集、35 万+ 图标**，目前没有任何竞品
- **极其隐蔽**：`@iconify/utils` 周下载 1608 万，大量工具间接依赖它。很多人天天在用，却不知道自己用了 Iconify

### 生态分层
```
数据层    Iconify（200+ 图标集，统一 IconifyJSON 格式）
   ↓
消费层    lucide-react 组件包 / @unocss/preset-icons / unplugin-icons / @iconify/vue
```

同一个数据层，上面可以有完全不同的消费方式。这是它能长期存活的原因——**消费方式会变，数据层不会**。

### 消费方式之争已经结束，赢家是组件包
- **赢家**：tree-shakeable SVG 组件（如 `lucide-react`，9780 万/周，一年涨 6 倍）
- **为什么赢**：类型安全；对 AI 生成代码友好——`import { Home } from 'lucide-react'` 比猜一个 `i-lucide-home` class 名可靠得多
- **但底层仍是 Iconify 生态**：Lucide 本身就是 Iconify 收录的图标集之一。很多人以为自己在"用 Lucide 而不是 Iconify"，其实两者是上下层

### Lucide 1.0 带来的变化（2026-06）
Lucide 发布 1.0 稳定版（1600+ 图标），**移除了全部品牌图标**，理由是法律压力与设计一致性。

后果：**需要品牌 logo 时，更依赖 Iconify 聚合了**（Simple Icons 等仍在 Iconify 里）。

## 我的理解 / 类比
Iconify 是图标界的 **USB 接口**：各家图标集是不同的设备，Iconify 定义了统一的插头标准，于是任何设备插上就能被识别、按需取用。

设备会更新换代（图标集有起有落），但只要 USB 标准还在，上层工具就不需要重写。

## 什么时候该直接用 Iconify
- 需要**多图标集混用**（Lucide + Carbon + 自建集统一写法）
- 需要**品牌 logo**（Lucide 1.0 后已不再提供）
- 需要把**自定义 SVG 目录自动变成组件**
- 需要**离线 / 内网部署**、不依赖远程 API

## 关联
- [[UnoCSS与Tailwind的定位差异]] —— `@unocss/preset-icons` 是 Iconify 的一种消费方式
