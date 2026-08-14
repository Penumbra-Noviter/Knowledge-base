---
type: study
tags: [headroom, ai-agent, token, proxy, zcode]
date: 2026-08-13
project: ZCode 环境
source: 实测（2026-08-13 本机 uv tool 隔离安装 headroom-ai 0.35.0，库模式 + wrap zcode 实测）
status: verified
---

# Headroom AI 上下文压缩工具：验证结论与使用手册

> 2026-08-13 本机实测。来源：D:\Desktop\downloads\headroom-main（GitHub: chopratejas/headroom，Apache 2.0）。

## 是什么

AI 代理的上下文压缩层：在 agent 与 LLM 之间加一层，压缩 agent 读取的工具输出/日志/文件/对话历史后再发给模型。本地运行、可逆（CCR 缓存原文）。宣传：JSON 省 60–95%，coding agent 省 15–20%。

## 2026-08-13 实测结论（重要，宣传 ≠ 默认效果）

| 场景 | 实测 | 说明 |
|---|---|---|
| 大 JSON 工具输出（120 条搜索） | 3926→2944，**省 25%** | smart_crusher，语义 3/3 保留 |
| 代码文件内容（Python 源码） | **0%** | **设计使然**，非故障 |
| 多轮会话（5 JSON + 5 代码） | 总 24.8% | 5×smart_crusher，代码全保护 |

- **代码读取零压缩是有意设计**：`content_router.py` 的 `_RELEASABLE_READ_TYPES` 只放行"确信非代码"的数据（JSON 数组/搜索输出/构建日志/git diff/HTML/表格），SOURCE_CODE 与 PLAIN_TEXT 一律原样保护——agent 要 patch 文件需精确字节（源码注释原话："we release only positively-identified data"）
- 宣传的 60–95% 需高冗余 JSON（如重复日志行）；coding agent 15–20% 是整体效果，需 `[ml]` 文本模型 + proxy 组合
- **基础包（无 extras）只有 JSON/搜索/日志压缩生效**；文本压缩需 `[ml]`（Kompress-v2-base，HuggingFace 下载）、代码 AST 压缩需 `[code]`（tree-sitter）、代理需 `[proxy]`
- tiktoken 词表下载本机 10s 超时 → token 计数为估算（比率仍有参考意义）
- 全部验证本地执行，零外部请求（未装 [ml] 前）

## 安装与位置

```bash
uv tool install --python 3.12 "headroom-ai[proxy]"   # 隔离安装（本次实测版 0.35.0）
```

| 位置 | 内容 |
|---|---|
| `~/.local/bin/headroom.exe` | 命令入口（uv shim，PATH 已含） |
| `%APPDATA%\uv\tools\headroom-ai\` | 包本体（隔离 venv，不污染系统 Python） |
| `~/.headroom/` | 运行时数据（日志、savings、CCR 缓存） |

卸载 = `uv tool uninstall headroom-ai`（无残留）。注意：`[all]` extra 依赖极大（本机 15 分钟未装完，已弃用），按需装单个 extra。

## 使用手册

### 模式 1：CLI 检查
```bash
headroom doctor    # 健康检查（proxy 未跑时报"未路由"属预期）
headroom perf      # 分析代理日志（需先跑过 proxy）
```

### 模式 2：Python 库模式（零侵入）
```python
from headroom.compress import compress
result = compress(messages, model="claude-sonnet-4-5-20250929")
# result.messages / tokens_saved / compression_ratio / transforms_applied
```
tool_result 为大 JSON 时最有效；代码内容默认保护不压缩。

### 模式 3：代理模式（wrap）
```bash
headroom wrap zcode        # 起代理 + 打印 ZCode 手动设置指引
headroom unwrap zcode      # 撤销（停代理）
```

## ZCode 接入（2026-08-13 实测）

`headroom wrap zcode` 实际只做两件事：**起本地代理 + 打印手动设置指引**。零写入：
- 不改 `~/.zcode/v2/config.json`（只读检测上游，本机检测为 Z.ai 默认端点）
- 不写 AGENTS.md / CLAUDE.md
- 输出指引：
  - OpenAI Base URL: `http://127.0.0.1:8787/v1`
  - Anthropic Base URL: `http://127.0.0.1:8787`
  - MCP 可选（CCR 取回原文用）：`{"headroom": {"type": "stdio", "command": "headroom", "args": ["mcp", "serve"], "enabled": true}}`
- 用户需手动在 ZCode 桌面端 Settings > Model Settings 填入 Base URL（wrap 不代劳 UI 操作）

**无损替代方案**（问题："要写全局提示词才能完整用吗？" → 不需要）：
1. 手动 `headroom proxy --port 8787` + 手动填 Base URL——与 wrap 效果完全等价
2. 不想用了把 ZCode 里 Base URL 改回原值即可，零残留
3. 写文件的只有独立命令 `headroom learn`（默认写 gitignored 的 `CLAUDE.local.md`；**只有显式 `--target AGENTS.md` 才碰全局治理文件**，不启用即可）

## 注意事项

- 代理是 MITM 型：LLM 流量过本地 8787 端口进程（README 声称 data stays local）
- 默认外联：每日一次 PyPI 更新检查（`HEADROOM_UPDATE_CHECK=off` 关闭）、`[ml]` 模型从 huggingface.co 下载、ONNX 从 cdn.pyke.io 下载
- wrap 是长驻进程，需保持运行；关掉即停（可 `--no-proxy` 复用已有代理）
- 收益预期：本机会话代码读取占比高（该场景零压缩），实际节省主要来自搜索输出/JSON/长日志，低于 README 的 15–20% 宣传值
