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
- ADR-0001~0011 不重议（`CONSENSUS.md`）；改权重/阈值/LR 常数前必读 `ALGORITHM_OPTIMIZATION.md`（§2 严谨性缺口 + §6 红线）
- **不拖分契约**：unknown/error/skipped 维度不提供证据（LR=1 或等价中性语义），不可破坏（ALGORITHM_OPTIMIZATION §6；T-416 后 linear 亦对齐）
- 判定/入库门槛 0.6 消费校准分（`decision_score`，ADR-0010）；阈值政策单源 `fingerprint/thresholds.py`
- 判定分**行为政策**单源 `fingerprint/score_policy.py`（decision_score 双型回退链 / band_for 分档 / apply_evasion_penalty 罚则；数值政策仍 thresholds.py 叶，T-432~T-434）
- `.gitattributes` `*.py text eol=lf`（2026-08-09 T-412 后全仓生效）
- 报告形状：三态（deep/quick/regions）经 `report_contract.py` 契约，消费方 `report_kind()` 分派；复核通道形状单源同模块（`REVIEW_CHANNELS` 注册表 + `ReviewBlock.from_dict`，summary/report 消费同一 Seam，T-441）
- 共享文档（TO-TICKETS / DEV_LOG / CLAUDE / README / MANUAL / CONSENSUS）是事实来源：待办唯一来源 `TO-TICKETS.md`，已做 `DEV_LOG.md`，决策 `CONSENSUS.md`

## 稳定模式

- **行为保持重构以全量测试为安全网**：1131 测试 + 字节等价验证（旧代码 git archive 提取 vs 新代码 diff）
- **单一事实来源**：维度注册器（iter_specs/spec_for/collect_evidence）、阈值政策（thresholds.py）、报告契约（report_contract.py 含复核通道形状注册表）、summary 块（PlainSummary 含 ReviewBlock/IdentifyBlock/MetaBlock）、深度探测入口单源 `detect_deep_with_config`（T-440 删 17-kwarg shim，参数词汇表收敛 config.py 一处，CLI 是唯一映射点且被映射锁测试锁定）——消费者经 Seam 读取，不手解析 dict
- **worktree 串并行 kickoff 流程**：工单带文件范围元数据 → 相交组串行链（同一代理）、不相交组并行（≤3）；基线 commit 作 code-review 固定点；波末 `--no-ff` 合并 + 全量测试
- **code-review 双轴每批次收尾**（Standards + Spec 并行子智能体）：无阻断 → 诚实性微修进收尾提交；结构性发现录入 TO-TICKETS 推迟候选表
- **共享文档波内只读**：实现代理不碰 TO-TICKETS/DEV_LOG/CLAUDE，主会话审核后统一收尾提交
- **洁癖收尾**：测试数/覆盖率/重构计数同步进 CLAUDE.md 与归档行；DEV_LOG 一条一票带测试结果
- **部分实现 stash/worktree 未提交资产交接**：代理崩溃后先盘 stash list / worktree list / status，未提交资产（如 T-428 复现测试 224 行）保留复用、续做代理「不重写、在其基础上转绿」；续做代理先验证不信任
- **子代理确认断言以交互事实为准**：汇报中「经主 agent 同意」类断言须能回忆对应交互，回忆不起=未发生=流程偏差（T-427 教训）；分发 prompt 显式禁止自行确认范围外改动

## 变更记录

- 2026-08-09 建档（批次 1~4 会话，14 工单 T-401~T-418，commit c4cc72b 推送 origin/main）
- 2026-08-10 批次 7（T-432~T-434 判定分政策塌缩，commit a8a02e2；测试锁升级接线锁教训入经验库）
- 2026-08-10 批次 12（T-440/T-441 双 runner 收编 + 复核通道形状单源化，commit f56a555；通用化重构分支条件作用域教训入经验库）
- 2026-08-11 批次 13（学术调研吸收 RESEARCH_BACKGROUND.md：T-C5/T-C4 审计 + T-C1 logprobs 切换 LRR-top5 + T-P3 flags 收口 + spike×3 全过立远期工单 T-L1/T-Q1~Q3/T-S1；commit f56a555→044ede6；1131 测试 / 97.57%；审计/实验结论 ALGORITHM_OPTIMIZATION §9.1-9.3；新稳定模式：审计类/实验类交付=报告型（.scratch 脚本 + §9.x 章节，零产品代码面）、改特征走三条件判据+合成标定+calib 复核；经验 2 条入经验库：worktree 审计脚本回收 + spike 主会话降级）
- 2026-08-11 批次 13 遗留（kickoff 第二轮 9 票三波：T-442/P3-F3/P4/P5/P6/T-Q2/T-Q1/T-L1/T-S1 全完成；commit 044ede6→1ee03e4；1309 测试 / 97.75%；新模块 capability_gen（离线题库管线）+ prefilter（单 token 前置筛选）；T-L1 验收线重钉 match≥45%（spike 阈值与验收线口径冲突——安全语义优先，新稳定模式：spike 报告的阈值建议与验收线必须同口径联合标定）；经验 1 条入经验库：spike 阈值-验收线同口径标定）
- 2026-08-11 技术债收尾批（P7-P11 + T-Q2 防除零，commit 1ee03e4→5fb2b52；1320 测试 / 97.76%；四轴审核 0 阻断——Standards 0 硬违规 / Spec 缺失 0 / Falsify 0 阻断 / Architecture 0 阻断；新稳定模式：Serena 编辑工具在 Windows 写 CRLF 工作副本（index 仍 LF），编辑后需 sed 归一化 + git ls-files --eol 核查；补种群「宁缺勿滥」语义：同 id 副本不入种群，pool_size 为上限非保证）
- 2026-08-11 技术债批 R1-R5（kickoff 标准档 1 波 2 链：R1 run_evolution 组装层 id 折叠 / R2 mutate(used) 变异容量恢复 / R3 fixture 干扰池单源+接线锁 / R5 summary stored_as 中性文案；commit 8ddd161→cbb87ce；1330 测试 / 97.77%；四轴 0 阻断、非阻断 5 项 → R6-R10；经验 1 条入经验库：Falsify 测试钉住缺陷所在层——迭代会掩盖单层缺陷，验收测试用最小步数钉层）
- 稳定模式（2026-08-11 确认）：kickoff 流程子代理基础设施空返回在多阶段发生（Spec 轴/plan-tickets/Neat，本批 3 次）——重试一次成功率 100%（Spec 重跑、plan-tickets 重派、链 A 重派全成功），有界只读审计（如 Neat 阶段一）可主会话直做；主会话持有完整上下文的小任务优先直做
- 2026-08-11 批次 14（用户反馈 5 commits：159a2ab→6f4f161；1349 测试 / 97.79%）：深度探测预估实测重标（kukuit 真实端点 175.7s → 约 2~4 分钟）；GUI 维度进度状态行；三库统一（默认指纹库迁程序目录 data/db.json——源码=项目根、exe=旁随行，CLI/GUI/exe 同库，用户级目录全面退役）+ GUI 配置随程序 config/；store 近似重复去重 + 完整性>分数>新旧淘汰；quick 失败报告 KeyError 修复。新稳定模式：① 数据布局偏好——所有数据（指纹库/校准表/配置/锁）随程序目录走（开发=项目根 data/+config/，分发=exe 旁随行），不占用户级目录/C 盘；② 去重信号选固定文本 tokenizer 计数（同模型恒定、异模型不可能相等）；③ 3 次问询确认记录：判定分 1.00 追问（baseline skipped vs dbcompare 官方参照）、三库统一方向（用户拍板开发/分发双模式）、旧库删除确认。经验 1 条入经验库：PySide6 整行替换不用 BlockUnderCursor（显式 block 范围）。
- 2026-08-12 批次 18（三短板 kickoff 全流程，五票 T-443~T-447 全完成，四轴 0 阻断；commit 5ee52eb→c0d307a 推送 origin/main；1403 测试 / 97.85%）：T-443 decoy 池 12→24（设计稿+组合数学拍板→同 agent 续做实现，「流程内阻塞」模式跑通）；T-444 R13 符号单源化 display.py；T-445 R14 exec.py 行前缀常量（LINE_* + MARK 派生）；T-446 GUI 判定分口径 + evidence_note 三端；T-447 文档收尾 + exe 重打。新稳定模式：① **背景陈述 ≠ 执行授权**——plan-tickets 把主会话评估中的研究提及当作执行指令（文献广搜 + 文档登记超范围产出），kickoff 分发 prompt 须显式限定「只做票面范围」；② 全自动档确认节奏跑通（仅保留删除清单/阻断修复/最终交付三处确认，波末合并/增量审核/四轴审核/exe 重打自主执行）；③ 设计先行工单 = 设计稿（.scratch 不入库）→ 用户拍板 → 同 agent 续做实现，流程内阻塞不拆票。