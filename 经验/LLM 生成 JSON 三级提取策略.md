---
type: lesson
tags: [LLM, JSON, 解析, 容错, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
status: verified
---

# LLM 生成 JSON 的三级提取策略 + 白名单字段过滤

## 症状
LLM 返回的解析结果偶尔包在 ```json 代码块里或带前后缀文字，直接 `json.loads` 失败；还会返回白名单外的字段。

## 根因
LLM 输出格式不稳定——即使提示「只返回 JSON」也不保证纯 JSON；字段集合必须由系统侧约束，不能靠模型自觉。

## 教训
对 LLM 结构化输出做**三级兜底提取 + 白名单过滤 + 类型容错**：
1. 直接 `json.loads`
2. 提取 ```json 代码块内容
3. 花括号范围截取再解析

解析结果过 `_PARSED_FIELDS`（frozenset）白名单过滤，逐字段校验类型后再落库。字段名/类型约束在系统侧，容错在解析侧，缺一不可。

## 代价
三级提取实现 + `test_brace_extraction` 等测试（`document_parser.py:150-180`）。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] `document_parser.py` 已实现三级提取 + 白名单
- 新的 LLM 结构化输出场景直接复用该模式（parse → filter → validate → persist）

## 关联
- [[连接测试必须测用户实际配置]]（同为「模型输出/配置不可信，系统侧兜底」族）
- [[response_model 统一驱动序列化]]（同为「字段清单单点化」思路）
