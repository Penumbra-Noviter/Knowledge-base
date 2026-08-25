---
type: lesson
tags: [LLM, 环境, pytest, 汇总, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
summary: Conver System 零散坑汇总——底层异常映射可读提示、Git Bash 遮蔽 MSVC linker、pytest Test* 命名避让、流式 UI 首 token 才建气泡、易退役常量集中配置
provenance: git 4f686a5（2026-08-08 知识库批量入库）· Conver System 2026-08-06 实践
status: verified
---

# Conver System 高频小坑汇总

Conver System 的其他零散小坑，按「能复用」程度整理。每条都是真实踩过的。

| 坑 | 教训 |
|---|---|
| relay 中转返回非标准结构时抛裸 `'str' has no attribute …`，用户看到无法理解的报错 | LLM 调用处对「属性访问类异常」（`has no attribute` / `not subscriptable`）单独映射为「请检查 API 地址是否为兼容协议的端点」的可读提示（`errors.py:66-72`） |
| Git Bash 里 `cargo build` 链接失败，cmd/PowerShell 正常 | Windows 下 Git Bash 的 `/usr/bin/link.exe` 遮蔽 MSVC linker——Rust/MSVC 工具链编译在 cmd/PowerShell 跑；装完工具链立即冒烟验证（`docs/tauri-setup.md:21-25`） |
| 定义 `TestXxx` 命名的 schema 后 pytest 告警「可能被误认为测试类」 | pytest 默认按 `Test*` 前缀收集测试类——非测试类（尤其 schema/模型）命名避开 `Test` 前缀（`ConnectionTestRequest` 可，`TestConnectionRequest` 不可） |
| 流式对话先创建空白 assistant 气泡；错误发生在首 token 前时 `assistantDiv` 为 null 崩溃 | 流式 UI 先显示 thinking 指示器，**第一个 token 到达才创建气泡**；错误在首 token 前发生要移除指示器并兜底建错误气泡（防 null 引用） |
| 换模型 `claude-sonnet-4` → `claude-sonnet-5` 需同步 11 个文件（含 `.env.example`、`state.js`、`index.html`） | 模型名/版本号这类「会退役」的常量应集中 config，否则每次换模型牵动全局——「会变的值」永远不散落 |
| 改完前端代码，打包后发现旧 UI 还在 | `dist/conver_backend/conver_backend.exe` 内嵌前端静态文件，但它的构建时间戳无任何提示可查（`build-backend.ps1` 产出、`build-desktop.ps1` 按需补齐）。前端改动后必须**主动重跑 `build-backend.ps1`** 再打壳，否则 `tauri build` 打包的是一个静默过期的后端包——没有构建失败、没有日志警告，直接交付旧 UI |

## 通用教训提炼

1. **用户可见错误要翻译**：SDK/底层异常在边界处映射为领域可读提示
2. **Windows 工具链环境坑先文档化**：Git Bash 遮蔽、PATH、行尾，装完即验证
3. **命名空间即契约**：pytest 收集规则是全局命名空间，业务命名主动避让
4. **流式 UI 的 DOM 时机**：气泡创建绑定「有内容」而非「流开始」
5. **会退役的常量集中化**（模型名、版本号）——散落即漂移，见 [[文档漂移]]

## 关联
- [[SSE 流式前端状态陷阱]]（首 token 气泡是其渲染时机侧）
- [[文档漂移]]（模型名散落 11 文件）
- [[测试与架构高频小坑汇总]]（收益计算器对应物）
