---
type: lesson
tags: [kickoff, 子智能体, 网关]
date: 2026-09-14
project: Conver System
source: 自身项目实践
summary: 本地网关并发上限（user concurrency limit exceeded/captcha/Model request failed）时并行派发子智能体连撞——降为串行 lane（同时只跑 1 个 Implement）稳定零重开
provenance: DEV_LOG mod-cg-wiring 批次避坑 1 · commit 0fb43ba · 2026-09-14 Conver 会话
status: verified
agents_md_feedback: 印证 kickoff skill「单波并行上限 + 空返回辨层级（无 usage = 网关层）」——网关层失败不计入工单 BLOCKED，策略性降并行
---

# 网关并发上限降串行 lane

## 症状
kickoff 全自动档首波并行派发 3 个 Implement 子智能体，连撞 `user concurrency limit exceeded`、`captcha verify failed`、`Model request failed`（运行 16 分钟中途断）。T1 重开 2 次仍触顶，T3 也失败。

## 根因
本地网关有并发上限（本环境约 2 个并发即触顶）。同一条消息里 `run_in_background` 启动 3 个实现子智能体，超过网关允许的并行数，后续派发/重开全部被拒。与 prompt 长度无关（有 usage 无内容的模型行为才改 prompt；这里是无 usage 的网关层异常）。

## 代价
首波 3 票中 T1 失败 3 次、T3 失败 1 次，反复重开烧掉约 20 分钟 + 数万 token，最终靠降并行才恢复。

## 教训
多子智能体并行派发前，先探路网关健康（首波发 2-3 个探路者）；一旦出现无 usage 的网关层异常（concurrency limit / captcha / auth），立即降为串行 lane——同一时间只跑 1 个实现子智能体，网关层失败不计入工单 BLOCKED 计数，也不走「同模型重试」升级路径。

## 防复发
- [x] 本会话已落实：mod-cg-wiring 与 arch-f123-126 两批次均改串行 lane，零重开
- [ ] 网关层失败（无 usage）与任务失败（有 usage 无内容）分层处置，前者查网关/降并行、后者改 prompt

## 关联
- [[worktree半成品现场续用重开模式]]
