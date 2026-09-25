---
type: experience
tags: [Qt, Windows, 单实例, 互斥, 网络]
date: 2026-09-25
project: 收益计算器
summary: MSYS2 MinGW Qt 6.11 的 QLocalServer/QLocalSocket 跨进程完全失效（同名 listen 两进程都成功、socket 探测连不上）——单实例互斥改用 Win32 CreateNamedPipeW(nMaxInstances=1) OS 级原子锁，恒恰一持有者；管道名与原版相同保留跨版本互斥
provenance: DFD-Cpp 波1b 审核 F-P1（200 轮双进程探针 101 轮双 primary 实测）· commit 4912a5b
status: verified
---

# MSYS2 Qt 的 QLocalServer 跨进程失效与原子管道互斥

## 现象
单实例锁（QLocalServer + QLocalSocket 100ms 探测）双进程同步启动时约 50% 两实例都成为 primary——互斥契约被击穿，双写同一 data.json 可致数据损坏。

## 根因（探针实证，比表面更深）
1. 表面：探测失败分支无条件 `removeServer` 会误杀并发赢家刚建立的 server。
2. 深层：**本机 MSYS2 MinGW Qt 6.11 的 QLocalServer 跨进程完全失效**——同名 `listen()` 在两进程都成功（无 OS 级互斥）；`QLocalSocket` 探测连不上已有跨进程 server（`Invalid name`）。而原生 `CreateNamedPipeW` 同名二次创建实测 `err=231 (ERROR_PIPE_BUSY)`、`CreateFileW OPEN_EXISTING` 实测成功——OS 层正常，坏在 Qt 封装层。

## 修复（更强方案）
Windows 主路径弃用 Qt 网络层，改 **raw named pipe 原子互斥**：`CreateNamedPipeW("\.\pipe\<锁名>", ..., nMaxInstances=1)`——OS 级原子，第二个创建者必得 ERROR_PIPE_BUSY，恒恰一持有者，无探测时序依赖。析构 CloseHandle 释放；进程崩溃由内核自动清理。**管道命名空间与原版 PySide6 QLocalServer 相同 → C++ 与 Python 版天然互斥（保护共享数据）**。非 Windows 回退探测→listen。

## 判别与预防
- 单实例/互斥类功能**必须做同时启动竞态测试**（双进程同步启动 N 轮断言恰一 primary）——顺序双开测试全绿掩盖竞态（本波 200 轮探针才暴露）。
- Qt 抽象层在某些构建/环境下可能有深层失效——关键互斥不要依赖未经竞态测试的封装层。
