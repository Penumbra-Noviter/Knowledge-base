---
type: lesson
tags: [PowerShell, 脚本, 参数传递, 调试]
date: 2026-08-11
project: Conver System
source: 自身项目实践
summary: powershell -File 模式下 @(...) 不被解析为数组字面量——整个字符串绑到位置参数导致 GetFullPath 非法字符错误；-File 传数组要用 -Command 或拆成独立参数
provenance: DEV_LOG 2026-08-11 P6.4 build-desktop.ps1 冒烟调用失败定位 · Conver System P6.4 kickoff
status: verified
---

# powershell -File 模式数组参数陷阱

## 症状

从 Git Bash 调用 `powershell -ExecutionPolicy Bypass -File scripts/build-desktop.ps1 -SmokeArgs @("-UseInstaller","-ForceKillStale")`，构建链全部成功，但冒烟启动即抛：

```
Exception calling "GetFullPath" with "1" argument(s): "Illegal characters in path."
```

## 根因

`powershell -File` 模式下，命令行参数按**字符串**绑定，`@(...)` **不被解析为 PowerShell 数组字面量**。整个字符串 `@("-UseInstaller","-ForceKillStale")` 被当作单元素数组传给 `-SmokeArgs`；脚本内 `& $SmokeScript @SmokeArgs` 展开时，该字符串变成**一个位置参数**，绑到冒烟脚本第一个参数 `$AppExe` → `[IO.Path]::GetFullPath('@("-UseInstaller","-ForceKillStale")')` → 非法字符异常。

交互式 PowerShell（或 `-Command` 模式）会正常解析 `@(...)`，所以脚本在交互式下无此问题——这是**调用方式**问题，不是脚本 bug。

## 教训

**`powershell -File` 传数组字面量：不要用 `@(...)`。** 三种正确姿势：
1. 拆成独立参数：`-SmokeArgs -UseInstaller -ForceKillStale`（[string[]] 会逐个绑定，switch 语义正确）
2. 用 `-Command` 包裹：`powershell -Command "& { .\build-desktop.ps1 -SmokeArgs @('-UseInstaller') }"`（-Command 模式解析完整 PS 语法）
3. 交互式 PowerShell 直接调用（无此问题）

## 代价

一次「构建链全绿但冒烟假失败」的定位（约 10 分钟）；错误信息指向 GetFullPath 而非参数绑定，误导排查方向。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）

- 从非交互 shell（bash/cmd）调 PowerShell 脚本传数组/复杂参数时，先确认 `-File` vs `-Command` 的解析差异
- 脚本错误是「字符串被绑到意外位置参数」时，先怀疑数组字面量传递
- 脚本入口参数默认值写「空即默认」防御（`if (-not $AppExe) { 默认路径 }`）能显著缩短此类定位

## 关联

- [[Tauri 壳方案打包验证铁律]]（同 P6.4 交付链的验证教训）
