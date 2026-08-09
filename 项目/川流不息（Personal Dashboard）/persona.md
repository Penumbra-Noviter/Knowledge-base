---
type: persona
tags: [persona]
date: 2026-08-09
project: 川流不息
status: active
---

# 川流不息（Personal Dashboard）Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好
- CLI 优先：核心逻辑先以命令行形式实现，再包装 UI
- 先结论后证据，简洁清晰；代码必须完整可运行，不写半截代码
- 业务逻辑英文命名，公开函数完整 type hints + docstring；不允许 `except: pass`

## 固定约束
- 框架：Tauri（v2）
- Rust 端：无内置 ORM，用 `Repository<T>` trait + `impl_repo!` 宏 + 通用 CRUD 命令消除样板（[[Rust 通用 CRUD 模式消除样板代码]]）
- 前端 IPC：显式 ESM import `@tauri-apps/api/core`（无 `window.__TAURI__`）；参数前端 camelCase → Rust snake_case，只作用于顶层参数（[[Tauri v2 IPC 前端调用差异]]）
- 测试 Mock：`vi.mock('@tauri-apps/api/core')`
- 静态资源：`root:"src"` 时放 `src/public/` 而非 `src/assets/`（[[Vite root模式静态资源路径]]）
- CSS 模块化：base → layout → components → animations → pages 分层（[[CSS 单文件到模块化拆分策略]]）
- E2E：tauri-driver 外部启动（`driverProvider: "external"`）+ onPrepare 先构建 debug 产物 + e2e 独立目录（[[Tauri v2 E2E 基建必踩配置]]）

## 稳定模式
- worktree 用完即弃，不留 Rust 编译缓存尸体（[[worktree膨胀26G教训]]）
- 并行 worktree 合并后共享文件（工具函数/共享组件/公共样式）必查「改动是否还在」，零冲突 ≠ 零丢失（[[并行 worktree 合并静默覆盖]]）
- 周历/日期边界动态计算，不写常数（[[ISO 周边界不能硬编码]]）

## 变更记录
- 2026-08-09 建档（来源：全局 AGENTS.md §二.3 + 知识库经验笔记）
