---
type: lesson
tags: [Windows, COM, 音量, 系统自动化, zcode环境]
date: 2026-09-08
project: 通用
source: 自身项目实践
summary: 本机 CoreAudio COM 被劫持（QI 全失败），音量设 0 改用 keybd_event 音量键 + 任务栏图标 UIA Name 验证
provenance: ZCode 会话 sess_6c098f55（2026-09-08 音量调 0 任务）· 对应 memory 条目 audio-mute-windows
status: verified
agents_md_feedback: 印证 AGENTS.md「View 报告是唯一视觉依据、不脑补」——但对于数值型读数（音量百分比/滑块位置），View 两次读同一张图给出矛盾数值，须找权威数值源（UIA Name），纯视觉估算不能作为判定依据
---

# 本机音量控制：CoreAudio COM 被劫持，改用 keybd_event + 任务栏图标验证

## 症状
- `CoCreateInstance(MMDeviceEnumerator)` 经四个独立路径（PowerShell Add-Type / csc 编译 exe / comtypes / pywin32）全部能创建对象（IUnknown QI 成功），但 QI → `IMMDeviceEnumerator` 一律 `E_NOINTERFACE (0x80004002)`；QI `IMMDevice` 成功但 vtable 调用崩溃（读 0x20 越界）。
- `waveOutSetVolume(0)`（winmm）只能把 WAVE/旧式混音器子系统音量设 0，**不影响走 WASAPI 的现代应用**——用户仍能听到声音。
- winmm mixer API 在现代 Realtek 栈读不到 speakers 主音量线（mixerGetLineInfo 失败 hr=11）。
- CoreAudio 读不到音量，任务栏音量浮层 View 读不到，sndvol 滑块是自绘 Pane（UIA 读不到 RangeValue）。

## 根因
- 本机 CoreAudio COM 层疑似被 hook/劫持：注册表侧完全正常（InprocServer32 → `%SystemRoot%\System32\MMDevApi.dll`、ThreadingModel=both、无 AppInit_DLLs），但 COM 对象行为异常（QI IMMDeviceEnumerator 失败、IMMDevice vtable 崩溃）。根因未定位，结论明确：**IAudioEndpointVolume API 设/读音量在本机走不通**。
- waveOutSetVolume 是旧 API，只作用于 WAVE 子系统，与 WASAPI 现代音频栈脱节。

## 代价
- 排查 COM 劫持 + 试不通 API：约 2 小时；期间 View 子智能体两次读同一张 sndvol 截图给出矛盾滑块读数（13% vs 24%），误导方向。
- 如无替代路径会卡死（音量无法设、无法验证）。

## 教训
- **系统级音量/静音控制在本机只信 keybd_event 音量键 + 任务栏图标 UIA Name 作为权威验证**，不试 CoreAudio / waveOut / mixer。
- **精确数值读取（百分比/滑块位置）以结构化权威源为准（这里是任务栏图标的 UIA Name），不靠视觉估算**——视觉模型读像素位置只能给量级，不能给精确值。

## 防复发
- 音量控制标准流程（2026-09-08 实测）：
  1. 用 keybd_event 模拟 VK_VOLUME_MUTE(0xAD)/DOWN(0xAE)/UP(0xAF)，keydown+keyup + 短 sleep 防系统限流；
  2. 验证用任务栏音量图标的 UIA Name（形如「音量 Speaker (Realtek(R) Audio): 已静音」或「…: 0%」）——直接含静音状态与百分比读数；
  3. **先取消静音再调音量**：静音态下按 DOWN 按键视觉无反馈 ≠ 已在 0，正确顺序 = 先 MUTE 一次确认非静音 → 按 DOWN 到底 → 读 Name 确认「0%」；
  4. sndvol 窗口截图可用 `capture_window.ps1`（PrintWindow，后台窗口也有效）。
- [x] 已落实（条目已写入 memory audio-mute-windows，含完整操作路径）

## 关联
- [[ctypes COM GUID 构造字节序坑]]
- [[GUI 读数验证用权威数值源而非视觉估算]]