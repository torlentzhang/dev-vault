---
type: concept
tags: [HTML, 语义化, 表格]
created: 2026-11-03
aliases: [table, 表格标签, HTML 表格, tr, td, th, thead, tbody]
source: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/table · https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tr · https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/th · https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/caption · https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/border-collapse · https://html.spec.whatwg.org/multipage/tables.html
---
# HTML 表格

## 一句话定义
`<table>` 表达**多维数据**：由行、列组成的单元格网格。里面每一层标签都不是在「画格子」，而是在回答同一个问题——**这一格的数据，属于哪个表头、属于哪一组**。

## 用于解决什么问题
把「姓名 / 方向」这类成对数据写成一段文字，读者要靠猜；写成两列网格，行列关系一眼可见。原生 table 的价值在**语义**：标签把「谁是表头、谁属于哪行哪列」写进结构，屏幕阅读器据此播报，浏览器据此画出网格。

## 要点

### 最简二维表：2 列 × 3 行
```html
<table>
  <tr>
    <th>姓名</th>
    <th>方向</th>
  </tr>
  <tr>
    <td>小明</td>
    <td>前端</td>
  </tr>
  <tr>
    <td>小红</td>
    <td>后端</td>
  </tr>
</table>
```
- 三行两列：第一行是表头，后两行是数据。
- 一个 `<tr>` 就是一行，行里的 `<th>` / `<td>` 按顺序对应各列。
- 这段没写 `<tbody>`，但浏览器会**隐式补上**（见下方易错点一）。

### 加上语义：caption / thead / tbody / scope
```html
<table>
  <caption>小组成员</caption>
  <thead>
    <tr>
      <th scope="col">姓名</th>
      <th scope="col">方向</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>小明</td>
      <td>前端</td>
    </tr>
    <tr>
      <td>小红</td>
      <td>后端</td>
    </tr>
  </tbody>
</table>
```
- `<caption>` 是表题，给整张表一个可访问名称；**有就必须是 `<table>` 的第一个子元素**。
- `scope="col"` 声明「这个表头管它所在的那一列」；管行的写法是 `scope="row"`（另有 `rowgroup` / `colgroup`）。
- `<thead>` / `<tbody>` / `<tfoot>` 是**行组**，把行按区域切开：表头区 / 数据区 / 汇总区。

### 骨架：一张表只有这几种位置
规范给 `<table>` 的内容模型规定了顺序：

```text
table
├─ caption            表题（可访问名）
├─ colgroup / col     整列（本例不需要）
├─ thead ─ tr ─ th    表头行
├─ tbody ─ tr ─ td    数据行（可有多组 tbody）
└─ tfoot ─ tr ─ td    汇总行
```

| 标签 | 角色 | 上例是否出现 |
| --- | --- | --- |
| `<table>` | 数据容器 | ✅ |
| `<caption>` | 表格标题 / 可访问名 | ✅ |
| `<thead>` `<tbody>` `<tfoot>` | 行组：表头区 / 数据区 / 汇总区 | thead、tbody ✅，tfoot ❌ |
| `<tr>` | 一行 | ✅ |
| `<th>` | 表头单元格（`scope` 决定管行还是管列） | ✅ |
| `<td>` | 数据单元格 | ✅ |
| `<colgroup>` / `<col>` | 按列统一样式，不承载数据 | ❌ |

### 三个易错点
1. **`<tr>` 直接挂在 `<table>` 下，会被浏览器补 `<tbody>`**。`<tr>` 若不是 `<thead>` / `<tbody>` / `<tfoot>` 的子元素，浏览器会隐式插入 `<tbody>`。结果是 `table > tr` 这类选择器可能选不中——手写时显式写 `<tbody>` 更省事。
2. **把第一行加粗用 `<td>` + CSS 顶上，不等于 `<th>`**。`<th>` 的本职是「表头」，`scope` 决定它覆盖哪些单元格，屏幕阅读器读数据格时会带上对应表头；`<td>` + 粗体只剩视觉，这层语义是丢掉的。
3. **表格不能拿来做布局**。规范原文 *Tables must not be used as layout aids*，属于不符合规范：用表格排版会让读屏工具和取数工具都拿到混乱的结果。版面分栏交给 CSS grid / flexbox / 多列 / 定位（见 [[Grid 布局]]）。

另外，`border`、`cellpadding`、`cellspacing`、`align`、`bgcolor` 都是**废弃属性**，原本的活儿都归 CSS。

### 让格子连成一张表：`border-collapse`
`border-collapse` 的初始值是 `separate`——相邻单元格各画各的边，中间还留缝。所以只给 `th` / `td` 写 `border`，拿到的是「双线 + 缝」：

```css
table {
  border-collapse: collapse; /* 相邻单元格共用边框 */
}
th,
td {
  border: 1px solid #999;
  padding: 8px 10px;
}
```

- `collapse`：相邻单元格共用边框（合并边框渲染模型）；`separate`：各画各的，间距由 `border-spacing` 决定。
- `border-spacing` 在 `collapse` 下**不生效**——两套模型的边界正好落在这里。
- 该属性适用于 `table` / `inline-table`，并且**可继承**。
- 规范还点明一个历史包袱：*不画边框是表格的常见默认*——所以边框、斑马纹这类可读性处理都得自己用 CSS 补。

## 我的理解 / 类比
把 table 当成一张**成绩单**：`<table>` 是整张纸，`<caption>` 是纸上那句「XX 班期末成绩」，`<thead>` / `<tbody>` 把纸切成「表头带」和「数据带」，`<tr>` 是一横排，`<td>` / `<th>` 是每个格子。
写这些标签时并不是在费劲画格子，而是在**给每个格子贴身份标签**：这格是数据，上面那格是它所在列的表头。浏览器和读屏软件都靠这份标签理解表格——**语义对不对**是标签的事，**长得像不像表**才是 CSS 的事。

## 关联
- [[Grid 布局]] —— 版面分栏的正解，也印证「table 不承担布局职责」
- [[CSS 选择器]] —— 表格斑马纹靠 `:nth-child` / `:nth-of-type`；`||` 列组合符的例子正是 `col.selected||td`
- [[文本截断]] —— 单元格里做省略号通常要先 `table-layout: fixed`
