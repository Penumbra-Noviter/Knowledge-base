---
type: lesson
tags: [Android, Flutter, 冒烟, 像素采样, conver-system]
date: 2026-08-30
project: Conver System
source: 自身项目实践
summary: 模拟器截图像素采样须用 PIL 解码——screencap PNG 为 RGBA（colortype=6）且每行带 filter 字节，裸解析全零；Image.convert('RGB')+getpixel 才是确定性证据
provenance: mobile DEV_LOG〈技术债消费批次 F-10~F-17〉避坑 #3 · .scratch/techdebt-f10-f17/evidence/smoke-gate.md · commit b9dc9bc · 2026-08-30 会话
status: verified
---

# 模拟器截图像素采样须用 PIL 解码——screencap PNG 的 filter 字节与 RGBA 色型

## 症状
F-10~F-17 批次冒烟用 `adb exec-out screencap -p` 截图做主题状态像素采样（背景 == token 精确值），手写 PNG 解析（struct 读 IHDR + zlib 解 IDAT + 按 stride 偏移取像素）得到**全部 (0,0,0)**——三个采样点全黑，与 UI 树显示的主题明显不符。改用 PIL `Image.open(...).convert('RGB').getpixel(xy)` 后得到正确值（`(23,21,18)` = 深色 token `#171512`、`(240,236,229)` = 浅色 `#F0ECE5`）。

## 根因
`adb screencap` 输出的 PNG 是 **colortype=6（RGBA 8-bit）** 而非 truecolor RGB：裸解析按 3 通道偏移取字节，把 alpha 通道当 RGB 读；更关键的是 PNG 每行扫描线前有 1 字节 **filter 类型**（Row filter byte），未先解码 filter（None/Sub/Up/Average/Paeth）时行数据是「预测编码差」而非直接像素值——两者叠加导致全零/错乱。`struct` 直读 + `zlib.decompress` 只拿到原始 filter 编码流，必须按 PNG 规范逐行解 filter 才能还原像素。

## 代价
一轮像素采样误判（全黑）+ 一次调试（检查 IHDR colortype、重写解析）后改用 PIL 才拿到正确证据；若未发现会带着「全黑=主题异常」的错误结论归档（好在冒烟门在归档前兜住）。

## 教训
**Android 截图像素采样直接用 PIL 解码，不手写 PNG 解析**——`Image.open(path).convert('RGB').getpixel((x,y))` 一次性处理 RGBA 色型 + filter 字节 + alpha 归一；手写 zlib+struct 解析必须处理 colortype 分派与逐行 filter 解码（None/Sub/Up/Average/Paeth），极易漏。像素采样仍是主题/语言/登录态等状态的**确定性证据**（背景色 == 已知 token 精确值），但「如何解码出像素」要用成熟库，别在采样工具链上造轮子。

## 防复发
- [x] 已落实（本批）：采样统一 `PIL Image.convert('RGB').getpixel`；证据 smoke-gate.md 记正确值
- [ ] 通用化：设备 GUI 冒烟采样脚本模板统一用 PIL（或 ui-automator 的像素接口），禁止裸 zlib 解析 screencap PNG

## 关联
- [[运行态冒烟先采样确认基线状态]]（同族：像素采样是确定性证据——本条补充采样工具链怎么解码）
- [[模拟器GUI冒烟tap坐标须UI树实测]]（同族：设备侧验证纪律——坐标不能推算，状态不能假设，像素不能误解析）
