---
type: lesson
tags: [ZCode, 子智能体, provider, 视觉, new-api]
date: 2026-08-19
project: ZCode 环境
source: 自身项目实践
summary: 无视觉主模型经子智能体看图时模型编造画面，多为图像未送达——先查 provider 的 kind/baseURL 与中继兼容性
provenance: 2026-08-19 ZCode 环境会话（View 子智能体视觉链路修复：kind anthropic→openai-compatible、baseURL 补 /v1）
status: verified
---

# 子智能体看图"编造画面"：先查 provider 图像序列化兼容性

## 症状
View 子智能体（kuku 中转 qwen3.7-plus）描述图片时凭空编造：合成图（白底黑字 QC-8731）被说成"卡通小动物、图片上没有任何文字"，还编造不存在的颜色；真实照片被说成"卡通小鸡"。请求日志无报错、运行正常完成。

## 根因
链路断在 harness → 中继这一段，两层问题：
1. provider `kind: anthropic` 时，harness 把图像序列化成自定义格式块（顶层 `mediaType`/`dataUrl`），new-api 中继不识别——直接发该格式到 `/v1/messages` 中继 panic；实际请求图像被静默丢弃，模型只收到"查看图片 X"的文字提示 → 凭提示词编造画面。
2. 改 `kind: openai-compatible` 后，baseURL 缺 `/v1`——`{baseURL}/chat/completions` 打到中继 Web 管理面板 HTML（HTTP 200），被当 SSE 解析成空流（`suspicious_empty`，finishReason: other）。

## 代价
先误判为模型幻觉（差点换模型），再排查配置缓存，两次重启验证，约 1 小时。

## 教训
模型"能读图" ≠ 链路能读图。子智能体视觉异常按三段定位：①模型能力（直连 API 用标准格式测）→ ②请求序列化（provider kind 决定图像格式）→ ③地址拼接（baseURL 语义）。**用户直连 API 正常是强信号：问题在 harness→中继段**，不是模型。

## 防复发
- [x] kuku provider 固定 `kind: openai-compatible` + baseURL 带 `/v1`（与 [[OpenAI base_url 拼接语义]] 同源：SDK/框架按服务根自动追加路径段）
- [x] 改 v2/config.json 后重启应用才生效（内存缓存）
- [x] 配置变更后用不可预测标记图复测（见 [[视觉能力证伪测试用不可预测标记图]]）

## 关联
- [[OpenAI base_url 拼接语义]]
- [[视觉能力证伪测试用不可预测标记图]]
