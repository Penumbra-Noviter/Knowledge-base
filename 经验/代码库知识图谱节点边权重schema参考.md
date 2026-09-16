---
type: lesson
tags: [代码库建模, 知识图谱, schema, 建模词汇表]
date: 2026-09-15
project: 通用
source: 外部仓库研究
summary: 代码库知识图谱的节点/边/权重 schema（13 节点 + 26 边 + 权重约定）可作建模词汇表参考
provenance: 阅读 Egonex-AI/Understand-Anything 仓库总结 · 2026-09-15 会话
status: candidate
agents_md_feedback:
---

# 代码库知识图谱节点边权重 schema 参考

## 症状
要建代码库的「全景图」，一开始就卡在「节点和边怎么分类」——文件、函数、类、配置、文档、数据库表、API 端点、CI 流水线全混在一起，边也分不清是「导入」「调用」还是「依赖」。

## 根因
代码库建模缺一套先验的分类词汇表，导致每个项目都从零发明一套 node/edge 类型，既不一致也不可迁移。

## 代价
Understand-Anything 沉淀了一套现成 schema：13 类节点、26 类边、按边类型给权重。若无此表，建模阶段就要消耗大量时间在分类命名上，且不同项目间图谱无法对齐。

## 教训
代码库建模的节点/边分类是可以预先固化的领域词汇，不必每次重新发明；直接复用成熟 schema 再按项目裁剪。

## 防复发
- [ ] 建代码库模型时，先套用现成 node/edge 词汇表（13 节点：file/function/class/module/concept/config/document/service/table/endpoint/pipeline/schema/resource），再增补项目特有类型
- [ ] 边类型按语义分类（structural/behavioral/data-flow/dependency/semantic/infrastructure/schema），避免「imports」「depends_on」混用
- [ ] 边权重统一约定（contains=1.0、imports=0.7、tested_by=0.5…），给下游图算法（社区检测/布局）稳定输入

## 关联
- [[静态分析确定性打底与LLM补语义的混合建模]]
- [[大对象增量更新用结构指纹而非全量重扫]]
