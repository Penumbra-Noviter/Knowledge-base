---
type: lesson
tags: [Windows, ACL, icacls, deny, GetFileAttributes, 探测, PowerShell]
date: 2026-11-13
project: ctrl 脚本管理器
summary: icacls deny ACE 拦得住目录枚举但拦不住 GetFileAttributes——它以零访问权打开目标读属性，探测函数在 deny 下不抛异常
provenance: DEV_LOG〈2026-11-13 td-consume-28 批次〉· T-139 evidence §4 concern① · 期末四轴复认 · 双侧实证（deny 落盘成功 + 枚举被拒 + 属性读取不抛）
status: verified
agents_md_feedback: 印证 §三.1「对抗性自检——夹具自检探针先行，自检不过 skip 绝不假绿」：本轮夹具按协议自检才免于一条假绿用例入库
---

# icacls deny 拦得住枚举，拦不住 GetFileAttributes

## 症状

用 `icacls <dir> /deny Everyone:(OI)(CI)F` 构造「拒绝访问」测试夹具后：目录枚举（`EnumerateFileSystemEntries`/GCI）确实被拒，但 `[IO.File]::GetAttributes(<dir>)` **不抛异常、正常返回属性**——依赖「属性读取失败 = failed」的测试探针永远探不到 failed，夹具自检失败。

## 根因

`GetFileAttributes` 语义是**以零访问权（0）打开目标**读属性元数据——它不请求 Read/List 权限，所以 FILE_READ_DATA/_LIST_DIRECTORY 类 deny ACE 对它无效。Windows 的属性读取几乎总是可行：这是 API 设计（deny READ_CONTROL 才可能拦住，而改它影响面更大）。「枚举被拒」≠「属性被拒」，两者是不同的访问面。

## 代价

一轮夹具返工 + icacls e2e 用例只能 `pytest.skip`（自检协议兜住，没假绿）；AC 契约改由另一条 failed 形状（通配符路径 → 扩展路径转换拒收 → 探针抛异常）兜底验证。

## 教训

在 Windows 上构造「探测失败」夹具时，**deny ACL 造不出 GetFileAttributes 失败**——可行的 failed 形状是路径语法层拒收（通配符/非法形态在转换层拒绝）或被占用句柄。反过来这也是好事：探测函数对 deny ACL 天然免疫误判「不存在」，failed 判定要靠真异常而非权限。

## 防复发

- [x] 已落实：夹具自检协议（构造后先用生产探测函数验证 failed，不过即 skip）——`_gate_fixture_probe_failed` 先行断言
- [x] 已落实：failed 形状改用通配符路径拒收法（`ConvertTo-ExtendedPath` 拒收 → 探针异常），icacls 用例保留供 deny 可拦截的环境
- [ ] 通用：凡「构造某种失败」的夹具，先验证失败信号真的会被目标 API 抛出，再写断言

## 关联
- [[对抗探针先沙箱重定向再探测]]
- [[Falsify测试要钉住缺陷所在层]]
