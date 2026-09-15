# 2026-09-15 GPT-6 Astra 工作流优化记录

Status: LONG-TERM WORKFLOW RATIONALE / NOT SCIENTIFIC STATE

## Background

OpenAI 官方文章 `Rethinking skills and prompts for GPT-6 Astra` 提醒：随着模型能力增强，旧模型时代不断叠加的 Skills、AGENTS.md 规则、强制读文档流程和逐步确认要求，可能变成 instruction debt。更合适的做法是缩短 Skill/规则描述、缩窄触发范围、渐进式披露、明确安全 decision boundary，并提前定义“什么叫完成”。

Official source:
- https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra

## What we observed in our repositories

这次不是因为文章“看起来有道理”就重写仓库，而是先检查了实际仓库，发现了真实 instruction drift：

1. `ccf-b-research202609/AGENTS.md` 仍硬编码较早的 architecture-transition active workflow，而科研主线已经进入 whole-network lifecycle / Bootstrap 后续阶段。
2. `ccf-b-emerging-architecture202609/AGENTS.md` 仍保留 initialization 阶段“不授权 Phase 0”的旧边界，而 Phase 0 已完成。
3. `research-context-hub202609` 的 README/INDEX 只登记两个正式科研仓库，漏掉新建的 `ccf-b-d2c-starlink202609`。
4. 主线 HANDOFF/NEXT 曾要求新会话固定预读十多个文件，即使当前 bounded unit 只需要其中少数。
5. mutable current state 同时散落在 AGENTS/HANDOFF/NEXT，导致旧状态可能残留。

这些问题证明真正需要优化的是 instruction architecture，而不是降低科研证据门槛。

## Design decision

### Keep strict scientific invariants

保留：
- GitHub 是当前科研状态唯一权威；
- evidence-first / precise locator / fact-vs-inference；
- stage-appropriate collision；
- lifecycle / semantic completeness；
- L4 前不设计 solution；
- KEEP=0 合法；
- killed direction 不可改名复活；
- dated artifacts 不静默重写；
- 一次只执行一个授权的 bounded unit。

### Remove agent micromanagement

减少：
- 每次固定读取整套 governance / scaffold / historical reports；
- AGENTS 里硬编码 Current Phase / current active method；
- HANDOFF 复制整段历史研究；
- NEXT 再复制 HANDOFF 的完整 scientific history；
- 每个安全子步骤都要求用户确认；
- 为每类研究动作继续增加新的泛化 Skill。

### New separation

- `AGENTS.md` = stable invariants + routing + decision boundary + completion contract。
- `HANDOFF.md` = current-state dashboard + latest bounded result + exact next unit + task-specific input pointers。
- `tasks/NEXT_CODEX_TASK.md` = Codex/Astra runtime authorization + next execution boundary；`NONE` 不再误解为 Web ChatGPT 在用户明确“继续”后也不能做 bounded research。
- dated reports / mechanisms / ledgers / registry = detailed scientific history and evidence。
- Context Hub = long-form cross-conversation memory, rationale, learning, sidequests; never current formal scientific state。
- ChatGPT Memory = stable personal/work preferences only; never precise current Phase/KEEP/L4/candidate/next action。

## Progressive disclosure rule

Default bootstrap for a formal research track:

> Read `AGENTS.md`, `HANDOFF.md`, and `tasks/NEXT_CODEX_TASK.md`; then read only the files needed for the current bounded unit. Complete that unit's completion contract, sync GitHub, and stop at the next scientific decision boundary.

Long prompts are reserved for intentionally changing the research contract, not for re-stating rules already stored in GitHub.

## Decision boundary

Within one authorized bounded unit, the agent can continue through safe substeps without asking after every step: public-source search/read, repository inspection, unit artifact/ledger updates, unit-local corrections, relevant audits, HANDOFF/NEXT synchronization, and commit/push when available.

The agent stops before a second bounded unit, a new Phase, unsupported maturity promotion, unauthorized experiment/acquisition, pre-L4 solution design, permanent methodology change, or unrelated/irreversible external action.

## Completion definition

A first source, first draft, or first implementation is not completion. A bounded unit is complete when the exact question/blocker is resolved/recorded, required evidence/collision/state artifacts are synchronized, GitHub current-state pointers agree, the result is committed when possible, and the next decision boundary is explicit.

## Repository changes made in this optimization

- Added `meta/research-memory-and-agent-workflow.md` to Context Hub.
- Updated Context Hub AGENTS/README/INDEX; registered all three formal research repositories and added public-repo privacy rules.
- Slimmed `ccf-b-research202609` AGENTS/HANDOFF/NEXT around progressive disclosure and the latest Phase-1 re-entry boundary.
- Slimmed `ccf-b-emerging-architecture202609` AGENTS/HANDOFF/NEXT; removed stale initialization rule and made D7-00 the current authorized-next-on-continuation boundary.
- Slimmed `ccf-b-d2c-starlink202609` AGENTS/HANDOFF/NEXT while preserving its reality-driven evidence model and current unresolved effective-config question.

No scientific verdict, KEEP/L4 state, candidate state, evidence record, or historical artifact was rewritten merely to implement this workflow optimization.

## Reusable lesson

When a model upgrade arrives, do not ask only “what new rules should we add?” First ask:

> Which rules encode domain truth or scientific invariants, and which rules merely compensate for limitations of an older model?

Keep the first class. Re-test and usually reduce the second.
