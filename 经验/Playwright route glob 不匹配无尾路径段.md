---
type: lesson
tags: [测试, 冒烟, playwright, conver-system]
date: 2026-08-13
project: Conver System
source: 自身项目实践
summary: Playwright route 拦截 glob 中 `**/api/chats/**` 不匹配无尾路径段的精确 URL（`/api/chats`）——mock 拦截必须同时注册精确路径与带尾段 glob，否则真实请求漏拦（触发真实外部 API）
provenance: DEV_LOG G18「td-arch-health 批次」· merge 48447e6（2026-08-13）
status: verified
---

# Playwright route 拦截 glob：`**/path/**` 不匹配无尾路径段

## 症状

GUI 冒烟用 `page.route('**/api/chats/**', ...)` 拦截聊天发送请求。流式端点 `POST /api/chats/stream` 被拦截 ✓，但**非流式端点 `POST /api/chats`（无尾路径段）真实发出**——后端有真实 API key，触发了一次真实外部 LLM 调用（api.kukuit.com），并往用户对话写入 2 条真实测试消息。

## 根因

Playwright route glob 中 `**/api/chats/**` 的尾 `/**` 要求**至少一个路径段**——`/api/chats` 本身不匹配。同批注册的 `**/api/chats/stream` 只覆盖流式端点；非流式 `POST /api/chats`（api.js `chat: (data) => request('POST', '/chats', data)`）成为漏网。

## 代价

1 次真实外部 API 调用（冒烟纪律「不触发真实外部 API」被违反）+ 用户数据库污染 2 条消息（14→16 条，需用户决定清理）。若被拦截的是带副作用端点（删除/写入），后果更严重。

## 教训

**mock 拦截要覆盖端点的全部形态**：同一资源既有带尾段的端点（`/chats/stream`）又有无尾段的（`/chats`）时，注册 `**/api/chats/**` + `**/api/chats` 两条（或直接匹配具体路径）。注册后**先验证拦截生效**（发一条请求看 network 面板是否命中 mock 而非真实后端）再进入正式测试。

## 防复发（惯例：冒烟环境准备时对照执行）

- [x] td-arch-health 批次：补注册 `**/api/chats` 精确路径后非流式失败路径（system 气泡）验证通过
- 后续冒烟：route 拦截清单按端点表逐一核对（含无尾路径段形态），注册后先发探针验证命中

## 关联

- [[连接测试必须测用户实际配置]]
- [[Tauri 壳方案打包验证铁律]]
