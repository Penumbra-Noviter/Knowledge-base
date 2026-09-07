---
type: project-page
project: DeepTutor
repo: D:\Desktop\downloads\DeepTutor\DeepTutor-main
date: 2026-09-05
---

# DeepTutor

Agent-native 智能学习伴侣（fork 自 HKUDS/DeepTutor 上游）——双层插件模型（Level 1 Tools / Level 2 Capabilities），CLI + WebSocket + Python SDK 三入口；本地在其上加 Tauri 桌面壳（PyInstaller sidecar + 静态导出前端）。

## 指针

- 仓库内文档体系：`AGENTS.md` / `DOCUMENTATION_STANDARDS.md`（完整档，2026-09-05 升档）/ `CODE_WIKI.md`（技术细节权威源）/ `TO-TICKETS.md` / `DEV_LOG.md` / `TECH_DEBT.md`
- 上游同步：fork 保持上游遗产文档（README / CONTRIBUTING 等）随上游；本地桌面层文档（`deeptutor-desktop/README.md`、`packaging/`）自维护
- 侧链：[[经验/工作区解析回归让数据看似消失]]（桌面端工作区偏好契约教训）

## 当前状态（2026-09-05）

- 活跃工单 0；技术债候选 3（TD-14 打包路径超长 / TD-15 虚拟滚动二轮 / TD-16 onedir 约束）
- 最近批次：UX 四特性（Fork/Background/Plan Mode/Interrupt & Redirect）、UX 三连修、desktop fixes r3、工作区回归修复、partner 扫码连接上游移植
