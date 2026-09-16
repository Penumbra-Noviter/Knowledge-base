---
type: lesson
tags: [git, merge, CODE_WIKI, Conver System]
date: 2026-09-14
project: Conver System
source: 自身项目实践
summary: merge 冲突时 `git checkout --ours` 会整体丢弃他方对同一文件的非冲突区增补（如 CODE_WIKI §3/§4/§5）——用 git diff 提取对方 diff 手工重放，再 doc_sync 刷新
provenance: DEV_LOG mod-cg-wiring 批次避坑 4 · commit 35ebe1d · 2026-09-14 Conver 会话
status: verified
agents_md_feedback: 印证 persona「merge 冲突全取会丢对方修改」——第三次实证（C3/C4/C8、mod-cg-wiring、arch-f123-126）
---

# merge 冲突 checkout --ours 丢对方增补

## 症状
多分支改同一文档（CODE_WIKI.md）时，`git merge` 报 CONFLICT。用 `git checkout --ours CODE_WIKI.md` 取自己一侧解决，结果他方分支对 CODE_WIKI 的 §3 目录树行、§4 新模块节、§5 测试文件行**整体消失**，直到 doc_sync --check 报「测试文件未出现在文档引用中」才发现。

## 根因
`checkout --ours` 是「整个文件取我方版本」，不是「只解决冲突行」。他方分支对同一文件**非冲突区**的增补（新模块节、新测试行）被一并丢弃——git 只把冲突行标记出来，其余区域的「我方旧版 vs 他方新版」差异由 --ours 粗暴抹平。这比「零冲突静默覆盖」更隐蔽：冲突提示让人以为「解决完就安全」，实际是手动全取一侧。

## 代价
T4 合并时丢失 §4.36.7 mod-css.js 节 + §5 mod-css.test.js 行，靠 doc_sync --check 报「源文件未出现在文档引用」才反向补回，一轮返工。

## 教训
merge 冲突解决时，**禁止 checkout --ours/--theirs 全取一侧**（除非确认对方对同文件零实质增补）。正确做法：`git diff <base> <their-branch> -- <file>` 提取对方 diff，把实质增补（新节/新行）手工重放到当前版本，再跑文档同步工具刷新机械标记。AUTO 计数类冲突可任取一侧（合并完由 doc_sync 统一刷新）。

## 防复发
- [x] 本会话已落实：后续 CODE_WIKI 冲突均走「提取对方 diff 手工重放 + doc_sync --update 刷新」
- [ ] 有机械标记文档（CODE_WIKI/doc_sync 类）的项目，合并冲突解决后必跑文档同步 --check 兜底

## 关联
- [[并行 worktree 合并静默覆盖]]
- [[merge失败工作区残留部分合并产物]]
