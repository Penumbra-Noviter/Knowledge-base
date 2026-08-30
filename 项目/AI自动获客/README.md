# AI自动获客（项目页）

- 仓库：`D:\Desktop\Craft\AI自动获客`
- 产品：自建抖音获客自动化工具（以商业软件 `阿里山AI获客` huoke-radar-app v2.9.89 为行为规格的自建等价产品）
- 技术栈：Electron 壳 + 接口拦截 + BrowserView DOM 自动化 + FastAPI + Vue3 + SQLite + OpenAI 兼容 AI 适配
- 状态：**首期（纵深核心）已交付**（2026-08-30，15 工单完成）；二期登记待规划

## 项目语境

- 共识决策：`CONSENSUS.md`（Q1–Q11）
- 行为规格：`BEHAVIOR_SPEC.md`（开发蓝本，节 1–5 首期）
- 架构：`ADR/ADR-001-architecture.md`
- 生命周期：`README.md` / `CLAUDE.md` / `PROJECT_REFERENCE.md` / `TO-TICKETS.md` / `TECH_DEBT.md` / `DEV_LOG.md` / `KNOWLEDGE_BASE.md`
- 首期实现记录：`.scratch/phase1/`（gitignored，spec/tickets/orchestration/evidence，审计追溯）

## Persona

- [[项目/AI自动获客/persona|Persona 画像]]

## 经验连接

- 经验笔记：`经验/` 目录（按 `project: AI自动获客` 检索）
- 关联经验（既有）：`Python 3.12 pyc 逆向用重编译对比闭环取代反编译器`（逆向方法学）、`spike有界任务子代理连续故障时主会话直做`、`spike阈值与验收线必须同口径联合标定`