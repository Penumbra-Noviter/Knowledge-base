---
type: lesson
tags: [Falsify, 容错, JSON, 边界值, conver-system]
date: 2026-08-30
project: Conver System
source: 自身项目实践
summary: 容错解析「非法值回退」必须显式 NaN 守卫——JSON 字面量 NaN 被 jsonDecode 宽容接受、double.tryParse 返回 NaN 非 null，clamp 后仍 NaN 入库存档，导出 jsonEncode 抛错
provenance: mobile DEV_LOG〈M3 kickoff 批次〉期末四项真缺 #1 · 修复 0057d9e · 期末四轴 Falsify 对抗发现 · 2026-08-30 会话
status: verified
---

# 容错解析「非法值回退」须显式 NaN 守卫——JSON NaN 字面量穿透裁剪契约

## 症状
M3 V2 角色卡导入服务 `_clampTemperature`：docstring 承诺「非数值/越界回退默认 0.7」，实现为 `double.tryParse(value.toString())` + `temp.clamp(0.0, 2.0)`。期末四轴 Falsify 构造输入 `{"temperature": "NaN"}` 发现：Dart `jsonDecode` **宽容接受字面量 NaN**，`double.tryParse('NaN')` 返回 NaN（**非 null**），`NaN.clamp(0,2)` 仍 NaN——docstring 承诺被绕过，NaN 温度入库存档。后果 (a) 导出 `jsonEncode(NaN)` 抛 `JsonUnsupportedObjectError`；(b) 编辑/向导温度 Slider 以 NaN 为 value 在 debug 触发断言。同源桌面 `float('nan')` quirk，仅恶意/宽容 JSON 输入可达。

## 根因
「非法值回退」契约的常见实现只处理了两类非法：null（空值）与解析失败（tryParse 返回 null）。但 `tryParse` 对 NaN/Infinity **解析成功**（返回非 null 的 double），`clamp` 对 NaN 也是透传（不是归并到边界）——「解析失败才回退」的守卫天然漏掉 NaN 这一类合法解析结果。

## 代价
一轮期末 Falsify 对抗发现 + 小额修复；若未发现，恶意角色卡可让 NaN 温度入库，用户编辑该角色时 debug 崩溃（release 静默异常）。

## 教训
**容错解析的「非法值」集合 = null + 解析失败 + NaN/Infinity，三缺一即穿透**。凡「解析 + 裁剪/范围守卫」型代码，解析结果须先判 `isNaN`/`isInfinite` 再进 clamp/范围逻辑；Falsify 构造输入要含 JSON 宽容接受的边界字面量（NaN/Infinity/超大数/科学计数），它们是「解析成功但语义非法」的一族。

## 防复发
- [x] 已落实（本批）：`_clampTemperature` 加 `temp == null || temp.isNaN` 回退 0.7；回归测试 case `('NaN', 0.7)`
- [ ] 通用化：其他「JSON 解析 → 类型容错 → 范围裁剪」路径（文档导入/游戏配置/设置导入）复用此守卫模式

## 关联
- [[Falsify测试要钉住缺陷所在层]]（同理：Falsify 用恶意输入打数据通路，价值在期末再证）
- [[LLM 生成 JSON 三级提取策略]]（同族：JSON 解析的容错边界）