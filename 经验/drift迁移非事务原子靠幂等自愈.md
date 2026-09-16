---
type: lesson
tags: [drift, sqlite, onUpgrade, 迁移, 事务, 原子性]
date: 2026-09-15
project: Conver System
source: 自身项目实践
summary: drift 2.34 onUpgrade 迁移并非事务原子——实测 DDL 裸发无 BEGIN/COMMIT；中断自愈靠 `IF NOT EXISTS` + drift 失败锁库 + user_version 成功后回写，勿在文档声称「默认迁移事务语义」
provenance: DEV_LOG〈人机恋阶段 2 批次〉· W1 波末增量审核 Falsify（logStatements 探针实测）· commit 5fe10e5 · 2026-09-15 会话
status: verified
agents_md_feedback: 印证 AGENTS.md「对抗性自检（证伪而非证实）」——威胁模型依赖的「默认迁移事务原子」被探针实测推翻；文档陈述必须以实测为准
---

# drift onUpgrade 迁移并非事务原子——真实自愈机制是幂等 DDL + 失败锁库 + user_version 后写

## 症状

阶段 2 schemaVersion 2→3 迁移，威胁模型 SR-07 写「drift onUpgrade 默认在迁移事务内执行」（沿某文档断言）。波末增量审核用 `logStatements` 探针实测迁移期 SQL：9 条 `CREATE TABLE/INDEX` **裸发、全程无 BEGIN/COMMIT**——「默认事务原子」陈述不实，中断时无事务回滚兜底。

## 根因

drift 2.34 的 `MigrationStrategy.onUpgrade` 在迁移期间不自动包裹事务（或包裹方式与文档描述不符，以实测为准）；迁移中断的原子性实际由三个独立机制兜底：① 全部 `IF NOT EXISTS`（重复 DDL 幂等）；② drift 打开时迁移失败会锁库（下次重试前库不可用）；③ `user_version` 在迁移**成功后**才回写（中断后未写 → 重开重跑迁移自愈）。

## 代价

一次评审轮次 + 技术债 2 条（F-78 纠偏表述、F-79 补「中断残留→重开自愈」用例）；若按「原子迁移」假设设计业务（如迁移中写业务数据依赖回滚）会踩真实数据损坏面。危害路径当前未见（幂等兜底成立）。

## 教训

drift/ORM 迁移原子性**以实测为准，不沿文档断言**：需要原子性时必须显式包事务；「迁移中断后重开自愈」要专门补测试（构造中断残留状态 → 重开 → 数据一致），不能只测完整成功路径。给威胁模型/文档写「默认 X 语义」前，先看 SQL 探针日志确认 X。

## 防复发

- [x] 已落实：迁移测试补「中断残留 → 重开自愈」用例（F-79 消费时落实）
- [x] 已落实：迁移相关文档/威胁模型表述改为「幂等 DDL + user_version 后写自愈」实测语义

## 关联

- [[审计快照过期需复核]]