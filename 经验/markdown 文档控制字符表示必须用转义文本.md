---
type: lesson
tags: [文档卫生, git, 编码, markdown]
date: 2026-08-13
project: Conver System
source: 自身项目实践
summary: markdown 文档里表示控制字符须用转义文本 \x00 而非字面字节——字面 NUL 让 git 判 binary、Read 工具拒读，且 heredoc 中 \\x00 会被折叠成字面 NUL
provenance: DEV_LOG#2026-08-13 TD-29~45 批次 · commit d96b4ce/c60ee3a · 波 2 增量审核 F8 发现
status: verified
---

# markdown 文档控制字符表示必须用转义文本

## 症状
TICKETS.md 与 DEV_LOG.md 中记录 `[\x00- ]`（字符类）与 `\u0000MDCB<n>\u0000`（占位符形态）时写入了**字面 NUL 字节**：git 对该文件判 binary（`Binary files differ`、`Bin 66066 -> 66072 bytes`），Read 工具报「Unsupported or binary text encoding」无法直接读，git diff 无法文本审阅；新增 heredoc 写入的 `\\x00` 经 bash 折叠成 `\x00` 再次引入字面 NUL，一度误以为替换未生效。

## 根因
- 文档作者用字面控制字符「表示」一个控制字符——字节级污染而非语义表示
- heredoc 与 JSON 参数多层转义：`\\x00` → bash 折叠 → python 字面量 `\x00` = 字面 NUL，替换逻辑自我抵消（replace NUL with NUL）

## 代价
- 两次修复 commit（d96b4ce TICKETS.md、c60ee3a DEV_LOG.md）
- 历史 blob 已含 NUL 的 commit 永远显示 binary diff（两侧判定），未来提交才恢复正常
- Read 工具不可读期间用 python 解码绕行（预检阶段额外步骤）

## 教训
**文档中表示字节/控制字符一律用转义文本（`\x00`、`\\x00`、`\u0000` 按目标读者语境），绝不写字面控制字节**；涉及反斜杠的写入（heredoc/JSON/脚本字符串）先验证目标文件字节级结果（`count(b'\x00')`），不要信任中间层转义。

## 防复发
- [x] 已落实：TICKETS.md / DEV_LOG.md 字面 NUL 全部转义为 `\x00` 文本
- [x] 已落实：写入脚本用独立 .py 文件（Write 工具）而非 heredoc，避开 bash/JSON 折叠层
- [ ] 待落实：未来写文档含控制字符描述时，提交前 grep 目标文件 NUL 字节

## 关联
- [[]]
