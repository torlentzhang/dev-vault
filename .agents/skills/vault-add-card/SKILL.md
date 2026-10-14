---
name: vault-add-card
description: 在 dev-vault 中新增一张原子知识卡片，并完成模板套用、MOC 导航、关联建立和索引同步。当用户要求“新建笔记”“新增卡片”“整理一个知识点到 vault”时使用。
---

# vault-add-card

在 `Notes/` 中新增一张原子知识卡，并将其接入现有知识结构。

全局规则、权限、知识校验、MOC 架构等以根目录 `AGENTS.md` 为准，本 Skill 不重复定义。

## 步骤

### 1. 确定卡片类型

根据知识性质选择 `Templates/` 中对应模板：

* 概念知识 → `concept-card.md`
* 操作 / 配方 → `howto-card.md`

如果 `Templates/` 中存在其他正式模板，则按对应模板创建。

不要自行创造未定义的卡片类型。

---

### 2. 检查已有卡片

先搜索 `Notes/`，确认：

* 是否已经存在相同主题的卡片
* 是否存在明显重复卡片
* 当前需求是否只是已有卡片的补充
* 是否应该修改已有卡片，而不是新建卡片

能够通过扩展已有卡片解决时，不创建重复知识。

---

### 3. 确定卡片名称

优先使用稳定、简洁、可检索的概念名称。

推荐：

```text
Corepack
pnpm
Event Loop
Hash 与 History
Vue Router
Sass 与 SCSS
```

当问题本身就是知识主题时，可以使用问题式标题，例如：

```text
为什么需要 Corepack
为什么 History API 可以改变 URL
```

避免大量使用：

```text
xxx是什么
xxx到底是什么
xxx怎么理解
xxx有什么用
```

优先根据知识对象本身命名，而不是直接照搬用户的提问方式。

---

### 4. 确定 Primary MOC

根据知识本质定位到现有 MOC：

```text
知识本质
  ↓
L1 Domain
  ↓
现有 L2 Subdomain
  ↓
现有 L3 Topic（如存在）
```

优先使用已有 MOC，不因为新增一张卡片就创建新的 MOC。

确定一个 `Primary MOC` 后，根据该 MOC 当前结构找到最合适的 `##` / `###` 分组。

例如：

```text
pnpm
→ MOC-Engineering-PackageManagement

Vue Router
→ MOC-Frontend-Vue

Event Loop
→ MOC-JavaScript
```

如果同一张卡片对其他 MOC 也具有稳定的导航价值，可以被其他 MOC 引用，但不要复制卡片。

如果发现现有 MOC 结构本身已经需要重构，不在本 Skill 中顺手完成 MOC 重构，应交给 `vault-refactor-card`。

---

### 5. 创建卡片

创建：

```text
Notes/<卡片名>.md
```

要求：

* 使用对应模板
* frontmatter 按模板填写
* 围绕单一知识主题
* 不重复已有卡片内容
* 保持当前 Vault 的命名和内容风格

---

### 6. 加入 MOC 导航

在 Primary MOC 的适当位置加入：

```markdown
- [[卡片名]]
```

优先加入现有小节。

如果该小节已经存在 `###` 分组：

> 加入最合适的现有分组。

不要为了单张新卡片自动创建新的 MOC 或新的 `###`。。

---

### 7. 建立卡片关联

只建立真正有知识价值的关联。

优先考虑：

* 因果关系
* 前置 / 依赖关系
* 对比关系
* 互补关系
* 抽象与实现关系
* 原理与实践关系

在卡片末尾维护：

```markdown
## 关联
- [[相关卡片]] —— 一句话说明两者关系
```

关联默认**单向维护**。
