---
type: moc
tags: [moc, css, mobile, 响应式]
created: 2026-10-14
source: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta/name/viewport · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/length#relative_length_units_based_on_viewport
---
# 移动端适配 · MOC

> 主题地图：移动端尺寸适配的入口。组织方式对应「地基 / 单位 / 策略」三层模型（模型本身与选型判断见 [[移动端适配]]），外加与尺寸无关的正交维度。

## 框架与选型
- [[移动端适配]] 三层模型、方案全景、**地基免疫性规律**与选型判断（建议从这里进）

## 地基：视口怎么声明
- [[meta viewport]] 布局视口宽度、`width=device-width`、`viewport-fit`；`vw` 的语义由它决定
- [[安全区适配]] `viewport-fit=cover` + `env(safe-area-inset-*)`，刘海屏与底部横条

## 单位：尺寸怎么写
- [[视口单位]] `vw` / `vh` / `vmin` / `vmax` 与动态视口单位 `svh` / `lvh` / `dvh`
- [[rem 适配]] 动态根字号（`1rem = 视口宽 / 10`）+ `postcss-pxtorem`
- [[vw 移动端适配]] 设计稿 px → vw 的构建期转换、插件生态现状与 WCAG 代价

## 策略：界面怎么随屏幕变
- [[媒体查询]] 原生条件规则（`@media`）：mobile-first 方向与断点选择
- [[断点变体]] 原子化 CSS 里写断点的前缀语法（Tailwind / UnoCSS 专属）
- [[容器查询]] `@container`：组件级适配，按容器而非视口判断
- [[Flex 布局]] 弹性布局机制
- [[CSS 数学函数]] `min()` / `max()` / `clamp()`：无断点的流体尺寸

## 正交维度（与尺寸缩放互不替代）
- [[1px 边框]] DPR 与 `border-width` 的设备像素取整、`hairlines`
- [[响应式图片]] `srcset` / `sizes` / `<picture>`：资源随屏幕变
- [[text-size-adjust]] 移动端浏览器的「字号自动放大」及其关闭方式
