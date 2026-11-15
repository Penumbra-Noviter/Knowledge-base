---
type: lesson
tags: [robocopy, junction, reparse-point, 数据丢失, PowerShell, 镜像删除]
date: 2026-11-13
project: ctrl 脚本管理器
summary: robocopy /MIR /XJ 只约束遍历、不保护目的端树内 junction 子项——镜像删除会清空链接目标的真实文件，/XJ 不是数据安全护栏
provenance: DEV_LOG〈2026-11-13 td-consume-27 批次〉· 波 1 增量审核 F1 · commit 01c800a（kickoff/T-136-shared-seam）· 主会话两次独立复现（raw robocopy + AST 提取驱动）
status: verified
agents_md_feedback: 印证 §三.1「对抗性自检——证伪而非证实」：该缺陷潜伏两个批次（T-134/T-135 评审均未击中），靠 Falsify 构造"树内含 junction"输入才击穿
---

# robocopy /MIR /XJ 穿透树内 junction，清空链接目标

## 症状

`robocopy <空目录> <dest> /MIR /XJ /R:0 /W:0` 清空目的端目录树时：`dest\sub\jlink`（junction → 树外 `outside\`）被删，且 **`outside\precious.txt`、`outside\keep.txt`（树外真实文件）一并消失**，rc=2（<8，不被判失败）。调用方随后 re-probe 目标不存在 → 返回成功，状态报「已清理、残留 0」。

## 根因

`/XJ`（exclude junction points）只约束 robocopy 的**遍历/复制**方向；`/MIR` 的镜像删除（purge）在目的端做「删多余项」时**不尊重 /XJ 对树内 junction 子项的保护**——删除路径进入链接目标。注释声称的 reparse 保护若只覆盖「删除目标本身是 junction」（顶层判定），树内嵌套 junction 完全不设防。T-135 评审 F2 实证过「目标本身是 junction」的变体，本轮 F1 才击中「树内子级 junction」变体——同一 API 的两个穿透面。

## 代价

潜伏两个批次（T-134 / T-135 评审均未击中）+ 一轮增量审核返工。生产形状可达且危险：pnpm/npm 的 `node_modules` 天然大量含 junction，若清理工具的删除兜底触发在这类树上，树外真实项目文件会被静默删除并谎报「已清理」。幸未发生真实数据损失（夹具先行）。

## 教训

把外部工具当「数据安全护栏」前，必须为**护栏语义的每个边界**单独构造证伪实验——`/XJ` 保护的是遍历不是删除；「排除」≠「不触碰」。凡涉及删除的目录遍历，宁可自己写逐层枚举 + 显式 `ReparsePoint` 判定（只删链接本体、永不下降），也不用镜像类工具的隐式行为。

## 防复发

- [x] 已落实：目录清空整块改 `.NET` 递归（`EnumerateFileSystemEntries` + 每层判 `[IO.FileAttributes]::ReparsePoint`，命中只删本体不下降），`& robocopy` 全文 0 可执行命中（commit 01c800a，`Remove-DirectoryTree`）
- [x] 已落实：回归用例 `test_ast_unit_remove_path_extended_does_not_cross_nested_junction`（树内 junction + 树外文件存活断言）；期末突变抽查 3（`IsReparse` 判定改恒 false → 5 用例全红，含「已清理」谎报变红）
- [ ] 通用：任何「删除 + 第三方工具」组合，护栏边界逐个证伪（遍历/删除/清除/压缩各方向分开测）

## 关联
- [[对抗探针先沙箱重定向再探测]]
- [[宿主进程句柄锁住经symlink链解析的物理目录]]
- [[参考实现的实证结论必须在本机复验]]
