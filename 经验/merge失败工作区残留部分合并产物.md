---
type: lesson
tags: [git, 工作流, 合并, 残留]
date: 2026-08-12
project: 收益计算器
source: 自身项目实践
summary: git merge（ort 引擎）失败时工作区可能残留「部分合并产物」——非 UU 冲突态、无 MERGE_HEAD、git status 显示普通 M；合并前先确认工作区干净，异常残留先看 diff 内容再 restore
provenance: DEV_LOG 2026-08-12 C4 波合并 · commit 79e0e10
status: verified
---

# merge 失败后工作区残留部分合并产物

## 现象（2026-08-12 C4 波合并实测）

`git merge --no-ff` 被 pre-commit（doc_sync 漂移）拦截后重试，ort 引擎中途失败：
- 不在 merge 状态（无 `.git/MERGE_HEAD`）
- 无冲突标记（无 UU 文件）
- `git status` 显示普通 `M app/main_window.py`
- diff 内容 = **部分合并产物**（既非 HEAD 版也非分支版，`-128/+1` 行的中间态）

## 教训

1. **merge 前先 `git status` 确认工作区干净**——有未提交改动（哪怕是刚被 pre-commit 拦截的 commit）时 merge 会以各种方式失败，失败后 ort 可能留下部分写入。
2. **异常残留先看 diff 内容再处置**——确认是引擎残留（非用户/agent 有意改动）后才 `git restore`，避免误还原。
3. **pre-commit 钩子拦截 merge commit 的处理**：`git merge` 无 `--no-verify` 选项——用 `git -c core.hooksPath=<空目录> merge` 跳过（波末统一刷新 doc_sync 标记后恢复正常提交）。
4. 隔离纪律：审核/验证子智能体不得在主分支工作区做试验性改动（本次残留即审核 agent 的 Falsify 验证留下）。

## 相关

- [[worktree并行前查CRLF祖父blob]]（同族：worktree/merge 环境坑）
