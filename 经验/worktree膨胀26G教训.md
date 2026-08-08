---
type: lesson
tags: [Tauri, Rust, 桌面应用, 磁盘空间, subagent, personal-dashboard]
date: 2026-08-04
project: 川流不息
source: 自身项目实践
status: verified
---

# .claude/worktrees/ 膨胀 26G 教训

## 症状

`npm run tauri:dev` 编译极慢，检查发现项目目录高达 **30 GB**。其中 `.claude/worktrees/` 占了 **26 GB**，而实际代码仅 ~200 MB。

## 根因

每个并行 subagent（`Agent` tool + `isolation: "worktree"`）都会创建一个独立 git worktree。每个 worktree 包含完整的 `src-tauri/target/` 目录（Rust 编译缓存），单个约 1.6~3.1 GB。前后启动了 12 个 agent，累积了 12 份编译缓存，没有自动清理。

## 代价

- 磁盘空间从 3.9 GB 膨胀到 30 GB（7.7 倍）
- 手动定位 + 删除耗时约 5 分钟
- `rm -rf .claude/worktrees/` 清理后恢复 3.9 GB

## 教训

**worktree 用完即弃，不留 Rust 编译缓存尸体。** 确认合并后立即清理 worktree 目录，尤其是 Tauri/Rust 项目——每个 worktree 的 `target/` 是主项目的数倍。

## 防复发

- [ ] 已完成合并的 worktree 目录立即删除（`rm -rf .claude/worktrees/<name>`）
- [ ] 可在 `.gitignore` 中确认 `.claude/worktrees/` 已排除（避免被 git 追踪）
- [ ] 批量删除所有已合并 worktree：`rm -rf .claude/worktrees/`
- [ ] 若 disk pressure 敏感，考虑每批次 agent 完成后立即清理，而非等全部结束

## 关联

- [[并行 worktree 合并静默覆盖]]（并行 worktree 的另一面：清理 vs 合并）