---
type: lesson
tags: [架构, 序列化, 重构, FastAPI, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
summary: FastAPI 序列化走 response_model(from_attributes=True)，手写 dict 映射是重复维护反模式——Schema 是字段清单唯一事实来源
provenance: git 4f686a5（2026-08-08 知识库批量入库）· Conver System 2026-08-06 实践
status: verified
---

# response_model 统一驱动序列化——退役手写 dict 映射

## 症状
后端的 `character.py` 和 `conversation.py` 各有一份手写字段映射 dict（`_char_to_dict()` 23 行、`list_conversations` 13 行），与 Pydantic Schema 的字段定义完全重复。新增字段需要改 3 个地方：ORM model、Schema、手写 dict。遗漏任何一处就产生静默不一致。

## 根因
早期代码在 FastAPI 的 `response_model` 机制外又包了一层 dict 转换，原因是"怕 ORM 对象直接序列化会暴露敏感字段或产生循环引用"。但实际上 Pydantic v2 的 `from_attributes=True` + `response_model` 已经处理了这些——Schema 只暴露显式定义的字段，ORM 懒加载属性不会触发。

## 代价
删除 `character.py` 23 行 + `conversation.py` 13 行手写 dict；`list_characters`/`get_character_with_count`/`list_conversations` 返回 ORM 对象 + 瞬态属性（`conversation_count`/`message_count`）；117 项测试全过，行为零变化。

## 教训
**FastAPI 项目中，序列化优先走 `response_model=*(from_attributes=True)`**，不要手动写 dict 转换。这意味着：

1. Schema 是字段清单的唯一事实来源
2. 新增字段只需改 ORM model + Schema 两处
3. 瞬态属性（如 `conversation_count`）通过 `@property` 挂在 ORM 模型上，Schema 对应字段声明即可
4. 手写 dict 是"重复维护"的反模式，重构时优先删除

同样适用于返回 `list[dict]` → 改为 `list[Schema]`，路由声明 `response_model` 即可。

## 防复发（惯例：开工预检时对照执行，无需勾选；动作项见条目内去向）
- [x] 后端所有 list 端点不再有手写 dict
- [x] 新增字段只需改 ORM model + Schema
- 新端点默认使用 `response_model` 驱动序列化

## 关联
- [[Provider 标识符 keyid 分离]]