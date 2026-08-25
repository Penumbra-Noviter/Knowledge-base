---
type: lesson
tags: [python, 枚举, spec, 实测, 回归]
date: 2026-08-26
project: ctrl
source: 自身项目实践
summary: (str, Enum) 多继承的 str() 输出是 "枚举类.成员" 而非成员值；spec 对语言行为的假设须实测验证后定稿，配回归锚测试
provenance: 会话 td-consume-2 · commit 623e9f8（T-021 Category 去重）· evidence/03.md · DEV_LOG 2026-08-26
status: verified
---

# (str, Enum) 多继承的 str() 显示行为——spec 假设须实测定稿

## 症状

`_cmd_info` 显示脚本工具类别回归为 `Category.CLEANUP` 而非 `cleanup`。spec「高不确定实现点」声称 `Category(str, Enum)` 多继承下 `str(Category.CLEANUP)` 返回 `"cleanup"`（"str 输出行为不变"）——实测打脸。

## 根因

Python 3.12 中 `str(Category.CLEANUP)` 返回 `"Category.CLEANUP"`（枚举类名.成员名），不是成员值 `"cleanup"`。`Enum.__str__` 优先于 `str.__str__`（MRO 与枚举元类行为），`(str, Enum)` 多继承只保证**等值/哈希按 str 内容**（`hash("cleanup") == hash(Category.CLEANUP)`），不保证 `str()` 输出=成员值。显示侧必须显式取 `.value`。

## 代价

- spec 一个高不确定实现点的论断错误，靠实现期实测发现（幸运）；若未实测直接照 spec 写，`ctrl info <脚本工具>` 类别显示全回归
- 额外修复：`_cmd_info` 加 `isinstance(tool.category, Category)` 分支 + `.value` 归一，补回归锚测试 `test_info_script_category_shows_raw_code`
- 影响波及内置工具（category 仍是裸 str）：归一分支必须兼容「枚举 or 裸字符串」两种形态

## 教训

对语言运行时行为的假设（`str()`、`repr()`、`==`、哈希、迭代顺序）**不可凭直觉写进 spec**，尤其是多继承/枚举/魔法方法交互。高不确定实现点要么标注「以实测为准」，要么实现期先做最小实验证伪再定稿——spec 定稿的语言行为假设 = 未来的回归面。

## 防复发

- [x] 已落实：回归锚测试 `test_info_script_category_shows_raw_code`（枚举 → `.value` 裸串）锁定显示行为
- [x] 已落实：spec 修订日志记录该假设被证伪（evidence/03.md 声称断言节）
- [ ] spec「高不确定实现点」撰写惯例：涉及语言行为的论断改写成「XX 行为以 Python 3.12 实测为准」而非断言输出值

## 关联

- [[工单验收标准避免行号与grep计数]]（验收锚点语义契约——本次锚定方法语义而非输出字面）
- [[技术债票面修复建议须实证复核]]