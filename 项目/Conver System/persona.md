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
- 桌面形态偏好：**壳方案**（Tauri 壳 + 复用现有后端，零业务改动）优先于任何「重写」方案（2026-08-11 P6.4 实证）
- 验收以「真实用户路径」为准：自动化冒烟须含**不注入环境变量的干净回归**（注入 env 会掩盖 prod 缺陷）
- kickoff 全自动档偏好：共识确认后全部阶段不等待、仅汇报；无回答的确认点按推荐清单继续（best judgment）
- 桌面交付形态：NSIS 安装器单产物（免管理员 currentUser）；数据目录 %APPDATA% + CONVER_DATA_DIR 覆盖
- 冒烟脚本纪律：绝不清理非自己启动的进程、残留检查限定自己的端口（2026-08-11 误杀网页版 uvicorn 教训）
- 打包回归断言要钉接线行而非 token（`datas=list(_FRONTEND_RUNTIME)` 能抓到 `datas=[]`，token 断言漏报）
- 评审驱动多轮收敛（三轮架构评审 11+5+3 候选零回滚），不设"重构周"
- 单一事实来源：待办只进 TO-TICKETS、已做只进 DEV_LOG、技术细节 CODE_WIKI
- 回退链设计：显式传参判定用 `model_fields_set` 而非值比较（[[Pydantic model_fields_set 区分显式传参]]）
- 用户可见错误在边界处映射为领域可读提示（[[Conver System 高频小坑汇总]]）
- LLM 结构化输出三级兜底提取 + 白名单过滤（[[LLM 生成 JSON 三级提取策略]]）
- 架构深化候选按 /improve-codebase-architecture 报告推荐强度**分批直落** kickoff 全自动批次（ARC-1~8 / ARC-9 Strong 批 / ARC-10 剩余批三次实证：先 Strong 后剩余，两批落地 14 候选全部完成、零回滚）
- 「对齐/镜像契约」类改动必须含真实消费者连接级用例（[[跨语言镜像契约须对真实消费者验证]]：SQLAlchemy 零解码 vs sqlite3 URI 会解码，镜像互相印证 = 盲区共享）
- 技术债区（TICKETS 非阻断遗留）清理走 kickoff 全自动批次：逐项 git grep 复核现状（审计快照惯例），可修的做、不可达/设计意图的「复核确认维持」关闭归档（2026-08-12 16 项清零实证）

## 变更记录
- 2026-08-12 TD-8~12 批次更新：子代理只读探索禁改主工作树（共享文件写权归主会话）；契约锁测试语义（基线绿非先红）与回归测试区分（来源：TD-8~12 kickoff）
- 2026-08-12 TD 批次更新：技术债清理连续两批实证（16 项清零 + TD-1~7 清零）；「先红后绿」硬验收在守卫类工单全面落地；文档同步前提须 grep 全仓验证（来源：TD-1~7 kickoff）
- 2026-08-12 技术债批次更新：技术债区清理 = kickoff 全自动批次 + 复核确认维持关闭惯例（来源：技术债区 16 项清零 kickoff）
- 2026-08-12 ARC-10 更新：审查候选分批直落模式扩展（Strong 批→剩余批；来源：ARC-10 全自动 kickoff）
- 2026-08-12 ARC-9 更新：Strong 候选直落 kickoff 批次 / 镜像契约须消费者级验证（来源：ARC-9 全自动 kickoff + 期末阻断修复）
- 2026-08-11 P6.4 更新：壳方案/干净回归/全自动档/冒烟纪律/接线断言（来源：P6.4 kickoff + 期末审核）
- 2026-08-09 建档（来源：全局 AGENTS.md §二.1 + 知识库经验笔记）
