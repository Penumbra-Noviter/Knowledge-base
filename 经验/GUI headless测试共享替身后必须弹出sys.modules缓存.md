---
type: lesson
tags: [GUI, headless, 测试隔离, sys.modules, 替身, 顺序依赖]
date: 2026-08-27
project: ctrl 脚本管理器
source: 自身项目实践
summary: GUI headless 测试跨文件共享替身类后，module 被先跑的 fixture 导入并缓存（绑定到其替身），后跑者断言本地类身份失败——每 fixture 导入被测模块前必须 sys.modules.pop 弹出缓存强制重绑
provenance: DEV_LOG「display-gui 批次」· T-032 合并后顺序依赖回归 be640b2 · 2026-08-27
status: verified
---

# GUI headless 测试共享替身后必须弹出 sys.modules 缓存

## 症状

T-032（主窗口）合并后，全量测试出现 **2 个顺序依赖失败**：`tests/test_gui_generic_page.py` 的 `test_build_form_creates_checkbox_and_lineedit`、`test_page_builds_ui_with_form_and_buttons`。单独跑该文件 **18 全过**；按「generic_page 先跑」的顺序也过；按「sidebar/main_window/app 先跑、generic_page 后跑」的顺序失败。测试结果随执行顺序变化，是典型的测试隔离污染。

## 根因

T-030 的 `test_gui_generic_page.py` 用自己的局部 PySide6 替身类（`_StubQCheckBox` 等），fixture `monkeypatch.setitem(sys.modules, ...)` 注入后 `from ctrl.gui import generic_page`。T-032 引入共享替身 `tests/_gui_stubs.py` 后，其测试（sidebar/main_window/app）先跑，把 `ctrl.gui.generic_page` 导入并**缓存**在 sys.modules（类绑定的是 `_gui_stubs` 的替身）。后跑的 generic_page fixture 再 `import` 拿到**缓存模块**——类身份是 `_gui_stubs` 的，与文件内局部 `_StubQCheckBox` 断言 `isinstance` 失败。

## 代价

一次假性回归排查（先误判为 T-032 实现缺陷，实际是测试顺序问题）+ 修复（fixture 导入前 `sys.modules.pop` 弹出缓存，强制重绑本 fixture 替身）。

## 教训

**headless mock 测试跨测试文件共享替身类后，每个 fixture 在导入被测模块前必须弹出缓存的 `ctrl.<pkg>.` 模块**（`sys.modules.pop(mod, None)`），使模块按本 fixture 注入的替身重新绑定——顺序无关性靠「先弹后导」保证，不能依赖 pytest 收集顺序。

## 防复发

- [x] 已落实？`test_gui_generic_page.gui` fixture 导入前弹出 `ctrl.gui` / `ctrl.gui.exec_worker` / `ctrl.gui.generic_page`（be640b2）；test 文件头注释申报；正序倒序双验证
- 新增 GUI 测试文件沿用「pop + 注入 + fresh import」模式；CI/全量跑完后再加一个逆序冒烟可彻底免疫

## 关联

- [[状态发布与文件落盘顺序契约]]（同为「顺序」类契约问题，领域不同）
- [[参考实现的实证结论必须在本机复验]]（同批「真机/全量才暴露」模式）