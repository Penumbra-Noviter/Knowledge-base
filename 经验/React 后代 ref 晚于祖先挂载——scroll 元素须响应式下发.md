---
type: lesson
tags: [react, virtual-scroll, ref, mount-order]
date: 2026-09-06
project: DeepTutor
source: 自身项目实践
summary: 传 ref 对象给后代库接线会因挂载时序拿到 null 且无重渲染机会，scroll 元素须经 state 响应式下发
provenance: DEV_LOG 2026-09-06 TD-15/F1 节 · commit a5a02fcd · evidence/03.md §8
status: verified
agents_md_feedback:
---

# React 后代 ref 晚于祖先挂载——scroll 元素须响应式下发

## 症状

虚拟化消息列表（@tanstack/react-virtual）在真实浏览器中零行挂载：spacer 按估算高度布局（totalSize 正常）、rail 55 tick 正常、API 数据完好、console 0 error，滚动后仍 0 行——只有真浏览器 GUI 冒烟暴露，tsc / next build / 源码契约测试全绿。

## 根因

React commit 的 layout 阶段按「后代先于祖先自身」的顺序挂载 ref。列表组件是 scroll 根 div 的**后代**，其 `useVirtualizer` 在挂载期读取 `scrollElementRef.current` 时该 ref 还没被 React 赋值（祖先 div 的 ref 在其后才挂载）；`getScrollElement` 拿到 null，rect/offset observer 从未接线。此后 pin 写 scrollTop 不触发 React 渲染，列表再无 re-render 机会，`getVirtualItems()` 永远为空。tsc/静态检查无法发现——纯运行时时序问题。

## 代价

GUI 冒烟全链路阻断（T1-T5 全部 blocked），一轮派回修复 + 真浏览器三接线对照实验取证（A 原接线复现 / B state 下发修复 / C pin 不设门 offset 模型失对齐），约半天。

## 教训

「把 ref 对象传给后代做初始化接线」的模式，在后代是**DOM 祖先的子孙**时不可靠：读取时机早于赋值，且失败后没有重渲染触发器。跨组件下传 DOM 元素给需要立即使用的库（virtualizer / observer / 测量类），必须经 state（callback ref + setState）响应式下发，让库的接线发生在元素就位后的那次 commit。

## 防复发

- [x] 已落实：scroll 根元素经 `callback ref → setState → prop 下发`（commit a5a02fcd）；2 条源码契约断言锁定「getScrollElement 禁止 `.current` 晚读、页面必须 state 发布」。
- [ ] 虚拟化/测量类库接入后必须真浏览器冒烟一次挂载（源码契约测试抓不到运行时时序）。

## 关联
- [[运行时反测抓静态检测看不见的运行态bug]]
- [[Falsify测试要钉住缺陷所在层]]
