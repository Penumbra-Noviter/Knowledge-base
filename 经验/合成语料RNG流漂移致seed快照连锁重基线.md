---
type: lesson
tags: [seed快照, 合成语料, RNG, 重基线, kickoff, 测试断言]
date: 2026-09-03
project: Model Fingerprint
source: 批次 43 期末四轴 + 波末范围核验（DEV_LOG 2026-09-03 批次 43 条目）；批次 50 复证（T-531 新维度注册）
summary: 任何改变 generate_batch 每记录 RNG 消费次数/长度的改动（套件长度、题库长度、新维度注册）都会使 seed-42 快照断言连锁失效——改动前先 grep 合成流依赖面预判，重基线须批准 + git 三处留痕 + 独立复现脚本
provenance: DEV_LOG#2026-09-03 批次 43 · commit 1a974a9/59a23d6 · T-521/T-522 重基线 5 处 · 期末四轴 Standards 阻断；批次 50 T-531 重钉 3 处（批准制）· 5380fba · 复现脚本 repro_rng_drift.py
status: verified
---

# 合成语料 RNG 流漂移 → seed 快照断言连锁重基线

## 症状

Model Fingerprint 批次 43 两票连续触发同类重基线：

- T-521（tokenizer 套件 6→9 段）：`tests/test_calibration.py` negative `{"capability"}`→`set()`、维持 18→19；`tests/test_threshold_policy.py` partial_high ≤0.015→≤0.030、fp 0.003→0.017。
- T-522（capability 题库 14→19 题）：`test_threshold_policy.py` raw FN 0.028→0.033。

5 处 seed-42 快照断言全部「漂移后变红」，实现者需逐个重基线；T-521 的重基线仅记在 git-ignored 的 `.scratch/evidence/` → 期末四轴 Standards 判「留痕缺失」阻断。

批次 50 复证（T-531 新增第 12 维 test_retest 注册）——第三种漂移源：新维度注册追加 5 基场景每记录 2 次 RNG 消费（skip 抽签 + unknown 单键采样，detail 零消费）→ 同机制 seed 流漂移 → 重钉 3 处（raw FN 0.033→0.025、direction 审计 positive/negative 集、非维持列表 +capability）。本批在 plan-tickets 阶段即预判（spec §4.2 Drift 实验 + 批准制），无波末/期末两轮抓出，一次落地。

## 根因

`generate_batch`（adversarial.py:263）按**逐项**消费各维度 synthetic RNG：每生成一条语料就调用维度合成函数（`_tokenizer_detail` 的 `rng.randint`、capability 命中带抽签等）。任一维度的**套件/题库长度变化**（T-521/T-522）或**每记录 RNG 消费次数变化**（T-531 新维度注册追加抽签）都会改变该维度消耗 RNG 的调用次数 → 整个 seed-42 语料流从变化点起整体漂移 → 所有依赖该流的下游快照断言（校准方向快照、阈值政策分档占比、FP/FN 率）连锁失效。这是合成流对「每记录 RNG 调用次数 + 长度常量」的隐性全局耦合：改动只声明了「加一个维度」，实际爆炸半径覆盖全部 seed 固定快照。

## 代价

- 批次 43：5 处重基线 + 2 次波末范围核验裁量（T-521 票外 3 文件记「记录警告」）+ 1 次期末四轴 Standards 阻断与修复轮（三处留痕补录）
- 批次 50：3 处重钉（批准制，一次落地零返工）+ 1 次 merge 后 freshness 锁抓出 stale calib.json（git-ignored 运行产物随 12 维流重生成）
- 更危险的是**断言弱化面**：partial_high 容差 2 倍放宽（0.015→0.030）、fp 5.7 倍漂移（0.3%→1.7%，属 0.6 门槛校准红线指标）——若未识别「漂移 vs 真实回归」会掩盖产品缺陷

## 教训

改动任何影响合成语料**长度或每记录 RNG 消费次数**的因素（题库/套件长度、新维度注册、合成函数内抽签次数）前，先 grep 依赖 `generate_batch` 流的 seed 快照断言面（校准方向、阈值分档、FP/FN 率），预判漂移范围并同票申报；重基线数字必须**批准在先**（主会话认可 = 漂移机制成立）+ **git 三处留痕**（TO-TICKETS 归档行 + DEV_LOG 遥测 + spec 修订日志），`.scratch` 证据文件 git-ignored 不算留痕；重钉必须有**独立复现脚本**（双段流 hash 对比）锁定归因。

## 防复发

- [x] 合成流影响面改动票在 plan-tickets 阶段声明「受影响 seed 快照测试文件」面（批次 50 T-531 预判成功，spec §4.2 Drift 实验提前量化；批次 43 T-521/T-522 未预判 → 波末与期末两轮抓出）
- [x] 重基线批准制：Implement 发现 seed 快照漂移先上报，主会话确认机制成立（RNG 流漂移 vs 真实回归）后才放宽断言；未批准即改 = 回退
- [x] 重基线三处留痕：TO-TICKETS 归档行（票条目含重基线说明）+ DEV_LOG 遥测（逐处对应）+ spec 修订日志（每处一行）
- [x] 独立复现脚本：`repro_rng_drift.py` 双段 11 vs 12 维流 hash + 校准 keeps/FP/FN 对照，每处重钉注释引用（批次 50 落地；批次 45 先例 kickoff-r73）

## 关联

- [[测试适配性改动须申报文件范围]]（同批：票外适配申报 + 批准纪律）
- [[验收线基线数字写语义不写死数值]]（相邻根因：快照数字脆弱性；本条目是「合成流漂移」机制面，该条目是「验收线写法」面）
- [[git-ignored运行时资产在worktree下与主工作树分离]]（同批：.scratch 证据 git-ignored 不可溯）
