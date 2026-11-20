---
type: concept
tags: [vue, nuxt]
created: 2026-11-20
aliases: [Nuxt 目录结构, Nuxt 约定, Nuxt 工程结构]
source: https://nuxt.com/docs/guide/directory-structure · https://nuxt.com/docs/guide/directory-structure/app/components · https://nuxt.com/docs/guide/directory-structure/app/plugins · https://nuxt.com/docs/guide/directory-structure/app/middleware · https://nuxt.com/docs/guide/directory-structure/shared · https://nuxt.com/modules
---
# Nuxt 工程目录与约定

## 一句话定义
Nuxt 的项目结构是一套「目录即约定」的映射：文件放在哪个目录、叫什么名字，直接决定它被自动导入、注册为路由、成为插件还是中间件，不需要在配置里手工登记。

## 用于解决什么问题
手写路由表、逐条注册组件与插件、区分服务端/客户端代码，本来都是要在配置文件里同步维护的样板；Nuxt 把这些登记动作交给文件系统，少一处永远要手动对齐的地方。

## 要点

### 目录骨架（Nuxt 4）

```text
my-app/
├── app/                    # 应用源码（Nuxt 4 默认 srcDir）
│   ├── app.vue             # 根组件
│   ├── app.config.ts       # 响应式应用配置（运行时可改）
│   ├── error.vue           # 自定义错误页
│   ├── pages/              # 文件路由
│   ├── layouts/            # 布局
│   ├── components/         # 组件（自动导入）
│   ├── composables/        # 组合式函数（自动导入）
│   ├── utils/              # 工具函数（自动导入）
│   ├── middleware/         # 路由中间件
│   ├── plugins/            # 应用插件
│   └── assets/             # 交给 Vite 处理的资源
├── server/                 # Nitro 服务端
│   ├── api/                # /api/* 接口
│   ├── routes/             # 非 /api 前缀的服务端路由
│   ├── middleware/         # 服务端中间件
│   ├── plugins/            # Nitro 插件
│   └── utils/              # 服务端工具函数
├── shared/                 # 前后端共用代码
│   ├── utils/              # 自动导入
│   └── types/              # 自动导入
├── public/                 # 原样输出的静态文件
├── modules/                # 本地 Nuxt 模块
├── layers/                 # 可复用层（自动注册）
├── nuxt.config.ts
└── .nuxt/                  # 构建产物，不要手改
```

同名目录分属两套运行时，是这一层最主要的混淆来源：

| 目录 | 属于 | 用途 |
| --- | --- | --- |
| `app/middleware/` | Vue 应用 | 路由中间件，跳转前执行 |
| `server/middleware/` | Nitro | 服务端中间件，每个请求执行 |
| `app/utils/` | Vue 应用 | 前端工具函数 |
| `server/utils/` | Nitro | 服务端工具函数 |

`shared/` 有硬限制：不能引用 Vue 或 Nitro 的代码，只放纯函数与类型。

### 命名即行为

这条是 Nuxt 最需要适应的部分——文件名不只是标识，它决定行为：

| 约定 | 写法 | 结果 |
| --- | --- | --- |
| 组件目录前缀 | `components/base/Button.vue` | 标签名是 `<BaseButton />` |
| 懒加载前缀 | `<LazyMountainsList />` | 组件代码延迟加载，延迟水合只对 `Lazy` 组件生效 |
| 关闭目录前缀 | `components: { pathPrefix: false }` | 标签名回到文件名本身 |
| 插件后缀 | `plugins/gtag.client.ts` / `.server.ts` | 只在客户端 / 只在服务端加载 |
| 中间件类型 | `middleware/auth.global.ts` | 全局中间件；不带 `.global` 的需在 `definePageMeta` 启用 |
| 中间件名归一 | `middleware/authCheck.ts` | 中间件名是 `auth-check` |
| shared 别名 | `import x from '#shared/foo'` | 只有 `shared/utils`、`shared/types` 自动导入 |

### 常见集成模块

| 需求 | 模块 | 维护方 |
| --- | --- | --- |
| UI 组件库 | `@nuxt/ui` | 官方 |
| 图片优化 | `@nuxt/image` | 官方 |
| 字体 | `@nuxt/fonts` | 官方 |
| 图标 | `@nuxt/icon` | 官方 |
| Markdown 内容站 | `@nuxt/content` | 官方 |
| 代码质量 | `@nuxt/eslint` | 官方 |
| 测试 | `@nuxt/test-utils` | 官方 |
| 调试工具 | `@nuxt/devtools` | 官方 |
| 第三方脚本 | `@nuxt/scripts` | 官方 |
| 最佳实践提示 | `@nuxt/hints` | 官方 |
| 可访问性 | `@nuxt/a11y` | 官方 |
| 状态管理 | `@pinia/nuxt` | 社区 |
| Vue 工具函数 | `@vueuse/nuxt` | 社区 |
| SEO（robots / sitemap） | `@nuxtjs/robots` 等 | 社区 |

官方模块统一在 `nuxt.config.ts` 的 `modules` 数组里声明启用。

### 常见坑
- Nuxt 3 教程把 `pages/`、`components/` 放在根目录，Nuxt 4 都在 `app/` 下
- `plugins/` 只自动注册顶层文件或子目录中的 `index`，其余需在 `nuxt.config` 显式声明
- `shared/` 自动导入只覆盖 `utils/` 与 `types/`，且不能引用 Vue/Nitro API
- 组件目录前缀会让文件名与标签名不一致，搜索时容易找不到
- 全局中间件默认按文件名字母序执行，固定顺序用 `01.setup.global.ts` 这类数字前缀

## 我的理解 / 类比
这套目录像公司工位：坐哪个区、工牌怎么写，决定了你是谁、能干什么——不用每天去行政登记。方便的前提是**别坐错区**：把服务端中间件放进 `app/middleware/`，就像把保安安排进了产品组。

## 关联
- [[Nuxt]] —— 前置：先有 Nuxt 的整体定位与渲染模型，再看目录怎么落地
