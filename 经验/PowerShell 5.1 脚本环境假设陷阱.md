---
type: lesson
tags: [PowerShell, Windows, 编码, 脚本]
date: 2026-08-29
project: 通用
source: 自身项目实践
summary: Windows 上写 .ps1 按 PS 5.1 假设：自动变量只读（$PID 不能赋值），中文须 UTF-8 BOM 否则按 ANSI 乱码
provenance: 2026-08-29 ZCode 会话（capture_window.ps1 开发实测：EnumWindows 回调 `$pid=...` 报只读错误；中文注释重编码 BOM 后正常）
status: verified
---

# PowerShell 5.1 脚本环境假设陷阱

## 症状
1. 脚本回调里写 `$pid = [uint32]0` 报 `Cannot overwrite variable PID because it is read-only or constant`
2. UTF-8 无 BOM 的中文注释 .ps1 在 Windows PowerShell 5.1 下显示乱码

## 根因
- `$PID` 是 PowerShell 内置**只读自动变量**（当前进程 ID），任何赋值都会抛错；与 `$pid` 大小写不敏感
- PS 5.1 读无 BOM 的 .ps1 按 ANSI（中文系统=GBK）解码，UTF-8 中文自然乱码；pwsh 7 默认 UTF-8 无此问题——同一文件两个宿主表现不同

## 代价
一次运行失败 + 修复；中文乱码若不知 BOM 机制，会误判为"文件损坏"而浪费时间重写。

## 教训
在 Windows 上写 .ps1 先确认宿主：**目标是 PS 5.1 时，含非 ASCII 的 .ps1 必须写 UTF-8 BOM**；脚本内不要用 `$pid`、`$host` 等自动变量名做变量（改用 `$procId` 类）。

## 防复发
- [x] 生成 .ps1 后校验头三字节 `EF BB BF`（PowerShell: `[IO.File]::ReadAllBytes(...)[0..2]`）；无 BOM 用 `UTF8Encoding($true)` 重写
- [x] 回调/脚本内变量命名避开全部自动变量（`$PID`、`$HOME`、`$HOST`、`$PSHOME` 等）

## 关联
- [[powershell -File 模式数组参数陷阱]]
- [[非前台窗口截图按捕获手段分层]]