---
type: lesson
tags: [Dart, Flutter, 工具链, 格式化]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: 长期迭代中的仓库非 format-clean——勿全库/多文件 dart format（14/15 票引入 1700-3000 行噪音 diff），只格式化本票新增/结构性修改行
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· NPD-04/SP-01 票 · commit 8ca55aa / a8fca09
status: verified
agents_md_feedback: 
---

# 勿全库 dart format：非 format-clean 仓库的格式噪音

## 症状
`dart format` 全库跑在非 format-clean 仓库引入 ~1700 行（全库）/ ~3000 行（仅触碰文件的多文件批量）无关格式噪音 diff；最终须逐文件还原 + restore-and-reapply 清理。

## 根因
长期多批次迭代的仓库中，存量文件存在 formatter 版本漂移（历史 F-103 实证：188 文件/223 检查格式差异为存量漂移，已拍板不归一）。全库或批量格式化会把存量漂移全部拖入本票 diff，掩盖真实改动，review 无法聚焦。

## 代价
14/15 票各一次：1700/3000 行噪音还原耗时 + diff 审阅干扰；误提交风险（噪音混入真实改动）。

## 教训
格式化工具作用于「文件」而非「改动」——在非 format-clean 仓库中，任何批量格式化都会污染 diff；正确粒度是「本票新增/结构性修改的行」。

## 防复发
- 禁止全库 `dart format` 与多文件批量格式化；只格式化本票新建文件或结构性修改的代码块。
- 提交前 `git diff --stat` 核对 diff 行数异常膨胀（远超预估变更量）即查格式噪音。
- 子代理派发 prompt 显式注明「勿全库 dart format」。
- [x] 已落实？

## 关联
- [[]]