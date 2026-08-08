---
type: index
tags: [personal-dashboard, project, 川流不息]
date: 2026-08-06
---

# 川流不息 · 个人工作台（Personal Dashboard）

产品名「川流不息」，英文名 Personal Dashboard，仓库目录 `D:\Desktop\Craft\Personal Dashboard`。知识库内 `project` 字段统一用「川流不息」（曾与目录名混用，见 [[项目/README|项目注册表]]）。

技术栈：Tauri v2 + Rust（rusqlite bundled）+ Vanilla JS ESM + Vite（`root=src`）+ Vitest + WebDriverIO e2e + SQLite WAL（`PRAGMA user_version` 迁移 V1-V5）。

状态：Phase 6 全部交付（10 功能模块 + 107 Rust 测试 + 70 前端测试），2026-08-06 架构加深完成，进入维护/迭代期。

**单一事实来源**：技术细节、ADR、TO-TICKETS、DEV_LOG 都在项目仓库。本目录只放经验索引与遗留问题指针。

## 相关经验

- [[CSS 单文件到模块化拆分策略]] — 2890 行 → 15 模块文件
- [[Rust 通用 CRUD 模式消除样板代码]] — `Repository<T>` + `impl_repo!` 宏
- [[Tauri v2 IPC 前端调用差异]] — 无全局 `__TAURI__`；顶层参数名 camelCase→snake_case
- [[Vite root模式静态资源路径]] — 静态资源放 `src/public/` 而非 `src/assets/`
- [[worktree膨胀26G教训]] — `.claude/worktrees/` 26G 编译缓存
- [[前端无框架 CRUD 页面工厂模式]] — `createCrudPage()` 工厂
- [[并行 worktree 合并静默覆盖]] — 「零冲突」≠ 零丢失
- [[ISO 周边界不能硬编码]] — 52/53 周边界，当前代码仍是错的
- [[Tauri v2 E2E 基建必踩配置]] — 外部 tauri-driver + onPrepare 构建

## 遗留问题（2026-08-06 知识库审计发现）

> 审计发现 ≠ 已入账。正式待办一律以仓库 `TO-TICKETS.md` 为准，以下为需用户拍板的指针：

- ISO 周 53 硬编码未根治（`src/pages/weekly_review.js:162-175`）→ 应入 TO-TICKETS
- e2e 年份断言 `toContain("2026")` 2027-01-01 必炸（[[固定日期测试是定时炸弹]]复发登记）
- 版本号 0.1.0 漂移 `package.json`/`tauri.conf.json`/`Cargo.toml` 三处（[[文档漂移]]复发登记）
- 开屏图被 `.gitignore` 忽略（`src/public/*.png`），全新克隆缺图、`splash.e2e.js` 会失败
- git 作者仍是 olina（昵称改名 Kami 未覆盖 git config）
- 项目文档缺 e2e 运行说明（tauri-driver 启动步骤）
