---
type: index
tags: [project, registry, moc]
date: 2026-08-06
---

# 项目注册表

知识库 `project` 字段 ↔ 仓库目录 ↔ 技术栈 的映射表（2026-08-06 审计建立）。**反查项目以此表为准**——按 project 标签找项目、按目录找笔记都从这里出发。

| project 字段 | 仓库目录 | 产品/全称 | 技术栈 | 状态 |
|---|---|---|---|---|
| 收益计算器 | `D:\Desktop\Craft\Profit Calculator` | 收益计算器 | PySide6 + pyqtgraph + PyInstaller | 已交付，持续迭代 |
| 衍生框架 | `D:\Desktop\Craft\Profit Calculator  衍生的框架` | pyside6-skeleton | PySide6 骨架 | 实验产物，弃置待清理 |
| Conver System | `D:\Desktop\Craft\conver system` | Conver System 角色对话系统 | FastAPI + SQLAlchemy + Vanilla JS + 自建 LLM Provider | 主力，Phase 6 |
| 川流不息 | `D:\Desktop\Craft\Personal Dashboard` | 川流不息 · 个人工作台 | Tauri v2 + Rust + Vite + Vanilla JS | 已交付，维护期 |
| （夭折） | `D:\Desktop\Craft\6a742fdf7e0125eccfa9b7fd` | Auto-reply（微信自动化） | Python 图像识别 | 夭折，残留未清理 |

## 命名史

- 「川流不息」与「Personal Dashboard」曾混用（产品名 vs 目录名）→ 统一为产品名「川流不息」
- 「衍生框架」目录名含两个空格，无独立 project 名——笔记挂 `project: 收益计算器` + `tags: 衍生框架`，本表承担映射
- 夭折项目（Auto-reply）不入册也不给 project 名，仅本表登记残留位置

## 项目页

- [[项目/Conver System/README|Conver System]]
- [[项目/川流不息（Personal Dashboard）/README|川流不息（Personal Dashboard）]]
- [[项目/Profit Calculator/收益计算器项目经验复盘|Profit Calculator（收益计算器）]]
- [[项目/Profit Calculator 衍生的框架/README|Profit Calculator 衍生的框架]]

> 新项目立项时在本表加一行（正式名/目录/技术栈），project 字段从本表取值。
