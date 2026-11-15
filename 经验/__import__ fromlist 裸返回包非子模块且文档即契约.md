---
type: lesson
tags: [python, import, 契约单源]
date: 2026-11-13
project: hanhua
source: 自身项目实践
summary: __import__ 带 fromlist 裸返回 module 参数命名的模块本身非属性；契约单源模块 docstring 失实等同代码缺陷
provenance: DEV_LOG 第一百五十三会话（arch-review-20261113）· T-4546 F1 + 期末四轴 E-658 · commit 866e0f6
status: verified
agents_md_feedback: 印证「对抗性自检」——文档语义失实由 Falsify 构造输入实测抓出（guarded_import("os.path") 返回 <module 'os'>）
---

# `__import__` fromlist 裸返回包非子模块，且契约单源模块的文档即契约

## 症状
T-4546 把六处 ImportError 守卫收拢进 `guarded_import` 后，psp-glyph handler 每次真实调用炸 `AttributeError: module 'hanhua.engines.psp_uni2' has no attribute 'main'`——但既有测试全绿（只钉了 `--verify-lookup` 离线分支）。修复轮之后，期末 Falsify 又用 `guarded_import("os.path")` 实测击穿 docstring 残留语义：声称返回点路径命名的模块，实际返回顶层包。

## 根因
CPython 语义：`__import__(name, fromlist=...)` 的返回值**永远是 `name` 参数命名的模块**；fromlist 只触发子模块属性导入（副作用），不改变返回值。要拿属性须再 getattr（即 `from module import attr` 整条语句 = import 副作用 + 属性绑定两步）。第一次翻车 = 调用方按 docstring「等价 from-import」误把裸返回当属性消费；第二次残留 = docstring 修了 attr 形态但 `attr=None` 点路径分支的反面声明没修。契约单源模块（六调用点共用的守卫）的 docstring 就是契约本体，失实会系统性误导未来每个调用点。

## 代价
波 1 增量审核抓出 1🔴（合并后 master 的 psp-glyph handler 已坏）+ 一轮修复；期末再抓一次 docstring 残留（E-658 落盘）。合计约一轮修复轮 + 双轮审核成本。

## 教训
`__import__` 带 fromlist 的裸返回值 = `module` 参数命名的模块本身；要属性必须 getattr。契约单源模块的 docstring 失实等同代码缺陷——修代码不修文档等于留一半根因。

## 防复发
- [x] 守卫层改 attr 绑定语义并复刻 IMPORT_FROM 错误（属性缺失 = ImportError 非 AttributeError）
- [x] 成功态消费属性防复发断言 3 条（守卫层真 import / handler 层桩 rc 42 / 兜底语义钉测）
- [ ] 新守卫/单源模块落地时，docstring 的返回值语义用一段可执行 smoke 实测校对（本次靠审核者手工实测，未结构化）

## 关联
- [[测试锁随单源化升级为接线锁]]
- [[同一产物文件多写入方必须共享格式契约单源]]
