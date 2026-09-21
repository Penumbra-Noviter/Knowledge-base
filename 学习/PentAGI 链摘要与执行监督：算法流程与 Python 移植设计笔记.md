# PentAGI 链摘要与执行监督：算法流程与 Python 移植设计笔记

> 来源：vxcontrol/pentagi（master，2026-09 抓取，zip SHA 见标注文件）
> 核心文件：`backend/pkg/csum/chain_summary.go`、`backend/pkg/cast/chain_ast.go`、`backend/pkg/providers/{performer.go,performers.go,helpers.go,providers.go}`、`backend/pkg/config/config.go`
> 用途：为自研上下文管理模块（对齐 stage3 向量召回）和执行监督层提供参照设计。

---

## 0. 一句话定位

- **csum**：确定性规则 + LLM 摘要混合的对话链压缩器。不靠 embedding，靠「结构化 AST + 字节预算 + 保留最新」的贪心策略压缩历史段，摘要失败时回退保留原文。
- **监督逻辑**：四层防护（硬上限 → 重复检测 → mentor 动态干预 → reflector 兜底），外加可选的 planner 预规划。

两者均与 LLM provider 解耦，通过 handler 函数注入。

---

## 1. 模块一：链摘要（csum）

### 1.1 数据结构（ChainAST 概念模型）

```
ChainAST
├── Sections[]: ChainSection        # 一个对话轮次区段
│   ├── Header
│   │   ├── SystemMessage           # 仅第一个 section 有
│   │   └── HumanMessage            # 用户请求
│   └── Body[]: BodyPair            # 一个 AI 响应 + 其工具调用序列
│       ├── Type: Summarization | RequestResponse | Completion
│       ├── AIMessage（含工具调用 parts、可选 reasoning）
│       └── ToolMessages[]
└── Size()  # 所有 section 递归求和
```

关键点：每个 section 是「human 头部 + 若干 body pair」，body pair 粒度保存工具调用完整性；`Size()` 按字节计算（`CalculateMessageSize` 递归 parts）。

### 1.2 算法流程（三步策略，实现在 `SummarizeChain`）

```
输入: chain[] MessageContent, handler SummarizeHandler, cfg
1. 解析 chain → ChainAST
2. summarizeSections: 除最后 KeepQASections 个 section 外，
   每个 section 的多个 body pair 并发合并为 1 个摘要 pair
   （跳过已是摘要的；Completion 型保留文本前缀标记）
3. summarizeLastSection（仅 PreserveLast=true）:
   a. 单个 body pair > MaxBPBytes 的先就地压缩（跳过最后一个 pair——保护 reasoning 签名）
   b. 整个 section 仍 > LastSecBytes 时:
      - 阈值 = LastSecBytes × (100 - 25%) → 留出未来消息余量
      - 从最旧向最新分配: 塞得进阈值就保留，否则标记为待压缩（压一个点之后全压）
      - 铁律: 最新 pair 永不压缩（Gemini thought_signature / Anthropic 加密签名依赖它）
      - 防呆: 若「保留部分 + 待压部分」总和不超限 → 放弃压缩（白压不干）
      - 防重复: 待压集合只有一个 Summarization pair → 放弃
      - 产出: body = [summaryPair] + pairsToKeep
4. summarizeQAPairs（仅 UseQA=true）:
   - 触发: len(Sections) > MaxQASections 或 ast.Size() > MaxQABytes
   - 保留计数: 从最新往回，先强制留 KeepQASections 个，
     再按 maxSections 与 (MaxQABytes-1000 buffer) 贪心增留
   - human 消息: SummHumanInQA=false 时原文保留；true 时才压缩
   - ai 消息: 全量生成一份摘要
   - 重组: [摘要 section(system+human)] + 保留的 sections（剥掉各自 system）
5. ast.Messages() 返回新链
```

### 1.3 摘要 prompt 协议（`GenerateSummary → messagesToPrompt`）

三段式指令，XML 包装：

- case1（human+ai）：`<instructions>` + `<tasks>`（human 原文）+ `<messages>`（ai 原文，含 `<tool_call>`/`<tool_call_response>` 结构化保留）
- case2（纯 ai）：自包含摘要，无 user 上下文
- case3（纯 human）：提炼要求/约束，指令式
- 所有 case 强制：保留技术细节（函数名、路径、URL、版本、数字）、完整代码示例、逐步步骤；已有 summarized 内容视为高优先级要并入不重复
- 输出前缀标记：`**summarized content:**\n`（Completion 型识别标志）

### 1.4 参数契约

| 环境变量 | 默认 | 含义 |
|---|---|---|
| `SUMMARIZER_PRESERVE_LAST` | `true` | 保留最近区段原文 |
| `SUMMARIZER_USE_QA` | `true` | 启用 QA 对压缩 |
| `SUMMARIZER_SUM_MSG_HUMAN_IN_QA` | `false` | QA 压缩时是否压 human |
| `SUMMARIZER_LAST_SEC_BYTES` | `51200` | 最近区段字节上限（50KB） |
| `SUMMARIZER_MAX_BP_BYTES` | `16384` | 单 body pair 上限（16KB） |
| `SUMMARIZER_MAX_QA_SECTIONS` | `10` | QA 区段上限 |
| `SUMMARIZER_MAX_QA_BYTES` | `65536` | QA 区段字节上限（64KB） |
| `SUMMARIZER_KEEP_QA_SECTIONS` | `1` | 保留的最近区段数 |

Assistant 通道独立配置，默认更宽：`76800 / 16384 / 7 / 76800 / 3`。内部固定参数：reserve 25%、keep-min-last 1、摘要前缀 `**summarized content:**\n`。

### 1.5 移植要注意的坑（Falsify 视角）

- 空 handler / 空消息必须显式报错（原始实现 `GenerateSummary` 对二者均返回 error，不静默）。
- 摘要失败的分支语义不同：section 级失败直接返回 error 中止；last-section 压缩失败则**回退为仅保留最新 pairs**（`lastSection.Body = pairsToKeep`）并返回 error——两层回退行为不一致，移植时须明确各自契约。
- 字节预算的 guard 是「阈值判断」而非「压缩后比较」——不要照 README 流程图实现成显式长度比较。
- reasoning 签名是硬约束：最后 pair 不压、压缩时 `ContainsToolCallReasoning` 决定补 fake signature、`ExtractReasoningMessage` 保留 reasoning_content（Kimi 等 provider 要求 ToolCall 前有 reasoning）。
- 并发压缩共享 AST 切片，写回需加锁（原实现用 mutex + 错误 channel），Python 移植用 `ThreadPoolExecutor` 时同理会踩。

---

## 2. 模块二：执行监督（四层防护 + 规划）

### 2.1 分层结构

```
L0 硬上限     maxCallsLimit（GA=100 / LA=20，按 agent 类型分级）
             └ 接近上限（limit-3）时不再调 LLM，直接注入终止消息走 reflector 优雅收尾
L1 重复检测   repeatingDetector：连续 3 次相同 name+args（args 规范化后比较）→ 返回提示
             └ 累计 3+4=7 次 → 硬中止链
L2 动态监督   executionMonitor（enabled，same=5 / total=10）
             └ 工具执行成功后计数；触发 → 调 mentor（adviser）→ 响应包装双段 → reset 计数
L3 兜底       reflector（调用 3 次重试失败 / 无工具调用 / 接近上限时介入）
             └ maxReflectorCallsPerChain=3，递归防护（context key 标记）
旁路 规划     performPlanner（AGENT_PLANNING_STEP_ENABLED）：执行前生成 3-7 步计划，
             包装 <task_assignment> 限制 scope creep
旁路 修参     fixToolCallArgs：工具执行报错 → 拿 schema 让 LLM 重建参数，最多 3 次重试
```

### 2.2 触发矩阵

| 事件 | 处理 | 连续性 |
|---|---|---|
| 同工具连续调用 ≥ 3（args 规范化） | 返回「is repeating, please try another tool」 | 继续 |
| 重复累计 ≥ 7 | 中止 chain（error） | 终止 |
| 工具执行成功且 monitor 触发（same≥5 或 total≥10） | mentor 介入；响应 = `<original_result>` + `<mentor_analysis>` | 继续（计数清零） |
| LLM 返回无 content 且无工具调用 | 判失败重试 | 重试 ≤3 |
| agent chain 调用失败 3 次 | performCallerReflector → performReflector | 1 次 |
| reflector 后仍无工具调用 | 递归防护，直接 error | 终止 |
| 接近迭代上限（limit-3） | 注入终止消息，reflector 优雅收尾 | 终止 |

### 2.3 mentor 输入构造（关键细节）

- 最近 30 条消息（`getRecentMessages`，按 body pair 倒序，取 `CompletedToolCalls`）
- 最近 10 次工具调用（name / args 排序字段 / result 截断 1KB）
- last tool 的 args（排序字段，每字段截 256）、result 截断 4096
- subtask 描述 + agent 原始 prompt（从 chain last section human 提取）
- mentor 失败仅 warn 不阻断（降级为普通工具响应）

### 2.4 参数契约

| 环境变量 | 默认 | 含义 |
|---|---|---|
| `EXECUTION_MONITOR_ENABLED` | `false` | 启用 mentor 监督 |
| `EXECUTION_MONITOR_SAME_TOOL_LIMIT` | `5` | 同工具连续阈值 |
| `EXECUTION_MONITOR_TOTAL_TOOL_LIMIT` | `10` | 总调用阈值 |
| `MAX_GENERAL_AGENT_TOOL_CALLS` | `100` | 通用 agent 硬上限 |
| `MAX_LIMITED_AGENT_TOOL_CALLS` | `20` | 受限 agent 硬上限 |
| `AGENT_PLANNING_STEP_ENABLED` | `false` | 启用任务预规划 |

内部常量：重试 3 / reflector 3 / 重复阈值 3 / 软重复容忍 4 / 关闭窗口 3 / 间隔 5s。实测口径（README）：监督开启后耗时与 token 2-3x，小模型（<32B）结果质量 +2x。

---

## 3. Python 移植接口草案

对齐项目约定：`from __future__ import annotations`、`__all__`、type hints + docstring、CLI 优先。以下为接口骨架，非完整实现。

### 3.1 消息与 AST（简化版）

```python
from __future__ import annotations

from dataclasses import dataclass, field
from enum import IntEnum
from typing import Protocol

__all__ = [
    "MessageRole",
    "BodyPairType",
    "Message",
    "BodyPair",
    "ChainSection",
    "ChainAST",
    "SummarizeHandler",
    "SummarizerConfig",
    "Summarizer",
    "ExecutionMonitor",
    "RepeatingDetector",
    "format_enhanced_tool_response",
]


class MessageRole(str, IntEnum):
    SYSTEM = 0
    HUMAN = 1
    AI = 2
    TOOL = 3


class BodyPairType(IntEnum):
    SUMMARIZATION = 0
    REQUEST_RESPONSE = 1
    COMPLETION = 2


@dataclass(slots=True)
class Message:
    role: MessageRole
    parts: list[str] = field(default_factory=list)
    """part 统一为 str：text / tool_call JSON / tool_response 序列化文本"""

    def size(self) -> int:
        """字节大小，对齐原实现 CalculateMessageSize 的递归语义。"""
        return sum(len(p.encode("utf-8", errors="replace")) for p in self.parts)


@dataclass(slots=True)
class BodyPair:
    typ: BodyPairType
    ai_parts: list[str] = field(default_factory=list)
    tool_parts: list[str] = field(default_factory=list)

    def size(self) -> int:  # pragma: no cover - 语义与 Message.size 一致
        return sum(len(p.encode("utf-8", errors="replace")) for p in (*self.ai_parts, *self.tool_parts))

    def messages(self) -> list[Message]:
        return [Message(MessageRole.AI, self.ai_parts), Message(MessageRole.TOOL, self.tool_parts)]


@dataclass(slots=True)
class ChainSection:
    system: list[str] = field(default_factory=list)
    human: list[str] = field(default_factory=list)
    body: list[BodyPair] = field(default_factory=list)

    def size(self) -> int:
        return sum(len(p.encode("utf-8", errors="replace")) for p in (*self.system, *self.human)) + sum(
            pair.size() for pair in self.body
        )


@dataclass(slots=True)
class ChainAST:
    sections: list[ChainSection] = field(default_factory=list)

    def size(self) -> int:
        return sum(section.size() for section in self.sections)
```

### 3.2 摘要器

```python
class SummarizeHandler(Protocol):
    """一次摘要调用：入参为构造好的 prompt 文本，返回摘要文本。"""

    def __call__(self, prompt: str) -> str: ...


@dataclass(frozen=True, slots=True)
class SummarizerConfig:
    """参数契约与 PentAGI env 对齐；None 表示取默认值（对齐 NewSummarizer 兜底）。"""

    preserve_last: bool = True
    use_qa: bool = True
    summ_human_in_qa: bool = False
    last_sec_bytes: int = 50 * 1024
    max_bp_bytes: int = 16 * 1024
    max_qa_sections: int = 10
    max_qa_bytes: int = 64 * 1024
    keep_qa_sections: int = 1
    reserve_percent: int = 25  # 非开放参数，保留可观测

    def with_defaults(self) -> "SummarizerConfig":
        return SummarizerConfig(
            preserve_last=self.preserve_last,
            use_qa=self.use_qa,
            summ_human_in_qa=self.summ_human_in_qa,
            last_sec_bytes=self.last_sec_bytes or 50 * 1024,
            max_bp_bytes=self.max_bp_bytes or 16 * 1024,
            max_qa_sections=self.max_qa_sections or 10,
            max_qa_bytes=self.max_qa_bytes or 64 * 1024,
            keep_qa_sections=self.keep_qa_sections or 1,
            reserve_percent=self.reserve_percent or 25,
        )


SUMMARIZED_PREFIX = "**summarized content:**\n"
MAX_STRING = 10**6  # 单条输入安全上限，防御性


class Summarizer:
    """三阶段压缩：sections 归一 → last section 大小管理 → QA 对压缩。"""

    def __init__(self, handler: SummarizeHandler, config: SummarizerConfig | None = None) -> None:
        self._handler = handler
        self._cfg = (config or SummarizerConfig()).with_defaults()

    def summarize_chain(self, ast: ChainAST) -> ChainAST:
        """返回新 AST（原实现原地修改后重建；此处返回副本更安全）。"""
        raise NotImplementedError("按 1.2 节流程实现；先 sections 归一，再 last-section，最后 QA")

    def _generate_summary(self, human_parts: list[str], ai_parts: list[str]) -> str:
        if not human_parts and not ai_parts:
            raise ValueError("cannot summarize empty message list")
        return self._handler(self._messages_to_prompt(human_parts, ai_parts))

    def _messages_to_prompt(self, human_parts: list[str], ai_parts: list[str]) -> str:
        # case1/2/3 指令 + <tasks>/<messages> XML 包装，见 1.3 节
        raise NotImplementedError
```

### 3.3 执行监督

```python
class ExecutionMonitor:
    """计数型 mentor 触发器；阈值对齐 EXECUTION_MONITOR_*。"""

    def __init__(self, *, enabled: bool = False, same_threshold: int = 5, total_threshold: int = 10) -> None:
        self.enabled = enabled
        self.same_threshold = same_threshold
        self.total_threshold = total_threshold
        self._same_count = 0
        self._total_count = 0
        self._last_tool = ""

    def should_invoke_mentor(self, tool_name: str) -> bool:
        if not self.enabled:
            return False
        self._total_count += 1
        self._same_count = self._same_count + 1 if tool_name == self._last_tool else 1
        self._last_tool = tool_name
        return self._same_count >= self.same_threshold or self._total_count >= self.total_threshold

    def reset(self) -> None:
        self._same_count = 0
        self._total_count = 0
        self._last_tool = ""


class RepeatingDetector:
    """连续重复工具调用检测：阈值 3 提示、7 中止；args 需规范化后比较。"""

    def __init__(self, *, threshold: int = 3, soft_abort_tolerance: int = 4) -> None:
        self._threshold = threshold
        self._abort_at = threshold + soft_abort_tolerance
        self._calls: list[tuple[str, str]] = []

    def detect(self, tool_name: str, args_normalized: str) -> str | None:
        """返回 None=正常；'retry'=提示更换工具；'abort'=应中止链。"""
        if not self._calls or self._calls[-1] != (tool_name, args_normalized):
            self._calls = [(tool_name, args_normalized)]
            return None
        self._calls.append((tool_name, args_normalized))
        if len(self._calls) >= self._abort_at:
            return "abort"
        if len(self._calls) >= self._threshold:
            return "retry"
        return None

    @staticmethod
    def normalize_args(args_json: str) -> str:
        """排序字段、剔除 'message' 键，防语义相同而文本不同导致的漏检。"""
        import json

        try:
            obj = json.loads(args_json)
        except json.JSONDecodeError:
            return args_json
        if not isinstance(obj, dict):
            return args_json
        obj.pop("message", None)
        return "\n".join(f"{k}: {obj[k]}" for k in sorted(obj))


def format_enhanced_tool_response(original_result: str, mentor_analysis: str) -> str:
    if not mentor_analysis:
        return original_result
    return (
        "<enhanced_response>\n"
        "<original_result>\n"
        f"{original_result}\n"
        "</original_result>\n\n"
        "<mentor_analysis>\n"
        f"{mentor_analysis}\n"
        "</mentor_analysis>\n"
        "</enhanced_response>"
    )
```

### 3.4 与 stage3 向量召回的衔接建议

- **分工**：csum 走确定性规则压缩保「连贯性」，向量召回（VR-01~03 已交付）保「语义检索」。建议压缩发生在写库前，向量写入用压缩后文本，两侧不重叠。
- **观察点**：`Summarizer` 的每阶段记 `(before_size, after_size, action)` 结构日志；「压缩后更大则不采纳」用测试固定，防引入曲线回归。
- **CLI 优先落地顺序**：`summarize_chain` 纯函数 → 单测（空链/单 pair/超限 pair/QA 触发/摘要失败回退）→ 接入现有 agent 编排。

---

## 4. 移植验收清单（对抗性）

- [ ] 空 handler 与空消息抛错，不静默返回原文
- [ ] 最后 pair 永不压缩（构造 2 条消息其中 1 条巨大，断言最新 pair 原文暴露）
- [ ] 单 body pair 超 16KB 被就地压缩，且不是最后 pair
- [ ] 摘要失败：section 级中止 vs last-section 回退两个分支各自断言
- [ ] 重复检测：同工具同参数（含 message 差异被规范化）触发 retry/abort
- [ ] mentor 触发后计数清零，且 mentor 异常不影响主流程
- [ ] reflector 递归防护：caller reflector 后再触发 reflector 直接 error

---

## 5. 原始代码锚点（复核用）

| 关注点 | 位置 |
|---|---|
| SummarizeChain 三步 | `backend/pkg/csum/chain_summary.go:107` |
| last section 分配算法 | 同文件 `:719`（determineLastSectionPairs） |
| QA 保留计数 | 同文件 `:639`（determineRecentSectionsToKeep） |
| 摘要指令三段 | 同文件 `:846` |
| size 计算 | `backend/pkg/cast/chain_ast.go:712,890` |
| 工具循环 + 优雅终止 | `backend/pkg/providers/performer.go:106` |
| mentor 触发 | performer.go `:374` + `backend/pkg/providers/helpers.go:153` |
| 重复检测 | helpers.go `:89` |
| planner | `backend/pkg/providers/performers.go:811` |
| mentor 输入构造 | performers.go `:873` |
| reflector | performer.go `:565` / `:730` |
| 参数契约 | `backend/pkg/config/config.go:97,260` |