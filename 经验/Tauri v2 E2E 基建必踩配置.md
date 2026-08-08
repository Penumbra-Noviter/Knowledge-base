---
type: lesson
tags: [Tauri, E2E, WebDriverIO, 测试基建, 川流不息]
date: 2026-08-06
project: 川流不息
source: 自身项目实践
status: verified
---

# Tauri v2 E2E 基建两处必踩配置

## 症状
`@wdio/tauri-service` 开箱配置连不上，报「连接被拒」难定位；e2e 首次运行要手动构建 debug 二进制才知道。

## 根因
Tauri WebDriver 自动化不是开箱即用：① tauri-driver 必须**外部启动**（`driverProvider: "external"`），由 wdio 服务拉起的方式在 v2 下不工作；② 被测的 debug 二进制要先构建（`onPrepare` 里 `tauri build --debug --no-bundle`）。两处都是「文档默认值 ≠ 实际行为」。

## 教训
Tauri v2 E2E 的**必踩配置清单**：
1. `driverProvider: "external"` + 手动先启动 tauri-driver（失败时先怀疑驱动进程）
2. `onPrepare` 钩子里先构建 debug 产物
3. e2e 独立目录 + 独立 package.json，隔离 wdio 全家桶依赖，不进主应用 devDeps

项目文档（CLAUDE.md / PROJECT_REFERENCE.md）完全没有 e2e 运行说明——下个项目/新机器首跑必踩。

## 代价
配置排查 + 文档缺失的隐性成本；`npm run test:e2e` 首次运行必踩。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- ⏳ e2e 运行说明（含 tauri-driver 启动步骤）→ 录入 川流不息 项目仓库 TO-TICKETS
- 新 Tauri 项目照此清单搭 e2e

## 关联
- [[Tauri v2 IPC 前端调用差异]]（同为 Tauri v2 文档默认值陷阱族）
- [[固定日期测试是定时炸弹]]（e2e 内同样有绝对日期断言，见该笔记复发登记）
