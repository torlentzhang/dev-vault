---
type: concept
tags: [markdown, 解析器, 前端工程化]
aliases: [markdownit, Markdown 解析器, md 渲染器]
source: 官方文档站 https://markdown-it.github.io/markdown-it/ · 仓库 https://github.com/markdown-it/markdown-it
created: 2026-10-21
---
# markdown-it

## 一句话定义
用 JavaScript 写的 **Markdown 解析器**：把 md 原文渲染成 HTML 字符串。它的原生中间产物是 **token 流而不是 AST**，扩展能力来自三条规则链（core / block / inline）和按 token 类型查表的 renderer。

## 要点

### 先划边界：它和邻居不是一类东西
| | 中间产物 | 目标 |
| --- | --- | --- |
| markdown-it | **token 流** | 渲染成 **HTML** |
| remark / unified | **mdast 语法树** | 解析、改写、lint **Markdown 本身**，还能序列化回 md |
| Shiki / Prism | — | 只给代码块着色，不解析 Markdown |

选型口诀：**要输出网页 → markdown-it；要分析/改写 Markdown 文档 → remark。**（据两者官方 README）

### 中间产物是 token 流，不是 AST
- `md.render(src)` → HTML 字符串；`md.parse(src)` → token 列表
- token 是扁平数组，`open` / `close` 分离成两个 token；行内内容嵌在 `inline` token 的 `children` 里
- 官方原文："Instead of a traditional AST, we use more low-level data representation — tokens"，"Why not an AST? It's not needed for our tasks"
- 想要 AST 得自己来：调不带 renderer 的 parser 再转换 token 流，或自写 renderer 输出 JSON

### preset 决定默认语法，别笼统说「支持 GFM」
| preset | 表格 / 删除线 | `html` |
| --- | --- | --- |
| `default`（`new MarkdownIt()`） | **默认开**（官方标 "Embedded (enabled by default)"） | `false` |
| `commonmark`（`new MarkdownIt('commonmark')`） | **关**（规则表里没有 table / strikethrough） | `true` |

- 基准是 **CommonMark**（14.1.0 起对齐 spec 0.31.2），再加少量语法糖
- GFM 只内建表格与删除线；footnote、deflist、emoji、容器、任务列表都要插件
- 所以「表格渲染不出来」的第一诊断是 **preset 选错了**，不是插件没装

### 三个默认关着的开关
| 选项 | 默认 | 开了会怎样 |
| --- | --- | --- |
| `html` | `false` | 允许源文档里的原始 HTML 标签直通（`commonmark` preset 下为 `true`） |
| `linkify` | `false` | 裸 URL 自动变链接（基于 linkify-it） |
| `typographer` | `false` | 引号/破折号美化，`quotes` 默认 `“”‘’` |

「全开」要手写：`new MarkdownIt({ html: true, linkify: true, typographer: true })`

### 安全：它没有白名单
- `MarkdownItOptions` 里只有一个布尔 `html`，**没有 allowlist / sanitize 参数**
- 它做的另一件事是默认拦危险协议：`javascript:`、`vbscript:`、`file:`，以及除 gif/png/jpeg/webp 外的 `data:`
- 官方只给两条路径：**① 不开 `html`**（默认，"Output will be safe without sanitizer"）；**② 开 `html` + 外部 sanitizer 包**
- 对应到框架：`v-html="md.render(userInput)"` 在 `html: true` 下就是一个 XSS 口子

### 插件机制：往规则链插规则
```text
core   （整体处理：normalize、linkify …）
block  （块级语法）
inline （行内语法）
```
- 每条链是一个 `Ruler` 实例，规则有名字，可 `enable` / `disable` / 新增
- 渲染阶段按 `md.renderer.rules[tokenType]` 逐 token 查表，可直接替换
- `md.use(plugin, params)` 只是 `plugin(md, params)` 的语法糖
- 因此插件能加语法（anchor、脚注、emoji），也能改渲染（给 `<a>` 补 `target="_blank"`）

### v15 的三处变化
- 当前稳定版 **15.0.3**（2026-10-16）；v15 为 **TypeScript 重写**并**内置类型声明**，`@types/markdown-it` 不再需要
- `exports` 提供 **ESM + CJS 双入口**（`dist/markdown-it.mjs` / `dist/markdown-it.cjs.js`），新增 `markdown-it/browser` 子路径
- **旧 `markdown-it/lib/*` 深路径导入已移除**——从 v13 直跳 v15 的项目，这类 import 会直接炸
- 浏览器侧：UMD 构建挂 `window.markdownit`，也可用 CDN 的 ESM

### 谁在用它
- **VitePress**：官方文档原文 "VitePress uses Markdown-it as the parser, and Shiki to highlight"，`markdown` 配置项可传底层实例配置
- **VuePress v2**：`markdown` 配置 "accepts all options of markdown-it"，默认挂 anchor、emoji 等插件
- **VS Code**：扩展通过 `markdown.markdownItPlugins` 贡献插件，`activate` 里返回 `extendMarkdownIt(md)` 再 `md.use(...)`
- ⚠️ 别记成「Vite 用 markdown-it」：Vite 官方文档无此表述，是 VitePress 在用。

### 性能与体积
- 官方 benchmark 是 **2013 款 MacBook Pro Retina、解析 7,774 字节 README** 的样本：`current` 743 ops/s、`current-commonmark` 1,568、`marked` 1,587（97~98 runs）。单机单样本，只能当量级参考。
- npm 上 `unpackedSize` ≈ 2,064,760 字节 ≈ 2 MB，是**解包体积**（含 dist、类型、browser 构建），不是压缩后体积，别拿它当「库很大」的证据。

## 我的理解 / 类比
markdown-it 像一条**只给零件、不给图纸**的流水线：原料（md 文本）进去，出来的半成品是一排贴好标签的零件（token），最终装配成 HTML。你可以往流水线上任意工位塞自己的工具（规则、renderer），但它不会给你一张能反复改的设计图（AST）——那是 remark 的路子。所以「要不要 AST」本质上是在问：你要的是**成品页面**，还是**能反复加工的图纸**。

## 关联
- [[unplugin-vue-markdown]] —— 库内唯一直接接触点：该插件底层用的 markdown-exit 是 markdown-it 的 TS 重写版并兼容其插件，且它把 `markdownOptions` 默认设成全开，正好踩中本卡「三个默认关着的开关」
- [[PostCSS]] —— 同构对照：都是「解析 → 插件链按序变换 → 生成」，且都强调「能力全部来自插件」；区别在 PostCSS 加工的是 CSS 且交给 AST，markdown-it 产出 token 流并渲染成 HTML
- [[编译时与运行时]] —— 判据卡：构建期把 md 预渲染成静态 HTML（VitePress 的做法）vs 运行期渲染用户/远端内容（此时安全与性能都归运行时管）
