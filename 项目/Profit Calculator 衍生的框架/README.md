---
type: index
tags: [衍生框架, pyside6, project, 收益计算器]
date: 2026-08-06
---

# Profit Calculator 衍生的框架（PySide6 骨架）

从收益计算器剥离业务逻辑提炼的 **PySide6 桌面应用骨架**（框架自称 `pyside6-skeleton`）：新项目只需替换 `domain/` 包、重写 `adapter.py`、改 `config.py` 即得一个可运行的 CRUD + 图表桌面应用。

- **目录**：`D:\Desktop\Craft\Profit Calculator  衍生的框架`（注意两个空格）
- **技术栈**：Python 3.10+ / PySide6 / pyqtgraph / pytest（覆盖率 ≥90%）/ PyInstaller onedir
- **架构**：`skeleton/`（通用不动）/ `domain/`（领域整体替换）/ `app/`（UI 层）/ `adapter.py`（唯一 seam，`typing.Protocol`）
- **来源**：2026-08-04 R 系列框架抽取实验——[[实验性重构不留痕]] 是该实验的教训本体
- **git**：非独立仓库，挂在源项目分支 `profit-calculator-derived-framework`（97 commits 共享历史，origin 仍指向 profit-calculator）
- **定位边界**（CONTEXT.md 声明）：单用户、纯本地、百级记录可；多用户/万级数据/流式不可

## 沉淀价值（「怎么做对」的正解登记）

本页是知识库中唯一「正解模板」型项目页——不是坑，而是坑的反面教材物化：

- **三包分离 + DomainAdapter Protocol seam**：UI 层绝不 import domain，adapter 的 `FieldDef/ColumnDef/SeriesDef` 配置驱动输入/表格/图表自动渲染——换领域不动 UI 层
- **五文档分工**：TO-TICKETS（唯一待办）/ DEV_LOG（已做）/ CODE_WIKI（技术细节）/ CONSENSUS（约定+决策）/ ADR（方向性决策）——见 [[个人项目文档架构模板]]
- **doc_sync.py 防漂移实现**：CODE_WIKI 的 `<!--AUTO:lines/tests/sig-->` 标记由脚本从 pytest collect + AST 提取自动刷新，pre-commit `--check` 拦截——[[文档漂移]] 笔记的正解配套
- **PyInstaller onedir 打包瘦身清单**：Qt 二进制白名单 + excludes 剔 matplotlib/PIL + spec 入库可复现
- **主题 token 化设计系统**：68 个通用 token（交互态/表面层级/语义色对/图表序列色），新信号类型只改映射不动 UI
- **存储保留/展示窗口解耦**：[[存储保留与展示窗口解耦]]（ADR-0003）
- **数据目录约定 + 幂等迁移**：[[打包覆盖丢数据]] 的落地正解

## 待办建议（未执行，需用户拍板）

- **弃置克隆状态**：41 个未提交文件——清理（`rm -rf`）或正式登记；主库 TO-TICKETS 声称 R-01~R-09 已完成但主实现只存在于本目录（对账见 [[实验性重构不留痕]]）
- **目录名双空格**：建议改名 `Profit Calculator 衍生的框架` → 无空格名
- **独立分发**：若作模板仓库需拆仓库（当前 origin 仍指向源项目）
- **复制毒物**：`cp -r` 会连带 `.git`（97 条业务历史）/`dist/`/`build/`/`.serena`——新项目用本模板第一件事是清理
