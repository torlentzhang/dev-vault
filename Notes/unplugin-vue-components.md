---
type: concept
tags: [vite, 前端工程化]
aliases: [unplugin-vue-component, 组件自动导入, vite-plugin-components]
source: https://github.com/unplugin/unplugin-vue-components
created: 2026-09-22
---
# unplugin-vue-components

## 一句话定义
unplugin 家族的构建插件（Anthony Fu  authored）：构建期按需自动导入 Vue 组件——模板里直接写 `<HelloWorld />`，由插件注入 import 与组件注册，免去手写样板代码。

## 要点

### 工作机制
- 启动时扫描组件目录（默认 `src/components`，递归子目录），建立「组件名 → 文件路径」映射；编译时匹配模板中的标签，自动注入 import 与注册代码
- 转换全部发生在编译期，运行时零开销；tree-shakable——只有实际用到的组件进入 bundle（对比 `app.component()` 全局注册：所有组件无条件打进产物）
- 父组件异步加载（如懒加载路由）时，自动导入的组件随父组件一起代码分割
- 同时支持自定义指令自动导入（`directives` 默认开启）

### Resolver：UI 库按需引入
- 内置 resolver 覆盖 Element Plus / Ant Design Vue / Vant / Naive UI 等：模板写 `<el-button>` 即自动从库中导入组件及其样式——这就是 Element Plus 官方「按需导入」方案的底层
- resolver 本质是 `(componentName) => ({ name, from })` 的映射函数，可自写；没有 resolver 时插件只认识本地 `dirs` 扫描到的组件
- 官方已不再接受新增内置 resolver，改由各库自己提供（如 `@vant/auto-import-resolver`、`@tdesign-vue-next/auto-import-resolver`）

### TypeScript 支持
- `dts: true`（装了 typescript 时默认开启）生成并自动维护 `components.d.ts`，使自动导入的组件在 TS / Volar 获得类型提示；需将其加入 `tsconfig.json` 的 `include`
- `types` 选项为已全局注册的组件（如 vue-router 的 `RouterLink`）只补类型注册、不做导入

### components.d.ts 只增不减（实测会踩）
- dts 不是每次启动全量重算，而是**随「组件被实际使用」逐步写入**：新增并用到组件 → 声明自动出现；**删除或重命名组件后，旧声明不会自动移除**——插件只管加，不管删
- 残留的声明是失效路径（如 `typeof import('./src/components/Foo.vue')['default']`），会污染类型检查，也让 Volar 对已删组件给出错误跳转/补全
- **处理办法：删掉当前的 `components.d.ts`，再重新启动 dev server**，插件会重新生成一份与现存组件目录一致的干净文件（卸载组件库后残留的 `ElButton` 之类声明同理）
- 同一原因也解释「改了组件文件名 / 大小写后 dts 不跟着变」：插件沿用它缓存里的旧路径，只改代码没用，删 dts 重建才是通用解法
- 反向现象已知：dev 下 dts 反而会把某些声明删掉、build 时才补齐（官方 issue #611），导致这个文件在 dev / build 之间反复变化。文件头那句 `We suggest you to commit this file into source control` 说的就是它

### 生态定位
- 一份配置跨 Vite / Webpack / Rspack / Rollup / Rolldown / esbuild 复用（入口不同，如 `unplugin-vue-components/vite`）
- 与 `unplugin-auto-import` 分工：那个管 API（ref / computed…），这个管组件，常成对配置
- Nuxt 项目不需要它：用内置的 `@nuxt/components` 即可
- 前身是 `vite-plugin-components`，旧名与旧选项（`customComponentsResolvers` 等）已过时

## 我的理解 / 类比
像给组件注册请了个「代写 import」的人：源码层面把本地组件和 UI 库组件当全局可用，直接写标签，插件在编译期补齐导入。与全局注册的区别是「用到才注册 vs 全部注册」——便利留在开发期，产物只含实际用到的部分。

## 关联
- [[unplugin-auto-import]] —— unplugin 家族姊妹插件，互补关系：一个自动导入 API，一个自动导入组件，常成对配置
- [[vite+Vue3工程的初始化与启动]] —— Vite + Vue3 工程的常用增强插件，属工程实践环节
