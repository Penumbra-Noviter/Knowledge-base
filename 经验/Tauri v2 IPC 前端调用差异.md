---
type: lesson
tags: [Tauri, Rust, 桌面应用, IPC, 前端, personal-dashboard]
date: 2026-08-06
project: 川流不息
source: 自身项目实践
summary: Tauri v2 不再注入 window.__TAURI__——前端显式 ESM import @tauri-apps/api/core；参数前端 camelCase→Rust snake_case（只作用于顶层参数）
provenance: git 4f686a5（2026-08-08 知识库批量入库）· 川流不息 2026-08-06 实践
status: verified
---

# Tauri v2 IPC 前端调用差异

## 症状

Tauri v1 时代的前端代码使用 `window.__TAURI__.invoke()` 调用后端，在 Tauri v2 下全部失效，报错 `window.__TAURI__ is undefined`。

## 根因

Tauri v2 **不再默认注入** `window.__TAURI__` 全局对象。前端必须通过 ESM `import { invoke } from "@tauri-apps/api/core"` 显式导入。同时，Tauri v2 自动将前端 camelCase 参数名转为 Rust snake_case，所以前端参数必须用 camelCase（如 `{entityName: "stats"}`），Rust 端接收 `entity_name: String`。

## 代价

- 定位问题约 30 分钟（文档理解 + 试验）
- 所有前端 IPC 调用点需批量修改
- 测试 Mock 层需同步更新为 `vi.mock('@tauri-apps/api/core')`

## 教训

**Tauri v2 不再有 `window.__TAURI__` 全局——前端必须显式 ESM 导入 `@tauri-apps/api/core`；参数名在前端用 camelCase、后端用 snake_case，Tauri 自动转换。** 升级或新项目直接按 v2 惯例来，不要套用 v1 经验。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）

- 新 Tauri 项目或升级时，先确认 `@tauri-apps/api` 版本，按 v2 导入方式写
- 前端 `invoke()` 参数使用 camelCase，Rust 命令参数使用 snake_case
- 测试 Mock 使用 `vi.mock('@tauri-apps/api/core', ...)` 而非 mock 全局对象

## 补充（ARC-06 实测）

camelCase → snake_case 转换只作用于**顶层参数名**：嵌套 data 按 serde 字段名透传。最终惯例：**单字顶层参数名 + snake_case 嵌套键**；`convertKeys` 这类「统一转换」函数因多此一举被移除——先确认转换边界，再决定要不要写转换层。

## 补充（2026-09-07，DeepTutor）

`withGlobalTauri: true`（tauri.conf.json）会重新注入 `window.__TAURI__` 全局——但**只保证应用页面（tauri.localhost）**：WebView 加载跨源页面（如回环后端 `http://127.0.0.1:8001` 提供的前端）时，全局 API 不保证存在。两条推论：

1. 纯注入型普通 JS（`include_str!` 注入、无法 ESM import `@tauri-apps/api`）不能赌 `window.__TAURI__`——用双路桥接：`window.__TAURI__?.core.invoke` 优先，`window.__TAURI_INTERNALS__.invoke`（每个 Tauri webview 恒在）兜底。
2. 跨源页面的前端代码要调用壳命令时，同样探测 `window.__TAURI__ || window.__TAURI_INTERNALS__`，别假设全局 API 在。

## 关联

- [[worktree膨胀26G教训]]
- [[Tauri 壳方案打包验证铁律]]
