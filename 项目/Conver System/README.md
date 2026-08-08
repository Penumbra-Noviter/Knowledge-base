---
type: index
tags: [conver-system, project]
date: 2026-08-02
---

# Conver System（角色对话系统）

当前主力项目。Phase 1-6 已完成（含 P6.1/6.2/6.3、P2.5）；Ollama 已于 2026-08-03 封存，活跃待办 P6.4 Tauri + P6.5 多 tab（以仓库 TICKETS.md 为准，2026-08-06 审计修正）。

技术栈：FastAPI + SQLAlchemy 2.0（同步 ORM）+ SQLite（`PRAGMA foreign_keys=ON`）+ Pydantic v2 + HTML/Vanilla JS ESM，LLM 走自定义 Provider 抽象层。

**单一事实来源**：技术细节、ADR、TO-TICKETS、DEV_LOG 都在项目仓库。本目录只放：

- 项目级经验复盘
- 踩坑原子笔记入口（打 `conver-system` 标签聚合）
- 外部来源引用

## 相关经验
- [[审计快照过期需复核]] — 审计快照与当前代码不一致，执行前需复核
- [[Provider 标识符 keyid 分离]] — 多 Provider 系统的标识符演进
- [[API 凭证跨协议兜底设计]] — 聚合平台凭证共享模式
- [[SSE 流式前端状态陷阱]] — ReadableStream done 分支漏处理
- [[前端模块化拆分中的循环依赖处理]] — 钩子函数模式
- [[response_model 统一驱动序列化]] — 退役手写 dict
- [[架构摩擦渐进发现与分批落地]] — 多轮轻量评审替代大重构
- [[连接测试必须测用户实际配置]] — 测试与真实使用路径必须同源
- [[LLM 生成 JSON 三级提取策略]] — 代码块/花括号提取 + 白名单过滤
- [[Pydantic model_fields_set 区分显式传参]] — 回退链用显式性判定，不用值比较
- [[无框架前端 fetch seam]] — API 层 setFetch 注入，Vanilla JS 可单测
- [[DB 枚举列按值存取]] — values_callable，存量 VARCHAR 零迁移
- [[OpenAI base_url 拼接语义]] — SDK 服务根语义 vs 用户根地址习惯
- [[FastAPI 静态挂载顺序契约]] — mount("/") 遮蔽后注册路由
- [[Conver System 高频小坑汇总]] — SDK 异常翻译 / Git Bash 遮蔽 / Test 前缀 / 首 token 气泡 / 模型名散落
