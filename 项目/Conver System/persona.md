---
type: persona
tags: [persona]
date: 2026-08-09
project: Conver System
status: active
---

# Conver System Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好
- CLI 优先：核心逻辑先以命令行形式实现，再包装 UI
- 先结论后证据，简洁清晰；代码必须完整可运行，不写半截代码
- 业务逻辑英文命名，公开函数完整 type hints + docstring；不允许 `except: pass`

## 固定约束
- 后端：FastAPI + SQLAlchemy 2.0 + SQLite（**同步 ORM，不使用 async**）+ Pydantic v2
- 前端：HTML + Vanilla JS (ESM)，不引入 React/Vue
- LLM：anthropic + openai SDK，自定义 Provider 抽象层（新增 Provider = 实现 BaseLLM → `__init__.py` 注册）
- 配置：pydantic-settings 读 .env；运行时配置通过 DB settings 表管理
- SQLite 默认不启用 FK → 必须 `PRAGMA foreign_keys=ON`
- 架构分层：api（只做 HTTP 映射）/ models（纯数据定义）/ schemas / services（ORM 操作全在此）/ config / database / main；路由不直接操作 ORM
- 所有包 `__init__.py` 必须有 `__all__`；模块要「深」

## 稳定模式
- 评审驱动多轮收敛（三轮架构评审 11+5+3 候选零回滚），不设"重构周"
- 单一事实来源：待办只进 TO-TICKETS、已做只进 DEV_LOG、技术细节 CODE_WIKI
- 回退链设计：显式传参判定用 `model_fields_set` 而非值比较（[[Pydantic model_fields_set 区分显式传参]]）
- 用户可见错误在边界处映射为领域可读提示（[[Conver System 高频小坑汇总]]）
- LLM 结构化输出三级兜底提取 + 白名单过滤（[[LLM 生成 JSON 三级提取策略]]）

## 变更记录
- 2026-08-09 建档（来源：全局 AGENTS.md §二.1 + 知识库经验笔记）
