# demo 知识库结构规范

本项目即 Obsidian 知识库（`D:\Desktop\knowledge base\demo`），记录软件项目开发的学习/经验/日记。

## 目录结构
- `Home.md` — 门户索引（MOC）
- `收件箱/` — 快速捕捉，每周清空（内含 `README.md` 说明）
- `归档/` — 废弃的每日日记（2026-08-08 起原始观察只记项目仓库 DEV_LOG，无日记；旧日记与日记模板已移入）
- `项目/` — 每项目一子目录：`Conver System`（当前主力）、`Profit Calculator`（已交付）、`川流不息（Personal Dashboard）`（Tauri，已交付）、`Profit Calculator 衍生的框架`（PySide6 骨架，实验产物）；另有 `项目/README.md` 项目注册表（project 字段 ↔ 目录 ↔ 技术栈 的映射，2026-08-06 建立）
- `经验/` — 原子避坑笔记（每条一坑）
- `学习/` — 学习笔记（CSS / Git / Skills / 提示词）
- `来源/` — 参考资料与来源 URL
- `附件/` — 图片等
- `模板/` — `日记模板.md`、`经验模板.md`、`项目预检模板.md`

> 命名历史：最初用编号（00/10/20/30/…）强制排序 → 用户嫌数字感 → 2026-08-02 改为**纯中文无前缀名**；**顺序固定靠 Obsidian 插件（File Tree Alternative 拖拽排序），不在文件夹名上加前缀**。

## frontmatter 规范
所有笔记必须带 frontmatter：
- `type`: `lesson` | `study` | `diary` | `index`
- `tags`: 小写 kebab-case，项目标签如 `conver-system` / `profit-calculator`
- `date`: YYYY-MM-DD
- `project`: 收益计算器 / Conver System / 川流不息 / 通用（lesson 必填；取值以 `项目/README.md` 注册表为准，衍生框架无独立 project 名，挂 `收益计算器` + `tags: 衍生框架`）
- `source`: URL，或「自身项目实践」
- `status`: `verified` | `unverified`
- `verify`: 验证方法（unverified 必填；写明怎么验、验什么后允许升格，如「对照本会话可用 skills 清单核对」；每次项目预检时顺手清一批）

## 单一事实来源原则
- **待办只在项目仓库 `TO-TICKETS.md`**；ADR / 技术细节 / CODE_WIKI 在项目仓库；vault 只引用不复制，绝不在 Obsidian 另建一套待办
- **原始观察唯一来源 = 项目仓库 `DEV_LOG.md`**（无日记，日记已废弃归档）；阶段完成时从 DEV_LOG 批量升格为 `经验/` 原子笔记
- **预检读取按 frontmatter `project:` 字段限定**（Obsidian 搜索 `path:经验 project:"<注册表项目名>"`），不是 tags——tags 回归主题维度，不承担项目归属（2026-08-08 审计：按 tag 读会漏读无项目 tag 的笔记）
- **经验笔记「防复发」段无 checkbox**：惯例项 = 开工预检对照执行的普通列表；动作项 = 指引去向（→ 对应项目 TO-TICKETS / [[项目预检模板]] §三），只有已落实事实才保留 `[x]`
- 经验模板字段：症状 → 根因 → 代价 → 教训 → 防复发 → 关联
- **AI 生成的知识必须标 `status: unverified`**；只有在本库项目里实测验证过，才允许升格为 verified 实践知识
- **知识库闭环契约**：全局 CLAUDE.md「3.5」+ `学习/项目驱动知识库模式.md`，本 memory 只索引不复制

## 关联
- `mem:global/dev-workflow` — 全局开发流程（memory 不存待办、TO-TICKETS 单点）
- `mem:global/cbm-usage` — CBM 用法（架构/死代码专用）
