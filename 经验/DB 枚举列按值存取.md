---
type: lesson
tags: [数据库, SQLAlchemy, 枚举, 迁移, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
status: verified
---

# DB 枚举列按「值」存取——存量数据零迁移兼容

## 症状
把 `Message.role` 从 VARCHAR 升级为 Enum 列时，存量数据是 `'user'`/`'assistant'` 字符串；SQLAlchemy 默认存枚举**成员名**，与存量值不匹配 → 要么迁移数据，要么读不出来。

## 根因
SQLAlchemy `Enum` 默认 `values_callable` 存成员名（`Role.user` → `"user"` 恰好巧合一致时掩盖问题），当成员名 ≠ 值时会错位。枚举升级的「值契约」没想清楚就建列。

## 教训
**枚举列用 `values_callable` 按 `member.value` 存取**（落库值 = 枚举定义值），存量 VARCHAR 数据无缝读出，零迁移。约定：数据库里的值永远用枚举的 `.value`，成员名只是代码内标识。

## 代价
零迁移（`models/message.py:29-34`）——做对了就是零成本。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] `message.py` 已按值存取
- 新枚举列一律声明 `values_callable=lambda enum: [e.value for e in enum]`，不依赖默认

## 关联
- [[response_model 统一驱动序列化]]（同为「存储/序列化契约单点化」族）
- [[Provider 标识符 keyid 分离]]（同为「标识符语义」族）
