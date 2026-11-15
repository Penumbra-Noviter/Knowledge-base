---
type: lesson
tags: [PowerShell, PS-5.1, Join-Path, 长路径, 扩展路径, \\?\]
date: 2026-11-13
project: ctrl 脚本管理器
summary: PS 5.1 Join-Path 不接受 \\?\ 前缀路径（ArgumentNullException/DriveNotFoundException），拼接扩展路径必须用字符串拼接
provenance: DEV_LOG〈2026-11-13 td-consume-27 批次〉· 期末修复轮次 1 stub 调试附记 · commit cf4f1fd · 双形态实测
status: verified
agents_md_feedback: 无冲突；印证 §二.2「动手前自检——API 行为查证不凭记忆」
---

# PS 5.1 Join-Path 拒收 \\?\ 扩展路径前缀

## 症状

`Join-Path '\\?\D:\x' 'y'` 在 PowerShell 5.1 下两种失败形态交叉出现：`ArgumentNullException("drive")` 与 `DriveNotFoundException '\\?\D' does not exist`——且报错信息完全不指向「前缀不支持」，看起来像路径不存在或参数为空。

## 根因

`Join-Path` 内部按盘符语义解析路径（`Split-Path -Parent/-Leaf` 同族），`\\?\` 扩展路径前缀不在其认知内，被当坏盘符处理。`.NET` 托管 API（`[IO.File]`/`[IO.Directory]`/`EnumerateFileSystemEntries`）原生接受 `\\?\`，但 **PowerShell cmdlet 层的路径工具链不认**——与「PS 5.1 cmdlet（Remove-Item/GCI）不接受 \\?\ 前缀」是同一家族限制（T-135 已实证）。

## 代价

测试 stub 初版用 `Join-Path` 拼 ExtPath 造出假红，排查一轮（两种异常形态交叉验证才定位到前缀）。

## 教训

在 PS 5.1 上，`\\?\` 前缀路径只在 **.NET 托管 API 层**可用；凡 PowerShell 层的路径工具（`Join-Path` / `Split-Path` / `Resolve-Path` / cmdlet 的 `-Path` 参数）都要默认它不认识扩展路径——拼扩展路径一律字符串拼接，且不要用 cmdlet 做二次解析。

## 防复发

- [x] 已落实：仓库枚举器 `Get-FsEntries` 契约中 ExtPath 自带 `\\?\` 前缀、全部下游用字符串拼接（`Remove-DirectoryTree` / `Get-DevMatches` / 计量入口均无 Join-Path）
- [x] 已落实：stub 用例注释记录本事实（`tests/test_clean_cache_plugin.py` AttrError 用例调试附记），下次写夹具先看到
- [ ] 通用：写 .ps1 时先确认宿主（PS 5.1），含 `\\?\` 的路径只走 [IO.*] 层

## 关联
- [[PowerShell 5.1 脚本环境假设陷阱]]
- [[powershell -File 模式数组参数陷阱]]
- [[robocopy MIR XJ 穿透树内 junction 清空链接目标]]
