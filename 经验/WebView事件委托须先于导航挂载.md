---
type: lesson
tags: [Flutter, WebView, 事件委托, 时序, conver-system]
date: 2026-09-07
project: Conver System
source: 自身项目实践
summary: WebView 事件（onPageFinished）只派发给挂载时已存在的委托、不回放过去事件——先 loadRequest 后挂委托会丢事件致 completer 永不完成（超时兜底掩盖）；委托先挂、导航后发
provenance: mobile DEV_LOG〈M5 kickoff 批次〉波末 B1（save_sheet）· commit 8e630d1 · 2026-09-07 M5 会话
status: verified
---

# WebView 事件委托时序：事件只派发给挂载时已存在的委托——先挂委托后导航

## 症状
移动端存档面板（SaveSheet）生产 WebView：`createSheetWebViewController` 先 `await inner.loadRequest(url)` 再返回，消费方返回后才 `setOnPageFinished` 挂导航委托——服务器正常、`http://127.0.0.1:8642/` 索引页加载完成时 onPageFinished 已触发，但事件只派发给「当时已设置」的委托 → `loaded` completer 永不完成 → `.timeout(8s)` 到期 → 面板显示「读取存档超时，WebView 未就绪」降级文案（实际服务器与 localStorage 均正常）。若 `loadRequest` 的 Future 在页面加载完成后才 resolve，则**每次必现**；否则窄窗口竞态偶发。

## 根因
行级单一：**WebView 事件回调是「挂载时注册制」**——`onPageFinished` 只派发给调用 `setOnPageFinished` 时已设置的委托，**不回放挂载前已发生的事件**。先导航后挂委托 = 事件在委托挂载前流失。波及面：任何「loadRequest 后挂事件」的 WebView 时序都中招；单测若用 fake 在「注册后」才派发事件（microtask），恰好编码了理想时序，覆盖不到真实窗口。

## 代价
一轮真实竞态排查（W5 波末审核 Falsify 构造时序才发现）+ 一轮修复（seam 重构 + 回归断言）；未修复会偶发「存档面板超时不可用」。

## 教训
- **WebView「事件驱动 + Future 等待」的装配序 = 先挂委托、后发导航**（`setOnPageFinished` → `navigate(url)`）；若控制器接口不支持先挂，则在 seam 层拆两步（先建控制器〔委托已挂〕→ 再 navigate）；
- **单测必须模拟「事件在挂载前已发生」的窗口**（fake 在 loadRequest 返回后、setOnPageFinished 挂载前的 microtask 序列派发事件），或至少用**调用序 spy**（`indexOf(setOnPageFinished) < indexOf(navigate)`）锁顺序契约；
- 超时兜底（loaded 永不完成 → timeout 降级）**掩盖**了事件丢失——「有兜底不挂死」不等于「时序正确」，兜底只保可见性不保正确性。

## 防复发
- [x] 已落实：save_sheet seam 改零参工厂 + `navigate()` 方法，`_bootstrap` 装配序 = 挂 `setOnPageFinished` → navigate；回归断言 = 调用序 spy + fake 在 navigate 内立即派发事件（若委托未先挂即静默丢事件 → 面板阻塞超时）
- [x] 已落实：W6 波末审核发现 run view（F-43）同构未修 → 落债
- [ ] 通用化：所有 WebView 委托挂载点自查「委托挂载是否先于导航发起」

## 关联
- [[WebView runJavaScriptReturningResult跨平台JSON编码契约与fake契约]]（同族：WebView 平台层真实行为 vs 测试理想假设）
- [[瞬态状态不可做终态断言]]（同族：假时钟/时序类缺陷，测试要打违规时序而非顺时序）