---
type: lesson
tags: [OpenAI, SDK, base_url, 规范化, 聚合平台, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
status: verified
---

# OpenAI 兼容 API 的 base_url 需规范化——SDK 拼接语义 vs 用户习惯

## 症状
用户只填聚合平台根地址（`https://api.kukuit.com`），openai SDK 自动拼 `/chat/completions` 打到 HTML 面板页面而非 API 端点。

## 根因
OpenAI SDK 的 `base_url` 语义是「服务根」——自行追加路径段（如 `/chat/completions`），而用户习惯填控制台根地址，不含 `/v1` 版本段。SDK 的拼接语义与用户心智模型不一致。

## 教训
接入 OpenAI 兼容 API 前先想清 SDK 对 `base_url` 的**拼接语义**，对用户输入做规范化：自动补 `/v1`，已含 `v1`/`v1beta` 不误改。凭证与地址的「兜底链」设计见 [[API 凭证跨协议兜底设计]]——本条的增量是**地址规范化**：用户输入是「控制台地址」，程序消费的是「SDK 地址」，中间必须有一层转换。

## 代价
一次通用化重构 + 5 例 normalize 单测（`openai.py:21-34`，`a83ef91`）。

## 防复发
- [x] base_url 规范化已实现并测试
- [ ] 新接入 SDK 时先查它的 base_url 拼接语义再写输入处理

## 关联
- [[API 凭证跨协议兜底设计]]
- [[连接测试必须测用户实际配置]]
