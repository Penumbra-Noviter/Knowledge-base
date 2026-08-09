---
type: lesson
tags: [前端, Vanilla JS, 架构, CRUD, 工厂模式, personal-dashboard]
date: 2026-08-06
project: 川流不息
source: 自身项目实践
summary: Vanilla JS 用 createCrudPage() 工厂函数（entity/fields/formTemplate/renderItem 配置）消除 CRUD 重复，特殊页面工厂返回后 override
provenance: git 4f686a5（2026-08-08 知识库批量入库）· 川流不息 2026-08-06 实践
status: verified
---

# 前端无框架项目的 CRUD 页面工厂模式

## 症状

10 个功能页面的 CRUD 操作模式完全一致：列表渲染 → 表单弹窗 → 创建/编辑 → 删除确认。每个页面都在重复同样的 `mount()` 函数结构，HTML 模板相似，事件绑定逻辑雷同。代码量膨胀，修改通用行为需要改 10 个文件。

## 根因

Vanilla JS 无框架项目没有组件化机制，每个页面独立编写时天然倾向于复制粘贴。CRUD 是最常见的重复模式，但团队成员（或单人）容易忽略提取公共模式的机会。

## 代价

- 重复代码量：每个页面约 80-120 行 CRUD 样板，2 页面重构后各降至 ~50 行
- 后续维护：通用行为修改（如 Toast 反馈、错误处理）只需改工厂一处

## 教训

**Vanilla JS 无框架项目通过工厂函数消除 CRUD 重复：`createCrudPage()` 接收配置对象（`{entity, fields, formTemplate, renderItem}`），返回 `{mount, unmount}`。** 工厂封装了 `loadList()`、`showForm()`、`handleCreate()`/`handleUpdate()`/`handleDelete()` 的完整生命周期。有特殊需求的页面在工厂返回后做 `override` 扩展。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）

- 新增纯 CRUD 页面时，优先使用工厂函数，不手写重复模式
- 工厂函数的配置对象保持简单（entity + fields + template + render 四个核心）
- 有特殊需求的页面使用工厂 + 覆盖（override）模式，而非绕过工厂重写

## 关联

- [[Rust 通用 CRUD 模式消除样板代码]]
