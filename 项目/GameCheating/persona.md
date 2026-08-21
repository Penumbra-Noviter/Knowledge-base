---
type: persona
tags: [persona]
date: 2026-08-19
project: GameCheating
status: active
---

# GameCheating Persona 画像（L3）

## 本人偏好

- 本地优先，不依赖云服务；优先 CLI，再包装 GUI。
- 先结论后证据，代码和文档保持可运行、可追溯。
- 版本控制用于保存阶段性基线，不把运行时用户数据放进仓库。

## 固定约束

- GameCheating 是通用 Ren'Py 存档修改器与两个专用工具的同一相关项目。
- 原存档不覆盖；未知格式、未知对象和不确定字段拒绝写入或保持只读。
- 代码事实以仓库文档为准，长期稳定偏好和约束维护在本 persona。

## 稳定模式

- 通用核心与专用工具共享领域边界，但专用行为不复制成仓库级规则。
- 测试从对应模块目录运行，避免 Python 相对导入路径造成假失败。
- 知识库只保留可跨会话复用的项目语境，开发流水记录保留在仓库 `DEV_LOG.md`。

## 变更记录

- 2026-08-19：建立 GameCheating 项目画像，并与仓库首个 Git 基线互链（来源：`DEV_LOG.md` / initial commit）。
