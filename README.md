# research-context-hub202609

这是跨 ChatGPT 对话的长期上下文 / 长记忆仓库，用来保存学习笔记、支线讨论、失败教训、方法经验和非正式历史；它**不保存也不裁决当前正式科研状态**。

记忆与仓库分层规则见：`meta/research-memory-and-agent-workflow.md`。

## 正式科研仓库

以下仓库各自拥有本线科研状态的唯一权威：

1. [`yanye-cmd/ccf-b-research202609`](https://github.com/yanye-cmd/ccf-b-research202609)：成熟地面网络机制 × D2C / LEO / NTN 主线。
2. [`yanye-cmd/ccf-b-emerging-architecture202609`](https://github.com/yanye-cmd/ccf-b-emerging-architecture202609)：Emerging / 6G Architecture 主线。
3. [`yanye-cmd/ccf-b-d2c-starlink202609`](https://github.com/yanye-cmd/ccf-b-d2c-starlink202609)：从现实部署 / 标准 / 商业与公开实现出发的 D2C / Starlink 主线。

Current Phase、KEEP/L4、candidate、TESTABLE SEED、EXPERIMENT ENTRY、current blocker 和 exact next action 必须从对应正式仓库最新 `AGENTS.md`、`HANDOFF.md`、`tasks/NEXT_CODEX_TASK.md` 恢复；不要从本仓库、旧聊天或模型记忆推断。

## 适合放进来的内容

- 经压缩整理、具有长期价值的跨对话讨论；
- 暂不构成正式 candidate 的支线 idea、观察和论文阅读笔记；
- 已被否决方向的历史记录与否决理由；
- 5G 核心网、网络、卫星、分布式系统的学习材料；
- 非敏感的组会/协作背景与沟通经验；
- 科研方法、ChatGPT 工作流、explanation style 与跨对话恢复规则；
- “原先判断 -> 改变判断的证据 -> 新结论 -> 可复用教训”这类长期 rationale。

## 不适合放进来的内容

- 任一正式科研主线的现行状态、正式 candidate 或最终 verdict 的替代副本；
- 会快速过期的 exact next action、当前标准 draft/version、临时 source blocker；
- 未经核实而补写的历史、猜测或 AI 推断；
- 全量机械复制聊天；
- 密码、token、账号标识、私人联系方式、私人医疗信息、机密沟通或其他敏感个人数据（本仓库是 public）。

## 新 ChatGPT 会话怎么读

不要默认把整个 Hub 当“入职培训”读完。

- 问长期背景、学习解释、旧讨论：先读 `AGENTS.md` + `INDEX.md`，再只打开与问题相关的 context 文件。
- 问某条正式科研线“现在做到哪了”：直接去对应正式仓库读 `AGENTS.md`、`HANDOFF.md`、`tasks/NEXT_CODEX_TASK.md`，再按当前任务指针补读必要文件。

简短 bootstrap prompt：

> 读取 `yanye-cmd/research-context-hub202609` 的 `AGENTS.md` 和 `INDEX.md`，只按我的当前问题读取相关 context。若问题涉及正式科研现状，改读对应正式科研仓库最新 `AGENTS.md`、`HANDOFF.md`、`tasks/NEXT_CODEX_TASK.md`；不要从旧聊天或 Context Hub 推断当前状态。

## 冲突处理

当本仓库、ChatGPT Memory、旧聊天和正式科研仓库冲突时，当前科研状态永远以对应正式仓库为准。本仓库保留历史属性并注明后续变化，而不是反向覆盖正式结论。
