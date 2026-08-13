---
type: lesson
tags: [安全, 前端, XSS, 转义, conver-system]
date: 2026-08-13
project: Conver System
source: 自身项目实践
summary: escapeHtml 类「先转义再拼串」不覆盖属性上下文——双引号在属性值里击穿边界（截断 + 属性注入面）；数据进属性须用 DOM dataset 赋值（数据通道单一化）而非 HTML 字符串拼接
provenance: DEV_LOG G18「td-arch-health 批次」· merge 9716c30（2026-08-13）
status: verified
---

# HTML 属性上下文转义：数据通道单一化

## 症状

FE-1 气泡工厂 `messageBubbleHtml` 把消息内容经 `escapeHtml(content)` 后拼进 `data-content="${...}"` 属性：内容含 `"` 时 dataset.content 在首个引号处**截断**（复制功能数据损坏），且同一通道可注入任意 HTML 属性（`hi" onclick="alert(2)` → onclick 属性成立，属性注入面 XSS）。`<script>` 等标签注入已被转义挡住，但属性注入是第二向量。

## 根因

`escapeHtml` 基于 `textContent → innerHTML` 序列化：文本节点中的 `"` 无需实体化，innerHTML 保留裸引号。`&`/`<`/`>` 转义后拼接进**双引号属性**时，引号击穿属性边界——「先转义再解析」只保证标签上下文安全，不覆盖属性上下文。旧流式路径用 `btn.dataset.content = content`（DOM 属性赋值）天然安全；三路径收口后流式骨架改走 escapeHtml 字符串通道——**从安全回归为不安全**，路径级回归。

## 代价

复制功能对含引号内容静默截断（用户可感知）；属性注入面存在（当前靠内容仍经转义无法注入新元素限制为属性级）。Falsify 波末审核抓到并修复（`9716c30`，数据通道单一化），未上线。

## 教训

1. **「先转义再拼 HTML 字符串」只对标签上下文安全**；数据进**属性**要么用属性级转义（`"` → `&quot;` 等全套），要么走 DOM API 赋值（dataset/attr——值直接存不经属性解析，无注入面无截断）
2. **数据通道单一化**：一个值只有一条进 DOM 的通道（本案例：内容一律不嵌 HTML 属性，由调用方统一 dataset 补写）；多路径渲染（全量重渲染/增量/流式）共用同一通道，避免路径间安全语义漂移
3. 收口类重构要对照**旧路径的安全形态**：合并三路径时，若其中一条旧路径是安全的（dataset 赋值），收口后不能让它回归到不安全形态——逐路径核对

## 防复发（惯例：前端 DOM 渲染代码评审时对照执行）

- [x] td-arch-health 批次：messageBubbleHtml 去 data-content 属性 + chat.js 三调用点 dataset 补写 + 引号/属性注入复现用例钉死（format.test.js + chat.test.js）
- 后续：任何新数据进 HTML 属性，先问「为什么不用 dataset/attr 赋值」；grep `data-content="${` 类字符串拼接模式

## 关联

- [[无框架前端 fetch seam]]
- [[SSE 流式前端状态陷阱]]
