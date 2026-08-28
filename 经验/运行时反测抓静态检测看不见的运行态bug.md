---
type: lesson
tags: [验证, 测试, 回归, UI]
date: 2026-08-29
project: 通用
source: 自身项目实践
summary: 静态门禁只证明"无表面 slop"，证明不了运行时正确；回归基线必须含真实交互与 reduced-motion 态反测。
provenance: finesse-ui 优化批次 ①②③ · commits 4aa0376/858105a/4080ad4/e755945 · 2026-08-28 kickoff 全自动
status: verified
---

# 运行时反测抓静态检测看不见的运行态bug

## 症状
为 finesse-ui 建立回归基线时，detect.mjs（正则静态扫描）对 4 个新构建页面全部 p0=0、findings=0，全绿；但 Playwright 交互 + reduced-motion 态实跑暴露 4 个真实运行 bug：
- B1 品牌落地页任务时钟输出**负数**（rAF 回调的 `DOMHighResTimeStamp` 与 epoch 时间戳混用）
- B3 工作流页预检查卡不随标题输入刷新（live 校验行残留「未填写」）
- B4 助手页 reduced-motion 下流式回答首帧**空**（`slice(0, i)` 在 `i+=` 之前取值，首帧 `slice(0,0)`）
- B4 审批卡 RM 静态度缺默认选中（默认态只在动效模式铺设）

## 根因
静态扫描只读"代码长什么样"（规则、黑名单、正则命中），看不见"运行时状态与交互是否正确"。这 4 个 bug 全在事件时序、动画状态机、reduced-motion 分支里——恰好是静态层不可见的那一层。全绿 ≠ 正确，是"检查过的错误"。

## 代价
4 个 bug 全部要等实跑才暴露；若没有回归基线，方法链后续改动会让它们反复出现且无门拦截。单个 bug 的发现成本 = 一次完整构建 + 一次人工/运行时巡检。

## 教训
静态门禁（lint / 正则 / 黑名单 / 覆盖率）证明"没有明显 slop"，不证明"运行时正确"。带状态机、动画、reduced-motion、实时数据源的界面，回归基线必须包含**真实运行**（浏览器交互 + 静态度），只跑静态门是把错误当成通过。

## 防复发
- 已建 `finesse-ui/examples/REGRESSION.md`：4 个代表性 brief + 通过标准 + 执行矩阵，实跑含 Playwright 点击走通与 reduced-motion 冻结态验证
- 方法链改动后按 runbook 重跑（单一 cwd 前提 + 完整路径，见 [[文档命令必须按文档实跑一遍再发布]]）
- 运行时反测的通过标准 = "拿到真实像素 / 真实状态"，不是"命令退出 0"

## 关联
- [[Falsify测试要钉住缺陷所在层]]
- [[对抗探针先沙箱重定向再探测]]
