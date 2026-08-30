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
- **UI 设计偏好：克制、理性、简约**（2026-08-28 明确）——这只是个工具，不用过多的元素堆砌，太多视觉堆料只会抢占注意力。GUI 主题 `ctrl.gui.theme`（QSS）按此落地：冷灰中性 ramp + 单一青玉强调色、hairline 边框、无渐变/紫光/装饰性堆叠；每页一个主操作高亮、其余描边幽灵按钮；finesse 只走 product register（SPECTACLE 低），不走 brand register

## 固定约束

- **提权永远分离**（ADR-004）：管理器自身永不请求管理员令牌；提权只在 runner/elevator 按工具元数据（`requires_admin`）触发；提权目标为 `python.exe <src/ctrl/_elev_bootstrap.py> [--hold-open] [--output-file <path>] -- <子命令>`（ADR-012 起：不经 cmd/bat 二次解析，消元字符注入面），bootstrap 在提权进程内由脚本自身路径注入 src 到 sys.path（**UAC 提权进程不继承调用进程 env**——PYTHONPATH/CWD 跨不过 UAC，实测证伪 2026-08-30），`--` 前专属参数翻译回进程内环境变量后进入 CLI
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

## 稳定模式（display-gui 批次，2026-08-27）

- **GUI headless 测试共享替身**：`tests/_gui_stubs.py` 统一实体替身 + `_FakeSignal` 真投递；**fixture 导入前必须弹出缓存的 `ctrl.gui.*` 模块**（`sys.modules.pop`）——多测试文件共用替身类后，先跑的 fixture 会把模块绑定到自己的替身，后跑者断言本地类身份失败（顺序依赖回归，be640b2 教训）
- **单实例 = Windows 命名互斥体**（`single_instance.py`，CreateMutexW `ERROR_ALREADY_EXISTS` 原子判定）+ QLocalSocket 聚焦管道：QLocalServer 二次 listen 在本机 PySide6 6.11.1 **不返回 AddressInUseError**（in-process 两次 listen 均 True），独占判定不可靠——实证后弃用
- **ICC 机制事实**（spike#1 实证，替代 win-hdr-fix 假设）：切换/还原 CURRENT_USER **免提权**、仅 `--import` 需管理员；`ColorProfileAddDisplayAssociation` **add-only（setAsDefault=true）直接生效**，remove→add 顺序非必需；错误码 0x800707DF 幂等 / 0x80070709 非显示 profile / 0x80070002 无关联；SYSTEM_WIDE 非提权「假成功」（S_OK 但未生效）代码层禁止
- **display 工具提权语义**：`ToolInfo.requires_admin=False` + 门面 `requires_elevation(action)` 仅 `--import` 返回 True（profile/restore/save_default 内联免提权，gamma 永不提权）；互斥校验先于提权
- **饱和度物理边界**：gamma ramp 是逐通道 1D LUT，3×3 饱和度矩阵行和=1 强制灰度中性 → `build_curve(b,c,g,0)==(b,c,g,3)`，**饱和度经 gamma ramp 不可见**（TD-33）；GUI/CLI 均不暴露饱和度，真实饱和度需驱动级（NVIDIA Digital Vibrance）

## 稳定模式（td-consume-4 批次，2026-08-27）

- **GUI 工单覆盖率口径怪癖**：Windows + pytest-cov + 假 PySide6 sys.modules 懒加载下，`--cov=src/ctrl/gui/X.py`（路径口径）报「module never imported / No data / 0%」；须用**模块名口径** `--cov=ctrl.gui.X`（同一文件）才正常采集。GUI 单工单测试一律用 `-o addopts="--cov=ctrl.gui.<模块> --cov-report=term-missing --cov-fail-under=90"`（覆盖 pytest.ini 默认全仓口径，避免稀释）
- **QtNetwork 可先于 QApplication 构造（F1 实证）**：`single_instance.acquire()` 在 create_app（建 QApplication）前构造 QLocalServer/QLocalSocket **不崩溃**——真机冒烟验证首实例稳定存活、二次实例正确检测并 exit 0；listen 失败走防御路径（中文警告），互斥体独占语义保持
- **GUI 导航单一信号**：`MainWindow` 导航只连 `currentItemChanged`（承载点击/键盘/程序化选中），不再连 `itemClicked`——一次选中仅一次 `show_tool_page`；通用页切页从不 delete 旧页（孤儿化 + QThread 销毁风险，TD-44 pre-existing）
- **display 刷新 Seam**：托盘/外部切换 ICC 后经 `MainWindow.refresh_display_page()` 公共方法刷新 display 页，不再 `getattr` 触 `_display_page`/`_refresh_profiles` 私有成员（TD-41）

## 变更记录

- 2026-08-27（td-consume-4 批次，project-kickoff 全自动档标准档）消费 TECH_DEBT TD-33~42：5 工单（T-035~039）全合并；547 tests / 99.69%；期末四轴 0 阻断；修复轮次 0/5；冒烟通过（含 F1 QtNetwork 前置真机验证）；Neat 清场 5 分支 + 5 worktree + scratch 批次
  - 本批最有价值发现：**F1 真机验证**（T-037 把 QtNetwork 构造前置到 QApplication 之前不崩溃，防御路径生效）+ 范围授权教训（TD-34 GUI 层验收的实现文件 generic_page.py 在 plan-tickets 文件范围外——验收语义指向的实现文件可能不在范围，plan-tickets 需交叉核对验收锚点与其实现文件）

- 2026-08-27（display-gui 批次，project-kickoff 全自动档标准档）9 工单（T-026 spike + T-027~034）全合并；534 tests / 99.69%；期末四轴 0 阻断；修复轮次 0/5；Neat 清场 21 项；TECH_DEBT 新增 TD-33~42 落盘
  - 本批最有价值发现：**spike#1 实测推翻参考实现 win-hdr-fix 的两条假设**（「需管理员」「先 remove 再 add 才生效」均不成立）——参考实现的实证结论必须在本机复验，不能照搬；以及 T-028 HRESULT 符号数缺陷（ctypes `c_long` 返回负值 vs 无符号常量比较永不匹配，真机 `--status` 才触出）