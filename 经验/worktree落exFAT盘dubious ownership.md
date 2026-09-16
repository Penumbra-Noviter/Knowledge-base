---
type: lesson
tags: [git, worktree, windows]
date: 2026-09-14
project: Conver System
source: 自身项目实践
summary: worktree 落在 exFAT 盘不记文件所有权 → git 报 dubious ownership 拒绝一切操作——加 safe.directory 例外 + 新 worktree 统一用 NTFS 路径（如 desktop/.worktrees/）
provenance: DEV_LOG mod-cg-wiring 批次避坑 2 · 2026-09-14 Conver 会话
status: verified
agents_md_feedback: 印证 AGENTS.md「平台注记（Windows）：独立工作树路径放短目录」——补强为「NTFS 路径，避开 exFAT 无所有权语义」
---

# worktree 落 exFAT 盘 dubious ownership

## 症状
`git worktree add "F:/Craft/conver system/worktrees/t1-cg-weight" ...` 把 worktree 建在了 exFAT 分区。后续在该 worktree 内跑任何 git 命令都报：
```
fatal: detected dubious ownership in repository at '.../worktrees/t1-cg-weight'
```

## 根因
exFAT 文件系统不记录文件所有权（owner），git 的 `safe.directory` 安全检查无法确认仓库归当前用户所有，于是拒绝一切操作。主仓库 `desktop/` 在 NTFS 分区正常，但 `F:/Craft/conver system/` 下的 worktree 目录落在 exFAT（同一盘符下 NTFS/exFAT 分区混杂）。

## 代价
T1 实现子智能体在该 worktree 内烧掉多次重试（git status/log/diff 全部被拒），16 分钟的中断现场几乎无法通过 git 诊断，直到手动加 safe.directory 例外才恢复。

## 教训
Windows 下 worktree 路径必须落在 NTFS 分区；exFAT/FAT32 不记所有权会让 git 全线拒操作。项目已形成惯例的 `desktop/.worktrees/<slug>`（NTFS）路径才是正确选择。

## 防复发
- [x] 本会话已落实：后续 worktree 全部改用 `desktop/.worktrees/`（NTFS）；对已存在的 exFAT worktree 加了 `git config --global --add safe.directory <路径>` 例外
- [ ] 新项目建 worktree 前先确认目标盘文件系统（`fsutil fsinfo volumeinfo <盘符>` 或直接看项目既有 worktree 惯例）

## 关联
- [[worktree并行前查CRLF祖父blob]]
- [[git-ignored运行时资产在worktree下与主工作树分离]]
