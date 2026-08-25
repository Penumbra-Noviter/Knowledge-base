---
type: lesson
tags: [nextjs, static-export, image]
date: 2026-08-24
project: DeepTutor
source: 自身项目实践
summary: Next.js output export 模式下 next/image 必须配 images.unoptimized，否则生成 /_next/image 请求 404
provenance: 2026-08-24 DeepTutor 桌面修复会话 sess_4fafa097 · next.config.js 修复 · web/out 构建产物 grep 验证
status: verified
---

# Next.js 静态导出必须关 next/image 优化

## 症状
应用侧边栏左上角 logo/banner 图标不显示（favicon 正常）。构建产物 HTML 中图片 src 为 `/_next/image?url=%2Flogo.png&w=48&q=75`，该请求返回 SPA fallback 的 HTML 而非图片。

## 根因
`next.config.js` 用 `output: "export"`（纯静态导出，FastAPI 直接服 `out/` 目录，无 Node 进程），但未设置 `images: { unoptimized: true }`。`next/image` 组件默认生成指向 `/_next/image` 优化端点的 URL——该端点是 Next.js 运行时 API，静态导出模式下不存在。后端 `SpaStaticFiles` 把未知路径 fallback 到 index.html，浏览器把 HTML 当图片渲染 → 破图。

## 代价
图标缺失问题潜伏多个版本未被发现（用户日常不在意侧边栏小图），本轮作为 5 个 bug 之一才定位。排查约 20 分钟。

## 教训
**静态导出（output: "export"）等于放弃所有 Next.js 运行时 API**——image 优化、middleware、ISR 全部不可用；任何依赖运行时的组件特性都要显式降级配置，而不是等 404 暴露。

## 防复发
- [ ] 已落实：`next.config.js` 加 `images: { unoptimized: true }`，重建 web/out 后 grep 确认产物无 `/_next/image` 引用
- [ ] 同类检查：静态导出项目引入新 Next 特性时先查该特性是否依赖运行时

## 关联
- [[Vite root模式静态资源路径]]
- [[新增前端运行目录必须同步打包面]]
