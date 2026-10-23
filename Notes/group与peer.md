---
type: concept
tags: [CSS, 原子化CSS, Tailwind, UnoCSS, 变体]
created: 2026-10-15
aliases: [group, peer, group-hover, peer-hover, 状态标记类, "group/", "peer/", 群组变体]
source: Tailwind Hover, focus, and other states https://tailwindcss.com/docs/hover-focus-and-other-states；CSS Selectors 4 特异性规则（:where()） https://drafts.csswg.org/selectors-4/#specificity-rules；MDN :where() https://developer.mozilla.org/en-US/docs/Web/CSS/:where；UnoCSS preset-wind4 https://unocss.dev/presets/wind4
---
# group 与 peer

## 一句话定义
`group` / `peer` 是原子化 CSS 里的**状态标记类**：它们自身不产生任何样式，只是给元素贴一个标记，让**后代**（group）或**后面的兄弟**（peer）借用这个元素的状态（hover / focus / checked …）来写样式。

## 要点

### 两者分别管谁
| 标记类 | 写在哪 | 作用对象 | 依赖的 CSS 组合子 |
| --- | --- | --- | --- |
| `group` | 父级 | 它的**后代** | 后代组合子（空格） |
| `peer` | **前面的**兄弟 | 它**后面的**兄弟 | 后续兄弟组合子 `~` |

实测（tailwindcss 4.3.3）：只写 `class="group"` / `class="peer"` 时，生成的 `utilities` 层**空空如也**——标记类本身是「零产物」的，它的全部作用体现在**别人**的选择器里。

### group：按父元素状态给后代上样式
典型场景：鼠标移到整张卡片上，卡里的标题和图标一起变色——hover 挂在父上，样式落在子上。

```html
<a href="#" class="group block p-4 ...">
  <h3 class="text-gray-900 group-hover:text-white">New project</h3>
  <p class="text-gray-500 group-hover:text-white">Create a new project …</p>
</a>
```

Tailwind v4 实际生成：

```css
@media (hover: hover) {
  .group-hover\:text-white:is(:where(.group):hover *) {
    color: var(--color-white);
  }
}
```

拆开这个选择器：

| 片段 | 作用 |
| --- | --- |
| `.group-hover\:text-white` | 目标元素自己的类（这里是 `<h3>`） |
| `:is(…)` | 把整段条件包起来（`:is` 只是分组，等价于括号） |
| `:where(.group)` | **父级**要有 `group` 类 |
| `:hover` | 那个父级正被 hover |
| ` *` | 目标是它的**后代** |

`group` 只是**标记名**，不含固定语义——起作用的仅仅是「父级有个叫 `group` 的类」。

### peer：按前面兄弟的状态给后面兄弟上样式
典型场景：表单校验纯 CSS 搞定，邮箱非法时输入框变红、错误提示显形。

```html
<label class="block ...">
  <input type="email" class="peer ..." />
  <p class="invisible peer-invalid:visible text-red-500">Please provide a valid email address.</p>
</label>
```

实际生成：

```css
.peer-invalid\:visible:is(:where(.peer):invalid ~ *) {
  visibility: visible;
}
```

`~` 是 CSS 选择器里的**后续兄弟组合子**（subsequent-sibling combinator），它只向「后」看：这正是 peer 方向性约束的根源。

### peer 的方向性硬约束（易错）
> **`peer` 只能标在「前面的兄弟」上**，因为它依赖的 `~` 只能向后找兄弟。标记写在目标后面时不生效：

```html
<!-- ❌ 不生效：标记写在目标之后 -->
<label>
  <span class="peer-invalid:text-red-500">Email</span>
  <input type="email" class="peer" />
</label>
```

Tailwind 官方文档专门点出了这一条。

### 命名标记：`group/{name}` / `peer/{name}`
嵌套 group 或多个 peer 混在一起时，用命名标记指定「是哪一个」。

```html
<li class="group/item ...">
  <a class="group/edit invisible group-hover/item:visible" href="tel:…">
    <span class="group-hover/edit:text-gray-700">Call</span>
  </a>
</li>
```

```css
.group-hover\/edit\:translate-x-0\.5:is(:where(.group\/edit):hover *) { … }
```

名字**不需要任何配置**，直接写在标记上即可，Tailwind 自动生成对应 CSS。

### 任意值：`group-[...]`
不需要 config，直接用自己的选择器片段：

```html
<div class="group is-published">
  <div class="hidden group-[.is-published]:block">Published</div>
</div>
```

```css
.group-\[\.is-published\]\:block:is(:where(.group):is(.is-published) *) { display: block; }
```

想更精确时用 `&` 指定 `.group` 应落在最终选择器的哪个位置（如 `group-[:nth-of-type(3)_&]:block`）。

### 关键机制：`:where()` 把标记部分特异性清零
`group-hover:*` 的选择器是 `.group-hover\:…:is(:where(.group):hover *)`。**`:where()` 的参数特异性恒为 0**（CSS Selectors 4 明确规定 `:where()` 不贡献特异性）。于是：

- 父级标记 `.group` 贡献 **0**；
- 最终权重只由「目标自己的类」+「`:hover` 这个伪类」决定——**与直接写 `hover:text-white` 完全同量级**。

实测（Chrome `CSS.getMatchedStylesForNode`）：`.group-hover\:bg-blue-500:is(:where(.group):hover *)` 与 `.hover\:bg-red-500:hover` 的特异性**完全相等**。若去掉 `:where()`，`.group:hover .child` 会是「2 类 + 1 伪类」，权重高于普通工具类，让工具类之间的覆盖关系被「父级有没有 group」污染——所以 `:where()` 是刻意的设计，不是装饰。

### 由此产生的排序规则：自身状态优先于父级状态
因为特异性被拉平，**源码顺序成了唯一裁决者**。实测 tailwindcss 4.3.3 的输出顺序为：

```text
group-hover: … → group-focus: … → group-active: … → peer-hover: … → peer-focus: … → hover: … → focus: … → active: …
```

即 **`group-*` / `peer-*` 一律排在普通 `hover:` / `focus:` / `active:` 之前**。所以同一元素上同时写两者、且父级与自身**同时** hover 时：

```html
<div class="group">
  <div class="hover:bg-red-500 group-hover:bg-blue-500">…</div>
</div>
```

**结果是红色**——`hover` 规则写在后面，同为 2 单位特异性，后写者胜。这条链条要整体记住：**`:where()` 拉平权重 → 顺序成唯一裁决者 → 框架把 `group-*` 排前 → 自身状态优先于父级状态**。只记「group 编译在前面」会变成玄学。

### Tailwind v4 把 hover 类包进了 `@media (hover: hover)`
v4 里 `hover:` / `group-hover:` / `peer-hover:` 全部长这样：

```css
@media (hover: hover) {
  .hover\:bg-red-500:hover { … }
}
```

含义：**宣称「无悬停能力」的设备（多数触屏手机）上这些规则完全不生效**，不会出现点一下颜色卡住的粘滞效果。这是 v4 的默认行为；**v3 默认生成裸 `:hover`**，需要显式打开 `future.hoverOnlyWhenSupported` 才会包成 `@media (hover: hover) and (pointer: fine)`（v3.4.17 源码 `src/featureFlags.js` / `src/corePlugins.js`）。排查「手机上 hover 样式没了」时先想起这句。

### 与 UnoCSS 的差异（同一诉求，产物不同）
实测 `@unocss/preset-wind4` 的输出：

```css
.group:hover .group-hover\:bg-blue-500 { … }     /* 裸后代组合子 */
.peer:invalid ~ .peer-invalid\:visible { … }      /* 裸兄弟组合子 */
```

| | Tailwind v4 | UnoCSS（preset-wind4） |
| --- | --- | --- |
| group 选择器 | `…:is(:where(.group):hover *)` | `.group:hover .…` |
| 特异性处理 | `:where()` 把标记清零 | 不加保护，标记**计入权重** |
| hover 能力检测 | 包 `@media (hover: hover)` | 不包 |

UnoCSS 在这个家族里还多了两个方向（官方 `@unocss/rule-utils` 源码）：`parent-*`（组合子 `>`）与 `previous-*`（组合子 `+`，即**紧邻**的前一个兄弟，比 Tailwind 的 `~` 更严格）。Tailwind 把 group / peer 固定下来，UnoCSS 把它做成可组合规则。

一句话：**Tailwind 的 group / peer 是「用 `:where()` 精心控制过权重的产品」，UnoCSS 的是同一思路的机制裸版**。

## 我的理解 / 类比
- **`group`**：给一栋楼挂门牌 `group`，楼里每个房间（后代）都可以写「只要**这栋楼**亮灯，我就换成白字」——条件写在房间上，状态来自楼。
- **`peer`**：像排队，**只有前面的人**能决定后面那个人的样子（`~` 只往一个方向看）；想让后面的人影响前面的人，CSS 做不到。
- **`:where()`**：给门牌加了一层「降权罩」——挂着牌子这件事本身不增加发言权，发言权只属于「我自己的类 + 当下的状态」。

## 关联
- [[断点变体]] —— 同属「工具类前缀变体」这条机制，那张卡讲条件生效里的**视口/容器条件**，本卡讲**父级或兄弟状态条件**
- [[UnoCSS与Tailwind的定位差异]] —— 本卡是两者「类名高度兼容但产物细节各异」的又一实例（`:where()` 与 `@media (hover: hover)` 的有无）
- [[原子化CSS的样式复用]] —— 同属原子化 CSS 的书写层：那张卡解决「类名怎么复用抽象」，本卡解决「状态怎么跨元素传递」
- [[媒体查询]] —— 本卡里 `@media (hover: hover)` 的能力检测，以及「同等特异性看源码顺序」的层叠规则，都在那张卡展开
