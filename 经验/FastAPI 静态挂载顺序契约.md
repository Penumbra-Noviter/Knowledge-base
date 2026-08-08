---
type: lesson
tags: [FastAPI, 静态文件, 路由顺序, 契约, conver-system]
date: 2026-08-06
project: Conver System
source: 自身项目实践
status: verified
---

# FastAPI 静态挂载顺序契约——`mount("/")` 会遮蔽后注册路由

## 症状
静态文件挂载点与 API 路由的注册顺序隐式耦合：顺序错误时 API 请求被 `index.html` 兜底吞掉（返回 HTML 而非 JSON）。

## 根因
`StaticFiles(directory=…, html=True)` 挂到 `/` 会匹配所有路径；FastAPI 按**注册顺序**匹配，之后注册的 `/api` 路由会被遮蔽。这不是报错，是静默的「请求进了错误的分支」。

## 教训
**挂载 StaticFiles 到根路径时，所有 API 路由必须在此之前注册且用 `/api` 前缀**；把注册顺序契约写成代码注释防回归。顺序耦合类问题的最佳防复发是「在出错的位置就地写下顺序契约」。

## 代价
本次仅补契约注释（`main.py:29-43`）——坑已踩过，代价已付。

## 防复发
- [x] `main.py` 注册顺序注释
- [ ] 新 FastAPI 项目：路由注册（含前缀）先于静态挂载，顺序写注释
- [ ] 同型契约检查：中间件顺序、异常 handler 顺序同理

## 关联
- [[测试与架构高频小坑汇总]]（同类「顺序/时序敏感」族）
- [[架构摩擦渐进发现与分批落地]]
