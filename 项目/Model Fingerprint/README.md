---
type: project
tags: [project, moc]
date: 2026-09-03
project: Model Fingerprint
status: active
---

# Model Fingerprint 项目索引

> 项目页 = 预检入口：先读 [[项目/Model Fingerprint/persona|persona]]（长期偏好），再按需进仓库文档。经验笔记见 `经验/`（按主题原子化）。

## 定位

识别 OpenAI 兼容接口**实际使用的模型**，对抗中转站偷换模型（用户买旗舰、商家换便宜）。多维指纹 + 反探测伪装 + 贝叶斯融合评分。CLI 工具 + 可选 PySide6 桌面 GUI，本地优先零外部服务。

## 仓库入口

- 仓库：`F:\Craft\model-fingerprint`
- 开发入口 / 架构总览：仓库 `PROJECT_REFERENCE.md`
- 待办唯一事实来源：仓库 `TO-TICKETS.md`（活跃表）；技术债候选池：仓库 `TECH_DEBT.md`
- 已做记录：仓库 `DEV_LOG.md`；需求与决策（ADR-0001~0011）：仓库 `CONSENSUS.md`
- 行为规矩：仓库 `AGENTS.md`；技术唯一来源：仓库 `CODE_WIKI.md`（doc_sync 机械标记防漂移）

## 文档分工

| 文档 | 职责 |
|---|---|
| `TO-TICKETS.md` | 唯一待办事实来源（活跃工单 + 批次归档 + 推迟候选） |
| `TECH_DEBT.md` | 技术债候选池（未立项 + 处置记录），消费 = 显式立项 |
| `DEV_LOG.md` | 只记「已做」（批次历史 / 遥测） |
| `CONSENSUS.md` | 需求 + ADR（决策记录，到 ADR-0011）+ 已知局限 |
| `AGENTS.md` | 行为规矩 + 当前状态指针 |
| `CODE_WIKI.md` | 技术唯一来源（模块 / 签名 / 数据格式 / 测试基线） |
| `MANUAL.md` | 用户手册（GUI 内嵌渲染） |
| `ALGORITHM_OPTIMIZATION.md` | 评分算法优化交接（§2 严谨性缺口 / §6 红线） |

## 状态速览

- 功能成熟可交付：1947 测试全通过，覆盖率 98.10%（门槛 90%）
- 最近批次：批次 42（2026-08-20 GUI 用户手册清理）；批次 5~42 历史见 DEV_LOG
- 推迟候选：T-Q3（capability 真实判别力验收，触发 = 真实端点样本到位，样本已 2/2+ 到位）
- 文档体系：完整档（CODE_WIKI + doc_sync/pre-commit 已生效）

## 相关笔记

- [[项目/Model Fingerprint/persona|persona]]（长期偏好 / 固定约束 / 稳定模式）
- 经验：`经验/` 暂无独立笔记（教训蒸馏从 2026-08-10 起持续，见 persona 变更记录）

>- 2026-09-03 建档：注册表仓库路径由 `D:\Desktop\Craft\model-fingerprint` 修正为 `F:\Craft\model-fingerprint`；agent guide 改名 CLAUDE.md → AGENTS.md；补建仓库 KNOWLEDGE_BASE.md。