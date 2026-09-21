---
type: lesson
tags: [Flutter, ListView, 滚动, 时序]
date: 2026-09-20
project: Conver System
source: 自身项目实践
summary: ListView 懒构建 + 消息行高变化时 _scrollToBottom 首跳基于未布局 extent 估算，跨 cacheExtent 边界须 post-frame 补跳（实测差 19.33px）
provenance: DEV_LOG〈移动端角色对话打磨批次 chat-polish-aigs〉· MS-05 票 _scrollToBottom 懒构建补跳 · commit 7866ce6
status: verified
agents_md_feedback: 
---

# 消息列表懒构建滚动：首跳估算 + post-frame 补跳

## 症状
消息列表在「追加消息后自动滚到底」场景：首跳位置基于未布局区段估算，跨 cacheExtent 边界后实际到底位置与估算差 ~19.33px（底部留白/最后消息未完全可见）。

## 根因
ListView 懒构建下，cacheExtent 之外的消息行尚未布局，`ScrollPosition` 对总 extent 的估算基于「行数 × 估算行高」而非真实测量；当消息行高变化（多行文本换行）或跨越 cacheExtent 边界时，估算与真实相差一个/多个行高，首跳落点偏上。

## 代价
「自动滚到底」体验缺陷（最后消息被新消息推起后未完全可见）；修复需补 post-frame 二次滚动，测试需先红后绿锁定（M3-04c 用例）。

## 教训
懒构建列表的「跳到底/定位」不能信任单次基于未布局估算的滚动——布局完成（post-frame）后内容边界可能移动，须按需补跳。

## 防复发
- 消息/可变行高列表的自动滚底：首次 scrollTo 后 post-frame 核对是否仍在底部附近，偏离则补跳。
- 边界覆盖：既有 M3-04c 用例先红后绿（跨 cacheExtent 场景显式断言）。
- [x] 已落实？

## 关联
- [[]]