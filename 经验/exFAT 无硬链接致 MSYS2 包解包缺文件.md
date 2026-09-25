---
type: experience
tags: [环境, Windows, MSYS2, exFAT, 构建]
date: 2026-09-25
project: 收益计算器
summary: exFAT 不支持 hardlink/symlink——MSYS2 的 gcc.exe/ld.exe/liblto_plugin/moc 等 hardlink 文件解包时全部缺失（pacman 报 Can't create）；从缓存包 tar 解出真身手动复制补齐；MSYS=winsymlinks:copy 只影响 symlink 不影响 hardlink
provenance: DFD-Cpp 全量复现 kickoff 2026-09-22 · F:\Craft\DFD-Cpp\docs\BUILD_NOTES.md
status: verified
---

# exFAT 无硬链接致 MSYS2 包解包缺文件

## 现象
F 盘 exFAT。MSYS2 装 gcc/qt6 后编译报缺文件：`g++: fatal error: '-fuse-linker-plugin', but liblto_plugin.dll not found`；`cannot find 'ld'`；moc/rcc 缺失。pacman 解包日志有 `Can't create ...` 警告（hardlink 条目）。

## 根因
exFAT 文件系统不支持 hardlink（与 symlink 都是 NTFS 特性）。MSYS2 包内大量 exe 是 hardlink（gcc.exe→cc.exe、g++.exe→c++.exe、ld.exe→ld.bfd.exe、liblto_plugin.dll→bfd-plugins 真身、qmake6.exe→qmake.exe、moc 在 share/qt6/bin）。pacman/tar 解包 hardlink 条目 → `Operation not permitted` 静默跳过或警告，文件缺失。

## 修复（通用套路）
```bash
# 从缓存包解出真身（tar -xf 对 hardlink 条目失败但真身会解出）
tar -xf /var/cache/pacman/pkg/<pkg>.pkg.tar.zst -C /tmp/x
# 按 tar -tvf 的「连接到」关系，把真身复制为缺失文件（hardlink 语义 = 内容相同，复制等价）
cp /tmp/x/mingw64/bin/cc.exe /mingw64/bin/gcc.exe   # 以此类推
```
`MSYS=winsymlinks:copy` **只影响 symlink，不影响 hardlink**——试过无效。

## 判别
`tar -tvf pkg | grep -E "连接到|link to"` 直接看 hardlink 关系；`ls` 缺失文件 + 同包真身大小相同即可对号。

## 相关
windeployqt 复制的是普通文件，exFAT 打包机无 hardlink 坑；但 git 在 exFAT 需 `safe.directory` 例外；Git Bash /tmp 映射 D:	mp（非系统 Temp）。
