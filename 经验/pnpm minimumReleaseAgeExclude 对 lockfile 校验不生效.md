---
type: lesson
tags: [包管理, pnpm, 供应链安全]
date: 2026-09-16
project: dsh（Deepseek harness）
source: 自身项目实践
summary: pnpm 11.7 的 minimumReleaseAgeExclude 写在配置里却不放行 lockfile 校验；绕过用 --config.minimum-release-age=0
provenance: commit 4114c57（插件市场 dshmarket 升级）· 会话 sess_da83070d-c180-4d3b-ac1d-2e198a558de3 · 2026-09-16
status: verified
agents_md_feedback: ""
---

# pnpm minimumReleaseAgeExclude 对 lockfile 校验不生效

## 症状

pnpm 11.7 启用供应链门禁（`minimumReleaseAge: 1d`）后要装一个昨天发布的包：

```
ERR_PNPM_MINIMUM_RELEASE_AGE_VIOLATION  1 lockfile entries failed verification:
  dshmarket@1.47.0 was published at 2026-09-15T04:56:17.000Z, within the minimumReleaseAge cutoff
```

按文档把该包加进白名单后**仍然报同样的错**：

```yaml
minimumReleaseAgeExclude:
  - dshmarket@1.47.0
```

`pnpm config get minimumReleaseAgeExclude` 明确返回 `["dshmarket@1.47.0"]`（配置读到了），连改成**纯包名** `dshmarket`（不做版本精确匹配）也照样被拒。

## 根因

pnpm 11.7 的 `minimumReleaseAgeExclude` 在 **lockfile 校验路径上没有生效**：读 `pnpm/dist/pnpm.mjs` 可见 `createPackageVersionPolicy` / `isExcluded` / `detectMinReleaseAgeViolation` 的实现逻辑本身是完整正确的（名字匹配 → 版本数组 → 命中即放行），但该校验阶段实际传入的排除策略没有覆盖到 lockfile 里已存在的条目。用最小复现验证过：一个只有 `package.json` + `pnpm-workspace.yaml`（含 exclude）的空目录，`pnpm install` 依旧失败——排除配置对这条路径是失效的。

（另一个独立陷阱：YAML 里以 `@` 开头的条目必须加引号，否则 js-yaml 报 `bad indentation of a sequence entry`——`- "@scope/pkg@1.0.0"` 而非裸写。）

## 代价

更新一个插件市场的版本，卡在门禁上约半小时：先怀疑格式（改引号）、再怀疑位置（改纯名）、再怀疑版本字符串（翻 lockfile），最后读 pnpm 源码 + 做最小复现才确认是工具侧缺陷。

## 教训

**安全门禁的「白名单」本身要验证真的放行**——配置读到了 ≠ 配置生效了。门禁类工具卡住时，先建一个最小复现（空目录 + 最小配置 + 单包）把「配置问题」与「工具缺陷」分开，再决定绕行还是改配置；否则会在格式/位置上反复空转。

## 防复发

- [x] 已落实：绕行方案——`pnpm --config.minimum-release-age=0 add/install <pkg>` 单次关闭门禁完成安装，`minimumReleaseAge` 配置本身保持不变（供应链安全不破坏）
- [x] 已落实：注意 dsh 的 `dsh plugin --profile web add` 内部转发 pnpm、传不了该 flag，需直接进 profile 目录跑 pnpm
- [x] 已落实：违规包**发布满 24h 后自动过龄**，门禁自然放行，无需长期绕行（记住这个自愈时间点即可）
- [ ] 待养成：在 pipeline/脚本里依赖 exclude 白名单时，加一步断言「装完后断言目标版本确实在锁文件里」，避免静默降级

## 关联

- [[PowerShell 5.1 脚本环境假设陷阱]]（同类：工具/环境的隐性行为与文档不符时，须以实测为准）
