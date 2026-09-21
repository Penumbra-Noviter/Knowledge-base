---
type: lesson
tags: [测试, 质量门禁, 子代理]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: 突变探针（MUTATION/PROBE/TEMP）会经子代理半成品残留进生产代码——波末/接手半成品第一件事 grep 清零扫描
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· 01 switchSwipe / 05 TEMP-EXPERIMENT 残留 · commit 9f68a9d
status: verified
agents_md_feedback: 印证「Falsify 快审」与测试诚实协议精神——突变验证的临时状态必须清场后复验，残留即测试环境泄漏进生产
---

# 突变探针残留：波末必 grep 清零

## 症状
批次内两票（01 switchSwipe、05 `TEMP-EXPERIMENT`）在生产代码中发现突变验证/实验探针残留——子代理完成「突变抽查后恢复」流程时漏恢复，半成品落盘带入生产。

## 根因
突变验证（删分支/改写实现使测试变红）是开发期临时状态，依赖「事后恢复」的自觉；子代理批量落盘时恢复步骤可能被跳过（或恢复不完整），残留探针随 commit 进入生产代码。全量测试绿不保证探针清除——探针可能恰好不影响测试结果。

## 代价
波末审核 grep 扫描拦下（本批 0 泄漏进发布）；若未拦截，突变探针会改变生产行为（如 switchSwipe content 未覆写）——用户可见的隐性缺陷，且极难定位（测试全绿）。

## 教训
突变验证的临时状态是「必须清场后复验」的一等流程：探针残留的检测不能依赖实现者自觉，必须是门禁级扫描。

## 防复发
- 波末/接手子代理半成品第一件事：`grep -rn "MUTATION\|PROBE\|TEMP-EXPERIMENT" lib/ test/` 清零扫描。
- 子代理派发 prompt 显式要求「commit 前 grep 扫描残留并清除」作为硬性提交前置。
- [x] 已落实？

## 关联
- [[]]