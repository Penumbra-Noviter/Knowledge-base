---
type: lesson
tags: [Flutter, State, tab切换, 闭包, conver-system]
date: 2026-09-07
project: Conver System
source: 自身项目实践
summary: HomeShell switch 直接切换 tab（无 IndexedStack）= 每次切回重新挂载 State；视图首挂载闭包捕获的 context 卸载后 defunct，点击即异常——接线须每次挂载刷新 + 闭包内 mounted 守卫
provenance: mobile DEV_LOG〈M5 kickoff 批次〉波末 B1（simulators_view）· commit 9630423 · 2026-09-07 M5 会话
status: verified
---

# HomeShell switch 切 tab 无 IndexedStack：视图闭包持有已卸载 State 的死 context

## 症状
移动端模拟器列表页 AppBar 四个入口（AI 生成/存档/导入/文件打开）在**首次进入后切到其他 tab 再切回**，点击任意入口 → 异常（release 下 `Navigator.of(null)` NoSuchMethodError / 对话框永不打开 / 入口失效）。单测 40 全绿（均为单次挂载）捕获不到。

## 根因
- `HomeShell` 用 `switch` 直接切换子视图（**无 IndexedStack 保活**）——切走 = 视图整体卸载、State defunct；切回 = **全新 State 挂载**；
- 视图 `initState` 里的接线闭包（`() => unawaited(_openGenerateDialog(context))`）捕获**调用时求值**的 `context`（首挂载 State）；闭包被存进控制器槽位后不随视图销毁；
- 切回时新 State 挂载，但接线代码 `if (slot != null) return;` 守卫**短路**——闭包仍指第一个已卸载 State → 点击落死 context。

## 代价
一轮 B1 修复（wireViewDefaults 重构 + 4 回归测试）；若未发现，切 tab 后模拟器页入口全部静默失效，是最易漏的「单测绿但交互坏」。

## 教训
- **switch 直接切换（无 IndexedStack）的 tab，视图状态每次切回都被重建**——「首次接线 + 唯一挂载」假设不成立；
- **闭包上下文安全三件套**：① 接线放「每次视图挂载都执行」的路径（`wireViewDefaults` 刷新槽位）；② 闭包内 `if (!mounted) return;`；③ 需要 context 时用 `this.context`（State 自身重取，取到当前有效 context）而非捕获字面量；
- **「单次挂载」的 widget 测捕获不到「卸载-重建」时序**——必须显式 `pumpWidget(挂载) → pumpWidget(占位卸载) → pumpWidget(重挂载) → 点击断言派发`；
- 设计取舍：若页面状态值得保活（滚动位置/加载态），用 IndexedStack；否则接受重建但必须做上述上下文安全。

## 防复发
- [x] 已落实：`SimulatorsController.wireViewDefaults` 每次视图挂载刷新槽位（外部注入恒优先）+ 四闭包 `if (!mounted) return;` + `this.context`；回归测试 = 挂载→卸载→重挂载→四入口点击均正常派发（修复前全红 + 突变 +0 -3）
- [ ] 通用化：审计 home_shell 其他 tab 视图是否也有「首挂载闭包逃逸」模式

## 关联
- [[模拟器GUI冒烟tap坐标须UI树实测]]（同族：UI 交互坏点只有真机/真实交互时序暴露）
- [[瞬态状态不可做终态断言]]（同族：生命周期/时序类缺陷，测试要打卸载-重建窗口）