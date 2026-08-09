---
type: lesson
tags: [前端, Vanilla JS, 测试, seam, 依赖注入, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
summary: API 层暴露 setFetch(fn) seam（默认 globalThis.fetch）+ 渲染抽成纯函数模块——无框架 Vanilla JS 也能高质量单测
provenance: git 4f686a5（2026-08-08 知识库批量入库）· Conver System 2026-08-06 实践
status: verified
---

# 无框架前端：API 层注入 fetch seam，让 Vanilla JS 可单测

## 症状
`api.js` 直接调 `globalThis.fetch`，Vitest 单测无法拦截网络层——要么打真实网络，要么测不了。

## 根因
模块级函数硬编码全局 fetch，无注入点。无框架项目没有框架自带的测试基建（如 React Testing Library 的 mock 生态），可测性要自己造。

## 教训
**API 层暴露 `setFetch(fn)` seam**（内部 `doFetch` 默认 `globalThis.fetch`，浏览器行为不变，测试注入 mock），配合「数据 → HTML 渲染抽成纯函数模块」（`format.js`），无框架前端也能上高质量单测：format 15 / utils 8 / api 5 用例。

## 代价
基建搭建 + 28 用例（`8098114`，`docs/architecture.md:115-116`）。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] `api.js` 已实现 seam + 纯函数拆分
- 新前端模块：可测性设计在写代码时同步做（IO 与纯函数分离）
- 同型：本库 [[前端模块化拆分中的循环依赖处理]] 的 setter 注入是同一模式

## 关联
- [[前端模块化拆分中的循环依赖处理]]（setter 注入 seam 同族）
- [[response_model 统一驱动序列化]]（后端侧对应：测试路径与真实路径同源）
