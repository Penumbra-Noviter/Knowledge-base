---
type: persona
tags: [persona]
date: 2026-08-21
project: 代订场
status: active
---

# 代订场 Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时最先读。

## 本人偏好
- 全自动流程：项目启动使用 project-kickoff 全自动档，只保留三处确认（Grilling 共识、Neat 删除清单、最终交付）
- CLI 优先：核心逻辑先用命令行形式实现，再包装 UI
- TDD 驱动：每个模块先写测试再实现，覆盖率 ≥ 90%
- 最小依赖：核心仅 requests + pycryptodome，不引入 python-dotenv 等可自写替代的依赖
- 支付前全自动：抢订到「待支付」状态即停，支付人工收尾

## 固定约束
- 目标场馆：成都市老体中心羽毛球馆（TARGET_VENUE=1543，sportId=4028f0ce5551abf3015551b0aae50001）
- 放场时刻：每天 08:00（RELEASE_TIME=08:00）
- 路由方案：C（API 直调），A（UI 图像识别）兜底但未实现
- 竞速策略：4 场并发 → 限流转串行指数退避 → 5 分钟重试窗口
- 验证码：选超级鹰打码平台（交付给客户用，稳定/按次计费），弃用 VLM
- 交付形态：Windows 计划任务每日 08:00 拉起 `python -m app.scheduler`，源码跑通
- token 策略：内存提取（probe/extract_token.py），不自动重登，过期弹窗提示人工处理

## 稳定模式
- 项目文档体系：CLAUDE.md + PROJECT_REFERENCE.md（无，用 spec 替代）+ TO-TICKETS.md + DEV_LOG.md
- 技术栈：Python 3.12 + requests + pycryptodome + pytest-cov
- spec 权威：specs/booking-robot-spec.md 是唯一 Spec 权威，修订日志记录每次变更
- 探针文件：probe/API_FINGERPRINT.md 是 API 契约权威，probe/extract_token.py 是 token 运维工具
- 配置模式：app/config.py 用 os.environ + Final 常量，环境变量注入
- 测试架构：fake ApiClient + 虚拟时钟 + injectable notifier，不触真实网络
- 并发模式：git worktree 隔离并行工单，波末 --no-ff 合并
- **测试环境**：跑测/验收必须干净 env（`unset TARGET_VENUE / SPORT_ID / NEED_CAPTCHA`）——`app/config.py` 在 import 期冻结 env 默认值，shell 污染会致 test_config 5 个无关红（波 5 / 波 8 两次实测遇到）；覆盖率命令用点号模块路径（`--cov=app.aj_captcha`，pytest-cov 不接受 `.py` 结尾）

## 变更记录
- 2026-08-20：初始创建，项目启动 | DEV_LOG#1
- 2026-08-21：波 3 补充 — 全自动档偏好、sportId 配置化、验证码开关、交付脚本自检机制、code-review 证伪修复流程 | DEV_LOG#7
- 2026-08-28：波 8 补充 — 测试环境干净 env 约束 + pytest-cov 点号模块路径（实测两次） | DEV_LOG 波 8
