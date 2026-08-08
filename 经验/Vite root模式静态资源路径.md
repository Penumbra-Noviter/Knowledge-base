---
type: lesson
tags: [Vite, 前端, 构建, 静态资源, personal-dashboard]
date: 2026-08-06
project: 川流不息
source: 自身项目实践
status: verified
---

# Vite `root: "src"` 模式下静态资源路径的陷阱

## 症状

将图片（如 `splash-mascot.png`）放入 `src/assets/` 目录，开发模式正常显示，但 `vite build` 构建后图片不在 `dist/` 中，页面 404。

## 根因

Vite 配置 `root: "src"` 时，构建的**静态资源根目录**变为 `src/public/` 而非项目根目录的 `public/`。`src/assets/` 是给 JS 动态 import 用的（Vite 会 hash 文件名），不会被自动拷贝到 `dist/` 根目录。只有 `src/public/` 下的文件才会被逐字复制到 `dist/` 根。

## 代价

- 排错约 20 分钟（尝试了多个路径组合）
- 需将图片从 `src/assets/` 移到 `src/public/`

## 教训

**Vite `root: "src"` 模式下，希望被逐字复制到构建产物的静态资源必须放 `src/public/`，而非 `src/assets/`。** `assets/` 只给 JS 动态 import 用。开发模式正常不代表构建正确。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）

- 新项目检查 Vite `root` 配置，确认 `public/` 路径相对于 root
- 静态资源（图片、字体等）放 `{root}/public/` 而非 `{root}/assets/`
- 构建后检查 `dist/` 确认资源文件存在

## 关联

- [[Tauri v2 IPC 前端调用差异]]
