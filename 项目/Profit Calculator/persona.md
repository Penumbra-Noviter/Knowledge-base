---
type: persona
tags: [persona]
date: 2026-08-09
project: 收益计算器
status: active
---

# 收益计算器 Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好
- CLI 优先：核心逻辑先以命令行形式实现，再包装 UI
- 先结论后证据，简洁清晰；代码必须完整可运行，不写半截代码
- 业务逻辑英文命名，公开函数完整 type hints + docstring；不允许 `except: pass`

## 固定约束
- 桌面端：PySide6 + pyqtgraph
- 数据：JSON 原子写入（write + rename）+ 滚动备份 + 损坏自愈 + 日志轮转
- 架构：UI 层（`app/`）与业务逻辑（`calculator.py`）严格分离；`app/` 不直接操作 `data.json`
- `DayRecord` 采用 frozen dataclass 作为不可变数据模型
- 所有模块定义 `__all__`
- Windows 下 QApplication 单实例需用 QSharedMemory
- 打包：PyInstaller（`delta_force_dashboard.spec` 入库可复现）；onedir + UPX 瘦身
- 运行态数据与程序安装目录分离（`Path.home()/收益计算器`）
- 测试：pytest，覆盖率目标 ≥ 90%

## 稳定模式
- 数据安全三问：程序目录与用户数据分离？测试路径与生产路径隔离？空环境能启动？
- 测试路径 100% 隔离：持久读写路径显式注入 tmp_path（[[测试夹具污染真实用户数据]]）
- 测试数据相对 now 生成，绝对日期是定时炸弹（[[固定日期测试是定时炸弹]]）
- 金额一律显式 Decimal 舍入（ROUND_HALF_UP），不用内置 round（[[Python 银行家舍入]]）
- 结构性防复发优先于再次修复：AST 断言钉死顺序（[[主题色 import 期冻结]]、[[空环境首启即崩]]）
- 用户输入映射为目录名/文件名：校验集覆盖全边界（符号+控制字符+长度上限），目录创建 try/except OSError 返回可读原因（[[输入映射文件路径的校验边界]]）
- 主题改动按「双主题 × 全部渲染路径」回归，色值用最严格消费者格式（[[双主题渲染路径回归]]）；2026-08-11 起升级为**树遍历契约**：启动期收集具 `apply_theme` 组件（父拥有子树防双扇出），refresh_theme 统一调用，新增主题化组件自动纳入（C1-08 落地）
- 测试注入 seam 优先于环境变量哨兵与私有属性 monkeypatch：构造注入（`client=None` 自建兜底）取代生产代码里的测试守卫（C2-02/03 落地）；测试与生产共享同一 stub 工厂
- 设置文件 schema 所有者：已知键白名单 + 未知键保留 + `update(patch)` 合并回写，窗口层键访问经常量（C3-10/11 落地）
- 装配与渲染收敛契约（C4）：仪表盘装配走 `build_dashboard` 直构（bundle 字段名锁定测试防漂移），MainWindow 只解包保留属性——新增组件改 bundle 不改 MainWindow；KPI 渲染唯一入口 KpiPresenter 三出口（update/apply_theme_styles/reset），共享纯函数单一来源经调用期延迟解析规避循环 import（C4-01/02 落地）
- KPI 动画 per-tile 槽契约（C4-债2/C4-债3，根治版）：数值落值三态——直落（数据不足/数值未变/动效关闭）/ 动画 count-up / 同槽替换即落终；动画槽 `_countup_anims: dict[QLabel, ...]` 每磁贴独立、在途动画始终可寻址（顶出截断语义不存在）；落值入口统一 pop + `setCurrentTime(duration)` 优雅落终（同磁贴再触发零陈旧帧窗口，直落分支同调用内被 setText 覆盖）；动画生命周期随状态收敛——自然结束 → finished 回调（**weakref 闭包破引用环**：强闭包在销毁路径无 reset 的生产代码（closeEvent 不调 reset）下会令 presenter+在途动画存活 → 迟到帧写已删 label → access violation）→ identity 检查移除 entry + deleteLater，reset 显式 stop+deleteLater，Qt children 与 dict 双双有界；先枚举在途动画寻址路径再定修复（C4-债1 两轮推翻：全局 stop 冻结中间值 / per-label 漏出槽动画）（C4-债1 收敛 → C4-债2 根治 → C4-债3 生命周期闭环）
- 动画生命周期收敛模式（C4-债3/债4 通用化）：动画对象生命周期随状态收敛——启动新动画前 `stop()` 旧动画（零帧零 finished 防残帧），`finished` 回调 identity 检查后 `deleteLater` 回收；跨目标 stop 需先分目标域（同目标 stop 无条件安全，跨目标会冻结中间值）；「有界」断言按实体生命周期口径写（dict 有界 ≠ 对象有界）；测试构造裸控件即时销毁须排水 qWait（GC abort hazard）；**2026-08-13 C4-债5 补充：DWS + 强闭包 finished 回调组合在「控件动画在途时销毁」路径确定性崩溃**（强闭包环 edit→_shake_anim→anim→信号→闭包→edit 依赖循环 GC 整链回收与 DWS 延迟删除互踩 → 双重删除 access violation）——finished 闭包一律 weakref 破环（kpi_presenter/input_panel 同款）；「已有 DWS 回收模式的组件补 finished 清理」不可照搬（fade_in_widget 同族风险，C4-债6），照搬前先枚举销毁路径
- 存储容错读统一 seam（C7）：所有 JSON 容错读走 `try_load_json`（读写对称含加密）；可选依赖异常类惰性持有（模块顶层零 import）；容错契约含解密失败（InvalidToken → None + on_error）（C7 落地）

- 2026-08-12 更新：KPI 动画落终契约并入（来源：C4-债1 技术债批次，commit 8bc4e68..8ba15eb）
- 2026-08-12 更新：KPI 动画落终契约 → per-tile 槽契约（来源：C4-债2 根治落地，commit d33def8..e6c8bc5）
- 2026-08-12 更新：per-tile 槽契约补生命周期闭环 + weakref 破环模式（来源：C4-债3 落地，commit d3fbeff..99e2f5a）
- 2026-08-12 更新：新增动画生命周期收敛模式稳定条目（来源：C4-债4 落地，commit dcb941e..8ecf654）
- 2026-08-13 更新：生命周期收敛模式条目补 DWS+强闭包环在途销毁崩溃实证 + weakref 破环定案（来源：C4-债5 落地，commit 641ab0c..6001a4a）
## 变更记录
- 2026-08-12 更新：装配/渲染收敛契约 + 存储容错读统一 seam 两条稳定模式并入（来源：C4→C7 架构批次，commit 98b2ee1..9916efb）
- 2026-08-11 更新：主题契约树遍历、注入 seam 优先、设置 schema 所有者三条稳定模式并入（来源：架构加深批次 C1/C2/C3，commit 633f549 起）
- 2026-08-09 建档（来源：全局 AGENTS.md §二.2 + 收益计算器项目经验复盘）
