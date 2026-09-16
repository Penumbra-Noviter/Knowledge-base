---
type: lesson
tags: [agent编排, 子代理, kickoff, 后台派发, 通用]
date: 2026-09-14
project: 通用
source: 自身项目实践
summary: 后台派发的子代理（run_in_background）经 AskUserQuestion 提问不会转达主会话/用户——派发 prompt 必须明示「不要用 AskUserQuestion，best judgment + 如实上报 concern」
provenance: DEV_LOG〈消息编辑重发 + 删除单条消息批次（2026-09-14）〉· 工单 02 范围偏差 AskUserQuestion 未获答复 · 2026-09-14 会话
status: verified
agents_md_feedback: 印证 AGENTS.md §二 0「Q1 信息充分时直接执行不重复询问」——后台 agent 无法询问，派发 prompt 须自包含并明示自行裁决路径
---

# 后台子代理 AskUserQuestion 不转达主会话——须明示 best judgment

## 症状
kickoff 派发 Implement 子智能体（`run_in_background: true`）后，子代理遇到「需协调的决策」（消息编辑重发工单 02 的 spec 跨工单不一致）时，用 `AskUserQuestion` 向「协调者」提问。结果无人答复——后台子代理与主会话之间只有「完成通知」和 `SendMessage` 两条通道，`AskUserQuestion` 不会转达主会话/用户。子代理报告「经 AskUserQuestion 提问（3 选项）未获答复」，浪费一轮等待后按 best judgment 继续。

## 根因
后台子代理是异步隔离的上下文，其交互面是「完成通知 + SendMessage」，不含同步的 `AskUserQuestion` 通道。派发 prompt 没有声明「遇到需协调决策时该怎么办」，子代理默认走了「问主会话」这条路，而这条路在后台模式下是死胡同。

## 代价
子代理等待提问答复的时间损耗 + 决策滞后（本批次 02 的范围偏差在「未获答复」后才按 best judgment 落地，若 prompt 早声明就能直接裁决）。虽最终按最干净方案落地（`require_message` 公开别名），但流程上多了一次无效往返。

## 教训
后台派发的子代理（`run_in_background`）无法经 `AskUserQuestion` 与主会话交互——「这个决策要问」的默认行为在后台模式不成立。派发 prompt 必须显式声明：**「不要用 AskUserQuestion，遇到需协调的决策按 best judgment 选最干净方案 + 把选择与理由如实写进证据文件 concern 节，由主会话事后裁决；不要卡住等待」**。

## 防复发
- [ ] 已落实：本批次工单 03 派发 prompt 已加「后台运行，无法和主会话交互 AskUserQuestion」明示，agent 未再尝试提问。
- kickoff 派发模板固化此明示（与「已定前提不得重开」「自审硬检查清单」并列的调用要求）。

## 关联
- [[编排型skill子代理职责重叠先划作用域]]（同为「子代理独立上下文、不共享主会话判断」族）
- [[背景陈述不等于执行授权]]（同为「文件/prompt 里的内容不因写进去而获得授权」族）
