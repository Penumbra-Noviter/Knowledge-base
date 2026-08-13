---
type: lesson
tags: [worktree, git-ignored, 运行时资产, 校准表, 并行, kickoff]
date: 2026-08-13
project: Model Fingerprint
source: 批次 20 波 3 增量审核（DEV_LOG 2026-08-12 批次 20 条目，commit a4626cb）
summary: git-ignored 运行时生成物（calib.json 等）在 worktree 并行模式下不可追踪——实现代理在 worktree 内重生成写入独立 data/，主工作树保持 stale 且 git diff 永远看不见，agent 汇报「已重生成」为假而不自知；必须配 freshness 锁测试（磁盘表存在即与 produce 函数逐值一致）兜底
provenance: DEV_LOG#批次20 波3 · commit a4626cb/3add3ca · 2026-08-12
status: verified
---

# git-ignored 运行时资产在 worktree 模式下与主工作树分离

## 症状

T-465（capability 题库 24→14）要求「calib.json 复核重生成」。实现代理在独立 worktree
（`.worktrees/b20-bank`）里跑 `write_calibration()`——`portable_calib_path()` 解析到
**worktree 自己的 data/**，新表写进 worktree；主工作树的 `data/calib.json` 保持
08-11 的旧表（24 题口径）。agent 汇报「已用 write_calibration 同口径覆盖，主 worktree
的旧文件已同步为新表」——**汇报为假而不自知**：它验证的是 worktree 内的文件。波末
增量审核实测 `produce_calibration(seed=7,count=300) != 磁盘文件`（296 分箱点差异）、
mtime 未变，才抓出「重生成」从未真正落到交付物上。

## 根因

`data/calib.json` 被 `.gitignore` 忽略（运行时生成物、首次探测自动生成的既有约定）。
git-ignored 文件**不进 diff、不进 merge、不产生任何版本痕迹**——worktree 是完整独立
checkout，各 worktree 与主工作树各自持有自己的 data/ 副本，谁改了谁都不知情。实现
代理的验证循环（生成 → 对比 → 汇报）全程在 worktree 内自洽，交付物（主工作树的
data/）从未进入它的视野。审核若只看 agent 汇报或 git diff 也必然漏过。

## 代价

一次阻断审核 + 主会话直做修复轮（重生成 + freshness 锁 + 全量测试）；更危险的是
**stale 校准表静默服役**：24 题口径 PAVA 表套在 14 题分布上，校准概率系统性偏离，
没有任何测试能捕获（既有 calibration 测试全用 tmp_path/monkeypatch，从不读真实
data/calib.json）。

## 规则（防复现）

1. **git-ignored 运行时生成物必须配 freshness 锁测试**：磁盘文件存在 → 与产生函数
   （produce_calibration）逐值相等；不存在 → skip（首启前无锁）。agent 在 worktree
   里怎么折腾都过不了主工作树的锁。
2. **worktree 模式下「重生成/写文件」类验收由主会话或波末审核在主管线上验证**——
   agent 的自证只在它的 worktree 内有效。
3. 生成物若需跨机器可审计（如打包分发依赖它），考虑纳入版本控制或钉校验和——
   依赖 git-ignored + 人工同步的组合必然漂移。
