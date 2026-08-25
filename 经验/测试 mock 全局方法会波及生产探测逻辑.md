---
type: lesson
tags: [测试, mock, pytest]
date: 2026-08-26
project: ctrl
source: 自身项目实践
summary: monkeypatch 全局类方法（Path.exists）会波及生产探测逻辑，探测改直读判定语义
provenance: 会话 sess_034cc4d1 · commit c76f8ce（T-013 修复轮次2 实现教训）· evidence/T-013.md
status: verified
---

# 测试 mock 全局方法会波及生产探测逻辑

## 症状

给降级护栏加守卫时首版用 `REGISTRY_PATH.exists()` 探测文件是否存在，结果被旧测试全局 monkeypatch 的 `Path.exists` 波及，大面积误判（存在性判定随测试替身漂移），红绿翻转混乱。

## 根因

`.exists()` 是全局可变类方法，任何测试替身都能改写其行为；生产代码依赖它做关键判定时，判定真实性由"当前测试环境碰巧怎么 patch"决定。同类陷阱此前已出现过一次（模块级 `from x import CONST` 按值绑定导致 fixture 补丁失效）——都是"探测/取值方式放大了测试环境的副作用"。

## 代价

实现者一轮返工 + 误判排查时间；若无回归测试锚定可能静默放过坏路径。

## 教训

生产代码的关键探测不用可被 mock 的便利方法（exists/is_dir…），直接做真实操作并以异常类型表达语义（FileNotFoundError=未损坏）；这同时让测试替身无法无意间改变生产行为。

## 防复发

- [x] 已落实：`registry_file_degraded` 直接 read 并以 FileNotFoundError 表达缺失；docstring 写明设计原因

## 关联
- [[不变量护栏下沉模块受保护写]]
- [[worktree并行前查CRLF祖父blob]]
