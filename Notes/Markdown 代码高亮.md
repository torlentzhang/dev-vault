---
type: concept
tags: [markdown, 代码高亮, 前端工程化]
aliases: [markdown-it-prism, Prism 高亮, 语法高亮]
created: 2026-10-23
source: https://unpkg.com/markdown-it-prism@4.0.0/README.md（发布包 README）· https://registry.npmjs.org/markdown-it-prism/latest · https://prismjs.com/
---
# Markdown 代码高亮

## 一句话定义
给解析器吐出来的代码块**加着色标记**的独立一层——它不属于 Markdown 解析器本身。在 markdown-it 这条线里，它与外界的交接点只有一个：`markdown-it` 的 `highlight` 选项；主流实现是 **markdown-it-prism**，而它只负责插标记，**样式表必须自己引**。

## 用于解决什么问题
md 里写一个 ` ```ts ` 代码块，解析器只输出 `<pre><code class="language-ts">` 加一段转义后的纯文本——没有颜色。可读性全靠这一层补上。

## 要点

### 它为什么是「另一层」
- markdown-it 的职责是 md → HTML 字符串，它不认语法（见 [[markdown-it]]）；代码块由 `highlight` 选项交给外部高亮器处理
- `highlight` 不设时，代码块就是原样转义输出，也就是**黑白一片**
- markdown-it-prism 的接法正是**接管这个钩子**：注册时把 `markdownit.options.highlight` 替换成自己的实现（据发布包 `build/index.js`）；README 的表述是「The plugin will insert the necessary markup into all code blocks」

### 在 unplugin-vue-markdown 里怎么接
官方 README 的 Options 示例本身就是 anchor + prism：
```ts
import MarkdownItAnchor from 'markdown-it-anchor'
import MarkdownItPrism from 'markdown-it-prism'

Markdown({
  markdownSetup(md) {
    md.use(MarkdownItAnchor)
    md.use(MarkdownItPrism)
  },
})
```
（旧名 `markdownItSetup` 仍可用但已 deprecated）

### 补两样东西才「看得见」
1. **样式表**：官方原话 "Include one of Prism's stylesheets in your HTML to get highlighted code"。`prismjs` 包的 `style` 字段直接指向 `themes/prism.css`；要更多主题另有 `prism-themes`（1.9.0，MIT）
2. **行内代码要高亮得显式开**：`highlightInlineCode` 默认 `false`
   ```ts
   md.use(prism, { highlightInlineCode: true })
   ```

### 常用选项（据官方 README 选项表）
| 选项 | 默认 | 说明 |
| --- | --- | --- |
| `highlightInlineCode` | `false` | 是否给行内代码着色 |
| `plugins` | `[]` | 要加载的 Prism 插件名数组 |
| `init` | `() => {}` | 设置完 prism 后调用，收到 prism 实例 |
| `defaultLanguageForUnknown` | `undefined` | 指定了 Prism 不认识的语言时退回哪个 |
| `defaultLanguageForUnspecified` | `undefined` | 没写语言时用哪个 |
| `defaultLanguage` | `undefined` | 上面两项的简写 |

### 语言怎么加载
包内 loader 遇到未知语言时会按需加载对应 grammar（据发布包 `build/index.js`）。官方只对 **Webpack** 明确了「必须手动 import 语言，且语言之间有依赖、要按正确顺序 import」。

### 限制与坑
- **依赖 DOM 的 Prism 插件不能用于本包**：官方点名 `line-numbers`（issue #1）→ 想要行号 / 复制按钮，这条线给不了
- 行内代码指定语言的写法是 `` `class Demo { };`{language=cpp} ``
- 与 `markdown-it-attrs` 搭配需 **≥ v5.0.0**，且配了 `allowedAttributes` 时必须包含 `language`，否则行内代码指定不了语言

### 什么时候跑：构建期
这条线里 `.md` 是在**构建期**被编译成 SFC 的（见 [[unplugin-vue-markdown]]），着色在构建期就写进产物，运行时不跑高亮——所以运行时没有高亮开销。

### 版本与口径（2026-10）
- markdown-it-prism **4.0.0**，MIT，`engines.node >= 20`
- peer 声明 `markdown-it >=7.0.1 <15.0.0`，可选 peer `markdown-it-attrs >=5.0.0`
- ⚠️ **存疑**：这条 peer 上限 `<15` 是声明层面的事实，但本线宿主已经是 `markdown-exit`，而 markdown-it 当前稳定版是 **15.0.3**；官方 README 未提这处口径不一致，是否真会冲突未验证
- Prism 本体为 `prismjs@1.30.0`；官方 README 写明正在做 **Prism v2**，v1 目前**只接受安全相关 PR**
- 官方 README **没有**给出 Shiki 之类的替代建议

### 替代：Shiki 那条线
- VitePress 用的是 Shiki（见 [[markdown-it]]）
- `@shikijs/markdown-it@4.5.0` 是 shiki 官方的 markdown-it 集成，`engines.node >= 20`、依赖 `markdown-it ^15`，带 `./async` 入口与可选 peer `markdown-it-async`
- 定位差别：Prism 走「打标记 + CSS 主题」，Shiki 走 TextMate 语法 + VS Code 主题；两者都要自己解决主题

## 我的理解 / 类比
解析器像把 md 原稿拆成一堆积木，高亮层是给积木刷漆——Prism 只送刷子，颜料（样式表）得自己买。而 `line-numbers` 那种「必须摸到真实 DOM 才肯干活」的插件在这条线里没得摸：md 在构建期就已经被编译成 SFC 了，浏览器里根本没有「渲染后再加工」这一步，所以官方干脆说不能用。

## 关联
- [[markdown-it]] —— 宿主与插件机制：代码块经 `highlight` 钩子交出去，本卡讲的就是这个钩子接谁
- [[unplugin-vue-markdown]] —— 接入点：通过 `markdownSetup(md)` 注册本插件，官方 Options 示例即 anchor + prism
- [[编译时与运行时]] —— 着色发生在构建期并写进产物，是「能编译期做就别留运行时」的又一实例