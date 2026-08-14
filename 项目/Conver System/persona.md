---
type: persona
tags: [persona]
date: 2026-08-09
project: Conver System
status: active
---

# Conver System Persona 画像（L3）

> 用途：本项目的长期稳定语境——预检时**最先读**；阶段蒸馏时把「稳定模式」并入此处，一次性教训才写 `经验/` 原子笔记。只记跨会话不变的内容，会变的内容进项目仓库文档。

## 本人偏好
- CLI 优先：核心逻辑先以命令行形式实现，再包装 UI
- 先结论后证据，简洁清晰；代码必须完整可运行，不写半截代码
- 业务逻辑英文命名，公开函数完整 type hints + docstring；不允许 `except: pass`

## 固定约束
- 后端：FastAPI + SQLAlchemy 2.0 + SQLite（**同步 ORM，不使用 async**）+ Pydantic v2
- 前端：HTML + Vanilla JS (ESM)，不引入 React/Vue
- LLM：anthropic + openai SDK，自定义 Provider 抽象层（新增 Provider = 实现 BaseLLM → `__init__.py` 注册）
- 配置：pydantic-settings 读 .env；运行时配置通过 DB settings 表管理
- SQLite 默认不启用 FK → 必须 `PRAGMA foreign_keys=ON`
- 架构分层：api（只做 HTTP 映射）/ models（纯数据定义）/ schemas / services（ORM 操作全在此）/ config / database / main；路由不直接操作 ORM
- 所有包 `__init__.py` 必须有 `__all__`；模块要「深」

## 稳定模式
- 桌面形态偏好：**壳方案**（Tauri 壳 + 复用现有后端，零业务改动）优先于任何「重写」方案（2026-08-11 P6.4 实证）
- 验收以「真实用户路径」为准：自动化冒烟须含**不注入环境变量的干净回归**（注入 env 会掩盖 prod 缺陷）
- kickoff 全自动档偏好：共识确认后全部阶段不等待、仅汇报；无回答的确认点按推荐清单继续（best judgment）
- 桌面交付形态：NSIS 安装器单产物（免管理员 currentUser）；数据目录 %APPDATA% + CONVER_DATA_DIR 覆盖
- 冒烟脚本纪律：绝不清理非自己启动的进程、残留检查限定自己的端口（2026-08-11 误杀网页版 uvicorn 教训）
- 打包回归断言要钉接线行而非 token（`datas=list(_FRONTEND_RUNTIME)` 能抓到 `datas=[]`，token 断言漏报）
- 评审驱动多轮收敛（三轮架构评审 11+5+3 候选零回滚），不设"重构周"
- 单一事实来源：待办只进 TO-TICKETS、已做只进 DEV_LOG、技术细节 CODE_WIKI
- 回退链设计：显式传参判定用 `model_fields_set` 而非值比较（[[Pydantic model_fields_set 区分显式传参]]）
- 用户可见错误在边界处映射为领域可读提示（[[Conver System 高频小坑汇总]]）
- LLM 结构化输出三级兜底提取 + 白名单过滤（[[LLM 生成 JSON 三级提取策略]]）
- 架构深化候选按 /improve-codebase-architecture 报告推荐强度**分批直落** kickoff 全自动批次（ARC-1~8 / ARC-9 Strong 批 / ARC-10 剩余批三次实证：先 Strong 后剩余，两批落地 14 候选全部完成、零回滚）
- 「对齐/镜像契约」类改动必须含真实消费者连接级用例（[[跨语言镜像契约须对真实消费者验证]]：SQLAlchemy 零解码 vs sqlite3 URI 会解码，镜像互相印证 = 盲区共享）
- 架构深化候选按 /improve-codebase-architecture 报告推荐强度**分批直落** kickoff 全自动批次（ARC-1~8 / ARC-9 Strong 批 / ARC-10 剩余批三次实证：先 Strong 后剩余，两批落地 14 候选全部完成、零回滚）
- 冒烟 mock 拦截 glob 陷阱：Playwright route glob `**/api/chats/**` **不匹配无尾路径段的精确路径**（`/api/chats`）——非流式请求漏拦触发 1 次真实外部 API（2026-08-13 td-arch-health 批次冒烟失误）；mock 拦截须同时注册精确路径与带尾段 glob
- 波末增量审核 Falsify 实证价值：3 轮审核抓到 2 个跨工单耦合真实缺陷（parse-document 422 wire 回归 / data-content 属性注入面——后者是「旧流式路径安全、收口后回归不安全」的路径级回归），「波内最该做的验证」成立；修复均先红后绿 + 防复发断言
- 视觉验证降级惯例：模型无图像输入能力时 GUI 冒烟以 DOM 结构化证据为准 + 截图存档供用户查看，如实标注降级（2026-08-13 td-arch-health 批次首次执行）
- 技术债区（TICKETS 非阻断遗留）清理走 kickoff 全自动批次：逐项 git grep 复核现状（审计快照惯例），可修的做、不可达/设计意图的「复核确认维持」关闭归档（2026-08-12 16 项清零实证）；**票面修复建议本身也须实证复核**——TD-15 票面建议 providerSelect.value === '__custom__' 被实测否定（provider 下拉永不为 '__custom__'，条件恒假会让守卫形同虚设），修正为 modelSelect 与裸读分支精确对齐（来源：TD-15~24 kickoff）
- **Falsify 须覆盖防御机制自身状态机**：占位符/序号/id 分配器等「碰撞循环兜底」类自审只测单实例正确性会漏掉跨实例不变量——碰撞循环内部消耗 vs 调用方计数器推进脱钩（TD-38 多代码块首块丢失，期末四轴 Falsify 抓到，来源：TD-29~45 批次）；凡有序分配器必测多实例交错 + 用户内容占用中间序号
- **文档控制字符一律转义文本表示**：`\x00` 文本而非字面 NUL 字节——字面 NUL 让 git 判 binary、Read 工具拒读；heredoc/JSON 多层转义会把 `\\x00` 折叠成字面 NUL（来源：TD-29~45 批次 TICKETS/DEV_LOG 修复）

- **模拟器集成方向偏好**：产品扩展优先「集成现成成熟玩法」（下载的单文件 HTML 模拟器直接静态托管 + iframe 运行）而非从零造玩法引擎；大方向先 prototype 验证链路再 kickoff（2026-08-14 U7 实证：原型验证 → 5 工单 3 波交付）
- **数据面预估须逐项核查**：原型阶段对 22 个游戏 grep 抽查估「约半数纯本地」，T2 实测 22/22 全为 AI 驱动（每款都有 API 配置面板）——静态资产分类等数据面预估，原型阶段就该逐项核查，不抽样估算（2026-08-14 U7-T2）
- **桌面验证自动化法**：Tauri dev 窗口可用 WebView2 CDP 自动化——`WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS=--remote-debugging-port=9222` 环境变量 + playwright-core `chromium.connectOverCDP` 驱动（入口/列表/iframe/返回全流程断言，两次实证 2026-08-14）

- **冒烟脚本扩展必须真实运行验证**：UI 工单的 smoke 脚本改动「node --check 通过」不算验证——U8-T2 的注入段/存档段仅语法检查交付，波末审核 F1/F2 抓到 2 个必现失败（select option 不匹配断言、步骤间视图未恢复），修复时才首次真实跑通并暴露 main 上 3 个从未跑通的冒烟阻塞点（baseUrl 作用域 bug/闭包泄漏/settings 恢复语义）——自动化冒烟以「真实数据真实跑通」为硬验收（2026-08-14 U8+U9）
- **单测 happy path 构造掩盖真实数据面差异**：select.value 赋值不在 option 集静默无效（key-injector 单测只构造匹配 option）；真实游戏 DOM（life-sim cfg-model 仅 deepseek 两选项）与测试夹具的差异只有冒烟/审核能暴露——测试夹具须对照真实资产数据面核查（2026-08-14 U8+U9 波 2 审核）
- **新增前端运行目录必须同步打包面**：PyInstaller spec 的 _FRONTEND_RUNTIME（前端运行子集清单）与 frontend/ 真实目录间须有**双向**防漂移锁——simulators/ 模块加入时漏更新 spec，桌面版游戏列表为空而网页版正常（单向「声明的都存在」拦不住「新增未声明」方向，2026-08-14 simulators 漏打包教训）；防复发：`test_packaging.py::test_frontend_runtime_dirs_all_shipped` 反向差集锁（新增目录未进 spec 即红）+「网页版能跑不证明打包态能跑」——桌面端变更必跑 smoke-desktop 打包态冒烟
- **安装包仅在明确提需求时打包**：常规「打包」= dist/ 根「双击即用」测试包（build-desktop.ps1 `-SkipInstaller` 开关，--no-bundle 仅编译壳）；NSIS 安装器默认不产（2026-08-14 用户重申，脚本已固化开关）

## 变更记录
- 2026-08-14 打包教训更新：新增前端运行目录必须同步打包面（simulators 漏进 spec → 桌面版游戏列表为空；反向差集防漂移锁 + 打包态冒烟硬验收）；安装包仅在明确提需求时打包（build-desktop.ps1 -SkipInstaller 开关固化）（来源：2026-08-14 会话 + commit 71f34b7/5e29f89/2f3dc7e）
- 2026-08-14 U8+U9 批次更新：冒烟脚本真实运行硬验收/单测夹具须对照真实资产数据面（来源：U8+U9 kickoff 全自动档）
- 2026-08-14 U7 批次更新：模拟器集成方向偏好/数据面预估逐项核查/WebView2 CDP 桌面验证法（来源：U7 kickoff 全自动档）
- 2026-08-13 TD-29~45 批次更新：技术债区 17 项清零（11 做 + 5 关闭 + 1 票面修正 TD-31）；期末 Falsify 抓 TD-38 占位符计数器失同步阻断（多实例不变量盲区 → 防复发断言 + 取号单一职责）；文档控制字符转义纪律（TICKETS/DEV_LOG 字面 NUL → \x00 文本）；审核代理空返回残留 _review_falsify.test.js 致 npm test 3 红（重开前须查工作区未跟踪文件）（来源：TD-29~45 kickoff）
- 2026-08-13 td-arch-health 批次更新：架构深化批次（8 工单 3 波，13 做 + 3 关闭）——错误映射/凭据解析/附件头/字段语义/气泡工厂/启动契约八项单点收口；波末 Falsify 抓 2 个跨工单耦合阻断（parse-document 422 回归 + data-content 注入面）；冒烟 mock glob 漏拦教训（`**/api/chats/**` 不匹配 `/api/chats`）；视觉验证降级惯例（来源：td-arch-health kickoff）
- 2026-08-13 TD-25~27 批次更新：平台守卫形态实证（断言级 skipif 不可表达 → 函数级修正，票面建议实证复核惯例再验证）；skipif 可见信号优于静默平台包裹；轻量档单文件测试改动 = 主树独立分支直行（无 worktree）（来源：TD-25~27 kickoff）
- 2026-08-12 TD-15~24 批次更新：票面修复建议实证复核惯例（TD-15 providerSelect 恒假被否定 → modelSelect 对齐裸读分支）；Windows worktree junction 清理坑（git worktree remove 失败 → cmd rmdir 拆链接）（来源：TD-15~24 kickoff）
- 2026-08-12 TD-8~12 批次更新：子代理只读探索禁改主工作树（共享文件写权归主会话）；契约锁测试语义（基线绿非先红）与回归测试区分（来源：TD-8~12 kickoff）
- 2026-08-12 TD 批次更新：技术债清理连续两批实证（16 项清零 + TD-1~7 清零）；「先红后绿」硬验收在守卫类工单全面落地；文档同步前提须 grep 全仓验证（来源：TD-1~7 kickoff）
- 2026-08-12 技术债批次更新：技术债区清理 = kickoff 全自动批次 + 复核确认维持关闭惯例（来源：技术债区 16 项清零 kickoff）
- 2026-08-12 ARC-10 更新：审查候选分批直落模式扩展（Strong 批→剩余批；来源：ARC-10 全自动 kickoff）
- 2026-08-12 ARC-9 更新：Strong 候选直落 kickoff 批次 / 镜像契约须消费者级验证（来源：ARC-9 全自动 kickoff + 期末阻断修复）
- 2026-08-11 P6.4 更新：壳方案/干净回归/全自动档/冒烟纪律/接线断言（来源：P6.4 kickoff + 期末审核）
- 2026-08-09 建档（来源：全局 AGENTS.md §二.1 + 知识库经验笔记）
