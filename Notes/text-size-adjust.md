---
type: concept
tags: [CSS, mobile, 字号]
created: 2026-10-14
aliases: [text-size-adjust, "-webkit-text-size-adjust", font boosting, 文字膨胀, text autosizing, 字号自动放大]
source: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/text-size-adjust · https://drafts.csswg.org/css-size-adjust-1/ · https://caniuse.com/text-size-adjust
---
# text-size-adjust

## 一句话定义
`text-size-adjust` 控制移动端浏览器对文字的**自动放大**行为（font boosting / text autosizing），用来关掉或调低这种「贴心但破坏设计」的缩放。

## 要点

### 成因（为什么会自动放大）
老式移动浏览器按 **800–1000px** 的宽度布局，再整体缩放到屏幕宽度上显示。布局是桌面级的，但文字被缩得极小；于是浏览器对**整宽文本块**自动放大 `font-size` —— **只放大文字，不改布局**。

- 这正是「**手机上某些段落里的字忽然比别处大**」的来源：被放大的通常是撑满整宽的文本块，而不是全部文字。
- 它是**浏览器单方面决定的启发式算法**，与设计稿无关，所以同一份样式在不同设备上可能字号不一致。

### 规范语义
规范（css-size-adjust-1）原文：

> **`text-size-adjust: 100%` 等价于 `none`**，即不做调整。

- 百分比值表示 `font-size` **乘以**该比例（负数无效）。
- 初始值 `auto`，**可继承** —— 因此通常写在根元素上，一次生效全局。

```css
html {
  -webkit-text-size-adjust: 100%; /* Safari / iOS Safari */
  text-size-adjust: 100%;         /* Chrome/Edge 54+；Firefox 忽略 */
}
```

### 只认前缀 + 兼容性
| 浏览器 | 情况 |
| --- | --- |
| Safari / iOS Safari | **至今只支持** `-webkit-text-size-adjust` |
| Chrome / Edge | **54 起**支持无前缀 |
| Firefox | **至今不支持** |
| 整体 | **不是 Baseline**（Experimental）特性；MDN 原话：其他浏览器会**忽略**这个属性 |

结论：**必须带 `-webkit-` 前缀**，无前缀那行只服务 Chromium 系；Firefox 上写了也不生效。

### 什么情况下 boosting 本来就不会发生（规范列出的抑制条件）
1. **文本过少** —— 不足以换行到一两行以上；
2. 祖先为 **`height` 非 `auto`** 的块级 / inline-block；
3. 祖先为 **`width` 非 `auto`** 的 inline-block；
4. `white-space: pre` / `nowrap`。

把这份清单和上面的成因放在一起看：「**整宽 + 多行**」的文本块才是 boosting 的高发区，一旦被约束了宽度、高度或换行方式，浏览器就不再插手。

### ❓ 存疑项（未核实，不作结论）
> **待核实：页面已经设了 `width=device-width` 时，Mobile Safari 是否仍会做 font boosting？**
>
> 本次**未找到一手来源**明确表述该条件下的行为，故**不写成结论**，保留为存疑项。
>
> 相邻的**已核实**事实（供将来验证时参考）：MDN 现行推荐写法是 `<meta name="viewport" content="width=device-width">`，并说明 `initial-scale=1` 通常不必写。
>
> 视口声明与 boosting 的关系正落在这条存疑项上 —— 在拿到一手依据之前，不要把「设了 `device-width` 就一定不 boosting」当作结论。

## 我的理解 / 类比
这是浏览器自作主张的「**字号自动补偿**」：它发现你按桌面宽度布局、文字被缩得太小，就偷偷把整宽段落的字号调大 —— 像手机在嘈杂环境里自动把音量调大。
`text-size-adjust` 就是那句「**别多管闲事**」，把字号的控制权交回给作者。

## 关联
- [[移动端适配]] —— 本卡是总览里「字号专门处理」那条支线的具体机制
- [[vw 移动端适配]] —— 字号与缩放体系的另一处冲突：那边是 WCAG F94（视口单位让文字不可缩放），这边是浏览器自动放大
- [[meta viewport]] —— 视口声明与 boosting 的关系，正是上面那条存疑项所在
