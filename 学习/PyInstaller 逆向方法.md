---
type: reference
tags: [逆向, PyInstaller, 安全, 方法论]
date: 2026-08-20
project: 通用
source: 吾爱破解论坛 · thread-2123942
summary: PyInstaller 打包产物逆向全流程——CArchive 手动解包、PYZ 解密、跨版本 .pyd 加载、动态 Hook 提取密钥、本地验证绕过。帖主完整还原了 A9-A11 Ramdisk 工具的本地验证流程。
---

# PyInstaller 逆向分析方法

## 来源

- 原文：[吾爱破解论坛 thread-2123942](https://www.52pojie.cn/thread-2123942-1-1.html)
- 目标：A9-A11 Ramdisk 工具（PyInstaller onefile + Cython .pyd + AES 加密 PYZ + Fernet 许可证验证）
- 帖主核心立场：**"只要找对方法，任何本地验证都藏不住"**

## 逆向路线图（可复用）

```
解包 CArchive → 解密 PYZ 存档 → 重构 Python 加载环境 → 动态 Hook 提密钥 → 反推验证逻辑 → 本地绕过
```

## 各阶段关键技术

### 1. 手动解包 CArchive（字节序陷阱）

**问题**：pyinstxtractor 工具直接失败，TOC 长度解析出 18 亿。

**帖主发现**：Cookie 中 `len`、`TOC` 等字段是**大端序（big-endian）**，pyinstxtractor 默认小端导致数值爆炸。手动按 `be32` 解析后得到正确值。

**关键教训**：
- 不要迷信解包工具。PyInstaller 不同版本可能切换字节序
- Cookie 魔数 `MEI` 可手工定位
- 十六进制校验比工具输出更可靠
- 正确解析后得到 1137 个 TOC 条目，提取出 main.pyd、fzip.pyd 等文件

### 2. 解密 PYZ 存档（AES-CTR）

**问题**：PYZ-00.pyz 有加密标志（1），marshal.loads 失败。

**做法**：
- 密钥在 `pyimod00_crypto_key` 模块硬编码字符串 `'AC10@3774NL!O91@'`
- 算法：**AES-CTR 模式**（密钥前 16 字节，IV 取自每段数据前 16 字节）
- 解密后再 zlib 解压
- 参考 PyInstaller 官方源码即可实现，不需要逆向破解加密算法

**关键教训**：PyInstaller 的加密是"防君子不防小人"——密钥在模块硬编码，算法在官方源码公开。

### 3. 重构 Python 加载环境（最大坑点）

**问题**：系统 Python 3.13 无法加载 cp38 编译的 .pyd 文件。

**方案**：
1. 下载 **embeddable Python**（匹配编译版本，此处为 3.8.10）
2. 自定义 `PyzLoader` 和 `PyzFinder`（继承 `importlib.abc`），实现双轨加载：
   - Python 模块从解密后的 PYZ 加载
   - `.pyd` 模块由系统正常加载

**常见坑**：
| 问题 | 根因 | 解决 |
|------|------|------|
| tkinter 缺失 | embeddable 不含 tcl/tk | 从 EXE 提取 tcl/tk，设环境变量 |
| pythoncom.pyd 加载失败 | 依赖 pywintypes38.dll | 复制并改名 |
| cryptography 属性缺失 | namespace 冲突 | 调整加载顺序 |
| 模块不是包 (ispkg) | 判断逻辑不完整 | 加 binary_mods 前缀匹配 |

### 4. 动态 Hook 提取密钥（核心突破）

**原理**：Cython 编译的 .pyd 函数难以静态分析，但 **动态 Hook 已有库** 可以绕过。

**帖主做法**：
- Hook `cryptography.fernet.Fernet.__init__`
- 在 `make_key`/`make_key2` 时捕获运行时生成的 Fernet 密钥
- 捕获到密钥：`b'b5vM0i-_XUVKFBchVuhACbph_xhx0KdDmViY0bCRB84='`
- 发现 `decrypt_file` 操作路径为 `%LOCALAPPDATA%\TeamViewer\Viewer\`——注册列表伪装成 TeamViewer 文件

**关键教训**：Hook 已知库的初始化函数，是提取密钥最优雅的方式，不需要反汇编 .pyd。

### 5. 本地验证绕过

**发现**：所有验证逻辑（包括"未注册"弹窗）都是**本地代码**，不是服务器阻挡。

**绕过方式**：
- `B()` 函数展示未注册弹窗并打开腾讯文档
- 一句绕过：`main.main.B = lambda self: None`
- 即使 Cython 编译的函数，Python 层面 lambda 赋值即可替换

**关键教训**：任何本地验证，只要程序能在本地跑，密钥和逻辑最终都会暴露。

## 防御维度（对抗帖主方法）

针对帖主的 5 阶段方法，各阶段的防御有效性：

| 阶段 | 攻击方法 | 简单防御 | 有效防御 | 根本限制 |
|------|---------|---------|---------|---------|
| 1 | 解包 CArchive | 无（格式公开） | 基本无 | 格式必须公开 |
| 2 | 解密 PYZ | 设置 `--key` 加密 | 密钥仍可提取 | 密钥硬编码在 bootloader |
| 3 | 重构环境 | 无 | 代码混淆（PyArmor） | 可动态加载读取 |
| 4 | 动态 Hook | 反调试/反Hook | 运行时保护 | 程序自解密后即暴露 |
| 5 | 逻辑绕过 | 运行时完整性检查 | 服务端验证 | 纯本地验证不可能完全防御 |

## 项目启示

- 对于 model-fingerprint 项目，帖主的方法可在小时内完成解包、提取所有常量
- 当前 PyArmor + base64 防护抵抗**静态分析**（casual decompilation），但不抵抗**动态分析**
- 核心防御不在代码混淆，而在于**检测方法论本身的对抗性设计**（decoy、adversarial 场景、多维融合）
- 若需加强，可参考下方关联的加固建议

## 关联

- [[项目驱动知识库模式]]