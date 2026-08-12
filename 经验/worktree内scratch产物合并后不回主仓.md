---
type: lesson
tags: [worktree, subagent, scratch, 审计, 回收]
date: 2026-08-11
project: ZCode 环境
source: 自身项目实践
summary: Implement 在 worktree 内产生的 .scratch 审计脚本合并时不随分支回主仓（gitignore 产物不入 commit）——文档写「复现：python .scratch/xxx.py」前须显式复制回主仓，否则复现命令不可验证
provenance: DEV_LOG「批次 13 波 1」条目（W2 警告）· commit 6cd0c7f/f7847c9 · 2026-08-11
status: verified
---

# worktree 内 .scratch 产物合并后不回主仓

## 现象

批次 13 波 1 三个 Implement 在各自 worktree 内完成审计/实验脚本（`fisher-independence-audit.py` / `fpr-audit.py` / `logprobs-rank-exp.py`），文档（ALGORITHM_OPTIMIZATION §9.x）写入「复现：`python .scratch/xxx.py`」。波末合并后**脚本遗留在 worktree 的 .scratch/**，主仓 .scratch 不存在——增量审核发现「复现命令不可验证」。

## 机制

`.scratch/` 在 .gitignore 中（一次性产物不入库），git merge 只带跟踪文件——worktree 内的 gitignore 产物**不随分支合并回主仓**。审计脚本恰好属于「gitignore 但被文档引用」的灰色地带：既是复现必需品（要保留）又不符合入库纪律（.scratch 惯例）。

## 规则（防复现）

1. 波末回收：主会话在合并后、文档定稿前，**显式把被文档引用的 .scratch 脚本从 worktree 复制回主仓 .scratch/**，并抽查可复现性（两跑一致性）——本批次 fpr-audit 抽查通过。
2. 分发 prompt 明确：实现代理的 .scratch 产物默认「随波末回收清单」申报，文档引用复现命令即视为引用清单项。
3. 增量审核的「交付物缺失」检查应包含：文档引用的复现路径在**主仓**可验证（worktree 现场不算）。

## 相关

- 与「子代理崩溃先盘 stash 与 worktree 资产」互补：崩溃场景盘 worktree 内未提交资产；本教训覆盖**正常合并后**的回收。
