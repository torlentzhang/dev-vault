---
type: concept
tags: [CSS, UnoCSS, Tailwind, 原子化CSS, 技术选型]
created: 2026-09-03
---
# UnoCSS 与 Tailwind 的定位差异

## 一句话定义
Tailwind 是开箱即用的原子 CSS **框架**，UnoCSS 是可自由组合规则的原子 CSS **引擎**——前者给你一套定好的类名，后者给你一套"生成类名"的机制。

## 要点

### 当年 UnoCSS 靠"快"出圈
UnoCSS 不扫描文件、不解析 AST，纯字符串匹配按需生成 CSS，热更新耗时几乎与项目规模无关。相比之下 Tailwind 的 JIT 要扫全项目源文件，项目越大越慢。

### 但这个优势已被 Tailwind v4 抹平
Tailwind v4（2025 年初）做了三件事：
- **CSS-first 配置** —— 不再需要 `tailwind.config.js`，直接在 CSS 里配
- **Lightning CSS** 编译，取代 PostCSS
- **Oxide 引擎** —— 用 Rust 重写，性能大幅提升

性能差距基本消失。所以**如果现在还拿"UnoCSS 更快"当选型理由，理由已经过期了**。

### 实测数据（2026-09-03，npm 季度日均下载）
| 包 | 2025-Q2 | 2026-Q3 | 增幅 |
| --- | --- | --- | --- |
| tailwindcss | 297 万 | 1632 万 | ×5.5 |
| unocss | 3.2 万 | 6.4 万 | ×2.0 |
| @unocss/preset-wind4 | 0.46 万 | 4.87 万 | ×10.5 |

两点解读：
1. **两者都在涨** —— 整个原子 CSS 品类在扩张，不是零和博弈。UnoCSS 没被 Tailwind v4 打死
2. 但量级差距悬殊（虽然 tailwindcss 的数值含大量间接依赖，实际项目数差距没这么夸张）

### 定位已迁移：从"更快"到"更可控"
UnoCSS 现在的护城河是 Tailwind 做不到的四件事：

1. **preset 可组合** —— 同一个项目里可以同时用多套工具类系统，Tailwind 不行
2. **纯 CSS 图标** —— 图标走 CSS mask，零运行时 JS（数据来自 [[Iconify为什么是图标基础设施]]）
3. **attributify 模式** —— 工具类写成 HTML 属性：`<div flex items-center>`，不再堆长串 className
4. **变体分组** —— `hover:(bg-blue text-white)`，Tailwind 得把前缀写两遍

### 一个很说明问题的信号
`@unocss/preset-wind4`（对齐 Tailwind v4 语义的预设）一年涨了 10 倍。

这说明大家在用它的方式是：**兼容 Tailwind 的类名习惯，但把控制权握在自己手里**。UnoCSS 从"Tailwind 的替代品"变成了"可以兼容 Tailwind 的引擎"。

## 我的理解 / 类比
Tailwind 是**精装房**——格局定好了，拎包入住，想改承重墙很难；UnoCSS 是**毛坯房 + 全套工具**——想怎么隔都行，代价是你要自己动手。

精装房适合绝大多数人；但如果你想在同一套房里同时做出中式厨房和西式厨房（多 preset 共存），只有毛坯能做到。

## 选型建议
- 用 shadcn / ui、daisyUI、Headless UI → **Tailwind**，这些组件库是为它写的，生态绑定
- Vue / Vite 技术栈、需要多套工具类共存或自定义 DSL → **UnoCSS**
- 考虑简历通用性 / 团队协作成本 → **Tailwind**
- 已经在用 UnoCSS 且没遇到瓶颈 → 不用换，它仓库很活跃（★18947，持续发版）

## 关联
- [[Iconify为什么是图标基础设施]] —— UnoCSS 的纯 CSS 图标能力，数据源头就是 Iconify
- [[Sass与Less与Stylus的定位差异]] —— 同属 CSS 工具链选型对比卡，可对照阅读（原子 CSS 框架 vs 预编译语言）
