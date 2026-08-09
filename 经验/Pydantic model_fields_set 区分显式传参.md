---
type: lesson
tags: [Pydantic, 回退链, 配置, 显式传参, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
summary: 回退链判断「是否显式传参」用 Pydantic v2 model_fields_set，不用值比较——显式传默认值与没传必须可区分
provenance: git 4f686a5（2026-08-08 知识库批量入库）· Conver System 2026-08-06 实践
status: verified
---

# Pydantic v2 `model_fields_set`——区分「显式传参」与「未传」

## 症状
用户显式选择 `default_provider=openai`（恰好等于默认值）时被静默覆盖回默认值——显式选择与「没选」无法区分。

## 根因
靠「值是否等于默认值」判断是否覆盖，无法区分「显式传了默认值」和「根本没传」。值比较丢失了「这个值从哪来」的信息。

## 教训
回退链（显式传参 → settings → config）判断字段是否被显式传入，用 Pydantic v2 的 `model_fields_set`，**别用值比较**。凡是有默认值的回退链参数，判定依据是「是否在请求中出现」，不是「是否等于默认值」。

## 代价
回退链重构 + 边界 bug 修复（`conversation.py:81-93`，`fa1f411`）。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] 回退链已改用 `model_fields_set`
- 新回退链/默认值逻辑一律以「显式性」而非「值」为判定依据

## 关联
- [[API 凭证跨协议兜底设计]]（同为配置解析链设计）
- [[连接测试必须测用户实际配置]]（同为「用户显式配置不被吞」族）
