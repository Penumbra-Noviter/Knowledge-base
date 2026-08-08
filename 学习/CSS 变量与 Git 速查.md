---
type: study
tags: [css, git, unverified]
date: 2026-08-08
source: 外部整理（AI 生成，未验证；原「CSS变量.md」「Git 使用.md」压缩合并）
status: unverified
verify: 在真实项目（Conver System 前端 / 任意 Git 仓库）实际用上某项后改写成自己的经验并升格 verified；长期用不上按 YAGNI 删除
---

> ⚠️ **未验证**：本笔记由外部教程/AI 摘要压缩合并而来，无本项目代码实例。
>
> **验证条件**：实际用上其中某项后再改写成自己的经验并升格 `status: verified`。
> **若长期用不上**：按 YAGNI 删除，别让「学而未用」堆积。

## CSS 要点（速查）

- **CSS 变量（自定义属性）**：在 `:root` 定义设计令牌，JS 改一处全局生效——动态主题（暗黑模式）首选。
- **`:is()` vs `:where()`**：前者优先级 = 内部最具体选择器；后者永远为 0。基础样式用 `:where()` 防覆盖，需保证优先级时用 `:is()`。
- **Flexbox 避坑**：图片压扁 → `flex-shrink: 0`；子项撑破容器 → `min-width: 0`；垂直居中需父元素有高度。
- **Grid vs Flex**：一维用 Flex、二维用 Grid；Grid 搭骨架、Flex 填细节；`grid-template-areas` 直观画线。
- **`:has()` 与容器查询**：`:has()` 让 CSS 具备逻辑判断（如输入错误变红无需 JS）；容器查询让组件按所在容器自适应，突破媒体查询一刀切。
- **`@layer` 与原生嵌套**：`@layer` 自定义层叠优先级顺序，解决 `!important` 滥用；原生嵌套兼容 Sass 语法、浏览器直接解析。
- **`clamp()` 流体排版**：字号平滑缩放，消除断点跳跃；移动端全屏用 `svh` 视口单位，避免地址栏收起导致的跳动。
- **滚动驱动动画**：`scroll-timeline` 绑定动画进度，视差/序列动画可替代复杂 JS。

## Git 入门步骤（速查）

首次推送到 GitHub：

```bash
cd "项目目录"
git config --global user.name "你的用户名"
git config --global user.email "你的GitHub邮箱"
git init
git add .
git commit -m "初始提交"
# GitHub 网页建空仓库后：
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin main
```

> 图片来源：GitHub 建仓界面截图（原 [[Git 使用]] 附）
