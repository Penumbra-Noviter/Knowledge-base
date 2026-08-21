---
type: experience
tags: [python, config, env, falsify]
project: 代订场
date: 2026-08-21
summary: 环境变量通过 os.environ.get("KEY") 获取时，若值为空字符串 ""，if X is None 校验会绕过（空字符串 ≠ None），导致下游静默使用空值。修复：使用 if not X 替代 if X is None。
provenance: DEV_LOG#7 / commit 0e72461 / code-review Falsify 轴
priority: high
---

# 环境变量空字符串绕过 `is None` 校验

## 现象

`app/config.py` 中 `require_sport_id()` 使用 `if SPORT_ID is None` 校验，但环境变量 `SPORT_ID=""` 时 `os.environ.get("SPORT_ID")` 返回空字符串 `""`，不等于 `None`，因此校验通过，下游静默使用空字符串传参。

## 放大因素

交付脚本 `install_scheduled_task.ps1` 的 `$EnvOverrides` 默认占位正是 `"SPORT_ID" = ""`，用户不编辑直接跑脚本即把空串写进 User 作用域环境变量——随后调度器会带着空 sport_id 静默走完抢订流程。

## 修复

```python
# 有问题的写法
if SPORT_ID is None:
    raise ConfigError(...)

# 修正后的写法
if not SPORT_ID:
    raise ConfigError(...)
```

`if not SPORT_ID` 同时覆盖了 `None`（缺失）、`""`（空字符串）和 `"  "`（空白字符串，若带 `.strip()`）。

## 适用范围

本项目 `require_target_venue()` 也有同款问题，一并修复（`if not TARGET_VENUE`）。

## 通用教训

所有使用 `os.environ.get("KEY")` 的配置校验，如果「空字符串」无意义（大多数配置项），一律用 `if not VALUE` 替代 `if VALUE is None`。环境变量没有「空字符串有意义」的常见场景——空值与缺失值在语义上对配置项等价。
