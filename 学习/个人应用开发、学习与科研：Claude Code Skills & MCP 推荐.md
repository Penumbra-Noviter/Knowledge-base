---
type: study
tags: [claude-code, mcp, skills]
date: 2026-08-02
source: 实测环境（2026-08-02 对照本会话可用工具清单核对）
status: verified
---

# 个人应用开发、学习与科研：Claude Code Skills & MCP 推荐

> **状态图例**：✅ 已安装（本环境实测存在）｜❌ 未安装（推荐候选，装前先确认是否需要）｜⚠️ 部分真实/已废弃
> 本文档 2026-08-02 重新核对过：**已装 / 未装 严格按当前环境标注**，避免照着推荐清单踩空。

---

## 科研 & 学习

| MCP | 状态 | 用途 | 推荐理由 |
|-----|------|------|---------|
| **arXiv MCP** | ✅ 已装 | 搜索和获取学术论文 | 科研必备，直接检索论文摘要和全文 |
| **Brave Search MCP** | ❌ 未装 | 网络搜索 | 比 WebSearch 更灵活，适合学术资料检索（装前评估：内置 WebSearch 是否够用） |
| **PDF Reader / Zotero MCP** | ❌ 未装 | 文献管理和阅读 | 管理参考文献、阅读 PDF 论文（当前用内置 Read 读 PDF 已够） |
| **Python REPL MCP** | ❌ 未装 | 交互式 Python 环境 | 数据分析、科学计算、快速原型验证（本机可直接开终端跑 python，优先级低） |

## 应用开发

| MCP | 状态 | 用途 | 推荐理由 |
| --- | --- | --- | --- |
| **SQLite MCP** | ❌ 未装 | 本地数据库 | 个人应用最实用的轻量数据库，直接通过对话操作 |
| **GitHub MCP** | ✅ 已装 | 仓库管理 | 管理个人项目、Issue、PR，不用离开终端 |
| **Filesystem MCP** | ✅ 已装 | 增强文件操作 | 比内置工具更强大的文件管理能力 |
| **Exa MCP** | ❌ 未装 | 语义搜索 | 搜索技术文档和代码示例，比关键词搜索更精准 |

## 🧩 内置 Skills（全部 ✅ 实测存在）

| Skill | 场景 |
|-------|------|
| **`deep-research`** | 多源交叉验证的深度研究报告，科研综述利器 |
| **`dataviz`** | 任何图表/可视化（matplotlib, plotly, D3），论文配图和仪表盘 |
| **`obsidian-vault`** | 管理 Obsidian 知识库（**本库改造用的就是它**） |
| **`domain-modeling`** | 设计应用时厘清领域概念和术语 |
| **`design-an-interface`** | 并行生成多种接口设计方案，对比选出最优 |
| **`prototype`** | 快速搭建原型验证想法（LOGIC / UI 分支） |
| **`code-review`** | 自审代码质量（收益计算器已验证其价值） |
| **`tdd`** | 测试驱动开发，red-green-refactor 工作流 |
| **`grilling`** | 让 AI 对计划/设计魔鬼拷问（注意：技能名是 grilling，不是 grill-me） |
| **`loop`** | 定时任务，定时检查/定时提醒复习 |
| **`simplify`** | 代码复用、简化、效率清理 |
| **`security-review`** | 代码安全检查 |

## 🧠 已内置 MCP 的用法（全部 ✅ 实测存在）

| MCP | 用法 |
|-----|------|
| **Context7** | 查任何库的最新文档，比凭记忆写代码靠谱得多 |
| **Sequential Thinking** | 复杂问题分步推理，适合算法设计、数学推导 |
| **Memory (知识图谱)** | 跨会话持久化记忆，积累学习和项目上下文 |
| **Playwright** | 浏览器自动化，测试 Web 应用、爬取网页数据 |
| **Serena** | 代码语义检索/编辑/架构（全局规范要求优先使用） |

## 🎯 按场景的推荐组合

### 场景 1：学一门新技术/框架
```
Context7（查文档） + deep-research（系统调研） + obsidian-vault（记笔记） + Memory（积累知识）
```

### 场景 2：开发个人应用
```
prototype（原型） → design-an-interface（接口设计） → domain-modeling（领域建模）
→ TDD（测试驱动开发） → code-review（自查） → GitHub MCP（管理代码）
```

### 场景 3：科研数据分析
```
Python REPL MCP（计算，未装） + dataviz（可视化） + arXiv MCP（文献） + deep-research（综述）
```

### 场景 4：写论文/报告
```
deep-research（调研） + obsidian-vault（素材管理） + grilling（论点拷问） + dataviz（配图）
```

## 🚀 安装决策清单

1. **SQLite MCP** — 只有当 Conver System 需要直接对话式查库时才装（当前 SQLite 用 SQLAlchemy 管理，优先级中等）
2. **Brave Search MCP** — 先试内置 WebSearch，不够再装
3. **其他（Exa / Python REPL / PDF Reader）** — 有明确场景再装，避免装一堆不用

> 依据 [[收益计算器项目经验复盘]] 的「YAGNI + 敢于关闭」原则：**工具不是越多越好，每个都要能回答「谁、在什么场景、用它解决什么」。**

---

*关联：[[Matt_Pocock 的 工程实用skills]] · [[技能决策树]] · [[个人桌面应用开发系统提示词]]*
