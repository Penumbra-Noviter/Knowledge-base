---
type: lesson
tags: [windows, 进程句柄, junction, symlink, 目录锁]
date: 2026-09-20
project: 通用
source: 自身项目实践
summary: Windows 宿主进程（Electron 壳）会话存活期间，经 junction/symlink 链解析过的物理目录项不可改名/删除，但子项可动；Restart Manager 不报告，须用 handle.exe 定位，交换等宿主退出
provenance: 2026-09-20 ZCode 会话（project-kickoff / universal-exam-cram-coach 链接关系交换 P0-4；handle.exe 实证句柄持有者清单）
status: verified
agents_md_feedback: 印证 §三.2「确认即保护」——处置中旧真身改名保留为回滚点而非删除；印证「Key 不外泄/无云依赖」无关；无冲突
---

# 宿主进程句柄锁住经 symlink 链解析的物理目录

## 症状

- 会话运行期间，对 `.cc-switch\skills\project-kickoff`（一个旧实体目录）执行改名/删除：
  `rm`、`mv`、`cmd ren`、PowerShell `Move-Item` 全部失败，报
  `Device or resource busy` / `The process cannot access the file because it is being used by another process`。
- 但**目录内的子项**（`scripts/`、`__pycache__`、`.serena/`）可以自由改名——只有目录项（名字本身）被锁。
- Windows Restart Manager 查询该路径返回 **NO-PROCESS-REPORTED**（不报告占用）。
- 对照组：同一次操作中，未被本会话使用过的 `universal-exam-cram-coach` 目录改名成功。

## 根因

- ZCode（Electron/node 壳）在会话启动时通过
  `~/.zcode/skills`（junction → `D:\Desktop\cc\.claude\skills`）解析
  `project-kickoff` 的 **SYMLINKD**（→ `.cc-switch\skills\project-kickoff` 物理目录），
  对**物理目标目录**建立了目录句柄（watch/枚举用，handle 类型 File 54/58），会话存活期间不释放。
- handle.exe 实证持有者：`ZCode.exe`（3 实例：20012/32228/31036）、`node.exe`、`cmd.exe`、多个 `python.exe`（MCP 子进程）。
- 关键判别特征：**锁在「目录项」而非「文件/子项」**——目录被某进程作为 watch/枚举目标持有时的典型形态；
  Restart Manager 对目录 watch 句柄不报告（它只报可重启进程的文件锁）。

## 代价

- 链接关系交换被迫分两阶段：真身先行（复制 + diff 校验 + 转正）+ 收尾脚本等宿主退出后执行；
- 需要下载 sysinternals handle.exe 才能精确定位（Restart Manager 误导为"无占用"）；
- 一度误判为 MSYS2/杀软问题，多轮排查（rm → cmd ren → PowerShell → RM → handle）。

## 教训

- Windows 上「目录项被锁、子项可动、Restart Manager 无报告」= **宿主进程目录句柄持有**的指纹，
  与「shell CWD 持有」（见关联）表现相似但根因不同；先 `handle.exe <路径>` 定位再决定处置时机。
- 宿主（Electron 壳）会话存活期间，不要尝试改名/删除「经 junction/symlink 链被宿主解析过」的物理目录；
  链接关系交换属于**宿主退出后**操作。

## 防复发

- [x] 迁移/改名「宿主加载路径下的目录」前，先 `handle64.exe -accepteula -nobanner <路径>` 确认无宿主句柄；
      宿主存活 → 改为「真身先行（复制+diff 校验）+ 幂等收尾脚本，宿主退出后执行」两阶段。
- [x] 链接方向设计：**单向链接**（受管侧 junction → 宿主侧真身目录）比交叉链接
      （宿主 junction 目录内再放 SYMLINKD 指向另一处物理目录）可维护——后者使宿主 watch 落点不可见。
- [ ] 目录锁排查顺序固化：`cmd ren` 快速确认（原生内核路径）→ `handle.exe` 定位 →
      判断释放时机（进程退出 vs 会话结束），不用 Restart Manager 作为排除依据。

## 关联

- [[无控制台进程的输出句柄继承与交互挂起]]（同为 Windows 句柄族，但为输出句柄继承，机制不同）
- [[zcode-bash-windows-cwd-delete]]（memory 指针：shell CWD 持有的姊妹坑，表现同为 Device or resource busy，判别：CWD 坑可先 cd 出去解决，句柄坑必须等宿主退出）
