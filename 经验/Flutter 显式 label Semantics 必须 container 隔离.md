---
type: lesson
tags: [Flutter, 语义, 无障碍]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: 气泡内组件带显式 label 的 Semantics 必须 container:true 隔离——无 container 时配置合并进兄弟语义节点，吞掉同气泡「回复中断」label（F-66 实证）
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· MS-05 票 F-66 语义吞并实证 · commit 7866ce6
status: verified
agents_md_feedback: 
---

# 气泡内显式 label Semantics 必须 container:true 隔离

## 症状
聊天气泡内消息操作按钮带显式 Semantics label，无障碍语义树中该 label 被吞并——同气泡「回复中断」label 不出现在语义数据中；对照实验（关按钮）后语义树恢复。

## 根因
Flutter `Semantics` 节点默认是「合并容器」：父容器（气泡）没有 `container: true` 时，子节点的显式 label/配置会向上合并进兄弟语义节点，多个显式 label 互相覆盖，后写的吞掉先写的。

## 代价
读屏用户漏读气泡操作按钮/状态 label；测试断言（a11y 语义树）需改实现才能过——若测试没断言到位，缺陷静默进入无障碍路径。

## 教训
「显式 label 的语义节点」与「纯语义孩子节点」不同：显式 label 必须自建隔离语义边界（`container: true`），否则合并语义会吞 label——这是 Flutter 语义树的通用陷阱，不限于气泡场景（一切「一个视觉容器内多个带 label 的交互组件」都适用）。

## 防复发
- 气泡/容器内带显式 label 的组件：`Semantics(container: true, ...)` 或等效隔离。
- 带显式 label 的语义测试必须断言**同容器多 label 并存**场景（对照实验：关掉一个组件验证另一个 label 仍存在）。
- [x] 已落实？

## 关联
- [[]]