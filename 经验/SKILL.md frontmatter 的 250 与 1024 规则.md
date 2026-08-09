---
type: lesson
tags: [ZCode, skill, frontmatter, 配置, 通用]
date: 2026-08-09
project: ZCode 环境
source: 自身项目实践
summary: SKILL.md description 超 1024 字符 skill 被整个丢弃，模型只见前 ~250 字符——只写电梯陈述，细节放 when_to_use
provenance: 2026-08-09 ZCode 环境会话：finesse-ui description 1083→243 字符修复 + validate_skills.py + /finesse 命令（无 git/DEV_LOG，过程见会话记录）
status: verified
---

# SKILL.md frontmatter 的 250 与 1024 规则

## 症状
- 客户端报 `Skill description is too long (>1024): finesse-ui`，skill 从 `/` 菜单与 Settings → Skills 中**消失**——不是降级，是加载失败被整体丢弃

## 根因
- SKILL.md frontmatter 的 `description` 有硬上限 **1024 字符**，超限 = 整个 skill 不加载
- 模型触发时 description 只展示前 **~250 字符**——超过 250 的内容对触发决策零贡献。1083 字符的 description 里约 760 字符无效且把风险顶到上限，纯赔不赚

## 代价
- finesse-ui 一次失效；本次修复 + 建闸门共一个会话

## 教训
- `description` 只写 ≤250 字符的"电梯陈述"：做什么（一句话）+ 最强触发词前置；完整触发词表、路由规则、动词命令移入 **`when_to_use`**（该字段展示给模型时**不截断**）
- 文档化的 frontmatter 键只有 `name/description/when_to_use/license/metadata`，但实践中 `version/user-invocable/disable-model-invocation/argument-hint/allowed-tools/compatibility` 被广泛使用且正常加载——**文档没收录 ≠ 无效，别乱删**（以实证为准，zcode-guide 文档集合不完整）

## 防复发
- [x] 已落实：`~/.zcode/tools/validate_skills.py`（description >1024 硬错误、>250 警告、缺必需键、解析错误）+ `/finesse` 短命令，升级后跑一次
- [ ] `~/.zcode` 纳入 git 后可挂 pre-commit hook 自动挡

## 关联
- [[高频升级资源配自动闸门]]
