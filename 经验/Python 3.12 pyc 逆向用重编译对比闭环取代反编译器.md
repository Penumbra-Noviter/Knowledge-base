---
type: lesson
tags: [逆向, Python, PyInstaller, bytecode, 验证]
date: 2026-08-23
project: 通用
source: 自身项目实践
summary: Python 3.12+ pyc 无反编译器可用时，用"字节码重建+重编译对比闭环"替代，语义等价可机器证明
provenance: 2026-08-23 抖音视频下载器逆向会话 · D:\Desktop\tools\抖音视频下载器_reversed\REPORT.md
status: verified
---

# Python 3.12 pyc 逆向用重编译对比闭环取代反编译器

## 症状

目标为 PyInstaller onedir 打包的 Python 3.12 程序，需要完整还原源码：
- `uncompyle6`/`decompyle3` 只支持到 3.8/3.9
- `pycdc` 系（含 `pydumpck`）后端 xdis 不识别 3.12 的 marshal 格式，直接报 `Unknown type 100 (hex 64)`
- `pylingual`（对 3.12 效果最好的反编译器）无本地 PyPI 包，纯在线服务，受无云依赖约束排除

## 根因

Python 3.10+ 字节码与 marshal 格式变动频繁，传统反编译器生态断层；而 PyInstaller 不 strip pyc——变量名、行号表、docstring、常量全部保留，`dis` 全量导出后信息足以人工重建。真正缺的不是信息，是"重建结果正确性"的证明手段。

## 代价

反编译器试错约 20 分钟；人工重建 3 模块 53 个代码对象（856 行）约 2 小时，但一步到位拿到机器证明的等价性，无一轮"看起来对其实错"的返工。

## 教训

**反编译器缺位时，闭环方向反转：不是"pyc→源码"而是"源码→pyc"——重建源码后重编译，与原 pyc 递归对比 co_code/co_consts/co_names/co_varnames/co_flags/异常表，逐指令一致即语义等价的机器证明。** 该方法质量上限高于任何自动反编译器（自动工具输出需人工校验，此闭环本身就是校验）。

## 防复发

- [ ] 已落实？本条为方法论文档，无代码守卫需求

落地清单（复用时照抄）：
1. `pyinstxtractor-ng` 解包 → 定位自有 pyc（PYZ 里 639 个模块中仅 3 个非依赖，先按顶层名单筛选，避免无效工作量）
2. dump 脚本导出全量 `dis`（**argrepr 不截断**）+ 完整 `co_consts`，两份文件就是重建的全部依据
3. 按反汇编行号（co_firstlineno/starts_line）反推源码行布局重建 .py，docstring/SQL/JS 从 consts 逐字符复制
4. `verify.py` 递归对比；不一致时用 difflib 对齐指令序列定位第一处真差异（跳转目标差是下游后果，先看指令增删）
5. 查捆绑 `python3xx.dll` 的 VersionInfo 拿精确小版本（本次 3.12.3），下载对应 embeddable 做终验

### 三个必踩的坑

| 坑 | 现象 | 解法 |
|---|---|---|
| `compile()` 继承调用方 future 状态 | verify 脚本自身的 `from __future__ import annotations` 污染被编译代码，co_flags 多出 0x1000000 | `compile(..., dont_inherit=True)` |
| 模块级 import 影响 LOAD_GLOBAL 编码 | 函数内 `os.startfile()` 的 NULL 位：模块级有 `import os` 产 `NULL+os`（A 编码），无 import 产 `NULL\|self+startfile`（B 编码）；对照实验 stub 忘写 import 则永远对不上 | 实验最小复现必须连同模块级 import 一起复制 |
| return 指令内联掩盖源码形态 | 3.12.0–3.12.10 全系把 try/if-else 中的 return 内联到分支尾；原 pyc 中"JUMP_FORWARD 跳到 try 范围外的 RETURN"布局无任何官方编译器可产生 | 逐一实测 11 个小版本仍不匹配 → 判定 pyc 被后处理，采用语义等价结构（`try/except[return]/else[return]`）并在报告注明差异 |

### 判断语义等价差异的门槛

差异指令 ≤2 条、控制流路径全覆盖一致（本例 5 条路径：双分支成功/异常/前置条件失败/未选中，行为全部一致）→ 接受并文档化；路径行为有任何分歧 → 必须继续修源码（此闭环曾抓出 `ok=True` 误放 with 块内、缺 `nonlocal`、`tk.Label` 写成 `Label` 三处真实语义差异）。

## 关联

- [[PyInstaller 逆向方法]]
