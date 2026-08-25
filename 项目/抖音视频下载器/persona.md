---
type: persona
tags: [persona]
date: 2026-08-23
project: 抖音视频下载器
status: active
---

# 抖音视频下载器 Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好
- 先结论后证据，简洁；代码必须完整可运行，不写半截代码
- 逆向还原项目的**行为零变更**是硬红线：用户可见行为（文案/状态串/交互）逐字保留，重构只动内部结构
- 全自动批处理偏好：一次性授权「全自动跑完」后，不再中途拦截（但删除类动作始终要确认）
- 验收偏好机器可验证（pytest 目标/命令 + grep 门），不可自动验证的项清单化供人工冒烟

## 固定约束
- 技术栈：Python ≥3.10 + Tkinter + Playwright(channel=msedge) + requests + sqlite3 + asyncio（无云依赖，本地优先）
- 源码是 exe 逆向还原产物：基线 commit `c424b75` 起不再维护字节码一致性；`dump/`、`verify.py`、`dump_all.py`、`diff_ins.py`、`test_variants.py` 为历史证明物**勿删**
- 任务池唯一权威 = 仓库 `TO-TICKETS.md`；技术债候选池 = `TECH_DEBT.md`（候选不自动进入认领）
- 测试全量口径 `python -m pytest -q tests/`（根目录裸跑会因 test_variants.py 依赖 git-ignored 逆向产物收集失败）
- GUI 属装配层，覆盖率数字门豁免（结构 grep 门 + 行为测试替代）

## 稳定模式
- 架构审查（improve-codebase-architecture）→ project-kickoff 全自动批次的流水线已跑通两次：审查报告定候选 → Grilling 共识（含 TD 裁决）→ plan-tickets 拆票带文件范围/共享文件/语义验收锚 → Implement 独立 worktree 并行 → 波末合并+范围核验 → 期末四轴 code-review
- 新模块命名短英文小写（capture/workflow/constants），新模块进 CONTEXT.md 领域词汇表
- 用户并发上限实测 2：单波并行 ≤2，波中事件（空返回/停止）用「同分支现场续用」重开，不丢半成品
- 增量审核网关连败时降级并入期末四轴 Falsify（不无界消耗重开预算）

## 变更记录
- 2026-08-23 首次建档：来源=架构优化批次2 kickoff 会话（全自动档实证）