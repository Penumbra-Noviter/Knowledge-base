---
type: lesson
tags: [windows, 安全, 提权, 进程]
date: 2026-08-26
project: ctrl
source: 自身项目实践
summary: 以 .bat 为提权/中转目标时参数必经 cmd 二次解析，无字符串层转义口径，须直达 .exe
provenance: 会话 sess_034cc4d1 · commit e2f6a42..c76f8ce（T-011 修复轮次1）· DEV_LOG 2026-08-26
status: verified
---

# Windows 提权传输避开 cmd 二次解析

## 症状

`ctrl run shutdown --minutes 5` 经 UAC 提权后按默认 30 分钟执行；更深的 Falsify 探针发现参数值含 `&`/`|` 时可在提权上下文注入第二条命令（实测 `--comment "a&ver"` 使 ver 执行）、`%VAR%` 连引号内都展开（`100%PATH%` → PATH 全文）、`^` 被吞噬。

## 根因

重启链路以 ctrl.bat 为 ShellExecuteW 目标：bat 必经 cmd.exe 解析，参数串经历「CRT 切分 → cmd 再解析」两次口径。`list2cmdline` 只按 CRT 规则加引号，给人"已安全"的错觉——cmd 元字符层完全不在其防护范围。字符串层面不存在能同时满足 CRT 与 cmd 两套解析的可靠转义口径（微软官方亦无 escapeshellarg 等价物）。

## 代价

评审两轮才定位到传输层根因（第一轮只修了 argv 风格不匹配）；若被实际利用是提权上下文任意命令执行。

## 教训

Windows 上需要"逐字节保真传参"的中转/提权场景，目标必须是 .exe 直达（如 `sys.executable -m <pkg>`）；任何 .bat/cmd 中转等于放弃转义防线。环境变量传递优先于命令行拼接。

## 防复发

- [x] 已落实：elevator.run_as_admin 结构断言"不得路由 .bat"永久锚定 + 12 组元字符语料的字节保真 oracle 测试（test_elevation_roundtrip_*）
- PYTHONPATH 改为写入子进程环境变量继承，不再依赖 bat 脚本设置

## 关联
- [[Falsify测试要钉住缺陷所在层]]
- [[Windows GUI 壳拉起控制台子进程闪窗口]]
