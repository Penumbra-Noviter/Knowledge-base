---
type: lesson
tags: [qt, pyside6, dwm, win32]
date: 2026-11-05
project: Delta Force Dashboard（收益计算器）
source: 自身项目实践
summary: WA_TranslucentBackground 可能引发 show 期原生窗口重建，show 前设置的 DWM 属性会丢——依赖原生句柄的 DWM 设置须 show 后补调一次
provenance: DEV_LOG 2026-11-05「钛空玻璃主题」节 · CODE_WIKI §4.28 · 2026-11-05 Delta Force Dashboard 会话
status: verified
agents_md_feedback:
---

# Qt 半透明属性 show 期重建窗口丢 DWM 属性

## 症状
`apply_glass_backdrop` 在 `show()` 之前置 `WA_TranslucentBackground` 并对 `winId()` 设置 DWM 属性（暗色标题栏 + Acrylic 背板），调用全部成功；但窗口显示后实测「系统背板在、暗色标题栏丢失」——show 前设置的部分 DWM 属性未生效。

## 根因
`WA_TranslucentBackground` 需要带 alpha 通道的原生窗口，可能触发 Qt/系统在 **show 期重建原生窗口**（机制为基于证据的最佳解释：现象每轮启动可复现，且对现存句柄**重设** DWM 属性后即可生效）；show 前设置的 DWM 属性挂在旧句柄上随重建丢失。DWM 属性是挂在具体 HWND 上的窗口状态，不是跟随 Qt 对象的逻辑属性。

## 代价
2 轮截图验证定位（r2 发现「背板在但标题栏亮」→ 与 r1 对比锁定 show 后丢失）。

## 教训
Qt 窗口属性与原生窗口句柄是两层生命周期；**依赖原生句柄的 DWM/系统设置，必须在 `show()` 之后再补调一次重设**（只重设 DWM 属性、不触碰 Qt 属性，避免再次引发重建），并以视觉/日志证据确认，不信「调用成功」。

## 防复发
- [x] 已落实：`main.py` 固定序列 `apply_window_backdrop() → show() → refresh_window_backdrop()`（refresh 只重设 DWM 属性不触 Qt 属性）；`tests/test_glass_backdrop.py` 锁定 best-effort 契约

## 关联
- [[Windows 自定义主题压制 DWM 标题栏 API]]（同为「调用成功 ≠ 生效」家族；本条在标准主题机器上即解，前者是环境级压制）
- [[MSYS2 Qt 的 QLocalServer 跨进程失效与原子管道互斥]]（同为 Qt 平台层与原生层边界问题）
