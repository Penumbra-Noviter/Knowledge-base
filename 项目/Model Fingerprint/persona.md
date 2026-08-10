---
type: persona
tags: [persona]
date: 2026-08-09
project: Model Fingerprint
status: active
---

# Model Fingerprint Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好

- 中文文档 + 中文 Conventional Commits（`refactor:` / `fix:` / `docs:` / `chore:`）
- 先结论后证据、简洁不修饰；公开函数 type hints + docstring + 模块 `__all__`
- 零外部依赖优先（运行时仅 `requests`）；覆盖率门槛 ≥90%（pytest.ini 强制）
- docstring / 注释引票号（T-XXX），删除点注释说明理由防回退

## 固定约束

- 领域词以 `CONTEXT.md` 为准（维度规约 / 维度结果 / 判定 / 证据 / 报告契约 / 复核通道 / 对抗信号）
- ADR-0001~0010 不重议（`CONSENSUS.md`）；改权重/阈值/LR 常数前必读 `ALGORITHM_OPTIMIZATION.md`（§2 严谨性缺口 + §6 红线）
- **不拖分契约**：unknown/error/skipped 维度不提供证据（LR=1 或等价中性语义），不可破坏（ALGORITHM_OPTIMIZATION §6；T-416 后 linear 亦对齐）
- 判定/入库门槛 0.6 消费校准分（`decision_score`，ADR-0010）；阈值政策单源 `fingerprint/thresholds.py`
- 判定分**行为政策**单源 `fingerprint/score_policy.py`（decision_score 双型回退链 / band_for 分档 / apply_evasion_penalty 罚则；数值政策仍 thresholds.py 叶，T-432~T-434）
- `.gitattributes` `*.py text eol=lf`（2026-08-09 T-412 后全仓生效）
- 报告形状：三态（deep/quick/regions）经 `report_contract.py` 契约，消费方 `report_kind()` 分派
- 共享文档（TO-TICKETS / DEV_LOG / CLAUDE / README / MANUAL / CONSENSUS）是事实来源：待办唯一来源 `TO-TICKETS.md`，已做 `DEV_LOG.md`，决策 `CONSENSUS.md`

## 稳定模式

- **行为保持重构以全量测试为安全网**：943 测试 + 字节等价验证（旧代码 git archive 提取 vs 新代码 diff）
- **单一事实来源**：维度注册器（iter_specs/spec_for/collect_evidence）、阈值政策（thresholds.py）、报告契约（report_contract.py）、summary 块（PlainSummary 含 ReviewBlock/IdentifyBlock/MetaBlock）——消费者经 Seam 读取，不手解析 dict
- **worktree 串并行 kickoff 流程**：工单带文件范围元数据 → 相交组串行链（同一代理）、不相交组并行（≤3）；基线 commit 作 code-review 固定点；波末 `--no-ff` 合并 + 全量测试
- **code-review 双轴每批次收尾**（Standards + Spec 并行子智能体）：无阻断 → 诚实性微修进收尾提交；结构性发现录入 TO-TICKETS 推迟候选表
- **共享文档波内只读**：实现代理不碰 TO-TICKETS/DEV_LOG/CLAUDE，主会话审核后统一收尾提交
- **洁癖收尾**：测试数/覆盖率/重构计数同步进 CLAUDE.md 与归档行；DEV_LOG 一条一票带测试结果
- **部分实现 stash/worktree 未提交资产交接**：代理崩溃后先盘 stash list / worktree list / status，未提交资产（如 T-428 复现测试 224 行）保留复用、续做代理「不重写、在其基础上转绿」；续做代理先验证不信任
- **子代理确认断言以交互事实为准**：汇报中「经主 agent 同意」类断言须能回忆对应交互，回忆不起=未发生=流程偏差（T-427 教训）；分发 prompt 显式禁止自行确认范围外改动

## 变更记录

- 2026-08-09 建档（批次 1~4 会话，14 工单 T-401~T-418，commit c4cc72b 推送 origin/main）
- 2026-08-10 批次 7（T-432~T-434 判定分政策塌缩，commit a8a02e2；测试锁升级接线锁教训入经验库）
