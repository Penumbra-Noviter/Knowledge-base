---
type: lesson
tags: [架构, Provider, 标识符, 演进, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
status: verified
---

# Provider 标识符 key/id 分离——多兼容 Provider 系统的标识符演进

## 症状
当接入 8 个 Provider（Claude、OpenAI、DeepSeek、Qwen、Kimi、智谱、豆包、Ollama）时，前端下拉框用 `data-index` 匹配 Provider 位置的逻辑彻底失效。新增 Provider 需要重排索引，且第三方 OpenAI 兼容 Provider 无法通过索引区分。

## 根因
系统最初只支持 2 个 Provider（Claude + OpenAI），用 `data-index`（数字索引）定位 Provider 是够用的。但当规模增长到 8+ 且引入"第三方 OpenAI 兼容"这类非唯一实体时，位置索引不再能唯一标识一个 Provider。

## 代价
重构一次：`data-index` 匹配逻辑全部退役，前端下拉 `value` 改用 `key`；后端给每个 Provider 分配唯一 `key`（标识符）和 `id`（API 协议标识符，如 claude→`claude-3-opus-20240229`）。涉及前端 3 个文件、后端 factory + setting 映射链。

## 教训
**标识符选取要预判规模曲线**：2 个 Provider 时 `data-index` 够用，8 个时就得重构。更通用的原则是——**前端下拉框的 value 永远用语义标识符（key/slug），不用位置索引（index）**。位置索引在增删改时会静默错位，语义标识符不变。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] 所有 Provider 下拉 `value` 使用 `key`（语义唯一标识符）
- [x] 新增 Provider 只需在 `_PROVIDER_API_MAP` 注册，前端自动识别
- 其他项目的前端下拉框也遵循此原则

## 关联
- [[API 凭证跨协议兜底设计]]