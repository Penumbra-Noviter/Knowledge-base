---
type: lesson
tags: [ctypes, HRESULT, 符号数, Windows, 测试假绿]
date: 2026-08-27
project: ctrl 脚本管理器
source: 自身项目实践
summary: ctypes 绑定返回 HRESULT 的 API 时 restype=c_long 是有符号返回——0x80070002 实返 -2147024894，与无符号常量 == 永不匹配，真机才触出；须按无符号语义归一（& 0xFFFFFFFF）并加带符号位模式回归测试
provenance: DEV_LOG「display-gui 批次」· T-029 真机触出 + 授权跨工单修复 · 2026-08-27
status: verified
---

# ctypes 绑定 HRESULT 必须按无符号语义归一

## 症状

T-028 `display_icc.py` 引擎在真机 `python -m ctrl run display --status` 返回 1，报「读取当前默认 profile 失败：HRESULT=0x80070002」——但代码里 `_HR_FILE_NOT_FOUND = 0x80070002` 与返回值的 `== 比较永不匹配`，`get_default_profile()` 落进错误分支抛 OSError。T-028 自己的 38 个测试**全绿**（seam 直接注入正数常量，从未经过真实有符号返回）。

## 根因

ctypes 绑定 `ColorProfileGetDisplayDefault` 等 mscms API 时 `restype = c_long`（**有符号** 32 位），真实返回 `0x80070002` 是 **-2147024894**；代码用**无符号**常量 `0x80070002`（2147942402）比较，两个值不等于任何意——错误码映射（幂等 remove `0x800707DF`、非显示 profile `0x80070709`、无关联 `0x80070002`）在真实系统上**全部失效**。`_color_add/_color_remove/_color_get_default` 三条 seam 全有符号。只有真机首跑（T-029 门面）触出，因为测试替身从未注入过有符号负值。

## 代价

引擎第一条真机路径即错；跨工单授权修复（3 行 `& 0xFFFFFFFF` 归一 + 4 条带符号回归测试）；T-029 因它暂停等待裁定。

## 教训

**ctypes 绑定返回 HRESULT 的 API：restype 用 `c_ulong` 或在 seam 返回处统一 `hr & 0xFFFFFFFF` 归一，错误码常量比较一律按无符号语义。** 更通用：mock 注入的真实 API 返回值必须覆盖「真实位模式」——有符号负数形态，否则测试假绿，缺陷留到真机暴露。

## 防复发

- [x] 已落实？三条 seam 返回处 `& 0xFFFFFFFF` 归一 + `tests/test_display_icc.py` 用 `ctypes.c_long(0x8007xxxx).value` 注入真实负值位模式的 4 条回归测试
- 新绑定 HRESULT 的 API 时先核对 restype 有符号性；测试 seam 注入用有符号位模式而非正数常量

## 关联

- [[参考实现的实证结论必须在本机复验]]（同批 spike 触发：真机复验才暴露）
- [[Falsify测试要钉住缺陷所在层]]（回归测试注入缺陷层（符号转换）的真实输入）