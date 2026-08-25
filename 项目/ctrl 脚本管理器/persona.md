---
type: persona
tags: [persona]
date: 2026-08-26
project: ctrl 脚本管理器
status: active
---

# ctrl（统一脚本管理器）Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档（PROJECT_REFERENCE.md / ADR.md / CLAUDE.md）。

## 本人偏好

- Windows 个人生产力工具：Python 3.12 + PowerShell 5.1（PS1 插件引擎）+ PySide6（托盘）；命令行入口 `ctrl.bat`（`PYTHONPATH=src python -m ctrl`）
- 中文注释 / 中文错误消息（200，无裸 traceback）/ 中文 Conventional Commits（feat/fix/refactor/docs/test）
- 先 ADR 后代码（架构级改动先 ADR.md 标状态）；先结论后证据、不含装饰性过渡语
- 公共函数 type hints + docstring；模块 `__all__`；文件头 `__future__ import annotations`
- 覆盖率目标 ≥90%（pytest.ini `--cov-fail-under=90` 已门禁）；CLI 优先、渐进式（MVP→稳定→体验→性能）

## 固定约束

- **提权永远分离**（ADR-004）：管理器自身永不请求管理员令牌；提权只在 runner/elevator 按工具元数据（`requires_admin`）触发；提权传输直达 `sys.executable -m ctrl`（不经 cmd/bat 二次解析，消元字符注入面），PYTHONPATH 经进程环境继承（追加语义 + `rstrip(";")` 消双分号，TD-02/TD-24 后）
- `plugins/clean_cache_v3.ps1` 是外部导入插件（660 行成熟 PS1），除非显式任务否则不改其内容
- 插件手动导入、不做自动扫描（ADR-002）；每插件一个 `.meta.json` 与脚本平级（ADR-003）；import/remove 失败零残留（补偿式事务）
- 参数透传映射放 meta（ADR-007）：CLI 小写连字符 `--scan` → 脚本 `-Scan`（meta `flag` 字段）；`--` 开头值构造源头拒绝（无法跨提权边界保真）
- 输出双模式（ADR-005）：CLI 当前终端 / 托盘新窗口（cmd /c start）
- 数据文件（registry.json / plugins/）是运行时资产：测试一律 monkeypatch 到 tmp，绝不读写真实文件

## 稳定模式

- **registry 损坏判定单源** `_parse_registry_text`（load_registry / registry_file_degraded / save_registry 受保护写三处共用）；save_registry 原子写（同目录**随机临时文件名** `uuid.uuid4().hex` + os.replace，多进程交错写互不竞争，TD-15 后；失败清理）且 degraded 拒写保护原始字节；读文件 utf-8-sig（消化记事本 BOM）、写 utf-8 无 BOM
- **类别映射单源** `types.Category(str, Enum)` + `CATEGORY_ORDER`/`CATEGORY_LABELS`；**反序列化时** `_meta_to_tool` 直接把 meta category 转枚举（未知值/缺省回落 IMPORTED，TD-25 后），cli.py / tray.py 不再有任何 Category 转换逻辑；`_cmd_info` 显示侧用 `isinstance(tool.category, Category)` + `.value` 归一（内置工具 category 仍为裸 str，脚本工具为枚举——实测 Python 3.12 `str(Category.X)` 返回 `"Category.X"` 非 `"X"`，显示侧必须归一）
- 工具名 `validate_tool_name` 同源净化（路径分隔符/冒号/`..` 拒绝）+ casefold 重名归一（仅误拒方向安全侧）
- **提权链路** `require_admin`：`except AttributeError` 分支带 `if not sys.platform.startswith("win")` 平台守卫（Windows 上 AttributeError 继续传播 fail-fast，不误判"不支持提权"，TD-17 后）；CLI `_cmd_run` 捕获 OSError 输出中文错误消息（UAC 取消不裸 traceback，TD-16 后——注意 catch 范围过宽会误标非提权 OSError，见 TD-28）
- **kickoff 全自动档批次跑通**（Grilling 共识 / Neat 删除清单 / 最终交付三处确认，波末仅汇报）；td-consume-2 批次小档（单 Implement 串行 3 工单），波末与期末零修复轮次，竣工 262 tests / 99.27%
- tray 纯逻辑测试化：`sys.modules` 注入 mock PySide6（_make_icon/_tool_action/_build_menu），run_tray 事件循环 `# pragma: no cover`
- 测试隔离锚点：`import ctrl.core.registry as R` 后 `monkeypatch.setattr(R, "REGISTRY_PATH", ...)`（模块内取值，防 fixture 替换失败）

## 变更记录

- 2026-08-26（td-consume-2 批次）TECH_DEBT 13 条消费 = 3 工单（T-019~021）+ 6 关闭；262 tests / 99.27%；commit 42d9081→ff85d8f；四轴 0 阻断、修复轮次 0/5；新候选 TD-28~32 落盘
  - 本批最有价值发现：spec 对 `(str, Enum)` 的 `str()` 行为假设不成立（Python 3.12 `str(Category.CLEANUP)` = `"Category.CLEANUP"`），实现期实测纠偏并加回归锚测试——高不确定实现点须实测后定稿