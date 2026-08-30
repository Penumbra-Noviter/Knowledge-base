---
type: lesson
tags: [Android, Flutter, 冒烟, UI自动化, conver-system]
date: 2026-08-29
project: Conver System
source: 自身项目实践
summary: 模拟器 GUI 冒烟 tap 坐标须 UI 树实测（推算坐标落系统导航区），相邻截图字节相同=内容未变自查信号
provenance: mobile DEV_LOG〈M0 kickoff 批次〉· .scratch/m0-kickoff/evidence/g0-gate.md 诚实节 · commit 4da2e3b · 2026-08-29 M0 会话
status: verified
---

# 模拟器 GUI 冒烟 tap 坐标须 UI 树实测——截图字节模式是内容未变的自查信号

## 症状
M0 门校验用 `adb shell input tap` 遍历底部导航 5 tab：推算坐标 (等分宽, 屏高−100) 执行后进程存活、崩溃缓冲区 0 条，但逐 tab 截图中 tab3/tab4 **字节数完全相同**（169053B），tab1/tab2 异常大（1.34MB）；事后 UI 树实读发现设备处于 launcher Recents 概览界面而非应用内。

## 根因
两层叠加：① 推算 y=2300 落在**系统导航条背景区**（三键导航 background 2274–2400），不在应用 NavigationBar（实测 2064–2274，item 中心 y=2169）——手势/系统区触摸被系统 UI 消费，触发误航；② 「进程存活 + 零崩溃」只能证明**没崩**，不能证明**切换发生**——缺一个正向的内容变更信号。

## 代价
一轮无效冒烟（5 张污染截图）+ 一次自查返工（UI 树实测重跑）；若未发现，G0 门会带着假证据放行。

## 教训
**设备 UI 自动化的每一个 tap 坐标都必须来自 UI 树实测**（uiautomator dump / UI Automator describe 的元素 bounds 中心），永不推算；**正向断言要有内容变更证据**——最廉价可靠的是相邻截图字节对比：两张截图字节完全相同 ≈ 界面没变（PNG 对相同内容确定性编码），操作"成功"但内容未动即自查命中。

## 防复发
- [x] 已落实（M0 G0 门）：tap 前 `android_ui_describe` 取元素中心坐标；每步 `screencap` + `cmp` 相邻差异；崩溃判定仍用 logcat crash buffer + pidof 三件套
- [ ] 通用化：任何"点击后应变化的界面"冒烟都可套用字节对比自查（桌面 GUI 同理）

## 关联
- [[冒烟脚本扩展必须真实运行验证]]（同族：语法通过≠真实跑通；本条为其设备端补充——跑通了≠点对了）
- [[Conver System 高频小坑汇总]]