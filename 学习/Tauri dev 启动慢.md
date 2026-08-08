---
type: study
tags: [tauri, 性能]
date: 2026-08-06
source: 外部整理（AI 生成，未验证）
status: unverified
---

`tauri dev` 启动慢或热更新慢是 Tauri 开发中非常常见的问题，通常由 **Rust 编译耗时**、**前端构建工具配置不当** 或 **系统环境问题** 引起。

以下是按优先级排序的排查和优化方案：

### 1. 确认“慢”的具体阶段

首先需要区分是哪一步慢，这决定了优化方向：

- **首次启动慢（Cold Start）**：Rust 需要完整编译所有依赖。这是正常的，后续增量编译会快很多。
- **热更新/重载慢（Hot Reload）**：修改代码后等待时间过长。这是最需要优化的场景。
- **前端打包慢**：Vite/Webpack 等前端工具本身的构建瓶颈。

---

### 2. Rust 编译加速（核心优化）

#### ✅ 使用 `mold` 或 `lld` 链接器（效果最显著）

默认的 `ld` 链接器极慢。替换为 `mold`（Linux）或 `lld`（跨平台）可将链接速度提升 **5-10 倍**。

```toml
# .cargo/config.toml
[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=mold"]

[target.x86_64-pc-windows-msvc]
linker = "lld-link"

[target.aarch64-apple-darwin]
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```

> 💡 **安装 mold**: `sudo apt install mold` (Ubuntu) / `brew install mold` (macOS)  
> 💡 **Windows**: lld 随 LLVM 安装，或通过 `winget install LLVM.LLVM`

#### ✅ 启用增量编译 & 调试优化

在 `Cargo.toml` 中确保 dev profile 开启增量编译并降低优化级别：

```toml
[profile.dev]
incremental = true      # 默认已开启，但确认未被覆盖
opt-level = 0           # 开发环境不要开优化
debug = true            # 保留调试信息
split-debuginfo = "unpacked"  # Linux/macOS: 加速调试符号生成
```

#### ✅ 减少重编译范围

- 避免在 `lib.rs` / `main.rs` 顶层频繁改动，将逻辑拆分到子模块
- 使用 `cargo watch` 替代手动重启：`cargo tauri dev` 本身已内置 watch，但确保没有触发不必要的 crate 重编
- 检查是否有 build script (`build.rs`) 执行了耗时操作

---

### 3. 前端侧优化

#### ✅ Vite 用户（推荐）

Tauri v2 + Vite 是最优组合。确保：

- 使用 `@vitejs/plugin-react-swc` 而非 Babel（SWC 比 Babel 快 10x+）
- `vite.config.ts` 中设置 `server.hmr.overlay = false` 减少 HMR 开销
- 排除 `node_modules` 和 `src-tauri` 目录的监听

#### ✅ Webpack 用户

- 开启 `cache: { type: 'filesystem' }`
- 使用 `thread-loader` 并行处理
- 考虑迁移到 Vite/Rspack/Turbopack

---

### 4. 系统与缓存问题

|问题|解决方案|
|:--|:--|
|**杀毒软件扫描**|将项目目录、`~/.cargo`、`target/` 加入白名单（Windows Defender 是头号杀手）|
|**磁盘 I/O 瓶颈**|确保项目在 SSD 上；机械硬盘下 Tauri 体验极差|
|**Cargo 缓存损坏**|运行 `cargo clean` 后重新编译（仅当怀疑缓存问题时）|
|**Node 版本过旧**|升级到 Node 20+，新版对 fs 事件监听有显著改进|
|**WSL2 跨文件系统**|⚠️ **关键**：项目必须放在 WSL2 文件系统内（`~/project`），**绝不能**放在 `/mnt/c/` 下，否则 I/O 性能下降 100 倍|

---

### 5. 诊断命令

运行以下命令定位瓶颈：

```bash
# 查看哪些 crate 编译最慢
cargo tauri dev --timings

# 查看前端构建耗时（Vite）
npx vite build --debug

# 检查链接器是否生效
cargo tauri dev -v 2>&1 | grep "link"
```

---

### 📊 预期性能参考

|场景|正常范围|异常信号|
|:--|:--|:--|
|首次冷启动|30s–3min（取决于依赖数量）|>5min → 检查链接器/磁盘|
|增量热更新|1–5s|>10s → 检查前端HMR/Rust重编范围|
|纯前端修改重载|<2s|>5s → 前端工具链问题|

如果以上优化后仍然很慢，请提供以下信息以便进一步诊断：

1. 操作系统 & 硬件（CPU/内存/SSD型号）
2. Tauri 版本（v1 vs v2）
3. 前端框架及构建工具
4. `cargo tauri dev --timings` 的输出摘要
5. “慢”具体发生在哪个阶段及大致耗时