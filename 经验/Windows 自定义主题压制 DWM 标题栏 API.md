---
type: lesson
tags: [win32, dwm, qt, 桌面应用]
date: 2026-11-05
project: Delta Force Dashboard（收益计算器）
source: 自身项目实践
summary: DWM 标题栏 API（IMMERSIVE_DARK_MODE/CAPTION_COLOR）hr=0 却可被系统自定义主题静默压制——定性窗口 chrome 异常先查主题状态再做涂色实验
provenance: DEV_LOG 2026-11-05「钛空玻璃主题」节 · CODE_WIKI §4.28 · 2026-11-05 Delta Force Dashboard 会话
status: verified
agents_md_feedback: 印证「对抗性自检」——用高对比红色涂色 + 像素采样做证伪实验，而非信 HRESULT
---

# Windows 自定义主题压制 DWM 标题栏 API

## 症状
Win11 24H2（build 26100，console 物理会话、非 RDP）上，为深色主题窗口设置 `DWMWA_USE_IMMERSIVE_DARK_MODE(20)`、`DWMWA_CAPTION_COLOR(35)`、撤销 `DWMWA_SYSTEMBACKDROP_TYPE(38)` 后再直涂，**三路调用全部返回 HRESULT=0（成功）**，但标题栏实测仍为浅蓝灰垂直渐变（Aero 风格）+ 黑字——`hr=0` 与渲染效果完全脱钩。

## 根因
注册表 `HKCU\...\Themes\Personalize` 的 `AppsUseLightTheme=0`（深色标志）具有误导性；真正的决定者是 `HKCU\...\Themes\CurrentTheme`——该机指向 `Custom.theme`（亮色 Aero 派生的「未保存」自定义主题）。**主题文件层的标题栏样式优先于运行时 DWM 属性**：属性被接受但不渲染。注册表深色标志与实际主题文件不一致时，任何按注册表判断系统主题的逻辑都会走错分支。

## 代价
约 3 轮截图验证 + 2 组对照实验（红色 CAPTION 涂色、撤背板后涂色）+ 注册表/主题服务排查，约 1 小时才定位到「环境级压制」而非代码缺陷。

## 教训
DWM API 的 HRESULT=0 只代表「属性被接受」，不代表「生效」；定性窗口 chrome 异常时，**先查 `CurrentTheme` 指向的主题文件与注册表主题标志是否一致，再用高对比涂色（如纯红 CAPTION）+ 像素采样做证伪实验**——若涂色无渲染即可断定环境级压制，停止代码侧排查。

## 防复发
- [x] 结论已固化：CODE_WIKI §4.28 记录压制结论；项目 memory 只留指针指向知识库本条
- [ ] 彻底解（frameless 自绘标题栏）未立项——涉及拖拽/最大化/按钮全套自绘，成本独立评估

## 关联
- [[Qt 半透明属性 show 期重建窗口丢 DWM 属性]]
- [[非前台窗口截图按捕获手段分层]]（同涉 DWM 合成，根因无关）
