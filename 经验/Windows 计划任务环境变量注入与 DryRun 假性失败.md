---
type: experience
tags: [windows, scheduled-task, env, powershell, dry-run]
project: 代订场
date: 2026-08-21
summary: Windows 计划任务（schtasks/Register-ScheduledTask）不在任务定义里设环境变量；User 作用域 SetEnvironmentVariable 对 Interactive LogonType 可见。但 -DryRun 演练模式若去读真实 User 环境变量会假性失败——DryRun 未实际写入，自检必须跳过或改走模拟。
provenance: DEV_LOG#7 / commit 0e72461 / code-review Falsify 轴
priority: medium
---

# Windows 计划任务环境变量注入与 DryRun 假性失败

## 现象

`scripts/install_scheduled_task.ps1` 的 `-DryRun` 分支调用 `Test-EnvVars` 读取**真实** User 作用域环境变量——但 DryRun 模式只是打印命令、不实际写入。全新机器上必然全空 → 自检失败 → exit 1。演练模式假性失败。

## 根因

- Windows 计划任务（`Register-ScheduledTask`）的 `$Settings.Environment = "SYSTEM"` 写法不生效（schedule 任务不在任务定义里设环境变量）
- 可行注入方式：`[Environment]::SetEnvironmentVariable(key, value, "User")` 持久化到用户级，配合 `LogonType Interactive`（交互式登录）让任务以用户上下文运行并可见
- DryRun 自检逻辑错误：DryRun 未写入 → 读回必然失败

## 修复

```powershell
# DryRun 分支：跳过自检（未实际写入）
if ($DryRun) {
    Write-Step "[DryRun] 注册后自检略过（DryRun 未实际写入，请通过真实注册路径验证）"
}

# 真实注册分支：注册后执行自检
Test-EnvVars
Test-ScheduledTask
```

另：`Test-EnvVars` 对空值从 `exit 1` 改为警告（与脚本头部注释「缺项打印警告但不阻止注册」一致）。

## 通用教训

**DryRun / 演练模式的自检逻辑，只能验证「将要执行的动作」，不能读真实系统状态断言结果**。DryRun 与真实路径共享自检函数时，必须给 DryRun 分支显式跳过（或注入模拟值），否则会出现「演练反而失败」的荒诞结果。
