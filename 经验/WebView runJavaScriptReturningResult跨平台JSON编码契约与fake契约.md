---
type: lesson
tags: [Flutter, WebView, 平台通道, 合约, conver-system]
date: 2026-09-07
project: Conver System
source: 自身项目实践
summary: WebView runJavaScriptReturningResult 返回值跨平台编码不一致（Android evaluateJavascript 返回带外层引号的 JSON 编码串、iOS 裸值）；单测 fake 必须复刻生产契约，否则全绿真机必坏
provenance: mobile DEV_LOG〈M5 kickoff 批次〉冒烟产线缺陷#1 · commit de38f60 · 2026-09-07 M5 会话
status: verified
---

# WebView runJavaScriptReturningResult：跨平台 JSON 编码契约 + fake 必须编码生产真实契约

## 症状
移动端 M5 模拟器存档面板（SaveSheet）在 **Android 上恒显示 0 个存档**：游戏 localStorage 实际写入正常（leveldb 实证有键），但 `runJavaScriptReturningResult` 枚举返回空 → 面板 0 键、导出/删除按钮禁用。单测全绿（29 桥测 + 13 sheet 测）未捕获，AVD 冒烟才暴露。

## 根因
- Android `evaluateJavascript` 返回 **JSON 编码串**：字符串结果带**外层引号**（如 `"[[...]]"`），`webview_flutter_android` 的 `runJavaScriptReturningResult`（android_webview_controller.dart L522）对字符串**原样透传**（`num.tryParse(result) ?? result`）；
- Dart 侧 `parseLocalStorageEntries('"[[...]]"')` → `jsonDecode` 得到 `String`（非 `List`）→ `FormatException` → catch 降级空 Map；
- **单测 fake evaluator 返回的是「无外层引号的裸 JSON 数组串」——编码了错误契约**：测试模拟的是理想平台行为，不是生产 Android 行为，全绿却掩盖真机必坏路径。

## 代价
一轮 AVD 冒烟返工（面板轴 0 键排查 + 修复 + 真机复验 3 步）；若未冒烟会带着「存档管理不可用」交付。

## 教训
- **跨平台通道返回值契约必须按平台分别验证**：Android evaluateJavascript = JSON 编码串（外层引号），iOS = 裸值——桥解析层做**解包容错**（`jsonDecode` 得 String 再 decode 一次；直接得 List 原样处理），docstring 注明平台契约；
- **单测 fake 必须复刻生产真实契约，而非理想契约**——fake 的返回形态就是被测试代码的「世界假设」，fake 编码理想行为 = 测试与生产各说各话。

## 防复发
- [x] 已落实：`save_bridge.dart` `_decodeLocalStorageEntriesJson` 单层解包（编码串再 decode / 裸值兼容）+ docstring 注明 Android/iOS 契约差异
- [x] 已落实：fake evaluator 改回 `jsonEncode(jsonEncode([...]))`（外层引号 = Android 契约）+ 新增编码串回归用例 + 保留裸 JSON 兼容用例（iOS 锚）
- [ ] 通用化：任何 `runJavaScriptReturningResult` 消费点，第一件事确认目标平台返回形态并写进 fake

## 关联
- [[Flutter测试碰平台依赖必须超时兜底]]（同族：平台通道在测试环境的真实行为 vs 理想假设）
- [[跨语言镜像契约须对真实消费者验证]]（同族：双端契约各自单测通过 ≠ 互操作正确）