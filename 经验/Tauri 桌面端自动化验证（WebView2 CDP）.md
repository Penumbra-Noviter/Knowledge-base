---
type: lesson
tags: [tauri, 桌面验证, 自动化]
date: 2026-08-14
project: Conver System
source: 自身项目实践
summary: Tauri dev 窗口可用 WebView2 CDP 自动化验证（--remote-debugging-port 环境变量 + playwright-core connectOverCDP）；dev 后端拉起需 CONVER_BACKEND_CMD 路径引号 + cwd 指仓库根
provenance: DEV_LOG 2026-08-13/14（模拟器原型 + U7 kickoff 会话）· commit 37affb8/48e6e6c/3ab60e6 · 2026-08-14 Conver 会话
status: verified
---

# Tauri 桌面端自动化验证（WebView2 CDP 法 + dev 启动坑）

## 症状

- 桌面壳（tauri dev 窗口）无法用 Playwright 常规方式控制——无 devtools 连接入口，GUI 冒烟只能目视
- tauri dev 拉起后端失败：`ModuleNotFoundError: No module named 'backend'` / `启动后端进程失败: 系统找不到指定的文件 (os error 2)` / `program path has no file name`

## 根因

1. **无自动化入口**：Tauri v2 dev 窗口是 WebView2 渲染，Playwright 的 launch 无法 attach；需要 WebView2 的 CDP 调试口
2. **dev 后端启动三坑**（壳在 src-tauri 目录 spawn `python -m uvicorn backend.app.main:app`）：
   - cwd 是 src-tauri → `backend` 模块不可导入 → 需 `CONVER_BACKEND_CWD` 指向仓库根
   - `python` 解析到系统 Python 而非 venv → 需 PATH 前置 venv 或 `CONVER_BACKEND_CMD` 显式指定 venv python
   - **CONVER_BACKEND_CMD 路径含空格**：parse_command_line 按空格拆 token，`D:\Desktop\Craft\conver system\.venv\...` 被拆断 → os error 2；cmd 的 `set "VAR=..."` 不做 `\"` 转义（`\"` 是字面量反斜杠+引号），正确写法是值内嵌干净双引号：`set "CONVER_BACKEND_CMD="D:\...\python.exe" -m uvicorn ..."`（cmd set 保留内部引号）
3. **MSYS 干扰**：Git Bash 下 `MSYS_NO_PATHCONV=1` 会禁用 `//c` → `/c` 转换，`cmd //c` 失效进交互模式；且 MSYS 路径转换会改写 `D:\...` 形式的 set 值——最稳做法是写 .cmd 脚本文件让 cmd 直接执行

## 代价

- 桌面版验证靠目视 → 无法断言、无法进 CI；排查 dev 后端启动坑耗时约 3 轮重启（os error 2 → program path has no file name → 成功）

## 教训

- 桌面端 UI 验证的自动化法：`WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS=--remote-debugging-port=9222` 环境变量（tauri dev 启动前设置）→ WebView2 开 CDP 口 → playwright-core `chromium.connectOverCDP('http://127.0.0.1:9222')` 驱动窗口（页面 = boot.html 跳转后的后端 URL；同源 iframe 可 frameLocator 深探游戏内部 DOM）
- 壳拉起的后端子进程随壳树杀（taskkill //T //F）连带回收；验证后 netstat 复核端口

## 防复发

- 桌面验证一律走 CDP 法（原型 + U7 两次实证，5 步断言全过）
- dev 后端启动参数（CONVER_BACKEND_CMD 引号形式 / CONVER_BACKEND_CWD / venv PATH）固化为 .cmd 启动脚本，不再手工拼
