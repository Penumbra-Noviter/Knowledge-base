---
type: lesson
tags: [LLM, playwright, 浏览器, 缓存, 验证, conver-system]
date: 2026-08-19
project: Conver System
source: 自身项目实践
summary: 合并代码后浏览器验证必须清 HTTP 缓存再导航——无 Cache-Control 的静态服务下 ESM 模块被启发式缓存，验证会拿到合并前的旧代码（fetch 探测模块内容可确诊）
provenance: Conver System 2026-08-19 模拟器 PC 阅读覆盖层批次（merge 42e4af9 后验证）
status: verified
---

# 浏览器启发式缓存 ESM 模块导致合并后验证拿旧代码

## 坑

模拟器覆盖层批次：merge 后 Playwright 打开游戏，断言「注入 link 不存在」——百思不得其解，因为服务器上的文件（curl 验证）明明是新版。

## 根因

- `python -m http.server` 不发送 `Cache-Control` 头，浏览器按启发式缓存策略缓存静态资源（含 ESM 模块）
- 页面在**合并前**已加载（模块被缓存），merge 后直接刷新页面 → **ESM 模块 URL 不变**（相对路径无 query），浏览器仍用缓存副本
- 页面级 `?t=xxx` query 只绕过 HTML 自身缓存，不作用于模块请求

## 确诊方法

在页面内 `fetch('/js/xxx.js')` 拿文本，与服务器文件内容比对（`curl` 或直接读文件）——浏览器 fetch 到的内容缺新符号即缓存命中旧版。本批实测：fetch 长度 17190 vs 文件 18473。

## 修复

Playwright 中先清缓存再导航：

```js
const cdp = await page.context().newCDPSession(page);
await cdp.send('Network.clearBrowserCache');
await page.goto(url, { waitUntil: 'networkidle' });
```

## 通用教训

- 合并代码后做浏览器验证，**第一步永远是清 HTTP 缓存**，再谈其他
- 验证用静态服务尽量带 `Cache-Control: no-cache` 响应头（或验证脚本里先清缓存）
- 浏览器「刷新」不等于「拿新代码」——ESM 模块缓存独立于页面缓存
