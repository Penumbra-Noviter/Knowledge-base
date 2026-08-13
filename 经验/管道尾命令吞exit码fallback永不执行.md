---
type: lesson
tags: [shell, git, 管道, exit-code, fallback]
date: 2026-08-13
project: Model Fingerprint
source: 自身项目实践
summary: `git push 2>&1 | tail -5 || fallback` 中管道 exit 码是尾命令（tail）的、不是 git 的——fallback 分支永不执行；fallback 应直接再跑第二条命令而非挂在管道后
provenance: DEV_LOG「批次 21」条目（推送环节）· commit c55603e→ceffe21 · 2026-08-13 批次 21 会话
status: verified
---

# 管道尾命令吞 exit 码，|| fallback 永不执行

## 症状
推送批次 21 时执行：
```bash
git push origin master:main 2>&1 | tail -5 || (echo "GFW 直连失败，用 proxy workaround" && git -c ... push ...)
```
git push 实际失败（代理 127.0.0.1:7897 连不上），但 fallback 分支**从未执行**——后续输出只有错误，没有 fallback 的 echo 与第二次 push。连续尝试 `-c http.proxy=`、`env -u HTTP_PROXY` 等变体均失败，浪费 3 次重试才定位。

## 根因
Bash 管道 `a | b || c` 的退出码是**管道最后一个命令 b（tail）** 的退出码，不是 a（git push）的。`tail -5` 永远成功退出（0）→ `||` 右侧永不触发。fallback 设计被管道语义静默吞掉，且无任何提示——比显式失败更隐蔽。

## 代价
- 3 次无效重试（每次 ~2-4s 超时 + 排查）
- 代理 key 定位额外耗时（git config 无 http.proxy，最终在 `D:\Desktop\cc\git\.gitconfig` 的 `http.https://github.com.proxy` 找到——按域配置的 proxy 用 `git config --get http.proxy` 查不到，需 `--get http.https://github.com.proxy` 或 `--list --show-origin | grep -i proxy`）

## 教训
需要 fallback 时，**把管道去掉**或**显式捕获原命令退出码**——正确形态：
```bash
git push origin master:main 2>&1 | tail -5
if [ ${PIPESTATUS[0]} -ne 0 ]; then git -c http.https://github.com.proxy= push origin master:main; fi
```
或最简：先直接跑第一条命令看结果，失败再跑第二条（两条独立命令，无管道无 fallback）。

## 防复发
- shell 中「命令 | tail | 处理失败」类组合：用 `PIPESTATUS` 或拆分独立命令，不写 `cmd | tail || fallback`
- 推送脚本/指令若需 workaround：两条独立命令顺序执行
- 查 proxy：`git config --list --show-origin | grep -i proxy`（按域 key 用 --get 全名）

## 关联
- [[git-ignored运行时资产在worktree下与主工作树分离]]
