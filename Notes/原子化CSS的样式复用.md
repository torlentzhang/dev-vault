---
type: concept
tags: [CSS, 原子化CSS, UnoCSS, Tailwind, 样式复用]
created: 2026-10-08
aliases: ['@apply', --at-apply, UnoCSS Shortcuts, 工具类复用, 原子化CSS抽象]
source: Tailwind @apply https://tailwindcss.com/docs/functions-and-directives；UnoCSS transformer-directives https://unocss.dev/transformers/directives；UnoCSS Shortcuts https://unocss.dev/config/shortcuts
---
# 原子化 CSS 的样式复用

## 一句话定义
原子化 CSS 把样式散进 className 后，需要一层「复用 / 抽象」机制来避免重复和超长类名——`@apply` / `--at-apply` 把工具类**内联进自定义 CSS**，Shortcuts 则把多个工具类**打包成一个新类名**。

## 用于解决什么问题
按钮的圆角或主色一改，散在各模板里的几十处样式得靠全局搜索逐个替换，漏掉的那处就成了线上颜色不一致的按钮。

## 要点

### 为什么需要复用
- **className 过长**：`py-2 px-4 font-semibold rounded-lg shadow-md` 这样一串四处重复，难读也难改
- **重复的组合**：同一个"按钮样式"在几十处被重写
- **第三方库覆写**：要给 `select2` 这类库的元素写样式时，仍想用设计 token 和熟悉的工具类语法（Tailwind 官方举的典型动机）
- **语义化**：模板里想要 `class="btn"` 这种能自解释的名字

### `@apply`：把工具类内联进自定义 CSS
Tailwind 内置指令，开箱即用：

```css
.btn {
  @apply px-4 py-2 rounded bg-blue-500 text-white;
}
```

产物是把这些工具类的声明**直接展开进 `.btn`**。

### `--at-apply`：UnoCSS 的等价替代
UnoCSS 通过 `@unocss/transformer-directives` 提供 `@apply`，并额外支持用 CSS 自定义属性书写：

```css
.btn {
  --at-apply: px-4 py-2 rounded bg-blue-500 text-white;
}
```

官方理由是「兼容原生 CSS」——`--at-apply` 本质是合法的自定义属性，不依赖对自定义 at-rule 的解析。默认别名 `--at-apply` / `--uno-apply` / `--uno`，可在 `transformerDirectives({ applyVariable })` 中修改或关闭。

含 `:` 的规则（如 `hover:`）需要整体加引号：

```css
.btn { --at-apply: 'hover:bg-blue-600'; }
```

### Shortcuts：把多个工具类打包成一个新类名
UnoCSS 的机制，在配置里声明，直接在 HTML 的 `class` 中使用：

```ts
// uno.config.ts
shortcuts: {
  'btn': 'py-2 px-4 font-semibold rounded-lg shadow-md',
}
```

```html
<button class="btn">提交</button>
```

动态写法用正则批量生成：

```ts
shortcuts: [
  [/^btn-(.*)$/, ([, c]) => `bg-${c}-400 text-${c}-100 py-2 px-4 rounded-lg`],
]
```

配好后即可使用 `btn-green` / `btn-red`。

### `@apply` 与 Shortcuts 的区别
| | `@apply` / `--at-apply` | Shortcuts |
| --- | --- | --- |
| 写在哪 | CSS 规则里 | uno 配置里 |
| 用在哪 | 自定义 CSS 的类名 | HTML 的 `class` 直接用 |
| 产物 | 声明内联进那条 CSS 规则 | 生成一条新的 class 规则 |
| 视角 | 工具类 → CSS | 工具类 → 新工具类 |

一句话：**`@apply` 是「工具类降维进 CSS」，Shortcuts 是「工具类换个短名」**——真正的"聚合类名"是 Shortcuts。

> Shortcut 的值里**可以直接写变体**：官方示例 `'btn-green': 'text-white bg-green-500 hover:bg-green-700'`。（unocss.dev `config/shortcuts`）

## 我的理解 / 类比
- `@apply` 像把一道菜的配料表抄进另一个食谱——配料被"内联"了
- Shortcuts 像给一道组合菜起名（"宫保鸡丁"），点单只写名字，后厨照旧按配料做

## 关联
- [[UnoCSS与Tailwind的定位差异]] —— 本卡是两者共同面对的"复用"诉求，那张卡讲的是两者的定位与选型
- [[为什么需要CSS预编译语言]] —— Sass 的 `mixin` / `%placeholder` 解决的正是同一问题，可对照理解"复用"的谱系
