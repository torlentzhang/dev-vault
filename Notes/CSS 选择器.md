---
type: concept
tags: [CSS, 选择器, 基础]
created: 2026-10-21
aliases: [选择器, selector, CSS选择器, 组合符, combinators, 嵌套选择器]
source: Selectors Level 4 https://drafts.csswg.org/selectors-4/；CSS Nesting Level 1 https://drafts.csswg.org/css-nesting-1/；MDN 选择器参考 https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors；MDN Pseudo-classes https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/Pseudo-classes；MDN Pseudo-elements https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/Pseudo-elements
---

# CSS 选择器

## 一句话定义
选择器是「在 DOM 树里描述要找哪些元素」的模式；按 Selectors Level 4 的定义，它由四层嵌套组成：**简单选择器 → 复合选择器 → 复杂选择器 → 选择器列表**——上层只是下层的组合，理解这四层比背选择器清单更有用。

## 要点

### 1. 四层结构（先分清层级，再记符号）

| 层级                 | 定义                        | 例子                                                | 关键约束                              |
| ------------------ | ------------------------- | ------------------------------------------------- | --------------------------------- |
| **简单选择器** simple   | 对元素的**单个条件**              | `div`、`.card`、`#app`、`[type="text"]`、`*`、`:hover` | 规范限定为类型 / 通配 / 属性 / 类 / ID / 伪类六类 |
| **复合选择器** compound | 同一元素上的**多个条件叠加**，无组合符、无空格 | `.card.is-active`、`a[href]:hover`                 | 类型或通配选择器**必须最前**，且只能有一个           |
| **复杂选择器** complex  | 用**组合符**连接的多个复合选择器        | `.card > p`、`h2 + p`                              | 组合符两侧各是一个复合选择器                    |
| **选择器列表** list     | 逗号分隔，匹配**任一**即命中          | `.a, .b, h1 + p`                                  | 一个非法成员会让**整条规则失效**                |

> 空格本身就是「后代组合符」，所以复合选择器内部**不能有空格**——`.card.is-active` 是一个元素，`.card .is-active` 是两层。

### 2. 简单选择器速查

| 类别 | 写法 | 例子 | 说明 |
| --- | --- | --- | --- |
| 类型 / type | 标签名 | `div`、`p`、`input` | 按元素名匹配，可用命名空间前缀 `ns\|div` |
| 通配 / universal | `*` | `*`、`.card *` | 任意元素；单独用特异性为 0 |
| 类 / class | `.` + 类名 | `.card`、`.is-active` | 一个元素可同时匹配多个类，可叠加成复合选择器 |
| ID | `#` + id | `#app` | 文档内应唯一；特异性高于类 |
| 属性 / attribute | `[...]` | `[type="text"]`、`[disabled]` | 见下方运算符表 |
| 伪类 / pseudo-class | `:` + 名字 | `:hover`、`:nth-child(2)` | 描述**状态或位置**，见伪类分类表 |
| 伪元素 / pseudo-element | `::` + 名字 | `::before`、`::marker` | 描述**元素的一部分或额外盒子**，规范建模为 pseudo-compound selector，**不计入**上面六类 |
| 嵌套选择器 / nesting | `&` | `&:hover`、`&.is-active` | 由 CSS Nesting Level 1 定义，代表**父规则匹配的元素**；也不是六类之一，详见第 5 节 |

### 3. 属性选择器运算符

| 运算符 | 含义 | 例子 |
| --- | --- | --- |
| `[attr]` | 存在该属性 | `a[href]` |
| `[attr="v"]` | 值**完全等于** v | `[type="text"]` |
| `[attr~="v"]` | 值是以空白分隔的词表，其中**有一个词**等于 v | `[class~="logo"]` |
| `[attr\|="v"]` | 值等于 v，或**以 `v-` 开头** | `[lang\|="zh"]` |
| `[attr^="v"]` | 值**前缀**为 v | `[href^="https://"]` |
| `[attr$="v"]` | 值**后缀**为 v | `[href$=".org"]` |
| `[attr*="v"]` | 值**包含** v | `[href*="example"]` |

- **大小写**：HTML 里属性名不区分大小写；`class` / `id` / `data-*` 的**值区分大小写**，多数 HTML 枚举型属性值不区分。可用标志位强制：`[href$=".org" i]` 忽略大小写、`[href$=".org" s]` 强制区分。
- 多个属性选择器可叠成复合选择器：`a[href^="https://"][href$=".org"]`。

### 4. 组合符

| 符号 | 名称 | 含义 | 例子 |
| --- | --- | --- | --- |
| 空格 | 后代 descendant | 任意层级后代 | `.card p` |
| `>` | 子 child | **直接**子元素 | `.card > p` |
| `+` | 相邻兄弟 next-sibling | 紧接在后的**那一个**兄弟 | `h2 + p` |
| `~` | 后续兄弟 subsequent-sibling | 之后**所有**同级兄弟 | `h2 ~ p` |
| `\|\|` | 列 column | 表格列关系，MDN 标为 **Experimental**，不要用于生产 | `col.selected\|\|td` |

- 组合符写在嵌套内层选择器的开头时，表示「相对父元素」：`.card { > p { } }` 等价于 `.card { & > p { } }`。

### 5. 嵌套选择器 `&`

| 写法 | 展开结果 | 场景 |
| --- | --- | --- |
| `&:hover` | `.card:hover` | 自己处于某状态 |
| `&.is-active` | `.card.is-active` | 同元素再叠加类 |
| `&::before` | `.card::before` | 自己的伪元素 |
| `& > p` / `& p` | `.card > p` / `.card p` | 显式写组合符（不加 `&` 时默认后代） |
| `.theme-dark &` | `.theme-dark .card` | 反向上下文：`&` 写在后面 |
| `& + &` | `.card + .card` | 相邻兄弟 |
| `&__icon` | **仅 Sass**：`.card__icon`；原生 CSS 非法 | BEM 后缀拼接 |

规则要点：
- **原生 CSS 的 `&` 等价于 `:is(父选择器)`**，特异性取父规则选择器列表中**最高**的那个；Sass 则做纯文本替换，列表逐条展开、各自算特异性。
- **原生不能拼接字符串**：未用组合符时嵌套选择器被当作类型选择器，所以 `&div` 非法，要写 `div&`；BEM 后缀拼接只有 Sass 能做。
- **`&` 不能代表伪元素**（与 `:is()` 同限制）：父选择器是 `.foo::before` 时，`&:hover` 只匹配 `.foo` 本身。
- 非嵌套语境下 `&` 等同 `:scope`。

### 6. 伪类分类（按 MDN 归类）

| 分类 | 成员（代表） | 用途 |
| --- | --- | --- |
| 元素基础 elemental | `:defined`、`:heading` | 元素身份 |
| 显示状态 display state | `:open`、`:popover-open`、`:modal`、`:fullscreen`、`:picture-in-picture` | 弹层 / 全屏等显示状态 |
| 表单输入 input | `:enabled`、`:disabled`、`:checked`、`:indeterminate`、`:placeholder-shown`、`:default`、`:autofill`、`:required`、`:optional`、`:read-only`、`:read-write`、`:valid`、`:invalid`、`:in-range`、`:out-of-range`、`:user-valid`、`:user-invalid` | 表单控件状态与校验 |
| 语言 linguistic | `:lang()`、`:dir()` | 内容语言与书写方向 |
| 位置 location | `:link`、`:visited`、`:any-link`、`:local-link`、`:target`、`:scope` | 链接与文档定位 |
| 资源状态 resource state | `:playing`、`:paused`、`:seeking`、`:buffering`、`:stalled`、`:muted`、`:volume-locked` | 媒体播放状态 |
| 时间维度 time-dimensional | `:current`、`:past`、`:future` | WebVTT 字幕等时间轴 |
| 树结构 tree-structural | `:root`、`:empty`、`:nth-child()`、`:nth-last-child()`、`:first-child`、`:last-child`、`:only-child`、`:nth-of-type()`、`:nth-last-of-type()`、`:first-of-type`、`:last-of-type`、`:only-of-type` | 在兄弟中的位置 |
| Shadow 结构 | `:host`、`:host()`、`:host-context()`、`:has-slotted` | Shadow DOM |
| 用户动作 user action | `:hover`、`:active`、`:focus`、`:focus-visible`、`:focus-within` | 交互状态 |
| 函数式 functional | `:is()`、`:not()`、`:where()`、`:has()` | 接受选择器列表作参数 |
| 自定义状态 custom state | `:state()` | 自定义元素 |
| 分页 page | `:left`、`:right`、`:first`、`:blank` | 打印分页（配 `@page`） |
| 视图过渡 view transition | `:active-view-transition`、`:active-view-transition-type()` | 页面过渡 |

> 伪类清单随规范演进，新名字（`:heading`、`:scroll-marker` 等）的浏览器支持以 MDN 兼容表为准。

### 7. 四个函数式伪类

| 伪类 | 参数 | 特异性 | 列表是否容错 |
| --- | --- | --- | --- |
| `:is()` | 选择器列表 | 取参数中**最高**者 | forgiving：跳过非法项 |
| `:where()` | 选择器列表 | **恒为 0** | forgiving |
| `:not()` | 复杂选择器列表 | 取参数中**最高**者 | 非容错 |
| `:has()` | 相对选择器列表 | 取参数中**最高**者 | 非容错；不能嵌套 `:has()` |

- `:has()` 的参数是**相对选择器**，隐式后代组合符：`a:has(> img)`、`dt:has(+ img)`。
- `:is()` / `:where()` / `:has()` 在 MDN 均标注 Baseline **widely available**。`:not()` 在 Level 3 只接受单个简单选择器，Level 4 起接受列表。

### 8. 伪元素分类

| 分类 | 成员（代表） | 用途 |
| --- | --- | --- |
| 排版 typographic | `::first-line`、`::first-letter`、`::cue` | 首行 / 首字母 / 字幕 |
| 高亮 highlight | `::selection`、`::target-text`、`::spelling-error`、`::grammar-error`、`::highlight()`、`::search-text` | 选区与文本标注 |
| 树内 tree-abiding | `::before`、`::after`、`::marker`、`::backdrop`、`::column`、`::scroll-button()`、`::scroll-marker`、`::scroll-marker-group` | 像普通盒子一样参与布局 |
| 元素支撑 element-backed | `::details-content`、`::part()`、`::slotted()` | 真实但无法直接选中的元素 |
| 表单相关 form-related | `::placeholder`、`::file-selector-button`、`::checkmark`、`::picker()` | 表单控件内部部件 |

- **伪元素必须位于选择器最末**：`p::first-line` 合法，`p::first-line > *` 非法（伪元素的「子元素」不可选）。
- 伪元素后面还能接伪类（`p:hover::first-line`）；规范把 `::before:hover` 这类建模为 **pseudo-compound selector**，`.foo::before::marker` 则是两个连续 pseudo-compound selector。
- 历史兼容：`::before`、`::after`、`::first-line`、`::first-letter` 四个仍接受**单冒号**写法，推荐双冒号。
- **伪类 vs 伪元素**：伪类描述**状态**，伪元素描述**部分或额外生成的盒子**——`::selection` 是选区，`:checked` 是状态。

### 9. 匹配从右往左

Selectors 4 的匹配算法（§17.3）规定：复杂选择器**按复合选择器从右到左**逐个匹配。最右侧的复合选择器是规则的「主体」——`.card :hover` 的主体是 `:hover` 那个元素，`.card` 只是它的祖先条件。这也是 [[CSS 嵌套与父选择器]] 里漏写 `&` 会「变成后代」的根源。

### 10. 特异性与失效

- **特异性只由选择器本身决定**，与源码顺序共同裁决胜负；继承来的声明特异性为 0，`@media` 不参与计算。详见 [[媒体查询]]。
- **选择器列表不容错**：`h1, h2:invalid-pseudo, h3 { }` 会让 h1 / h3 一起失效；只有 `:is()` / `:where()` 接收 forgiving selector list，会跳过非法成员。
- 特异性速记：ID > 类 / 属性 / 伪类 > 类型 / 伪元素；`:where()` 归零，`:is()` / `:not()` / `:has()` 取参数最大值，`&` 同 `:is()`。

## 我的理解 / 类比

选择器像 SQL 的 `WHERE`：先描述「要哪些行」，再由引擎去找。但它和 SQL 不同的是**从右往左求值**——像先锁定嫌疑人，再往回查他有没有符合要求的祖先。`&` 则是其中的代词：`&:hover` 是「我」，省掉它就是「我里面的某个东西」。

## 关联
- [[CSS 嵌套与父选择器]] —— 嵌套里的 `&` 就是在复杂选择器中替换掉最左侧那一段
- [[媒体查询]] —— `@media` 不参与特异性计算，只影响声明是否生效
- [[group与peer]] —— 原子化 CSS 用 `:where()` 把标记类特异性清零，建立在这套选择器匹配规则之上
