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

- 2026-08-12 批次 20（工程共识批次，同类仓库调研落地，12 票五波全自动 kickoff，全自动档三度跑通；commit 89d8854→26a8b31 推送 origin/main；1623 测试 / 97.99%）：D1 题库 24→14（spike 门控 + 阈值-验收线联合标定 k=11/j=5 同口径落地，calib 重生成 + freshness 锁）；D4 判定性常数全量收敛 constants.py（D4a/D4b 63 名 base64 + facade re-export 保真，语料级 321 行逐字节等价）+ PyArmor 按模块混淆（D4c，维度实现公开——审计工具方法论公开可信原则）；D5 摘要折叠 top-3；I1 cache_replay_trace（期末四轴 F1：强档只保留 nonce 模板动态文本——固定文本跨 session 首见命中是诚实共享缓存常态不计分）；I2 response_poisoning（四类模式 + 上下文豁免 + 零 FP）；I3 --watch。新稳定模式：① **git-ignored 运行时生成物（calib.json 等）在 worktree 并行模式下不可追踪**——agent 重生成写进 worktree 的 data/ 而主工作树 stale、波末审核才抓出；必须配 freshness 锁测试（磁盘表存在即与 produce_calibration 逐值一致）；② 信号语义设计以诚实场景常态为基准（固定文本跨 session 命中 = 诚实共享缓存 → 不计分），防「检测信号本身制造系统性 FP」；③ 调研驱动立项流程（GitHub 竞品对比 → 增量/减量候选 → Grilling 拍板）跑通。

- 2026-08-09 建档（批次 1~4 会话，14 工单 T-401~T-418，commit c4cc72b 推送 origin/main）
- 2026-08-10 批次 7（T-432~T-434 判定分政策塌缩，commit a8a02e2；测试锁升级接线锁教训入经验库）
- 2026-08-10 批次 12（T-440/T-441 双 runner 收编 + 复核通道形状单源化，commit f56a555；通用化重构分支条件作用域教训入经验库）
- 2026-08-11 批次 13（学术调研吸收 RESEARCH_BACKGROUND.md：T-C5/T-C4 审计 + T-C1 logprobs 切换 LRR-top5 + T-P3 flags 收口 + spike×3 全过立远期工单 T-L1/T-Q1~Q3/T-S1；commit f56a555→044ede6；1131 测试 / 97.57%；审计/实验结论 ALGORITHM_OPTIMIZATION §9.1-9.3；新稳定模式：审计类/实验类交付=报告型（.scratch 脚本 + §9.x 章节，零产品代码面）、改特征走三条件判据+合成标定+calib 复核；经验 2 条入经验库：worktree 审计脚本回收 + spike 主会话降级）
- 2026-08-11 批次 13 遗留（kickoff 第二轮 9 票三波：T-442/P3-F3/P4/P5/P6/T-Q2/T-Q1/T-L1/T-S1 全完成；commit 044ede6→1ee03e4；1309 测试 / 97.75%；新模块 capability_gen（离线题库管线）+ prefilter（单 token 前置筛选）；T-L1 验收线重钉 match≥45%（spike 阈值与验收线口径冲突——安全语义优先，新稳定模式：spike 报告的阈值建议与验收线必须同口径联合标定）；经验 1 条入经验库：spike 阈值-验收线同口径标定）
- 2026-08-11 技术债收尾批（P7-P11 + T-Q2 防除零，commit 1ee03e4→5fb2b52；1320 测试 / 97.76%；四轴审核 0 阻断——Standards 0 硬违规 / Spec 缺失 0 / Falsify 0 阻断 / Architecture 0 阻断；新稳定模式：Serena 编辑工具在 Windows 写 CRLF 工作副本（index 仍 LF），编辑后需 sed 归一化 + git ls-files --eol 核查；补种群「宁缺勿滥」语义：同 id 副本不入种群，pool_size 为上限非保证）
- 2026-08-11 技术债批 R1-R5（kickoff 标准档 1 波 2 链：R1 run_evolution 组装层 id 折叠 / R2 mutate(used) 变异容量恢复 / R3 fixture 干扰池单源+接线锁 / R5 summary stored_as 中性文案；commit 8ddd161→cbb87ce；1330 测试 / 97.77%；四轴 0 阻断、非阻断 5 项 → R6-R10；经验 1 条入经验库：Falsify 测试钉住缺陷所在层——迭代会掩盖单层缺陷，验收测试用最小步数钉层）
- 稳定模式（2026-08-11 确认）：kickoff 流程子代理基础设施空返回在多阶段发生（Spec 轴/plan-tickets/Neat，本批 3 次）——重试一次成功率 100%（Spec 重跑、plan-tickets 重派、链 A 重派全成功），有界只读审计（如 Neat 阶段一）可主会话直做；主会话持有完整上下文的小任务优先直做
- 2026-08-11 批次 14（用户反馈 5 commits：159a2ab→6f4f161；1349 测试 / 97.79%）：深度探测预估实测重标（kukuit 真实端点 175.7s → 约 2~4 分钟）；GUI 维度进度状态行；三库统一（默认指纹库迁程序目录 data/db.json——源码=项目根、exe=旁随行，CLI/GUI/exe 同库，用户级目录全面退役）+ GUI 配置随程序 config/；store 近似重复去重 + 完整性>分数>新旧淘汰；quick 失败报告 KeyError 修复。新稳定模式：① 数据布局偏好——所有数据（指纹库/校准表/配置/锁）随程序目录走（开发=项目根 data/+config/，分发=exe 旁随行），不占用户级目录/C 盘；② 去重信号选固定文本 tokenizer 计数（同模型恒定、异模型不可能相等）；③ 3 次问询确认记录：判定分 1.00 追问（baseline skipped vs dbcompare 官方参照）、三库统一方向（用户拍板开发/分发双模式）、旧库删除确认。经验 1 条入经验库：PySide6 整行替换不用 BlockUnderCursor（显式 block 范围）。
- 2026-08-12 批次 18（三短板 kickoff 全流程，五票 T-443~T-447 全完成，四轴 0 阻断；commit 5ee52eb→c0d307a 推送 origin/main；1403 测试 / 97.85%）：T-443 decoy 池 12→24（设计稿+组合数学拍板→同 agent 续做实现，「流程内阻塞」模式跑通）；T-444 R13 符号单源化 display.py；T-445 R14 exec.py 行前缀常量（LINE_* + MARK 派生）；T-446 GUI 判定分口径 + evidence_note 三端；T-447 文档收尾 + exe 重打。新稳定模式：① **背景陈述 ≠ 执行授权**——plan-tickets 把主会话评估中的研究提及当作执行指令（文献广搜 + 文档登记超范围产出），kickoff 分发 prompt 须显式限定「只做票面范围」；② 全自动档确认节奏跑通（仅保留删除清单/阻断修复/最终交付三处确认，波末合并/增量审核/四轴审核/exe 重打自主执行）；③ 设计先行工单 = 设计稿（.scratch 不入库）→ 用户拍板 → 同 agent 续做实现，流程内阻塞不拆票。- 2026-08-12 批次 19（技术债 R15-R20 收口，全自动档 kickoff；commit c0d307a→89d8854 推送 origin/main；1431 测试 / 97.85%）：六票全完成（R15 regions evidence_note 行 / R16 跨题库冲突断言 / R17 caption 模板单源化 display.py / R18 英文显式列表 / R19 FAIL 行单行化 / R20 quick 形状契约锁）；四轴 1 阻断（R19 手写界符清单漏 //-）已修复。新稳定模式：① **格式/解析边界清单必须与消费侧内建语义单源**——手写「行界符/分隔符」清单必然漂移，`" ".join(splitlines())` 与 GUI 消费链同源对齐；② 全自动档二度跑通（Grilling 共识后零等待，仅 Neat 删除清单/阻断修复/最终交付三处确认）；③ AskUserQuestion 连续跳过 = 按推荐执行（三批一致模式）。- 2026-08-13 批次 21（技术债 R23-R25 收口，全自动档 kickoff **小档首跑**；commit c55603e→ceffe21 推送 origin/main；1628 测试 / 97.99%）：T-467 R23 流式聚合 cache shape 保真 + CacheObservation TypedDict 类型化（三处收敛 + 3 防复发）；T-468 R25 capability 答案去 [:200] 存全长（投毒 payload 后段检出链路锁）；T-469 R24 exec_pipe 变体漏检已知口径 docstring 文档化（零行为，R12 同型）。四轴 0 阻断；非阻断 R26-R28 落档（均弱）。新稳定模式：① **小档（2-3 票）跑通**——文件零相交票 + 阻塞票同批，单 Implement 一次调用按阻塞序连续完成（T-467→T-468→T-469），无波次/增量审核机制，期末四轴照常；② **git push fallback 管道坑**——`git push 2>&1 | tail || fallback` 中 exit 码被管道尾命令吞掉、fallback 永不执行；正确做法是直接再跑第二条命令。推送注记：直连间歇可用，代理 key 在 `D:\Desktop\cc\git\.gitconfig`（http.https://github.com.proxy=127.0.0.1:7897，Clash 端口）——直连失败时 `git -c http.https://github.com.proxy= push origin master:main`（清 key 直连），或开 Clash 7897。
- 2026-08-13 批次 22（技术债 R26-R28 收口，全自动档 kickoff **小档二度跑通**；commit ceffe21→db4d750 推送 origin/main；1636 测试 / 97.99%）：T-470 R26 usage_cache_hit 负值钳 0 且保 shape（纯函数单点修，非流式对齐流式，docstring 两口径并存：负整数=钳 0 / 非有限值=跳过形态）；T-471 R27 流式全链测试（stream → observation → records → cache_replay_trace 接线 hop 锁，纯测试票零源码改动）；T-472 R28 [ERROR] 答案截断回 200（_ERROR_ANSWER_MAX_LEN 模块级常量，成功分支全长保留、扫描器零动）。四轴 0 阻断；非阻断 R29（spec 测试数漂移，已顺手同步修订日志）/R30（1e308 大正值 pre-existing，spike §5.1 已知弱点）落档。新稳定模式：① **小档二度验证**——2-3 票零阻塞边批次 = 单 Implement 一次调用按序连续完成（commit 序列清晰、无波次机制、期末四轴照常 0 阻断），文件零相交即并行安全、相交即串行（同 agent）；② **技术债区自洽闭环**——每批落档的弱强度 Falsify 观察（R26-R28）由下一批原样消费（做/关闭拍板），技术债区成为持续改进的队列而非沉淀池。
- 2026-08-13 批次 23（技术债 R29/R30 复核关闭，全自动档 kickoff 纯文档收尾；commit db4d750→547461f 推送 origin/main；1636 测试 / 97.99%）：R29 关闭（已修复复核——批次 22 期末四轴顺手同步 spec 修订日志 + 负浮点测试双在，本批零动作）；R30 关闭（信任边界接受——端点可伪造任意 usage 值，单点数值防御无本质安全增益，spike §5.1「依赖端点如实回报」；docstring 已知口径注记防再发现 client.py:76-77 零行为改动）。新稳定模式：**弱候选关闭裁决路径**——Speculative/弱强度 Falsify 观察的关闭裁决 = Grilling 拍板（复核现状 + 一句话理由）+ 若涉及信任边界加 docstring 已知口径注记防再发现（R12/R24/R30 三例同型），零代码行为改动批次可主会话直做（轻量档），不走子智能体。
