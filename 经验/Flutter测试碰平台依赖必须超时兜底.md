---
type: lesson
tags: [Flutter, 测试, 平台通道, conver-system]
date: 2026-08-29
project: Conver System
source: 自身项目实践
summary: Flutter widget 测试碰平台依赖必须超时兜底——通道在测试环境挂起而非抛错，try/catch 管不了
provenance: mobile DEV_LOG〈M1 kickoff 批次〉避坑 #2 · commit de9e5de（3s 超时兜底）· 2026-08-29 M1 会话
status: verified
---

# Flutter widget 测试碰平台依赖必须超时兜底——通道挂起而非抛错

## 症状
设置页引入真实依赖（flutter_secure_storage + drift/path_provider）后，既有 widget 冒烟测试（tap 遍历 tab）在设置 tab 上 `pumpAndSettle timed out`；且该失败发生在假时钟 ~10 分钟预算内（真实仅 3 秒）—— CircularProgressIndicator 持续调度帧推进假时钟。

## 根因
测试环境无平台通道实现时，`flutter_secure_storage.read` / `path_provider.getApplicationDocumentsDirectory` 的 MethodChannel 调用**永久挂起**（Future 永不完成），**不是抛 MissingPluginException**——try/catch 与 catchError 对"挂起"完全无效；await 链上第一环挂起 → 整个加载 Future 永不完成 → FutureBuilder 永远 loading。

## 代价
一轮误诊（先按"抛错→catch"修防御，无效）+ 临时探针测试定位；若不知情可能反复改防御代码。

## 教训
**Flutter widget 测试碰任何平台依赖（插件通道/path_provider/设备 API），加载路径必须加 `.timeout(d, onTimeout: () => 兜底值)`**——异常防御（try/catch）只覆盖"抛错型"失败，覆盖不了"挂起型"失败。双层防御（内层 try/catch 抛错型 + 外层 timeout 挂起型）才完整。

## 防复发
- [x] 已落实：`settings_view.dart` 回显加载双层防御（内层 try/catch + 外层 3s timeout），注释声明原因；生产端安全存储读取远快于 3s，兜底不影响正常体验
- [ ] 新页面引入平台依赖时，widget 冒烟前先过一遍"挂起面"清单

## 关联
- [[Conver System 高频小坑汇总]]
- [[模拟器GUI冒烟tap坐标须UI树实测]]（同族：设备侧验证的坑）