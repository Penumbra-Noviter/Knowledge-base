---
type: lesson
tags: [subagent, worktree, stash, 崩溃恢复, 编排]
date: 2026-08-09
project: ZCode 环境
source: 自身项目实践
summary: 子代理崩溃恢复先盘资产再动手——stash pop 后崩溃会让部分实现只存在于该 worktree 工作区，重派前查 stash list + worktree list
provenance: DEV_LOG「T-406」条目（stash 交接注）· commit 4e43f96 · 2026-08-09 批次 2 会话
status: verified
---

# 子代理崩溃的 stash/worktree 资产盘点恢复

## 症状
批次 2 串行链代理（T-406~T-410）空返回（无输出无提交）。检查发现：repo 的 stash 列表**为空**——该代理已执行了 `git stash pop`（把上一代理的部分实现从主仓 stash 移交到其 worktree 工作区）然后崩溃；部分实现存活于 `$TMP/mf-wt-chain` 的未提交工作区，若直接重派全新 worktree 或重新 pop 就会丢失或冲突。

## 根因
崩溃发生在「setup 阶段」与「干活阶段」之间：stash pop 是幂等性最差的交接动作——pop 后 stash 记录即删除，资产迁移到 worktree 工作区（未提交、无分支保护）。编排方不盘资产直接重派，会基于错误前提（stash 还在 / worktree 干净）操作。

## 代价
一次盘点（stash list + worktree list + 逐文件核对 13 个修改文件 vs 预期部分实现）确认资产完好；重派代理指示「worktree 已存在、勿再 pop、验证后续做」——零丢失零返工。

## 教训
**子代理崩溃后、任何重派/清理前，先盘三样资产**：`git stash list`（stash 是否已消费）、`git worktree list`（哪个 worktree 持有未提交工作）、`git status`（工作区内容是否匹配预期进度）。交接用 stash 时把「pop 后 stash 即消失、资产只在目标 worktree 工作区」当默认事实。重派 prompt 必须写明「资产在哪里、不要再做交接动作、先验证再续做」。

## 防复发
- [x] 已落实？崩溃恢复流程 = 盘资产（stash/worktree/status）→ 重派到**现有** worktree → 验证；编排层检查清单

## 关联
- [[worktree并行前查CRLF祖父blob]]（同属 worktree 编排家族）
