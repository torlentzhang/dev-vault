---
type: moc
tags: [moc, web, html, 语义化]
created: 2026-10-25
---
# HTML 语义化 · MOC

> 子领域地图：用原生 HTML 表达内容与交互含义的入口（语义化基础 / 原生交互元素 / 浮层与反馈 / 可访问性）。
> 「浮层与反馈」按**语义载体**视角组织：先由行为（模态性 · 是否自动消失 · 能否交互）定出该用哪种载体，再看 `<dialog>`、`popover` 属性、`role="tooltip"`、`aria-live` 各表达什么。

## 语义化基础
（地标与标题层级、`section` / `article` / `nav` / `main` 的边界、列表与表格 …）
- [[HTML 表格]] 原生 table 的最简结构、元素分工与收边方式

## 原生交互元素
（`button` vs `a`、`label` 与表单控件、`details` / `summary`、`input` 类型 …）

## 浮层与反馈
- [[浮层的分类与命名]] 九类划分与五套体系的名字对照（先认类别，再对名字）
（tooltip / toast / dialog / drawer：模态性 · 自动消失 · 能否交互 三轴 → 对应的原生载体与 ARIA 语义）

## 可访问性
（`aria-describedby` 关联、`aria-live` 播报、焦点陷阱与 `inert`、`title` 属性在键盘与触摸下的缺口 …）
