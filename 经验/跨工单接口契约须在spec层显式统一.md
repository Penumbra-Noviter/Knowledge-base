---
type: lesson
tags: [agent编排, 契约, 跨工单, plan-tickets, conver-system]
date: 2026-09-14
project: Conver System
source: 自身项目实践
summary: 跨工单接口契约（service 函数签名参数 vs 端点 URL/body 参数）必须在 spec 层显式统一并交叉核对——签名需 conversation_id 而端点无此参数，实现时被迫加解析 seam 成唯一范围偏差
provenance: DEV_LOG〈消息编辑重发 + 删除单条消息批次（2026-09-14）〉· commit 3bcd80f · 2026-09-14 会话
status: verified
agents_md_feedback: 印证 AGENTS.md §二 2「验收标准为语义契约」——跨工单契约是模块边界上的真实 seam，语义契约须在 spec 层对齐而非各票自洽
---

# 跨工单接口契约须在 spec 层显式统一——service 签名 vs 端点参数

## 症状
消息编辑重发批次：工单 01 锁定的 service 函数 `edit_and_resend(db, conversation_id, message_id, content)` 需要 `conversation_id` 参数，但工单 02 的端点契约为 `PUT /api/messages/{message_id}`（URL 与 body 均无 conversation_id，前端 `messages.edit(messageId, content)` 同证）。两票各自按 spec 的不同部分实现，各自单测全绿。实现到端点票才发现：路由要委托 service，必须先由 `message_id` 反查 `conversation_id`，而 message.py 没有公开的「message_id → conversation_id」解析函数、路由又要求不碰 ORM——被迫在 message.py 加 11 行公开别名 `require_message`（`_require_message` 的转发）来补解析 seam。这是本批次唯一的「范围偏差」（触碰了标「只读」的共享文件）。

## 根因
spec 跨工单不自洽：service 签名与端点契约的参数不一致，且没有任何工单声明「解析 conversation_id 的归属」。plan-tickets 拆票时没有把 service 层签名与端点 URL/body 参数逐参数交叉核对；波内各票文件范围互不相交，每个 Implement 只在自己文件范围内自洽，接口参数错位在对接时才暴露。

## 代价
实现阶段才发现（非 spec 阶段），加 `require_message` 空心别名 + 三查冗余（路由 require_message → `_resolve_edit_target` 再查 → `update_message` 三查同一消息）。期末四轴 Standards/Architecture 双 MEDIUM（零行为透传别名 Middle Man + 目标解析知识散布），落债 F-130。若在 spec 拆票阶段就对齐，这本可避免。

## 教训
跨工单传递的接口契约（函数签名参数 vs 端点 URL/body 参数）与字段命名契约一样，是**模块边界上的真实 seam**：必须在 spec 层显式统一（声明「端点经 require_message 解析归属后委托 service」），不能「service 票跟随函数签名、端点票跟随 URL 契约」两头飘。参数错位不像字段命名那样静默丢字段，但会逼出「空心解析别名」这类补丁结构。

## 防复发
- [ ] 已落实（本批次事后接受）：`require_message` 公开别名 + 三查冗余已落债 F-130。
- plan-tickets 拆票前，对「跨 service/route 两层的功能票」做前置断言：git grep 实证 service 签名参数与端点契约参数一致（对齐 persona 既有「plan-tickets 表述与 ORM/schema 现状可能不符——拆票前置断言须 git grep 实证」）。

## 关联
- [[跨工单字段契约须在数据契约层统一并锚定]]（同根因族：spec 跨工单契约不统一的另一形态——字段命名）
- [[中间转换层转圈丢字段]]（接口/数据契约静默丢失的相邻形态）
