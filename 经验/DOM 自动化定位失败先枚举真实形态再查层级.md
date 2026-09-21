---
type: lesson
tags: [dom自动化, 定位, 探针]
date: 2026-09-21
project: AI自动获客
source: 自身项目实践
summary: DOM定位失败「先枚举元素形态再查层级」——占位容器点击激活形态致五条选择器全灭，iframe/shadow 假设反而证伪
provenance: DEV_LOG 2026-09-20 P3轮2+轮3 · commit 825dcea · AI自动获客会话
status: verified
agents_md_feedback: 印证 AGENTS.md 对抗性自检「证伪优先」——错误假设（iframe/closed shadow）先被探针证伪，真实根因（属性形态）后现
---

# DOM 自动化定位失败：先枚举真实形态，再查层级（iframe/shadow）

## 症状
抖音 PC 端评论区输入框**视觉存在**（用户截图确认、占位文字「留下你的精彩评论吧」可见），但自动化五条输入选择器（`[contenteditable]` / `textarea` / `[role=textbox]` / `[class*=DraftEditor]` / `[data-e2e*=comment-input]`）**全灭**，`located.input=false`，两条发送指令均 `not_sent`。连续两轮真机补验未定位。

## 根因
真机评论输入框是**「占位容器需点击激活」形态**：未点击时是普通 `div[class*=comment-input-inner-container]`，占位文字是 div **直接渲染的文本节点**（非 contenteditable、非 placeholder 属性）；**点击后才进入编辑态**——挂出 `public-DraftEditor-content[contenteditable=true]`。未激活时页面 `ceAny:0 / textarea:0 / shadowHosts:0`。
而轮 2 的推理被「视觉可见但 document 查不到」误导，按概率排了 **iframe / closed shadow root** 两个「结构层」假设——探针 v2 多 frame 遍历 + closed shadow 嫌疑检测把两者都**证伪**（iframe 全排除、shadowHosts=0、closed shadow 判据 8 个全是页面正常空壳 div 误报）。
**真实根因在「元素属性形态」，不在「元素所在层级」**——占位容器一直在主文档里，只是未激活时没有 contenteditable 等可写标记。

## 代价
- 轮 2 整个方向（多 frame 遍历 + closed shadow 检测）建在错误假设上，约 1 轮真机补验 + 6 个探针测试白做（探针本身仍有诊断价值，但未指向根因）；
- 失误诱因：合成 fixture（happy-dom）里输入框是 `div[contenteditable=true]` 直接形态，与真实结构不同形——**fixture 编码的是对格式的想象**（AGENTS.md 对抗性自检原文：fixture 会掩盖真实分布形态）。

## 教训
**DOM 定位失败时，先「枚举 + 属性原值 dump」真实形态，再「按层级」猜（iframe / shadow / 新层）**。选择器查询返回 0 不等于元素不在——第一刀应是无预设的候选枚举（`[contenteditable]` 任意值含 plaintext-only / 空值、占位文本即 textContent 的容器、所有 `[class*=input]`），并记录每个候选的属性原值（ce 原值 / class / rect / text），而不是增量扩选择器。

## 防复发
- 探针只 dump、不预设：`buildCommentProbeScript` 输出**候选元素全量枚举 + 属性原值**（ce 属性值含 `null`/`inherit`/空值等真实形态），右上角留真机验证路径（`.scratch` 探针 dump 归档）；
- 合成 fixture 必须按**真机 dump 的结构**重建（占位容器 + 点击后挂 contenteditable），不只改选择器字符串；
- 占位容器「点击激活」形态的通用激活协议（mousedown/mouseup/click + focus → 轮询编辑态）已入 `buildCommentSendScript`，同类富文本输入框复用。
- [x] 已落实（commit 825dcea：探针 v2 多 frame + 激活链路 + fixture 按真实结构重建）

## 关联
- [[Falsify测试要钉住缺陷所在层]]
- [[React 受控输入写入必须补 input 事件——事件在页面与测试沙箱的双重不可见]]