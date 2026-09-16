---
type: lesson
tags: [worktree, subagent, 断线重连, 现场续用, kickoff]
date: 2026-08-23
project: 抖音视频下载器
source: 架构优化批次2（DEV_LOG 2026-08-23 条目）
summary: 子智能体被网关「停止/空返回」中断时，其 worktree 半成品（未提交改动）常是健康可续的——重开用「同分支现场续用」模式（先审 diff 健康度，健康则续做不返工），一次成功（T2-06 恢复 ±42 行改动）；无谓的 git checkout -- . 重来会浪费大半工作量
provenance: DEV_LOG#批次2 W3 事件 · commit 5887b6d（T2-06 线下续用）· 2026-08-23
status: verified
---

# worktree 半成品现场续用重开模式

## 症状

批量子智能体执行到一半被网关层中断（"Model request failed" / "no text no usage" / 被 stop）。工单重开时若简单从头重做，会丢弃已落盘的半成品。T2-06 中断时已在 worktree 留下未提交改动（downloader.py ±42 行 + test_downloader.py +45 行新测试）——占总工作量约六成。

## 处理

重开 prompt 内置**现场续用指令**：先 `git status`/`git diff` 审查半成品健康度（改动方向对、无半截语法错）→ 健康则直接续做（跑测试、补证据、commit）；不健康（方向错/语法残）才 `git checkout -- .` 从头。结果：一次续用成功，commit 5887b6d 落地，与从零做等价质量。

## 规则

- 中断重开的默认动作 = 现场续用，不是重来；「重来」是审查后的降级选项
- 审查健康度的快速判据：diff 内容是否对准工单验收锚、是否出现语法错/半截语句、测试文件是否已含新增用例骨架
- 现场续用省下的不只是时间——保留上下文连续性的心理锚（代理对半成品的熟悉度）

## 边界

- 仅适用于「已落盘在独立 worktree」的半成品；主工作树的脏改不适用（主树要与交付态保持一致）
- 多代失败的连续中断仍按原协议升级（重开 ≤2 次 → 人工裁决），现场续用不改变重开上限计数

## 补充实证（2026-09-08 M6 kickoff，Conver System mobile）

**新场景：用户主动暂停 + TaskStop 后的接续**——「保留进度先暂停开发」→ 主会话 TaskStop 两在途 agent（07 B1 修复 5 文件未提交改动 + 票 11 视觉评审的 12 张截图/mock SSE 脚本均在磁盘）→ 恢复时 **SendMessage 不可续接**（TaskStop 后报 `No active local_agent task`）→ 必须**新 agent + 现场核查指令**：先 `git worktree add <路径> <分支>`（去 -b）复用 → `git status` 核对半成品 → **WIP 先 `git commit` 固化**（防 merge main 时丢失）→ `git merge main` 同步 → 接续完成。两路均半成品接续成功零丢失。与「网关中断」的差异：TaskStop 杀掉的 agent 会话不可恢复（非完成态），恢复动作固定为派新 agent，但磁盘现场保全不变。
- 半成品健康度判据扩展：未提交改动 = 修复进行到一半的产物（方向对、无语法残）→ 接续；WIP 固化是「暂停恢复」场景新增的保全前置（常规中断重开不强制，因 agent 会自己续做）。