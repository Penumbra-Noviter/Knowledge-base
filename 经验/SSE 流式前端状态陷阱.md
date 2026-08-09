---
type: lesson
tags: [前端, 异步, SSE, 流式, 状态管理, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
summary: ReadableStream reader.read() 必须处理 done 结束分支（触发 onDone 重置状态），controller.close() 要显式调用——否则流式 UI 状态悬挂
provenance: git 4f686a5（2026-08-08 知识库批量入库）· Conver System 2026-08-06 实践
status: verified
---

# SSE 流式前端状态陷阱——异步读取结束条件不完整导致状态悬挂

## 症状
SSE 流式对话中，AI 生成完成后发送按钮永久保持「⏹ 停止」状态，不再恢复为「➤ 发送」。用户卡死，无法发送新消息。

## 根因
`reader.read()` 返回 `{done: true}` 时前端代码直接 `break`，从未触发 `onDone` 回调，`isStreaming` 标志和 `btnSend.disabled` 状态永不重置。`{done: true}` 是 ReadableStream 的正常结束信号，但代码只处理了数据帧分支，漏掉了结束分支。

## 代价
1 个 Bug 修复 + 1 次测试补全（Playwright 手工触发结束流验证）。线上用户若遇到此问题会困惑并刷新页面，丢失未保存的上下文。

## 教训
**ReadableStream 的 `reader.read()` 永远要处理两个分支**：`{done, value}` 中 `done` 不是异常，是正常结束信号。正确的模式是：

```js
while (true) {
  const { done, value } = await reader.read();
  if (done) { onDone?.(); break; }
  // 处理 value...
}
```

另外，SSE 流的 `controller.close()` 必须显式调用，否则 `reader.read()` 永不返回 `done`，正常完成路径无法验证。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] 前端 `chatStream()` 已修复：`done` 分支触发 `onDone`
- [x] 新增 `completed` 标记，流结束未收 `done` 时兜底重置状态
- ✅ 检查项已固化于 [[项目预检模板]] §三（跨项目，开工对照执行）

## 关联
- [[前端模块化拆分中的循环依赖处理]]