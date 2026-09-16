---
type: lesson
tags: [flutter, provider, 装配, lazy, 死代码]
date: 2026-09-15
project: Conver System
source: 自身项目实践
summary: Flutter 装配层用哑 Provider<Object?> 挂启动副作用时，无消费者则 create 默认 lazy 永不执行——「冒烟不抛」是假阴性；副作用 Provider 必须 lazy:false
provenance: DEV_LOG〈人机恋阶段 2 批次〉· W6 波末增量审核 F1 · commit 5fe10e5 · 2026-09-15 会话
status: verified
agents_md_feedback: 印证 AGENTS.md「对抗性自检」——审核者用探针实证 create 未执行，装配冒烟「不抛」被证明是假阴性（verify create 执行而非只断言无异常）
---

# 哑 Provider 挂启动副作用：无消费者时 default-lazy 永不执行，冒烟「不抛」是假阴性

## 症状

Flutter + provider 装配层为「启动路径副作用」（通知初始化 + SR-08 排程恢复、冷启动深链接线）挂了两个 `Provider<Object?>(create: ...)`——create 内 `unawaited(...)` 执行副作用并返回 null，没有消费者。单元/装配测试跑「冒烟不抛」全绿、不崩溃。波末增量审核用探针（在 create 内加计数器）实测：`created=0`（有消费者的对照 Provider=1）——**副作用从未执行**，冷启动深链接线与启动排程恢复是真机死代码。

## 根因

provider 的 `Provider` 默认 `lazy: true`：**只有被 consumer 读取时才执行 create**。`Object?` 且无人 `context.read` 的哑 Provider 永远不会被实例化——「冒烟不抛异常」只证明 create 没跑，不证明副作用路径没问题（装配冒烟假阴性）。

## 代价

本批交付前由波末审核抓到：深链点通知进对话、启动恢复排程两个已「交付」功能为死代码；修复 = 两处 `lazy: false` + 兼容性回归修正。若未抓出，功能直接带病上线且测试全绿。

## 教训

装配层用 Provider 挂「fire-and-forget 启动副作用」时，**必须显式 `lazy: false`**；更稳的做法是副作用挂在有消费者的 Provider（如 `_prewarm` 先例的 create 内联）或 App 生命周期回调。装配冒烟必须断言「副作用真实发生」（create 计数 / seam 调用记录），不能只断言「不抛异常」。

## 防复发

- [x] 已落实：装配类任务（app.dart 挂启动副作用）在自审与增量审核中检查 Provider 是否 `lazy: false` 或有消费者
- [x] 已落实：冒烟测试对启动副作用类 Provider 断言 create 实际执行（副作用真实发生），不满足于「不抛」

## 关联

- [[Flutter测试碰平台依赖必须超时兜底]]