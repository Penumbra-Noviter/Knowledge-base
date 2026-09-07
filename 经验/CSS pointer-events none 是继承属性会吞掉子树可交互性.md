---
type: lesson
tags: [CSS, 注入脚本, 可交互性, WebView]
date: 2026-09-07
project: DeepTutor
source: 自身项目实践
summary: pointer-events:none 是继承属性，一道 none 会吞掉整棵子树的点击与 hover——注入浮层须在交互子树显式恢复 pointer-events:auto
provenance: commit f6523fea · DEV_LOG 2026-09-07 TD-28 节 · 2026-09-07 DeepTutor 会话
status: verified
---

# CSS pointer-events:none 是继承属性，一道 none 吞掉子树可交互性

## 症状

注入式窗口控制条（`include_str!` 注入的 `titlebar.js`，shadow DOM）设计为「默认透明、hover 显示」，宿主 `position:fixed` 设了 `pointer-events:none`。结果是：用户永远看不到它（`mouseenter` 永远不触发、opacity 保持 0），按钮也点不动——主界面看起来「完全没有窗口控制」。

## 根因

`pointer-events` 是 **CSS 继承属性**：宿主设 `none` 后，整棵子树（含 shadow DOM 里的 `.bar` 和按钮）都继承 `none`，除非子元素显式设回 `auto`。宿主自己因 `none` 也收不到 `mouseenter`，于是「hover 显示」的机制本身死掉，不是单纯「没显示」——是整条交互链被切断。

## 代价

- 主界面无窗口控制的问题被用户当作「找不到」，实际是「永远不可达」
- 修复时把「hover 显示」整段设计推翻，改为始终可见 + 显式 `pointer-events:auto`

## 教训

**注入型浮层的透明容器可以设 `pointer-events:none`（防挡下层），但所有「需要可交互/可 hover」的子元素必须显式恢复 `pointer-events:auto`——继承会静默吞掉它们**。想表达「默认隐藏、hover 现身」时，先确认 hover 目标本身能收到指针事件，再谈 opacity 动画。

## 防复发

- 给浮层宿主设 `pointer-events:none` 时，对交互子树（`.bar`、按钮、drag 区）逐个显式写 `pointer-events:auto`
- 可用性自查：任何「hover 才出现」的 UI，先验证触发 hover 的元素不在 `none` 子树内
- 注入脚本（无法走框架测试）改完先实机点一遍
- [x] 已落实？（titlebar.js 重写为始终可见 + `.bar` 显式 `auto`）

## 关联

- [[Tauri v2 IPC 前端调用差异]]（同为注入式窗口控制条的实现配套）
