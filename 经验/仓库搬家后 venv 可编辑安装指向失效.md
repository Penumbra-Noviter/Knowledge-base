---
type: lesson
tags: [Python, venv, 可编辑安装, 仓库迁移]
date: 2026-09-07
project: DeepTutor
source: 自身项目实践
summary: 仓库搬家后 venv 的可编辑安装仍指向旧路径——只有从仓库根目录运行时靠 cwd 遮蔽能 import，换 cwd 即 ModuleNotFoundError；pip install -e . --no-deps 重指即可
provenance: commit f6523fea · DEV_LOG 2026-09-07 TD-28 节（打包取证段）· 2026-09-07 DeepTutor 会话
status: verified
---

# 仓库搬家后 venv 可编辑安装指向失效

## 症状

PyInstaller 打包第一步就失败：`packaging/deeptutor-server.spec` 第 3 行 `from deeptutor import __file__` 抛 `ModuleNotFoundError: No module named 'deeptutor'`。但同一个 venv 在**仓库根目录**下 `python -c "import deeptutor"` 完全正常。

## 根因

仓库曾从 `D:\Desktop\downloads\DeepTutor-main\DeepTutor-main` 搬家到 `D:\Desktop\downloads\DeepTutor\DeepTutor-main`，而 `.venv` 的可编辑安装（`__editable__.deeptutor-1.5.11.pth` + finder）仍指向旧路径（已不存在）。在仓库根目录能 import 是 **cwd 遮蔽**：`python -c` 把当前目录放 sys.path 首位，恰好命中同名包；一旦 cwd 换成 `packaging/`（PyInstaller 按约定在 packaging 目录下跑），sys.path 首位是 packaging，找不到 deeptutor → 失败。`pip list` 显示的可编辑路径也是旧值，是同样的线索。

## 代价

- 打包首次运行即失败，多花一轮定位
- 症状（"换个 cwd 就 import 失败"）很有迷惑性，容易误判成依赖缺失

## 教训

**仓库目录被移动/重命名后，必须重跑 `pip install -e . --no-deps` 重写可编辑安装指向**。验证导入不能用「站在仓库根目录」的方式——那是 cwd 遮蔽假象；要从**非仓库目录**（或打包实际使用的 cwd）验证。

## 防复发

- 任何「从根目录 import 正常、换目录就失败」的 Python 环境问题，先查 `site-packages/__editable__*.pth` 与 finder 里的绝对路径是否仍存在
- 仓库迁移后把 `pip install -e . --no-deps` 列入环境修复清单；打包类脚本的失败日志里包含 spec 行号时先看导入链
- [x] 已落实？（`.venv` 已重指；后续打包正常）

## 关联

- [[GUI headless测试共享替身后必须弹出sys.modules缓存]]（同为 Python 环境路径类坑）
