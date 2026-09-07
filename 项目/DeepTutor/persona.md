---
type: persona
project: DeepTutor
date: 2026-09-05
---

# DeepTutor — Persona

## 项目偏好（长期）

- **fork 纪律**：上游遗产文件（README/CONTRIBUTING/CLI 语义）不随手改；本地新增层（`deeptutor-desktop/`、`packaging/`、文档体系）与上游改动分文件或分提交，降低合并冲突面。上游移植走显式 port 提交（先例：ce96ffbd partner 扫码连接）。
- **桌面端验收必须冒烟真实用户工作区**：打包后用 `--home D:\Desktop\tools\DeepTutor` 启动并断言数据非空（工作区回归两次教训，详见 [[经验/工作区解析回归让数据看似消失]]）。
- **用户偏好契约**：自定义工作区路径（`D:\Desktop\tools\DeepTutor`）是用户数据契约，任何重构不得单方面改默认值。
- **测试环境事实**：Windows 开发机——`resource` 模块不存在（sandbox runner 测试 skip 是正常态）；tests/ 重名 basename 依赖 `--import-mode=importlib`（pyproject addopts，勿清空）。
- **API key**：`data/user/settings/model_catalog.json` 含真实 key，永不提交公开仓库。

## 协作风格

- 中文交流；先结论后证据；文档更新与代码改动同提交。
