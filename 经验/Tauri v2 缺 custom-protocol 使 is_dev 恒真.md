---
type: lesson
tags: [tauri, rust, desktop, release-build]
date: 2026-08-24
project: DeepTutor
source: 自身项目实践
summary: Tauri v2 release 构建必须启用 custom-protocol feature，否则 is_dev() 恒 true，WebView 加载 devUrl 报 localhost 拒绝连接
provenance: 2026-08-24 DeepTutor 桌面修复会话 sess_4fafa097 · Cargo.toml 修复 commit · tauri-2.11.5/src/lib.rs:308 源码确认
status: verified
---

# Tauri v2 缺 custom-protocol 使 is_dev 恒真

## 症状
双击 release exe 启动即显示 Chromium 错误页 "localhost 拒绝连接 / ERR_CONNECTION_REFUSED"，无加载页、无自定义标题栏；后端 sidecar 实际启动正常（8001 可 curl 通）。每次启动稳定复现。

## 根因
`Cargo.toml` 中 `tauri = { version = "2", features = ["tray-icon"] }` 缺 `"custom-protocol"` feature。Tauri v2 源码（tauri-2.11.5/src/lib.rs:308）：

```rust
pub const fn is_dev() -> bool {
  !cfg!(feature = "custom-protocol")
}
```

缺 feature → `is_dev()` 在 release 构建也返回 true → lib.rs 中 URL 选择走 dev 分支：`WebviewUrl::External("http://localhost:1420")`（Vite dev server 地址）而非 `WebviewUrl::App("index.html")`（内嵌资源）。1420 无服务 → 连接拒绝。同一开关还连带污染工作区解析（回 repo root 而非 CUSTOM_DEFAULT_WORKSPACE）和 spawn 策略（legacy 而非 sidecar）。

## 代价
三轮排查约 2 小时：先误判为快捷方式指向已删目录、再误判为端口残留进程、再误判为 `cfg!(debug_assertions)` 不可靠（换成 `tauri::is_dev()` 反而把症状做实）。用户明确批评"别乱猜原因了"后才用 PrintWindow 截窗 + netstat + 读 tauri 源码锁定。

## 教训
编译期 feature flag 缺失时，错误表象（网络连接拒绝）与根因（构建配置）可以完全无关——**"稳定复现的环境差异"优先查构建配置差异，而不是运行时状态**。Tauri v2 项目 release 构建的 feature 清单是隐性契约。

## 防复发
- [ ] 新 Tauri 项目初始化时核对 features 含 `custom-protocol`（`tauri build` 官方模板默认带，手写 Cargo.toml 容易漏）
- [ ] 已落实：DeepTutor `Cargo.toml` 已加；lib.rs 三处 `cfg!(debug_assertions)` 改为 `tauri::is_dev()`（语义与 feature 对齐）

## 关联
- [[Tauri 壳方案打包验证铁律]]
- [[Tauri v2 E2E 基建必踩配置]]
- [[Windows GUI 壳拉起控制台子进程闪窗口]]
