---
type: lesson
tags: [skill, 生命周期, 落地策略, ZCode]
date: 2026-09-12
project: ZCode 环境
source: 自身项目实践
summary: 新造/不成熟的 skill 落全局时，用 disable-model-invocation 让模型不自动触发，靠 ask-matt/静态 MAP 人工索引保证可发现，成熟后再放开
provenance: 2026-09-12 AI自动获客会话 · product-facets 落地决策（用户拍板）：disable-model-invocation + 接入 ask-matt 路由 + 更新 SKILL_MAP（66→67）
status: verified
agents_md_feedback: <!-- 无 -->
---

# 新 skill 不成熟期用 user-invoked + 人工索引落地

## 症状
- 新造的 skill 知识层尚薄（部分分支协议还是草图），但已落全局目录。若立即 model-invoked，模型可能在它不成熟的场景自动触发，产出不可靠结果且用户难以察觉。

## 根因
- skill 的**成熟度**与「是否让模型自动路由」是两件事。成熟度不够时放开自动路由 = 用未验证的能力污染模型的自动决策。

## 代价
- 直接放开：模型在不成熟分支上给不可靠输出，因是自动触发，用户不易察觉来源。

## 教训
- **不成熟 skill 的落地策略**：`disable-model-invocation: true`（只用户主动调用）+ 人工索引保证可发现，成熟（真实用过几轮、知识层补齐）后再去掉该字段放开自动路由。
- 人工索引要**两处**都登记，二者互补：`ask-matt`（动态路由：按情境提问路由）+ `SKILL_MAP`（静态速查图：按「要做的事」反查）。
- 放对位置：立项类 skill 挂 `ask-matt` 的 on-ramps（起始情境），不是 Standalone（后者定义是「完全在主流程之外」）。

## 防复发
- [ ] 新 skill 落地默认 user-invoked，并在 ask-matt + SKILL_MAP 两处登记。
- [ ] 把「成熟后放开自动路由」作为显式后续动作记在 skill 待办里。

## 关联
- [[SKILL.md frontmatter 的 250 与 1024 规则]]
