---
type: lesson
tags: [架构, 凭证管理, 回退链, 聚合平台, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
status: verified
---

# API 凭证跨协议兜底设计——聚合平台场景下的凭证共享

## 症状
用户使用聚合 API 平台（如 kukuit、one-api）时，一个 API Key 和 Base URL 可以同时服务 OpenAI 协议、DeepSeek、Qwen、Kimi 等多个模型。但系统要求每个 Provider 单独填 key/url，用户被迫重复填写相同信息，体验极差。

## 根因
系统的凭证模型是"每个 Provider 独立存储"的，没有考虑到"一个 key 通吃多个同协议 Provider"的聚合平台场景。存储键用 Provider 名限定（如 `deepseek_api_key`），导致用户需要为每个 Provider 填相同值。

## 代价
一次设计 + 实现（~2 天）：`setting.py` 重写解析链为三级回退——「Provider 特定键 → 同协议槽位 → 跨协议兜底」；`api_key` 再叠 `.env` 兜底。前端设置面板加通用提示文案。8 项新测试覆盖全部回退路径。

## 教训
**当系统支持多个同协议 Provider 时，凭证解析必须设计为"共享优先"模式**，而非"每个 Provider 独立"模式。三级回退链（Provider 特定 → 同协议槽位 → 跨协议兜底）是通用解法：

```
provider 特定键 → 同协议槽位（如 openai_api_key） → 另一协议槽位（如 claude_api_key）
```

这样用户只需填一个 key，所有同协议 Provider 自动共享；高级用户仍可单独覆盖。

## 防复发
- [x] 系统已实现三级回退链 + 8 项测试
- [x] `base_url` 同样走此回退链（自动补 `/v1` 规范化）
- [ ] 新项目设计凭证系统时直接采用共享优先模型

## 关联
- [[Provider 标识符 keyid 分离]]