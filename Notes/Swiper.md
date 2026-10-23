---
type: concept
tags: [Swiper, 轮播, UI组件库]
aliases: [Swiper.js, 触摸滑块库]
source: https://swiperjs.com · https://github.com/nolimits4web/swiper
created: 2026-10-10
---
# Swiper

## 一句话定义
Swiper 是当前最流行的移动端触摸滑块（轮播）库，架构特点是**核心 + 按需装配模块**：默认只带核心，Navigation / Pagination / Scrollbar 等功能都要显式引入并注册；同一内核可跑在 vanilla、React、Vue 和 Web Components 上。

## 用于解决什么问题
手写轮播要自己处理触摸拖动、惯性回弹、循环首尾和 resize 重算，还得在真机上兼容各家浏览器的手势差异——这些边界几乎没人一次写对；Swiper 把它们封装成可直接调用的能力。

## 要点

### 三层 DOM 结构（必需）
```html
<div class="swiper">            <!-- 初始化目标：容器 -->
  <div class="swiper-wrapper">  <!-- 必需中间层，整体位移靠它 -->
    <div class="swiper-slide">…</div>
    <div class="swiper-slide">…</div>
  </div>
  <!-- 可选 UI，放在 wrapper 之外 -->
  <div class="swiper-pagination"></div>
  <div class="swiper-button-prev"></div>
  <div class="swiper-button-next"></div>
  <div class="swiper-scrollbar"></div>
</div>
```
- `.swiper-wrapper` 不能省略；少了它 Swiper 无法识别滑动结构
- 容器类名旧教程写作 `.swiper-container`，现版本统一为 `.swiper`

### 核心 + 按需模块（最关键的认知 & 最常见踩坑）
`import Swiper from 'swiper'` 只带核心，Navigation / Pagination / Scrollbar 等都不在里面，要额外引入并注册：
```js
import Swiper from 'swiper';
import { Navigation, Pagination } from 'swiper/modules'; // 模块入口
import 'swiper/css';
import 'swiper/css/navigation';    // 每个模块的样式也要单独引
import 'swiper/css/pagination';

new Swiper('.swiper', {
  modules: [Navigation, Pagination], // 注册模块
  pagination: { el: '.swiper-pagination', clickable: true },
});
```
- 「配了 `pagination` 却不出分页点」的根因，几乎都是漏了 `modules: [...]` 或漏了模块 CSS
- 从 v10 起模块统一从 `swiper/modules` 导入（v10 之前从 `swiper` 导出）
- 想省事可全量：`import Swiper from 'swiper/bundle'` + `import 'swiper/css/bundle'`（体积更大）
- Pagination / Navigation / Scrollbar 只是众多模块中的三个；还有 Autoplay、Effect（fade / coverflow / cards / cube …）、Thumbs、FreeMode、Keyboard、Mousewheel、A11y、Zoom、Lazy、Parallax 等

### 一套内核，多种外壳
- vanilla：`new Swiper('.swiper', {…})`
- React：`swiper/react` → `<Swiper><SwiperSlide>`
- Vue：`swiper/vue`
- Web Components：`swiper/element` → `<swiper-container>`

内核相同，只是外壳适配不同框架；按知识本质归类，它是前端交互库而非某个框架的专属能力。

### 版本状态（2026-10）
- 最新 v14（v14.3.0）；v14.0.0（2026-06-26）是整个代码库的 TypeScript 重写，跳过了 v13
- 官方说明 v12 → v14 **无需改代码**，差异在更严格的类型和更窄的浏览器基线（Chrome / Edge 110+、Safari 16.4+、Firefox 110+）

## 我的理解 / 类比
Swiper 像一台**模块化相机**：机身（核心）只负责转场和触摸手势，分页点、左右按钮、滚动条都是另购配件——不给机身插上对应配件（`modules`）并装上配套外观（模块 CSS），接口就在但功能不亮。早期版本把这些配件全焊在机身上（bundle），现在改成按需装配，所以「明明配了参数却没反应」几乎都出在没插配件。
