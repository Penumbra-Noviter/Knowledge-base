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

- **提权永远分离**（ADR-004）：管理器自身永不请求管理员令牌；提权只在 runner/elevator 按工具元数据（`requires_admin`）触发；提权目标为 `python.exe <src/ctrl/core/acl.py> [--hold-open] [--output-file <path>] -- <子命令>`（ADR-012 起：不经 cmd/bat 二次解析，消元字符注入面；ADR-013 起 bootstrap 逻辑整体迁入 core.acl，`_elev_bootstrap.py` 保留为旧地址兼容垫片——模块级 `__getattr__` 惰性委托、导入零副作用，elevator._bootstrap_script() 指 core/acl.py），bootstrap 在提权进程内由脚本自身路径注入 src 到 sys.path（**UAC 提权进程不继承调用进程 env**——PYTHONPATH/CWD 跨不过 UAC，实测证伪 2026-08-30），`--` 前专属参数翻译回进程内环境变量后进入 CLI；`acl._assert_env_translation` 校验 hold-open/output-file 请求与进程内 env 一致性，断链 exit 2 诚实失败（防跨进程静默断链）
- `plugins/clean_cache_v3.ps1` 是外部导入插件（732 行成熟 PS1，T-065 加固后），除非显式任务否则不改其内容
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

## 稳定模式（2026-08-28 ~ 09-04 批次）

- **测试 env 隔离根因修复**（T-079）：`tests/conftest.py` autouse `_restore_environ` fixture——yield 前快照 `dict(os.environ)`、yield 后 `clear()+update()` 整体恢复，根治「`monkeypatch.delenv` 恢复语义不覆盖测试中途直接写 `os.environ[...]`」泄漏；新测试不必再写补救式 env 清理 fixture（红/绿哨兵锚已钉）
- **registry 只读层去环 + 守卫成对**（T-085，ADR-013）：`list_tools` 模块顶层导入（实测 registry→builtins→display 顶层全链无环，延迟导入是历史防御）；`assert_registry()` 严格守卫（损坏/结构非法抛 `RegistryDegradedError`、缺失/0 字节放行——空注册表合法初始态）与 `registry_file_degraded`（宽容探测）成对（读侧严格 + 写侧拒写同型同源）
- **ACL 检查点接入**（T-086/087，ADR-013）：读路径（list/info/run/tray/gui）dispatch 前过 `_guard_registry`（损坏 → 中文错误 + exit 1；GUI 捕获模式仍写 done marker 面板不空转）；import/remove 保留内部早停（写路径严格语义在指令内部）；GUI `run_gui` 单实例 acquire 后、create_app 前过 assert_registry（损坏 → return 1 不建窗）；bootstrap 逻辑单点迁 `core/acl.py`（`_elev_bootstrap.py` 为惰性委托兼容垫片）
- **行流容错单点 ProcessLineReader**（T-071）：`core/process_io.py`（PIPE + errors=replace + iter_lines/exit_code/terminate），runner._run_script_forward 与 ExecWorker.run 复用，中断/强杀语义全保持；`_run_script_forward` 脚本执行统一 `Popen(stdout=PIPE, stderr=STDOUT, text=True, errors="replace")` 逐行转发当前 sys.stdout 并 flush——GUI 捕获模式输出真正进 CTRL_OUTPUT_FILE（clean_cache 面板空转修复，2026-08-28）
- **display gamma 基准磁盘持久化**（T-076~078）：基准存 `~/.ctrl/display_gamma_baseline.json`（3×256 曲线形状保真、写失败静默 warning、reset 优先级 = 进程内→磁盘→兜底、兜底不落盘防误报）；`_validate_ramp` int-only（float 抛 ValueError 走诚实兜底）；reset_ramp 磁盘分支 `_baseline = disk` 移至 `apply_ramp(baseline)` 成功之后——GDI 写失败不污染内存 `_baseline`（未生效不缓存）、apply 抛 OSError 照常上抛；`_execute_gamma` 门面层 OSError 容错（save_baseline 读失败 warning 不阻断 apply）
- **clean_cache_v3.ps1 操作纪律**（T-065 + 空转修复）：ps1 无 main 封装（顶层即流程）——**验证仅用 `-Scan` 或函数隔离提取，dot-source 整文件会触发真实 clean**（2026-08-29 事故）；cleanCommand 经 `Invoke-Clean-Command`（Start-Job + Wait-Job 超时 180s + 退出码哨兵 `__CTRL_EXIT__:N` + npm.cmd shim 兼容）；`Get-DirSize`/手写递归跳过 ReparsePoint（手写递归穿 junction 是 11.5h 卡死真凶，`-Recurse` 自带防护、`-Attributes !ReparsePoint` 为显式双保险）
- **脚本工具交互等待契约**（T-059）：仅 `$Host.UI.RawUI.ReadKey` 在无控制台（pythonw）下无限挂起，`Read-Host`/`cmd /c pause` 安全——`detect_interactive_wait` 探测集 = ReadKey 单 token；含 `IsInputRedirected` 守卫视为已防护不命中；notes 提醒不阻断导入、`_cmd_info` 实时探测显示（脚本本体为单一事实源，不落 meta 冗余标记）
- **提权链 env 单源**（T-075）：`capture.HOLD_OPEN_ENV` 与 `OUTPUT_FILE_ENV` 单源（镜像定义）；`_fail` 流安全（`sys.stderr or sys.stdout` 回退、双流皆 None 仍 exit 2）；`--output-file ""` 双层拒绝

## 稳定模式（td-consume-25 批次，2026-10-05）

- **技术债全量消费打法**：候选区一次清空 = 8 立项 6 工单 + Speculative 复核关闭；Worth exploring 由 Grilling 拍板、Speculative 关闭须 git grep 一句话理由——票面修复建议是候选假设，Grilling 阶段必须读码实证（TD-154「第三处同形」触发未达成被否定即实证例）。
- **合并票两 commit 粒度压缩串行波次**：共享同文件族的串行票（显示组 4 票共享 display_*）合并为一张票、票内两个独立 commit（每 commit 各自 TDD + 范围覆盖率），波次从 5 压到 3——减少串行 worktree/merge 摩擦，合并票验收条数票级合并（≤8）。
- **check-complete.py 只认四状态 token**：编排文件未开工票**不写 STATUS 行**（PENDING 会被判「未知状态」报错）；派发后才按 DONE/DONE_WITH_CONCERNS/NEEDS_CONTEXT/BLOCKED 落盘。
- **提权判定进程级触发式**（TD-152 消费面归位）：`needs_elevation` 是语义源（不含 is_admin），进程级触发 = `needs_elevation ∧ not is_admin()`，runner 与 GUI 通用页共用——已提权进程不再多跳 UAC。
- **状态判据单源化**（TD-149）：CLI/GUI 的 gamma 恒等判据收敛到引擎 `is_identity_ramp`，GUI refresh 捕获 `save_baseline()` 返回值存硬件快照（零新增 IO）；gamma ramp 不可反解回因子 vs DWM 矩阵可反解 = 方向性不对称属物理约束显式记录。

## 稳定模式（arch-review-20261024 批次，2026-10-24）

- **架构评审候选落地形态**：走查（热点定向 + 复核关闭表去重）→ HTML 报告（%TEMP%，Tailwind+Mermaid）→ 用户按强度顺序拍板 → Grilling 决策账本（D1–Dn，含备选代价）→ 4 票并行 + 收尾票。候选「第 2 次建议」必须重估上次关闭前提（本批 C1 的 TD-66/78 关闭理由因消费面变化而过期，C4 的 TD-62 关闭前提被 ADR-025 抽走）。
- **量化/判据单点化的收尾套路**：谓词化（is_identity_ramp / requires_elevation 消费）→ 中间人删除（build_cli_tokens 删、GUI 直调 codec）→ 重复收口（`_run_dwm_action` 一类 helper 收「警告捕获 + 失败措辞」，回弹/事件位留槽内）。
- **审核子智能体连续 2 次空返回 = 主会话直做**（本批波 1 增量审核与期末四轴均命中）：只读审计任务可脚本化直做（shared-file diff 零改 / grep 残留 / 点名锚 / 边界测试），不需第三次派发。
- **TECH_DEBT 表格批注必须在单元格内**：行尾 `|` 之后追加文本会破坏列数（pool_cleanup_check 报「表格格式异常」）——批注并入状态单元格后再补行尾 `|`。

## 变更记录

- 2026-10-24（arch-review-20261024 批次，/improve-codebase-architecture → 5 候选落地）：C2 提权判定归位 / C1 删 build_cli_tokens 转发（环消除）/ C3 DWM 量化收归 binding / C4 DisplayPage 动作族收口 A 层 / C5 cli HOLD_OPEN helper；4 工单（T-130~133）+ 收尾票；全量 1229→1227 passed / 99.08%；四轴直做通过（0 阻断）；Neat 清场 4 分支 + 4 worktree + scratch
  - 本批最有价值发现：**架构评审的「复核关闭前提」会随代码演进失效**——C1 的 TD-66/78 关闭理由（「runner 与 GUI 双入口共用 build_cli_tokens」）因 T-061/T-067 后消费面已变而过期；C4 的 TD-62 关闭前提（状态模型在页面）被 ADR-025 抽走。走查时对「第 2 次建议」必须核对关闭前提是否仍成立，否则会重复关闭真实摩擦

- 2026-10-05（td-consume-25 批次，project-kickoff 全自动档标准档）消费 TECH_DEBT TD-149~157：8 立项 6 工单（T-122~127，T-126/T-127 为两 commit 合并票）+ TD-154 复核关闭，候选区清空；1223 tests / 99.01%；波 1 首派 3 票全中断/失败重开 3 次全通过、波 2/3 首派即过；期末四轴需修 Recommended 无阻断（docs 2 项 + 代码级 2 项落债 TD-162/163）；真机 GUI 冒烟 SMOKE_OK；Neat 清场 11 分支 + 6 worktree + 2 scratch；新债候选区 TD-158~163
  - 本批最有价值发现：**波 1 首派三票全被网关层/中断打断，重开 3 次全通过零返工**——全自动档并行波次首波应先发探路票验证网关健康（skill 已有此规则，实证应验）；合并票两 commit 粒度把 5 串行波压到 3 波，显示族串行链成本减半

- 2026-08-27（td-consume-4 批次，project-kickoff 全自动档标准档）消费 TECH_DEBT TD-33~42：5 工单（T-035~039）全合并；547 tests / 99.69%；期末四轴 0 阻断；修复轮次 0/5；冒烟通过（含 F1 QtNetwork 前置真机验证）；Neat 清场 5 分支 + 5 worktree + scratch 批次
  - 本批最有价值发现：**F1 真机验证**（T-037 把 QtNetwork 构造前置到 QApplication 之前不崩溃，防御路径生效）+ 范围授权教训（TD-34 GUI 层验收的实现文件 generic_page.py 在 plan-tickets 文件范围外——验收语义指向的实现文件可能不在范围，plan-tickets 需交叉核对验收锚点与其实现文件）

- 2026-08-27（display-gui 批次，project-kickoff 全自动档标准档）9 工单（T-026 spike + T-027~034）全合并；534 tests / 99.69%；期末四轴 0 阻断；修复轮次 0/5；Neat 清场 21 项；TECH_DEBT 新增 TD-33~42 落盘
  - 本批最有价值发现：**spike#1 实测推翻参考实现 win-hdr-fix 的两条假设**（「需管理员」「先 remove 再 add 才生效」均不成立）——参考实现的实证结论必须在本机复验，不能照搬；以及 T-028 HRESULT 符号数缺陷（ctypes `c_long` 返回负值 vs 无符号常量比较永不匹配，真机 `--status` 才触出）

- 2026-08-28（gui-theme 批次）T-058：finesse 产品寄存器 QSS 主题落地（青玉强调 + 冷灰中性，UI 偏好区已并入）；645 tests / 99.72%
- 2026-08-28（arch-deepen 批次）T-060~064：capture 深模块 / param_codec encode-decode 往返 + decode 行为修正 / GAMMA_MIN-MAX 公开 / _ACTIONS 单声明 / refresh() 公开；733 tests / 99.56%；期末四轴 0 阻断，落债 TD-70~79
- 2026-08-28（clean_cache 面板空转修复 + td-consume-9）：PIPE 转发 + IsInputRedirected 守卫根治面板空转；detect_interactive_wait 契约（T-059）；679 tests / 99.45%
- 2026-08-29（clean_cache 加固批次）T-065（ADR-011）：覆盖面 uv/腾讯视频/npx/TRAE + cleanCommand 超时 / ReparsePoint 跳过 / uv --force；746 tests / 99.56%；dot-source 误触真实 clean 事故教训
- 2026-08-29（td-consume-11/12 批次）T-067~072：ProcessLineReader 行流单点 + ParamSpec to_dict/from_dict 单源 + redirect OSError 中文错误；777 tests / 99.57%，候选区 TD-01~79 全处置
- 2026-08-30（elev-bootstrap 批次）T-073（ADR-012）：UAC 提权不继承调用进程 env 实证（No module named ctrl 根因）+ bootstrap 引导链，真机 UAC 冒烟通过；785 tests / 99.57%
- 2026-08-31（td-consume-13/14 批次）T-074~077：bootstrap 流安全 + CTRL_HOLD_OPEN 单源 / gamma 基准磁盘持久化 + int-only 收紧 + 兜底不粘；810 tests / 99.58%
- 2026-09-04（td-consume-15/16 + acl-system + td-consume-17 + 墓碑 + 人眼验收批次）：T-078 reset_ramp 时序 / T-085~087（ADR-013）ACL 检查系统 / T-079 conftest env 快照恢复 / 删 2 死常量 / GUI 桌面手动验收 + display 肉眼确认闭环；831 tests / 98.82%；候选区 TD-01~100 全处置里程碑（17 批次）
- 2026-09-23（td129-td132 批次，全自动档标准档）：TD-129/TD-132 折回消费——T-120（GUI DWM 读回降级警告可见化：catch_warnings + 状态位 + 状态标签锚）/ T-121（巨 int 逃逸校验契约归一：float 前置归一四处同形，写侧中文 ValueError / query None / 谓词 False；10**400 防复发锚 + 2**200 float32 回归锚）；1138 tests / 98.96%；波末增量 0 阻断、期末四轴 Critical 0 / Recommended 1（＝文档同步缺口，收尾 docs commit 闭环）；候选区 TD-128/130/131/133~135/137~141（脚注「当前最大 TD-141」）
  - 本批教训：**批次文档收尾须与代码合并同一窗口落盘**——docs 收尾晚于期末审核窗口时，审核把「收尾迟到」记为 Recommended 文档同步缺口（S1/SP1 双轴同源）。下批把 TO-TICKETS/TECH_DEBT/PROJECT_REFERENCE/DEV_LOG 的收尾放在「波末合并后、期末审核派发前」，不要等审核回来才写
## 稳定模式（td-consume-27 批次，2026-11-13）

- **clean_cache 生态位定型**（ADR-028 修订后）：加载期零动态编译（`Get-PathProbe` 三态探测：missing/failed/present，探测失败绝不零计成成功）；目录清空唯一通道 = `Remove-DirectoryTree` .NET 递归（逐层判 ReparsePoint 只删本体，**robocopy 已绝迹**——`/MIR /XJ` 会穿透树内 junction 清空链接目标，见经验笔记）；枚举唯一原语 = `Get-FsEntries`（计量/匹配/开发垃圾三消费方共用，junction 只披露「保留 N 个链接」不删本体）；兜底触发唯一入口 = `Test-FallbackWarranted`（`-Scenario Tree|File` 两口径逐位等价，表驱动 oracle 锁定）；状态诚实 = `Failed==0 且残留>0 或 Links>0` 不得「已清理」；`-WhatIf` 对 agreeRequired 项输出「需 -Agree，将跳过」。
- **PS1 单测接缝 = AST 提取**：`Parser::ParseFile` 提取函数定义 + 传递依赖闭包 + `Invoke-Expression`，测试在 tmp_path 真调用（harness `_AST_HARNESS`/`_run_ast_unit`，勿重写）——生产不加守卫不拆文件（plugin_store 单文件模型约束）；拼接 `\?\` 路径禁用 Join-Path（PS 5.1 拒收，字符串拼接）。
- **删除工具的护栏边界必须逐个证伪**：外部工具的「排除」语义只保护它声称的方向（/XJ 保护遍历不保护镜像删除）；凡删除链引入新依赖/新参数，先构造穿透实验再采信。
- **链中重启实操**：同一 Implement 上下文连跑多票后连续上游超时 → 交接摘要落盘（现状 + 已诊断红点 + 剩余工作 + 已定前提）→ 新调用接续，一次通过；未提交工作原样保留是前提（不回滚不重建）。

## 稳定模式（td-consume-29 批次，2026-11-15）

- **clean_cache 覆盖清单单源化**（TD-181 消费，ADR-028 修订）：`plugins/config.json` 为运行时事实源（存在即优先），`Get-DefaultConfig` 内置默认表与其**同步维护为同一并集、逐字段对齐**（name/root/caches/process/risk/note，`_doc` 元数据豁免）——双源 diff 测试守护两源 apps 集合相等（`_APPS_ALIGN_FIELDS` 固定 6 键常量，**新增字段需显式扩展否则不被守护**，期末四轴 Falsify 🟡 记录）；并集即上限、无第三条路径（删除面不扩宽回归锚）；导入环境无 config 时回退内置默认表（`plugin_store.import_tool` 不复制 config.json，docstring 声明）。
- **PS 记账/出词收敛套路**：`$script:scanTotal +=` 12 处直写收敛为 `Add-ScanTotal` 单点（非正数不进账，语义不变）；包管理器 clean 分支不再硬编码「已清理」改三态出词（Format-RemoveStatus 口径，目录残留披露「部分(…，残留 N)」）；apps gateFailed 并入聚合（同一行不得「已清理」与「探测失败」同现，失败计数只算删除失败不误计）；devGarbage 根 failed/missing 两态分离。PS 反向锚计数法：`scanTotal +=` 字面 == 1（helper 内）、`switch ($script:mode)` 9 = 分派 8 + 横幅 1。
- **npx 进程防护**（TD-183 消费）：`process:["node"]` 而非 `["node","npx"]`——**Windows 无 npx 同名进程**（是 npx.cmd 包装，`_npx` 真实写入者是 node.exe），`"npx"` 名单项在 Get-Process 下无效；`"node"` 命中所有 node.exe 属过度防护但方向安全（宁可跳过不可删错）；进程检查提取 `Test-ProcessRunning` 共享 helper（空白过滤 + `Get-Process -ErrorAction SilentlyContinue` 非空判定）。
- **DisplayPage 状态刷新归属**（TD-179 消费）：变更路径补显式 `_update_status()`（gamma 拖动/还原对齐 DWM 成功路径）；`_run_dwm_action` 失败详情经结构化异常传递、**不拿 QLabel 当消息总线**（removeprefix 文本回读是反模式）；gamma/DWM 回滚收敛共享 `_rollback_to_prev` helper（恰 1 def 2 消费，S4 源码锚）；`sync_ui_from_applied` 全仓零生产调用的陈旧死代码删除——「外部同步」真路径是 `refresh()`/`refresh_display_page()`。
- **display 双表交叉不变式**（TD-182 消费）：`display_schema.py` DISPLAY_PARAMETERS 13 名 vs `display.py` `_ACTIONS` 11 动作 params 平铺——各有自洽锚但零交叉断言是盲区；交叉不变式测试（双向 + Falsify 诊断含具体参数名）守护，只加一侧即报警（schema 多出→CLI「没有指定动作」；_ACTIONS 多出→提权重启静默丢参）。
- **完成通知 ≠ 完成事实，以磁盘为准**（T-141 网关超时中断实证）：Implement/审核子智能体汇报前断连（「等待上游恢复超时」）≠ 失败——先查 worktree commit + 证据文件 + 全量跑；工作已完成仅未汇报 → 主会话磁盘核验 + 复核后不重开（重开仅为汇报仪式）、直接按完成走。这是「子代理证据落盘必须磁盘核验」的完成侧变体。

## 变更记录（续）

- 2026-11-15（td-consume-29 批次，全自动档标准档）消费 TECH_DEBT TD-179~183 全量：5 立项 4 工单（T-140/141/142/143，波 1 三票并行 + 波 2 串行合票），候选区清空里程碑；1292→1327 passed / 99.08%；波 1 增量审核无阻断（F1 note 键已吸收）；期末四轴 0 Critical / 0 Recommended（2 🟡 Falsify 观察 + 1 nit docstring 笔误已修）；T-141 汇报前网关超时中断但磁盘核验完整（未重开）；主会话独立真机 -Scan 冒烟（独立验证不采信汇报）
  - 本批最有价值发现：**删除链证伪输入必须覆盖「进程运行态」「覆盖清单双源」「状态词合法组合」**三新维度（前批已证链接拓扑）——本批 Falsify 最佳输入 = npx 进程在跑（防删错）、双源漂移（防清单失真）、「已清理」+「探测失败」同现（防状态谎报），三者都被波内实现/审核拦在合入前
