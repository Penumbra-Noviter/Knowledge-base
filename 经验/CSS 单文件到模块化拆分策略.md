---
type: lesson
tags: [CSS, 前端, 架构, 重构, personal-dashboard]
date: 2026-08-06
project: 川流不息
source: 自身项目实践
status: verified
---

# CSS 单文件到模块化拆分策略

## 症状

一个 `style.css` 文件膨胀到 2890 行，包含全局样式、布局、组件、页面样式、动画等所有内容。查找特定页面的样式需要在 3000 行中搜索，修改时不确定影响范围，合并冲突频繁。

## 根因

项目早期为了快速迭代将所有 CSS 放在一个文件中。随着功能模块从 1 个增长到 10 个，单文件模式没有跟随扩展。Vanilla JS 项目没有 CSS 模块化机制（如 CSS Modules / styled-components），需要自建拆分策略。

## 代价

- 重构结果：2890 行 → 15 个模块文件
- 查找效率大幅提升（页面样式在 `css/pages/` 下找）
- 修改范围清晰（`base.css` 改变量，`components.css` 改组件）

## 教训

**CSS 模块化拆分按层递进：`base.css`（变量 + 重置）→ `layout.css`（侧边栏 + 内容区）→ `components.css`（卡片/按钮/表单/弹窗等通用组件）→ `animations.css`（动画关键帧）→ `pages/*.css`（各页面专属样式）。** 一个汇总文件 `style.css` 做 `@import` 集中管理。规则：变量/重置放 base，布局放 layout，跨页面复用放 components，页面专属放 pages/。

## 防复发

- [ ] 新项目一开始就建立 CSS 模块化结构，不等到 2000+ 行再拆
- [ ] 明确每个 CSS 文件的职责边界（base / layout / components / animations / pages）
- [ ] 页面专属样式不注入 JS 字符串（`PAGE_STYLES`），放入 `css/pages/` 文件
- [ ] 汇总文件 `@import` 顺序：base → layout → components → animations → pages

## 关联

- [[前端无框架 CRUD 页面工厂模式]]
