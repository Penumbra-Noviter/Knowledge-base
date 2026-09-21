---
type: lesson
tags: [React, 自动化, 测试基建, 合成事件]
date: 2026-09-21
project: AI自动获客
source: 自身项目实践
summary: React 受控输入直写不触发 onChange 须补 input 事件；vm 测试沙箱无 DOM 事件构造器，脚本事件分支必须注入才被测到
provenance: DEV_LOG 2026-09-20 P3轮3 · commit 825dcea（helpers/dom-run.js 注入） · AI自动获客会话
status: verified
agents_md_feedback: 印证 AGENTS.md「测试诚实协议」——事件分支假绿（从未实际执行）与真绿的区别是本条的核心
---

# React 受控输入写入必须补 input 事件——事件在页面与测试沙箱的双重不可见

## 症状
对象（Electron + 页面 JS 自动化，写入抖音 React 评论区）：
1. 页面端：往 contenteditable `textContent = text` 直写后，框架不感知——发送（Enter）时命中空内容或 React 状态未更新；
2. 测试端：注入脚本 `new KeyboardEvent('keydown', …)` 走 Enter 兜底分支——测试**从未真正执行过该分支**（vm 沙箱里 `KeyboardEvent is not defined`，被 try/catch 静默吞掉，`sent=true` 由 sendButton.click() 其他分支造成，测试仍绿）。

## 根因
1. **React 受控组件**（contenteditable / textarea / input）的值由 React state 驱动：直写 DOM 的 `textContent` / `.value` **不触发 onChange**——React 监听的是 `input` 事件（contenteditable 经 input 事件同步 innerHTML/textContent）。写入后必须补发 `new InputEvent('input', {bubbles:true, inputType:'insertText', data})`（低版本兜底 `new Event('input')`）。
2. **Node vm 沙箱无 DOM 构造器**：`vm.runInNewContext(script, {document})` 的 context 不含 `MouseEvent / KeyboardEvent / InputEvent`——页面脚本的事件构造全 ReferenceError，被各分支 try/catch 吞掉 → 分支「假执行」→ 测试绿 ≠ 分支被测到。这是「改代码不许改测试断言骗过门禁」的镜像陷阱：**不是断言洗白，是构造器缺位导致分支静默跳过**。

## 代价
- Enter 兜底分支（真实页面回复框无独立发送按钮的核心路径）在测试里从未被执行，直到 P3 轮3 注入构造器才发现——若该分支有回归，测试不会拦住；
- 调试时误以为「合成事件在页面有效」——先测了页面侧才意识到测试侧缺构造器，多一轮往返。

## 教训
1. **写入 React 受控输入框：直写内容 + 补发 `input` 事件**（带 `inputType`/`data` 更贴近真实）——这是 React 与非 React DOM 自动化的分界线；
2. **vm/沙箱测试环境必须注入页面脚本依赖的 DOM 构造器**（Event/CustomEvent/UIEvent/FocusEvent/MouseEvent/KeyboardEvent/InputEvent），否则 try/catch 包裹的事件分支全部假绿；
3. **自查「分支是否真被执行」**：对含 try/catch 吞错的分支，用「少一个分支就失败」的方式验证（如把注入的构造器注释掉 → 对应测试应 `fail` 而不是仍绿）。

## 防复发
- `helpers/dom-run.js` 的 `runInDoc` 统一注入 happy-dom 构造器（已落实）——后续所有走 vm 的页面脚本测试自动获得真实事件环境；
- 新增页面脚本事件类分支时，测试必须真实触发该分支并断言副作用（写入文本 / sent 事实），不允许仅「不抛错」即通过；
- 写 React 受控页面脚本时，写入后补发 input 事件作为**标准动作**（同 writeText 模板）。
- [x] 已落实（commit 825dcea：dom-run 注入 + writeInputTemplate 补 input 事件 + 3 个激活链路测试真实走事件分支）

## 关联
- [[Falsify测试要钉住缺陷所在层]]
- [[DOM 自动化定位失败先枚举真实形态再查层级]]