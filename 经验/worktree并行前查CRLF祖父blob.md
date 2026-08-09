---
type: lesson
tags: [worktree, git, 行尾, subagent, 并行]
date: 2026-08-09
project: Model Fingerprint
source: 自身项目实践
summary: worktree 并行分发前必查 CRLF 祖父 blob——.gitattributes 声明 ≠ blob 事实，全新 checkout 会把整文件暴露为噪声
provenance: DEV_LOG「T-412」条目 · commit f6adc68/c9f3f06 · 2026-08-09 批次 2~4 会话
status: verified
---

# worktree 并行前查 CRLF 祖父 blob

## 症状
批次 2 三个并行 worktree 代理各自报告 `fingerprint/client.py`、`fingerprint/similarity.py`、`tests/test_client.py` 等 5 个文件为 modified——`git diff --ignore-space-at-eol` 为空（纯行尾伪差异），但代理误判为脏树、需手工排除，每个代理都踩一遍；主仓库工作区却显示干净（stat cache 掩盖）。

## 根因
5 个文件的 **blob 以 CRLF 入库**（`i/crlf`），而 `.gitattributes` 声明 `*.py text eol=lf`——属性文件只约束未来写入，不回溯历史 blob。主仓库靠 stat cache（mtime）掩盖差异；任何**全新 checkout**（worktree 即是）会重算内容，把整文件暴露为 modified。合并刷新 index 后主仓库也会暴露（test_logprobs.py 即此例）。

## 代价
批次 2 三个代理各花一轮排查噪声；一个代理为保住 CRLF blob 用 `hash-object --no-filters` 特技提交；最终 T-412 用 `git add --renormalize` 一次性修复（+952/-951 全为行尾，内容零差异）。

## 教训
**启动 worktree 并行分发前先跑 `git ls-files --eol "*.py" | grep crlf`**——有 `i/crlf` 就先 `git add --renormalize` 修掉再开波，否则每个新 checkout 都会暴露整文件噪声、污染每个代理的视野。

## 防复发
- [x] 已落实？分发前 EOL 预检写进 kickoff 分发计划检查清单（T-412 修复后本仓 `i/crlf` 为零，未来提交若再犯预检即红）

## 关联
- [[worktree膨胀26G教训]]（worktree 家族第二坑：前者是缓存体积，本条是行尾噪声）
