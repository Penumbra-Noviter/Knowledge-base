---
type: lesson
tags: [rAF, cleanup, leak, smooth-scroll]
date: 2026-09-06
project: DeepTutor
source: 自身项目实践
summary: rAF 驱动的动画飞行须双保险终止：元素 detached 自终止 + 卸载/重入时调用 cancel，否则永久帧率空转
provenance: DEV_LOG 2026-09-06 TD-15/F2 节 · commit 3780989d · 期末四轴 Falsify#1 · evidence/03.md §9
status: verified
agents_md_feedback:
---

# rAF 飞行循环必须带卸载取消与 detached 自终止

## 症状

平滑滚动飞行（rAF 渐进逼近目标 scrollTop）期间切换会话（路由段变化 → 整页卸载）后，rAF 循环以帧率永久空转到页面刷新；每次「跳转中离开」叠加一个循环 + 一组 wheel/touchstart 监听，主线程渐进退化。console 无任何报错。

## 根因

飞行的收敛条件是 `|target - el.scrollTop| < snap`，但**detached 元素的 scrollTop getter 恒读 0、写入无效**——diff 永不进 snap 分支；cap 兜底（达上限后 k=1 直冲）写的也是无效值，循环条件永不满足。cancel 函数有返回但调用方（jumpToTurn）丢弃，卸载清理路径不存在。

## 代价

期末四轴 Falsify 轴唯一阻断项，一轮派回修复（+86 行含回归断言）；若合入，用户每次中断跳转都累积一个永久帧率循环。

## 教训

依赖「读 DOM 属性收敛」的 rAF 循环，在元素可能脱离文档后收敛条件**在数学上不可达**，cap 不是终止保证（cap 只改变逼近速度，不改变条件判断）。任何 rAF 飞行需要双保险：①循环体内每帧检查 `el.isConnected`，detached 即自终止并拆手势监听（不依赖调用方纪律）；②调用方持有 cancel，在卸载 effect 与「新飞行覆盖旧飞行」时显式调用。

## 防复发

- [x] 已落实：双保险 + 2 条回归断言锁定（cancel 接线锚点，commit 3780989d）。
- [ ] Falsify 类审查对「循环 + DOM 状态收敛」模式固定检查一个问题：**元素脱离文档后这个循环还能停吗？**

## 关联
- [[React 后代 ref 晚于祖先挂载——scroll 元素须响应式下发]]
- [[共享动画槽竞态修复的寻址边界]]
