---
type: concept
tags: [CSS, 前端样式, 样式重置]
created: 2026-08-14
source: https://github.com/necolas/normalize.css · https://meyerweb.com/eric/tools/css/reset/
---
# normalize.css 与 reset.css 的区别

## 一句话定义
两者都是前端样式初始化的基础方案，用来抹平浏览器默认样式差异；核心区别在"保留"还是"清零"。

## 要点
- **normalize.css**：保留浏览器中一些有用的默认样式（如标题字号、列表样式），只做跨浏览器的一致性修正，不破坏语义化的默认表现
- **reset.css**：把几乎所有元素的默认样式完全清零（margin / padding / 字号等归零），让开发者从绝对零开始
- 选型倾向：想要"统一但不破坏"用 normalize；想要"彻底自定义、不要任何默认干扰"用 reset（现代项目也常用 Tailwind 的 preflight 等替代）

## 我的理解 / 类比
normalize 像"把房间打扫干净但家具留着"，reset 像"把房间清空到毛坯"。

## 关联
- [[vite+Vue3工程的初始化与启动]] —— 前置关系：搭工程前先定样式基线（本卡讲「保留还是清零」），那张卡讲从零起 Vite + Vue3 工程的步骤。
