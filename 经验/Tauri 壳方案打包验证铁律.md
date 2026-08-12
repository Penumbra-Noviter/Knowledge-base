---
type: lesson
tags: [Tauri, Rust, PyInstaller, 打包, 测试基建, 桌面应用]
date: 2026-08-11
project: Conver System
source: 自身项目实践
summary: Tauri 壳方案三铁律：resources 编译期校验→构建顺序前置；Windows resources 装 _up_ 子目录保留相对结构；冒烟必须含不注入 env 的干净环境回归
provenance: DEV_LOG 2026-08-11 P6.4 期末审核（阻断1/2 + 复审整改）· Conver System P6.4 kickoff
status: verified
---

# Tauri 壳方案打包验证铁律

## 症状

P6.4 期末四轴 code-review 发现 2 条阻断，全部在「打包链」上，且冒烟全绿却抓不住：
1. 壳 prod 无条件 `spawn("python -m uvicorn ...")`——干净用户机（无 python）安装后双击必失败；冒烟全绿是因为**冒烟自己注入了 CONVER_BACKEND_CMD 环境变量**，掩盖了真实用户路径。
2. 打包后端 `datas=[]` 不随包挂载前端 → webview 就绪后跳转根路径 404，验收 9 在交付物上无法执行；冒烟只测 `/api/models` 等 API 契约，不测 `GET /`。

## 根因

「壳 ↔ 打包后端 ↔ webview」三端契约没有在自动化中闭环：冒烟测的是**带 env 注入的开发者路径**，不是**真实用户双击路径**；打包配方（spec datas）与运行态假设（UI 挂载）未联动验证。

## 教训

1. **冒烟必须含「不注入任何环境变量的干净环境回归」**——注入 env 的冒烟全绿 ≠ 真实用户路径可用。安装器形态冒烟 + 不注入 CONVER_BACKEND_CMD + 断言 runtime.json ready + API 200 + GET / 200，才是交付验收的正式形态。
2. **tauri.conf.json 加 `bundle.resources` 后，tauri-build 在编译期校验资源路径存在性**——`cargo test` 在干净检出（无 dist/）直接失败。构建脚本必须把资源产出（build-backend）前置到任何 cargo 编译之前，否则一键构建链在干净检出/CI 必挂。
3. **Tauri Windows 把 resources 装在安装目录 `_up_` 子目录且保留相对路径结构**（`%LOCALAPPDATA%\Conver System\_up_\dist\conver_backend\conver_backend.exe`）——随包资源的运行时定位必须做候选探测（实测布局 + 平铺兜底），不能写死安装目录。
4. **打包回归断言要钉「接线行」而非 token**：`datas=list(_FRONTEND_RUNTIME)` 能抓到 `datas=[]`（原始缺陷只改接线行、不改定义），而「断言 spec 文本含 frontend/index.html token」的写法对 `datas=[]` **漏报**（token 全在定义里）。

## 代价

期末审核 + 修复 + 复审 + 全链重跑：约半天工作量；2 条阻断修复均为小改动（Rust prod 分支 + spec 1 行 + 冒烟断言），但暴露了「冒烟绿 ≠ 交付可用」的验证盲区。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）

- 任何壳/打包类交付：验收冒烟必须含**干净环境（不注入 env）安装器形态**用例
- 加 bundle.resources → 同步检查构建脚本顺序（资源产出先于 cargo 编译）
- 打包配方（spec datas）改动 → 断言必须钉接线行 + 打包态 GET / 端到端断言
- 新 Tauri 项目/换 Tauri 版本 → 复核 resources 安装布局（_up_ 行为可能随版本变）

## 关联

- [[打包覆盖丢数据]]（同为桌面打包铁律族）
- [[Tauri v2 E2E 基建必踩配置]]（同为 Tauri 测试基建坑）
- [[Tauri v2 IPC 前端调用差异]]（同为 Tauri v2 文档默认值陷阱族）
