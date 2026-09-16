---
type: lesson
tags: [agent编排, kickoff, 网关故障, 韧性, conver-system]
date: 2026-08-29
project: Conver System
source: 自身项目实践
summary: 网关故障期 kickoff 批次韧性组合拳——探针定窗口/降压串行/触线后主会话接续半成品/审核错峰补位
provenance: mobile DEV_LOG〈M1 kickoff 批次〉· .scratch/m1-kickoff/evidence/05-08.md · commit 4e8db1b..78b8a94 · 2026-08-29 M1 会话
status: verified
---

# 网关故障期 kickoff 批次韧性处置——探针定窗口 + 降压 + 主会话接续

## 症状
kickoff 波 3 执行期网关进入不稳 episode：子智能体连续死亡呈三种形态——长会话中期死亡（13min，留半成品）、即刻早亡（76s/84s，worktree 已注册但零工作）、captcha 拦截（99s/47s）。工单 05 触 3 次重开上限、工单 06 重开配额耗尽，批次面临停摆。

## 根因
网关不稳具有**时间聚集性**（episode 持续数分钟到小时），对长会话调用尤其不友好；且机械重开上限（3 次）在故障期会被正常消耗触线——触线 ≠ 工单不可做，而是"换个执行形态"的信号。

## 代价
波 3 拖长（两轮重派 + 一次冻结窗口）；一次人工裁决点（用户无回答，按 persona best-judgment 继续）。半成品无损（现场保全到位）。

## 教训
网关故障期有一套可复用的韧性组合拳，按序使用：
1. **健康探针定窗口**：最小 agent 探针（"只回复 OK"）测试网关，通了再派关键路径——失败期烧重开配额等于白扔；
2. **降压串行**：并发从 2 降 1，关键路径单飞；
3. **触线后主会话接续半成品**：子代理长会话抗不住抖动，主会话短交互每步失败立即可见——接续而非重写（半成品往往已近完成，M1-05 的 546 行半成品经验证**直接全绿**）；
4. **审核错峰补位**：审核是后台 lane 不占关键路径，等实现槽位释放再补；
5. **增量提交防中断**：故障期工单要求"实现可用即先 commit"，别把全部工作押在最后一次 commit。

## 防复发
- [x] 已落实（M1 波 3-5 实证走通）：全程记录于 orchestration progress 区（重开计数/降压决策/探针时点），批后遥测入 DEV_LOG
- [ ] kickoff skill 层面：可考虑把"网关故障韧性组合拳"固化进 project-kickoff QUICKREF 失败路径表（当前仅有空返回/重开上限条目，无 episode 级处置编排）

## 关联
- [[编排型skill子代理职责重叠先划作用域]]（同族：编排故障的另一根因）
- [[Conver System 高频小坑汇总]]

## 补充实证（2026-09-08 M6 kickoff，全自动档）

**新形态：并行首派 TLS 断连三连**——W1 三 agent 并行（峰值 3）全部 `Cannot connect to API: Client network socket disconnected before secure TLS connection was established`，运行 ~36min 后断连、无 usage（网关/API 层异常非模型行为）→ 处置：**立即重开（不等同波）+ 现场核查**（worktree 已建但零提交；04/06 有未提交半成品——colors.dart 已改 + contrast 测试新建 / wire+service+测试六文件已改）→ 半成品接续 DONE。判层依据：无 usage = 网关层，改 prompt 无用；同批三连 100% 空返回 → 下波降并行观察网关健康。与 M1 故障期组合拳的衔接：探针/降压/主会话接续原则不变，本批增量 = 「并行峰值是 TLS 断连的诱因之一，首派三连后下波保持 2 并发即稳定」。