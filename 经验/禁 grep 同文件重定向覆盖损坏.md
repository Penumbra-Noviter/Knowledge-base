---
type: lesson
tags: [shell, 数据安全, 子代理]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: 禁 grep pattern file > file 同文件重定向——shell 重定向在命令执行前截断目标，源=目标时先清空再读致文件覆盖损坏（850 行实证）
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· SP-01 票 chat_service_test 覆盖事故 · commit a8fca09
status: verified
agents_md_feedback: 
---

# 禁 grep pattern file > file 同文件重定向

## 症状
子代理执行类似 `grep xxx test_file > test_file` 的命令（想「过滤文件保留匹配行」），`test/services/chat_service_test.dart`（4102 行）被覆盖为 850 行垃圾，git 状态变 M，随后 checkout 恢复。

## 根因
Shell 重定向（`>`）在**命令执行前**由 shell 打开并截断目标文件——源文件与目标文件为同一路径时，grep 尚未读到任何内容文件已被清空，随后读到空输入输出空/部分内容，原文件内容丢失。

## 代价
4102 行测试文件损坏 → git checkout 恢复（幸有版本控制）；若文件未提交过则数据不可恢复。

## 教训
同文件原地过滤在 shell 里是数据破坏操作，不是编辑操作——任何「读 A 写 A」形态的命令都违反 shell 的文件语义。

## 防复发
- **禁 `grep pattern file > file` / `sed -i` 类同文件重定向**；输出到临时文件（`> file.tmp`）再改名，或直接用编辑器工具。
- 子代理派发 prompt 显式列出此禁令（本批已固化）。
- [x] 已落实？

## 关联
- [[]]