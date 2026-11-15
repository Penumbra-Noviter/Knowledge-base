---
type: persona
tags: [persona]
date: 2026-10-24
project: deepseek-harness
status: active
---

# deepseek-harness Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好

- CLI 优先、渐进式开发；先结论后证据，简洁清晰
- 双实例架构决策（ADR-001）：检索 19527 lite 忠实面 / coding 摄入 19528 pro 分离，不合并
- 第三方 SDK 集成前先实测 HTTP 透传面（client 签名支持 ≠ 服务器/路径行为支持）
- 知识库经验实时检索（kb-search 向量优先，需 19527 在跑）

## 固定约束

- **双实例分离**（ADR-001，不可回退）：19527 lite 检索 zcode/kb（chat 路）+ 19528 pro coding 摄入（coding 路），独立数据目录
- **markdown 唯一事实源**：Hy-Memory 只是检索索引 + 摄入管线，不改动 md 源文件
- **凭证纪律**：key 只走 `.env` / `.dsh/.credentials.yaml` / 环境变量，永不入库
- **hy_server.py 是自持 HTTP seam 薄补丁**（subclass 补透传，不绕层自建、不膨胀为行为代理）
- **SDK workspace_id 语义**（2026-10-24 实证）：仅 coding 路生效，lite/chat 路忽略——工具对 lite 非空 workspaceId 显式拒绝（workspace-filter-unsupported），不静默假过滤

## 稳定模式

- HTTP seam 字段透传面补丁随仓库版本化（T-01 薄补丁路线）
- 索引器带文件锁（并发 fail fast、stale 回收，stdlib O_EXCL）
- 自测隔离：self-test 用隔离 user + 唯一内容标记 + 生产库快照基线守卫（双 {-1,-1} fail-fast）
- 修复轮纪律：修复带回归断言；波末增量审核 + 期末四轴（Standards/Spec/Falsify/Architecture）
- 技术债候选池（TECH_DEBT.md）与任务池（TO-TICKETS.md）分离，消费走 kickoff 显式立项

## 变更记录

- 2026-10-24：建档（来源：技术债消费批次 kickoff 完成段，handoff 后续项 4 补登记）