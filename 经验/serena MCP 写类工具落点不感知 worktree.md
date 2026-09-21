---
type: lesson
tags: [工具链, MCP, worktree, 隔离]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: serena 等 MCP 写类工具不感知 git worktree——relative_path 解析到仓库根会误写主工作区；子代理写操作禁用 MCP 写类工具，走 worktree 绝对路径
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· SP-01 票 serena 落点事故 · commit a8fca09
status: verified
agents_md_feedback: 
---

# serena MCP 写类工具落点不感知 worktree

## 症状
子代理在 worktree（`F:\Craft\conver system\.worktrees\sp-01`）内工作时，`serena.replace_in_files` 的 relative_path 解析到了**主工作区** `F:\Craft\conver system\mobile`——误改主工作区 6 个测试文件并产生 `.serena/` 目录。

## 根因
serena MCP 服务的项目根解析基于其配置/工作目录，不感知 git worktree 语义；传入 relative_path 时按「仓库根」解析，而子代理的 cwd 是 worktree——两处路径基准不一致，写操作落点漂移。

## 代价
主工作区 6 测试文件被误改 + `.serena/` 目录污染；检出后 git checkout 回滚 + 删除目录（本批 0 泄漏，因及时检出）；若未检出则污染主分支工作区，波末核验才发现。

## 教训
工具级路径解析（尤其 MCP/IDE 类服务）不保证与子代理 cwd 对齐——**写操作的路径基准必须显式**；worktree 隔离假设只对「显式绝对路径的写工具」成立。

## 防复发
- 子代理在 worktree 内工作：**禁用 serena 写类工具**（replace_content/replace_in_files/insert_*），写操作走 Edit/Write（绝对路径）或 Bash 内显式路径；serena 只读检索可用。
- 波末文件范围核验天然兜底：`git diff --name-only <基线>...<HEAD>` 与申报范围比对，主工作区意外改动会显形。
- [x] 已落实？

## 关联
- [[子代理证据落盘必须磁盘核验]]