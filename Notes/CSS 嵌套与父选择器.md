---
type: concept
tags: [CSS, Sass, 选择器, 嵌套]
created: 2026-10-21
aliases: ["&", 父选择器, CSS嵌套, Sass嵌套]
source: Sass 嵌套 https://sass-lang.com/documentation/style-rules/#nesting；Sass 父选择器 https://sass-lang.com/documentation/style-rules/parent-selector/；MDN & nesting selector https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/Nesting_selector；MDN Using CSS nesting https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Nesting/Using；MDN Nesting and specificity https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Nesting/Nesting_and_specificity
---

# CSS 嵌套与父选择器

## 一句话定义
嵌套是「内层选择器继承外层选择器」的书写糖，默认用**后代空格**把两层拼起来；`&`（父选择器）是其中指代「外层选择器本身」的占位符，出现即被原样替换，用来把内层选择器**贴紧**外层——`&:hover` 是「我自己被 hover」，漏掉 `&` 就变成「我的后代被 hover」。

## 用于解决什么问题
没有嵌套时，要么给每个后代另起一个不冲突的类名，要么把 `.card .title` 这种全路径在十几处重复写；有了 `&` 又常漏写它，`:hover` 悄悄变成后代 hover，看着生效、位置却不对。

## 要点

### 1. 嵌套的默认行为：拼接空格

```scss
.card {
  p { color: white; }   // → .card p
  &:hover { … }         // → .card:hover
  :hover { … }          // → .card :hover（等价 .card *:hover）
}
```

| 写法 | 编译 / 解析结果 | 匹配对象 |
| --- | --- | --- |
| `p { }` | `.card p` | 卡片**内部**的 p（后代） |
| `& p { }` | `.card p` | 同上，`&` 在这里可省略 |
| `&:hover { }` | `.card:hover` | 卡片自己被 hover |
| `:hover { }` | `.card :hover` | 卡片内任意**后代**被 hover（最常见的 bug） |
| `&.is-active { }` | `.card.is-active` | 同一元素再叠加状态类 |
| `.theme-dark & { }` | `.theme-dark .card` | 反向上下文：`&` 写在后面 |
| `& + & { }` | `.card + .card` | 相邻兄弟 |

- **`&:hover` 的触发范围比字面更大**：Selectors 4 明确规定，后代匹配 `:hover` 时祖先也匹配，所以鼠标落在卡片的标题、按钮上，`.card:hover` 依然成立——这正是「移入卡片就变色」的底层原因。
- **漏写 `&` 的后果取决于 DOM**：`p` 是直接子元素时 `.card :hover p` 永远不成立（`:hover` 必须落在 `.card` 的后代上，而 `.card` 自己不算自己的后代）；多套一层内层容器时，鼠标停在那层容器上才会触发。所以准确说法是「hover 必须发生在 p 或包着 p 的内层祖先上」。
- 嵌套越深，选择器越长、产物越大，Sass 官方建议保持浅层。

### 2. `&` 是纯文本替换

Sass 官方定义：内层选择器中的父选择器会被替换成外层选择器，而**不走**普通嵌套规则。选择器列表会逐个展开：

```scss
.a, #b { &:hover { … } }   // → .a:hover, #b:hover
```

### 3. Sass 专属：后缀拼接（BEM）

```scss
.accordion {
  &__copy { … }        // → .accordion__copy
  &--open { … }        // → .accordion__copy--open（在 &__copy 内嵌套时）
}
```

前提是外层选择器以字母数字结尾（class / id / 元素选择器）。原生 CSS **做不到**，见第 5 点。

### 4. 限制：`&` 的位置规则，两套实现走相反方向

- **Sass**：`&` 只允许出现在复合选择器**开头**（即类型选择器可以出现的位置），所以 `span&` 是非法写法。写成 `&span` 虽然语法合法，但它是「给外层选择器追加后缀」，得到的是新类名 `.cardspan`，**不是**「卡片里面的 span」。
- **原生 CSS**：不能拼接字符串，类型选择器必须在前，所以要写 `element&` → `element.my-class`；写 `&element` 会让整条规则失效。

同一个「`&` 与元素选择器组合」的需求，Sass 靠 `&span` 拼类名、靠 `selector.unify()` 或直接展开选择器来合并元素选择器；原生 CSS 则必须写 `element&`。这条差异最容易在迁移时踩到。

### 5. 原生 CSS 的 `&`：语法一样，语义更窄

- **可用性**：CSS Nesting 由浏览器直接解析，MDN 标注 Baseline **widely available**（2023 年 12 月起全主流可用），不需要预处理器。
- **不能拼字符串**：`&__icon` 在原生里非法——未用组合子时，嵌套选择器会被当作类型选择器处理，而允许拼接会破坏这条规则。BEM 后缀只能留在 Sass。
- **特异性按 `:is()` 算**：`&` 的特异性取关联选择器列表中**最高的那个**。

```css
#a, b { & i { color: blue; } }   /* 特异性 1-0-1，两条都按 #a 计算 */
.foo i { color: red; }           /* 0-1-1，输给上面 */
```

而 Sass 会展开成 `#a i, b i` 两条独立规则，各自按**实际匹配的那条**计算特异性（`b i` 只有 0-0-2）。这是两者最实质的行为差异：原生 `&` 会被列表里的 id 选择器「连坐」提升权重。
- **非嵌套位置**：`&` 表示 scoping root（`:scope`）。Sass 从 Dart Sass 1.100.0 起也支持这种写法，原样输出交给浏览器处理，且此时不能加后缀。

### 6. 排查口诀

状态样式不生效，先数空格：`&:hover` 是自己，`:hover` 是后代；再查特异性：原生嵌套的 `&` 按 `:is()` 取最大值，Sass 按展开后各条选择器分别计算。

## 我的理解 / 类比

`&` 像口语里的「我」：`&:hover` = 「我被 hover」；省略后主语变成了「我里面的某个东西」。它也像模板里的占位符，编译器把外层选择器的**字符串**原样填进去——所以 `&__icon` 能拼出新类名；而浏览器不玩字符串拼接，它只处理选择器之间的关系，所以原生 CSS 拼不了 BEM。

## 关联
- [[Sass与SCSS的语法区别]] —— 嵌套是 SCSS 与 CSS 在语法层面的第一处差异
- [[为什么需要CSS预编译语言]] —— 嵌套是预编译器补上的能力之一，原生 CSS 已收回大半，唯 BEM 后缀拼接仍是 Sass 独有
- [[媒体查询]] —— 嵌套里的 `@media` 同样不参与特异性计算，同等特异性下仍看源码顺序
- [[group与peer]] —— `:where()` 把标记类特异性清零，是同一套特异性规则在原子化 CSS 里的应用
