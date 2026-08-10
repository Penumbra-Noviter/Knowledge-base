---
type: lesson
tags: [ZCode, skill, 硬链接, 文件同步, 通用]
date: 2026-08-10
project: ZCode 环境
source: 自身项目实践
summary: .cc-switch/.zcode/cc 三处 skills 是同一 inode 硬链接，编辑前 ls -li 验证，防复制漂移
provenance: 2026-08-09~10 ZCode 会话（project-kickoff P0-P4 优化，`ls -li` 验证三副本 inode 58828270132689331 相同）
status: verified
---

# 多入口 skill 副本先验硬链接

## 症状
- `C:\Users\Administrator\.cc-switch\skills`、`C:\Users\Administrator\.zcode\skills`、`D:\Desktop\cc\.claude\skills` 三处 project-kickoff 内容一致——但一度不确定是「硬链接同步」还是「三份独立副本」。

## 根因
- 多入口工具链（cc-switch / zcode / cc）各自指向同一技能仓库，装法可能是 `ln`（硬链接）也可能是 `cp`（复制）。复制出来的副本会随单点编辑漂移：用户以为改了全局，实际只改了一处。

## 代价
- 无（本次编辑前用 `ls -li` 验证三文件 inode 相同 → 单点编辑即三处同步）；
- 但若未验证就单点编辑，漂移副本会在未来某次跑出「与文档不符」的行为，排查成本按小时计。

## 教训
- 多入口共享资源（skill / 命令 / 配置）编辑前先验证同步方式：`ls -li` 对比 inode——相同 = 硬链接，可单点编辑；不同 = 副本，需逐处同步或提醒用户合并回硬链接。

## 防复发
- 本项目固定约束：三处 skills 同一 inode，编辑单点即可（已并入 persona）；
- 每次编辑涉及多入口文件时 `ls -li` 复查 inode 是否仍相同（某次重装可能变复制）。
- [ ] 已落实？

## 关联
- [[]]
