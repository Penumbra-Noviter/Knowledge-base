---
type: lesson
tags: [SDK集成, HTTP seam, 字段透传, 契约, hy-memory]
date: 2026-10-23
project: zcode 环境
source: 自身项目实践
summary: 集成第三方 SDK 先实测内置 HTTP 层透传面；client 签名支持≠服务器支持，且能力声明按场景/路径分裂——subclass 补丁收编为自持 seam
provenance: DEV_LOG〈2026-10-23 · hy-memory 批次（kickoff 全流程，T-01..T-06 完成）〉+〈2026-10-24 · hy-memory 技术债消费批次〉· docs/ADR-001-hy-memory-接入.md（方案 B）· 2026-10-24 第二次实证
status: verified
agents_md_feedback: 印证 AGENTS.md「动手前自检——API 签名/行为不凭记忆写」：内置 HTTP 层的实际透传面是 client 签名与服务器行为的分裂点；能力声明还可能按调用场景/路径分裂，两层均须实测核实
---

# 第三方 SDK 内置 HTTP 层透传面须实测——client 签名支持≠服务器支持，能力声明按场景分裂

## 症状
hy-memory 2.0.4 接入（ADR-001 方案 B：SDK 内置 server + 薄脚本）。`HyMemoryClient` 的 add/search 签名支持 source/meta/scene/workspace_id/branch/include_sources 等字段，但经内置 HTTP server 实际写入/检索时这些字段**全部丢失**——HTTP 层只透传 user_id/agent_id/session_id/metadata。后果：kb-search 的 source/scene/workspace_id 过滤静默失效、coding 摄入的 include_sources 作用域丢失。

**第二次实证（2026-10-24，技术债批次 T3）**：修好透传面后，`workspace_id`/`branch` 仍无法在 lite 检索实例起效——`client.search` 签名接受两键，但 SDK 只在 `decided_scene == "productivity/coding"` 时经 `_search_coding` 应用过滤（client.py:3377-3392），chat 路（normal/schema/proactive）**双端忽略**（add L1621-1623 / search L1755-1761 docstring 明示「Coding 路径专用（chat 路径忽略）」）。实测错仓泄漏：lite 实例带 `workspace_id=smoke-b` 检索仍返回 smoke-a 与 kb。**SDK 能力声明按调用场景/路径分裂**，且自定义 `scene` 参数在 search 里是「显式设置」而非注入。

## 根因
Python client 与底层 core 都支持这些字段，但内置 HTTP server 的 handler 只把**自己代码里声明转发**的字段传给 core——HTTP 边界是一个薄透传层，透传面由 server handler 的参数转发决定，不由 client 类型签名决定。「client 签名支持 = 服务器支持」是集成第三方的错误默认。**第二层分裂**：即便透传面修好，SDK 内部对同一参数的能力声明仍可能按调用场景（chat vs coding）分裂——docstring 与签名是「能力声称」，实际行为按场景分派决定。参数签名支持 ≠ 该参数在你要走的场景/路径生效。

## 代价
第一次：检索过滤与摄入作用域在真实调用里静默失效，薄补丁 `hy_server.py`（subclass 补透传）收编，需字段级回归复测。第二次：为「lite 按仓库过滤」自建了索引标签 + 自动注入三层代码（seam/标签/注入），实测证伪后 R3 收窄债义——revert 标签 hunk + 删除自动注入 + 工具改显式拒绝 `workspace-filter-unsupported`。若先按场景实测，可省一整层实现。

## 教训
集成第三方 SDK 时不要默认「client 签名支持=服务器支持」；先用字段级往返实测（add 后 search 核对各字段是否保留）核实其内置 HTTP 层的实际透传面。**进一步：即便透传面就绪，还要按你要用的调用场景/路径实测能力声明**——签名支持 ≠ 该场景生效；字段被吞/场景不支持时，优先用 subclass 补丁把 HTTP 边界改造成**自持 seam**，且**对不支持的能力做诚实化拒绝（显式报错码）而非静默 no-op**——不静默假过滤，避免交付「看起来生效实际空转」的虚假能力。

## 防复发
- [x] 已落实：接入时做过 add/search 字段往返核对，确认 7 字段丢失清单
- [x] 已落实：薄补丁落 tools/hy-memory/hy_server.py，随仓库版本化
- [x] 已落实（2026-10-24）：T3 实测证伪 workspace_id 场景分裂 → R3 收窄债义；工具对 lite 非空 workspaceId 显式拒绝（workspace-filter-unsupported）；hy-coding-server.md 明示「仓库级过滤仅 coding 实例生效」
- [ ] 补丁加字段级回归断言：source/meta/scene/workspace_id/branch/include_sources 往返保留
- [ ] 「核对内置 HTTP 层透传面 + 场景/路径能力声明」加入第三方集成前置检查清单

## 关联
- [[中间转换层转圈丢字段]]（同为中间层字段丢失，但那是自产 DTO 转圈，本文是第三方 HTTP 边界）
- [[跨语言镜像契约须对真实消费者验证]]（同为主张：契约预期须按真实消费者行为核实，不按声明/镜像）
- [[第三方引擎静默失败面验收须产物三查]]（同批 hy-memory 集成：成功旗标/签名支持均不承载真实能力，行为以产物与实测为准）