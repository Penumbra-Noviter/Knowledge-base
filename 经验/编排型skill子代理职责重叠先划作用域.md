---
type: lesson
tags: [ZCode, skill, 编排, 子代理, 职责边界, 通用]
date: 2026-08-10
project: ZCode 环境
source: 自身项目实践
summary: 流程型 skill 调多个子代理时，必须显式声明「已定前提不重开」+「分层验证作用域」，否则同一决策被访谈两遍、每层审核重做上一层
provenance: 2026-08-10 ZCode 会话：project-kickoff 子代理职责重叠分析（wayfinder×Grilling 重复访谈、Falsify/Standards 三层无作用域、冒烟×3），SKILL.md 7 处修订后三副本 md5 52a9f5ac 校验通过（无 git/DEV_LOG，过程见会话记录）
status: verified
---

# 编排型 skill 子代理职责重叠先划作用域

## 症状
- project-kickoff 分支 A：wayfinder 内部已跑多轮 grilling（destination 命名 + BFS 勘探 + grilling 类 ticket 全交 Grilling），回主线后又完整重跑一次 Grilling——同一决策空间被访谈两遍。
- Falsify 在三个层级重复跑（Implement 三轴自审 → 波末增量审核 → 期末全量），Standards 跑两层（Implement 自审硬清单 → 期末 Standards 轴），各层都没写清自己该看什么。
- 冒烟验证 N+2 次（Implement worktree 内 → 每波合并后 → 期末 4.5）。
- 计数式表述维护漂移：plan-tickets 调用要求写死「四项要求」，加一项要改数字。

## 根因
- skill 演进是**增量叠加**：加 wayfinder 分支时没有回头改主线 Grilling 的输入契约；加分层审核时没写各层职责边界。调用约定只写了「做什么」，没写「已定前提不重开」「每层只做上层未覆盖的部分」。
- 子代理是独立上下文，不共享主会话的判断——「这个已经问过了」必须写进调用 prompt，否则每个子代理都从零开始。

## 代价
- 用户被重复访谈，第二次答案可能与第一次漂移，共识反而变模糊；
- 每层审核重做上一层 → 边际价值趋零，审核成本线性翻倍；
- 本次定位重叠约半小时；若上线运行，每个项目都要多付一遍访谈与审核成本。

## 教训
- 编排型 skill（一个流程调多个子代理）的调用契约必须显式声明两件事：①**已定前提清单**——哪些决策已 settle、不得重开，新环节只审增量；②**分层验证作用域**——每层只做上层没覆盖的那部分（自审=单工单内部，波末=跨工单耦合+共享文件，期末=系统级+跨波交互）。
- 加新子代理/新阶段时，反查已有阶段的输入契约是否被破坏（wayfinder 加入后主线 Grilling 就该改成增量模式）。

## 防复发
- 编排节点新增清单：新节点是否重问已定决策？新验证层是否重复上层作用域？
- 调用约定用「以下要求」不用「N 项要求」，防计数漂移；
- 使用 memory 的「xxx 不存在」类断言前先 `ls`/`find` 反查现状（本次 memory 声称 wayfinder/grilling/prototype 无 SKILL.md，实际已存在——[[审计快照过期需复核]] 同源）；
- 多入口 skill 编辑后 md5 复查三副本一致（[[多入口skill副本先验硬链接]]）。
- [ ] 已落实？

## 关联
- [[审计快照过期需复核]]
- [[多入口skill副本先验硬链接]]
- [[流程skill优化按契约-退化-硬化分层]]
