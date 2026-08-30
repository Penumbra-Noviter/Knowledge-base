---
type: persona
tags: [persona]
date: 2026-08-30
project: AI自动获客
status: active
---

# AI自动获客 Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好
- 先结论后证据，简洁清晰；代码必须完整可运行，不写半截代码
- 业务逻辑英文命名，公开函数完整 type hints（Python）/ JSDoc（JS）+ docstring
- 关键决策走 ADR（先 ADR 再代码）；测试覆盖率基线 ≥90%
- CLI 优先、渐进式开发、本地无云依赖

## 固定约束
- 技术栈：**Electron 壳 + 接口拦截（通道A 页面 hook）+ BrowserView DOM 自动化（通道B）+ 本地 FastAPI + Vue3 前端 + SQLite**；AI 接用户自己的云端 Key（`DOUYIN_AI_KEY/BASE_URL/MODEL` 环境变量，不入代码）
- 架构边界（ADR-001，勿重开）：FastAPI=业务核心（任务/线索/审核/AI/调度/历史；SQLite 唯一写者）；Electron 主进程=浏览器执行器（BrowserView/登录态/hook 注入/DOM 执行/代理）；Vue3 渲染=纯视图（数据走 HTTP、浏览器控制走 IPC）
- 双通道分工：数据抓取只走通道A 拦截（快/稳/干净）；交互副作用（评论/私信/关注/点赞）只走通道B DOM；审核队列硬约束「人工确认后才发送」（T-04）
- 复用规范：注入脚本用页面主世界 hook 而非 webRequest；userKey 去重键按规格优先级；三态状态机 stopped 可重启、completed 为终态
- SQLite 运行产物不入库；Key/凭证走环境变量；token 绑定 127.0.0.1

## 稳定模式
- **行为规格复用**：以逆向对象的 BEHAVIOR_SPEC 为唯一行为蓝本，不复制专有代码、不接触原版授权体系与远端服务（合规红线）
- **真实环境补验**：自动化用 fixture/构造 DOM/本地 stub 达成工程性门槛（用户不在场时）；真实抖音页/真实 AI Key/真实扫码留待用户人工补验——验收汇报必须标注「已知降级」而非失败
- 双波纵深交付节奏：先数据主线可演示闭环，再 AI/DOM/调度完善；每波结束 Falsify 对抗审核（跨工单耦合是波末审核重点）
- kickoff 全自动档偏好：仅 Grilling 共识 / Neat 删除清单 / 最终交付三处必确认；其余仅汇报