---
type: persona
tags: [persona]
date: 2026-08-09
project: ZCode 环境
status: active
---

# ZCode 环境 Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进环境本身（SKILL.md / 命令文件）。

## 本人偏好
- 命令名 / 入口要**极短**：`/validate-skill finesse-ui` → `/finesse`（2026-08-09 明确）
- 短命令的默认行为贴合最高频用法：`/finesse` 缺省只校验 finesse-ui，可追加其他目标
- 升级高频资源后立即跑校验（`/finesse`），不靠"小心一点"记规则

## 固定约束
- SKILL.md frontmatter：`description` ≤1024 字符（硬限，超限整个 skill 被丢弃）；触发文案控制在 ~250 字符可见窗口内，完整触发词/路由/命令细节放 `when_to_use`（[[SKILL.md frontmatter 的 250 与 1024 规则]]）
- 扩展键有效：`version` / `user-invocable` / `disable-model-invocation` / `argument-hint` / `allowed-tools` / `compatibility`——zcode-guide 文档集合不完整，以实证为准，不因文档未收录而删除
- 校验闸门：`~/.zcode/tools/validate_skills.py` + `/finesse` 命令（[[高频升级资源配自动闸门]]）
- 多入口 skill 副本为**硬链接**：`.cc-switch/skills`、`.zcode/skills`、`D:\Desktop\cc\.claude\skills` 三处同一 inode（2026-08-10 验证 58828270132689331），编辑单点即可；每次编辑前 `ls -li` 复查 inode 仍相同（防重装变复制）（[[多入口skill副本先验硬链接]]）

## 稳定模式
- 高频升级资源配自动闸门：硬错误（会导致失效）→ 退出码 1；软警告（降级风险）→ 只提示（[[高频升级资源配自动闸门]]）
- 全量基线：53 个 user skill 校验 0 错误（2026-08-09）
- 优化按 **P 批次推进**：以「优化Px项」按批下发，每批原子落地 + 验证 + 汇报，批间按优先级排序
- 流程/skill 优化**先契约对齐**（逐条对照全局 AGENTS.md）再扩展；未证实的观察项不预埋，留给真实痛点（[[流程skill优化按契约-退化-硬化分层]]）

## 变更记录
- 2026-08-09：项目登记 + persona 首版——极短命令名偏好、SKILL.md frontmatter 250/1024 规则、校验闸门模式。来源：ZCode 环境会话（finesse-ui description 修复 + validate_skills.py + /finesse）
- 2026-08-10：project-kickoff skill P0-P4 优化（16 项）蒸馏——新增固定约束（三处 skills 硬链接同 inode）、稳定模式（P 批次推进 / 契约对齐优先、不预埋）；经验笔记 3 条（共享约定先查全局引用 / 多入口副本先验硬链接 / 流程优化分层方法论）。来源：ZCode 环境会话（project-kickoff 全流程优化）
