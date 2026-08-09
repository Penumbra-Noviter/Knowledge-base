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
- 主题改动按「双主题 × 全部渲染路径」回归，色值用最严格消费者格式（[[双主题渲染路径回归]]）

## 变更记录
- 2026-08-09 建档（来源：全局 AGENTS.md §二.2 + 收益计算器项目经验复盘）
