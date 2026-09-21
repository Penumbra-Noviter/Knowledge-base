---
type: lesson
tags: [Flutter, Windows, 测试, 进程]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: Windows 下 flutter test 被 kill 会残留 flutter_tester 进程锁 sqlite3.dll——重跑/清理前 Get-Process flutter_tester | Stop-Process -Force
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· 本批多次（flutter_tester 残留进程文件锁清理）· commit 9f68a9d
status: verified
agents_md_feedback: 
---

# flutter_tester 残留进程锁 sqlite3.dll（Windows）

## 症状
`flutter test` 运行中被 kill/中断后，后续重跑或 worktree 清理失败：sqlite3.dll 被占用（文件锁）——残留的 `flutter_tester.exe` 进程持有句柄。

## 根因
flutter test 以 flutter_tester 子进程承载测试，kill 主命令不保证杀死子进程树；Windows 下 SQLite 库文件被残留进程持有句柄 → 重跑（加载 dll）与文件操作（删除/重建）失败。

## 代价
重跑前需先杀进程（本批多次）；未杀则误判为「测试环境损坏」浪费排查时间；worktree 残留文件无法删除。

## 教训
Windows 上 shell 进程树的子进程残留是常态——测试进程尤其如此（子进程持有资源锁）；重跑前清理残留进程是环境前置，不是可选步骤。

## 防复发
- flutter test 重跑/清理前：`powershell -Command "Get-Process flutter_tester -ErrorAction SilentlyContinue | Stop-Process -Force"`。
- 子代理派发 prompt 显式注明此清理步骤（本批已固化）。
- [x] 已落实？

## 关联
- [[]]