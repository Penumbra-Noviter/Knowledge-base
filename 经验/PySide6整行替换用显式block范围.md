---
type: lesson
tags: [PySide6, GUI, QTextEdit]
date: 2026-08-11
project: Model Fingerprint
source: 自身项目实践
summary: PySide6 的 BlockUnderCursor 选中范围是 [pos-1, pos+len-1)（含前块分隔符），多行文档整行替换会吃掉换行；用显式 setPosition 范围
provenance: DEV_LOG#批次14 UX收口 · commit 159a2ab · 2026-08-11
status: verified
---

# PySide6 整行替换用显式 block 范围，不用 BlockUnderCursor

## 症状
QPlainTextEdit 日志区做「维度进度状态行就地更新」：`QTextCursor(block)` + `select(BlockUnderCursor)` + `insertText` 替换整行——多行日志（含中文/emoji）时替换结果把**上一行的换行吃掉**，两行文本并成一行（如「[*] 维度进度：🔄 logprobs 运行中」）。

## 根因
PySide6 的 `QTextCursor.SelectMode.BlockUnderCursor` 实际选中范围是 `[block.pos - 1, block.pos + block.length - 1)`——**包含前一个块的段落分隔符**（\n/U+2029），不包含本块的分隔符。纯 ASCII 单行测试看起来正常（前块 \n 被删、本块 \n 恰好补位，视觉结果侥幸正确）；一旦前块内容含中文/emoji（UTF-16 多 code unit），范围偏移导致换行被吞。

## 代价
一次「实验钉住」循环：3 轮最小复现脚本对比才定位到 select 范围语义（约 20 分钟）；GUI 测试断言一度失败（「🔄 与 ✅ 同行」）。

## 教训
QTextEdit 系整行替换不要依赖 `select(BlockUnderCursor)`——显式设置 selection 范围：
```python
block = doc.findBlockByNumber(n)
cursor.setPosition(block.position())
cursor.setPosition(block.position() + block.length() - 1, QTextCursor.KeepAnchor)
cursor.insertText(text)
```
`block.length()` 含块分隔符，`-1` 恰好选中纯块内容，分隔符保留。

## 防复发
- 涉及文本块操作的代码一律显式 position 范围，不用 select mode
- 行替换测试断言用真实多行 + 中文/emoji 文本（不是单行 ASCII）

## 关联
persona.md
