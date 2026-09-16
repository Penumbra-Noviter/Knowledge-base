---
type: lesson
tags: [Windows, COM, ctypes, GUID]
date: 2026-09-08
project: 通用
source: 自身项目实践
summary: ctypes 构造 COM GUID 时 Data1-3 须按 hex 串 big-endian 解析为整数；按 little 解析得到字节反转的错 GUID，CoCreateInstance 报类未注册
provenance: ZCode 会话 sess_6c098f55（2026-09-08 音量调 0 任务，排查 CoreAudio COM 时实证）
status: verified
---

# ctypes COM GUID 构造字节序坑

## 症状
- 用 ctypes 定义 `GUID(Structure)` 并按 hex 字符串构造后调 `CoCreateInstance`，返回 `REGDB_E_CLASSNOTREG (0x80040154)`（类未注册），即使注册表里 CLSID 明确存在、DLL 路径正确。
- 若用 `int.from_bytes(b[0:4], 'little')` 解析则 Windows GUID 的 Data1-3 字节序被反转，传出的 CLSID 无效。

## 根因
- Windows 注册表里的 GUID 字符串按 **big-endian** 语义书写（Data1 是 `x86` 风格十六进制），而 COM 接口的 GUID 结构体中 `Data1/Data2/Data3` 在内存里按 **little-endian** 存储。
- ctypes 必须用 `int.from_bytes(hex_bytes[0:4], 'big')`（Data2/Data3 同理）把字符串解析成整数，字段本身由 ctypes 按宿主字节序落下——misjudge 方向有一半概率出错，而 QI/创建失败往往表面上是「注册表没这个类」。

## 代价
- 在排查本机 CoreAudio COM 被劫持的会话里，先因 GUID 字节序错误把 `CoCreateInstance` 也误判为「类未注册」，额外增加一轮排查。

## 教训
- **手写 COM 接口/CLSID 的 ctypes GUID 时，Data1-3 一律 `int.from_bytes(..., 'big')`**；写完先用 `string_at(byref(guid), 16).hex()` 打印内存字节与期望 hex 串对比，确认再调 CoCreateInstance。

## 防复发
- 写一个 `GUID.from_str()` 帮助函数（big-endian 解析）并单元断言内存字节 == 期望 hex，任何 COM 开发都复用。
- 排查 `REGDB_E_CLASSNOTREG` 时，先打印 GUID 内存字节确认构造正确，再怀疑注册表/权限。
- [x] 已落实（帮助函数思路已记录在 memory audio-mute-windows「通用坑」节）

## 关联
- [[本机音量控制 CoreAudio COM 被劫持改用 keybd_event]]