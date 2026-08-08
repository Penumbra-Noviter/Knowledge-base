---
type: lesson
tags: [Rust, 架构, 桌面应用, CRUD, 模板宏, personal-dashboard]
date: 2026-08-06
project: 川流不息
source: 自身项目实践
status: verified
---

# Rust 通用 CRUD 模式消除样板代码

## 症状

10 个实体每个都需要重复实现 `get_by_id`、`list`、`create`、`update`、`delete` 方法，每个方法的实现模式完全一致（SQL 拼装 + 参数绑定 + 结果映射）。48 个 Tauri 命令中有大量透传调用的模板代码。

## 根因

Rust 没有内置 ORM，每个实体天然需要重复的 CRUD 操作。如果逐一手写，每个实体约 15-20 行样板代码，10 个实体就是 150-200 行。同时，每个实体的 CRUD 操作又需要独立的 Tauri 命令，导致命令文件膨胀。

## 代价

- 重构前：48 个 Tauri 命令，10 个命令文件，`lib.rs` 注册 48 行
- 重构后：5 个通用 CRUD 命令 + 12 个专有命令，`lib.rs` 注册 16 行
- 净删 3375 行代码（含其他优化项）
- Rust 测试从 101 增至 107

## 教训

**Rust 桌面应用（Tauri/SQLite）消除 CRUD 样板的标准模式：`Repository<T>` trait + `impl_repo!` 宏（生成 SELECT_COLUMNS、row_to_entity 及 CRUD 方法） + 通用 CRUD 命令（通过 `entity: String` 调度到具体 repo）。** 专有命令只保留有业务逻辑的操作（如签到中的连续天数计算），纯 CRUD 不走专有路由。

## 防复发

- [ ] 新增实体时三步走：`models/X.rs` 定义模型 → `impl_repo!` 生成样板 → `generic_crud.rs` 注册路由
- [ ] 不创建纯 CRUD 的专有命令；只在有业务逻辑时开新命令文件
- [ ] `lib.rs` 的 `generate_handler![]` 只注册通用命令 + 专有命令

## 关联

- [[Tauri v2 IPC 前端调用差异]]
