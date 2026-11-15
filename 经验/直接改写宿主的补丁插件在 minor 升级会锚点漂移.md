---
type: lesson
tags: [排查方法, 插件, 升级, 补丁]
date: 2026-11-10
project: dsh（Deepseek harness）
source: 自身项目实践
summary: 直接改写宿主文件的补丁插件，minor 跳版也会锚点漂移；稳妥做法是用原始备份逐条比对锚点并显式适配，不能只跑重放看状态
provenance: DEV_LOG〈2026-11-10 · 核心升级 0.2.1-rc.5 → 0.2.2-rc.1 与 purge 三条锚点漂移适配〉· commit 75a14d1 · 基线 b8176f7 · 2026-11-10 dsh 升级会话
status: verified
agents_md_feedback: 印证「对抗性自检（证伪取向）」——重放后 28/37 与基线 31/37 的差异才是信号，"重放成功、无报错"不是证据；也印证「真实样本验证」——判据必须取自上游真实产物（原版文件），而非自己对格式/契约的想象
---

# 直接改写宿主的补丁插件在 minor 升级会锚点漂移

## 症状

`dsh-purge` 是**直接改写宿主核心包文件**的补丁插件（37 条规则、23 个目标文件，改写前留 `.dshpurge.bak` 原件）。核心从 `0.2.1-rc.5` 升到 `0.2.2-rc.1` 后重放补丁：

```
补丁 28/37 已清洗, 4 待清洗
```

对照升级前基线是 `31/37 已清洗, 1 待清洗`。**三条规则静默失效**——状态从 `applied` 退化为 `pending`，即这三项「去限制」在新核心上压根没生效，但整个重放过程**没有任何报错**，`--apply` 退出码为 0。

失效的三条及其上游改动：

- `#12 APPROVAL_ASK_SENTENCE_BYPASS`：上游把 `ASK_SENTENCE` 文案由 `"Approval policy: ask. Operations that require approval may ask…"` 改为 `"Approval prompts are enabled in this session: operations that require approval may ask…"`。语义未变，仅前缀改写——锚点是整句字面量，于是不再命中。
- `#14 ESCALATION_GRANT_UNCONDITIONAL`：`displayReason` 由内联对象 `{ en: …, zh: … }` 抽成助手调用 `escalationDisplayReason(mode, justification)`（顺带新增 ja/es/fr/de/ru/pt）。锚点是包含 `displayReason: {` 的整段请求体，形状一变即失配。
- `#17 FS_OBSERVATION_INTENT_FREE`：`editIntent` 重构为共用助手 `requirePresent(target, actor, verb)`，并**新增 `deleteIntent` 走同一道 read-first 门**。锚点指向旧的内联实现，失配。

## 根因

1. **字面量锚点与上游文案/结构强耦合。** 补丁插件为了精准落点，锚点通常是整段代码或整句提示词的逐字匹配。上游做一次「语义等价的重构或文案润色」——在 minor 版本里完全合理——锚点就整体失配。
2. **失效是静默的。** 规则不命中只体现为「该条未应用」，不影响其他规则、不抛异常、退出码 0。若只看「重放跑通了」，会误判为成功。
3. **半补丁态（本次实际踩到）。** `#17` 有多个 `replacement`（旧的 `writeIntent`、旧的 `editIntent`）。重放后 `writeIntent` 命中了、`editIntent` 没命中——**同一个文件被改了一半**。而整体状态只报「未应用」，看不出文件已处于不一致的中间态。

## 代价

- 三条限制静默失效，若非逐项比对基线，会带着「升级成功」的结论收工——功能缺失被完整掩埋。
- 定位耗时主要在「上游到底改了什么」：手头只有**已打过补丁**的旧核心，无法直接看出原版长什么样；靠猜锚点会反复试错。

## 教训

**「重放成功」不是证据，逐条状态与基线的差异才是。** 对直接改写宿主的补丁式插件，任何宿主版本变动（哪怕只是 patch/minor 跳版）都必须把重放后的**逐条状态**与升级前基线做全量 diff；出现 `applied → pending/missing` 的降级，就是必须显式适配的信号，不是可接受的噪声。

**诊断上游改动，用原始备份做两侧对照。** 补丁插件通常会在改写前留存原件的备份（本项目的 `.dshpurge.bak`）。同时持有「旧版原件（备份目录内）」与「新版原件（新装核心内）」时，直接 diff 这两个**未被污染**的文件，就能一眼看出上游到底改了什么，不必猜测或反推。

## 防复发

- [x] 升级流程固定为：重放后 `--status` 与升级前基线**逐项 diff**（本项目已作为升级收工的硬判据；本次即由该步捕获 3 条降级）
- [x] 诊断步骤固化为：`diff <旧备份>/*.dshpurge.bak <新核心>/*.dshpurge.bak`，直接读出上游改动再写新锚点
- [x] 适配时**保留旧 pattern 并追加新 pattern**（第 3 个 `replacement`），使同一规则同时兼容新旧版式，不破坏旧核心上的可用性
- [x] 多 `replacement` 的规则，新锚点的替换文案**沿用原 marker 字符串**，让「已应用」判定在新旧版式下一致——否则半补丁态会被 markers 误判为「已完成」
- [ ] 未落实：为补丁规则加「锚点缺失即硬失败」的模式（当前 `pattern_not_found` 属软失败，退出码仍为 0）

## 关联

- [[集成插件静默跳过等于从未生效]]
- [[依赖活服务的自检须排在服务重启之后]]
