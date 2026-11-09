---
type: moc
tags: [moc, engineering, testing]
created: 2026-11-06
---
# Testing · MOC

> 子领域地图：前端测试的入口——按粒度分层（单元 / 集成 / e2e），每层各有一套工具。
> 「分类」回答该在哪一层写检查，「单元测试」「端到端测试」分别对应最细与最粗两层的工具落地。

## 测试分类与维度
- [[前端测试分类]] 粒度分层（单元 → 集成 → e2e）、三个维度（全面 / 持续 / 自动化）、六类测试的分工

## 单元测试
- [[单元测试框架]] Vitest / Jest / Mocha 的定位与取舍，Vite 项目的配置复用
- [[Vitest 生命周期钩子]] 8 个钩子分三类、执行顺序、返回值即清理函数
- [[组件挂载与 stub]] Vue 侧挂载方式：mount / shallowMount 的取舍与三个坑
- [[组件测试的输入与输出]] 只测 props / slots / emits / attrs，不测实现细节
- [[测试 v-model]] 双向绑定的测法：setValue + 挂 onUpdate 回写、v2 无 setChecked
- [[测试定时器]] 假定时器推进时间、setSystemTime 与 waitFor 的分工

## 端到端测试
- [[端到端测试工具]] Cypress 与 Playwright 的对照，浏览器覆盖范围的精确口径

## 集成测试、性能 / 可用性 / 兼容性
（组件交互的测试组织、性能基准、可访问性与跨端验证 …）
