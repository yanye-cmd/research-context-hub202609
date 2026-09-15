# Research Memory & Agent Workflow

Updated: 2026-09-15

This document defines how ChatGPT memory, this Context Hub, and the formal research repositories divide responsibility. It is intentionally small and stable.

## 1. Three memory layers

### Layer A — ChatGPT Memory: stable personal/work preferences only

Good fits:
- durable research goals and resource constraints;
- preferred explanation style and beginner assumptions;
- stable workflow preferences such as evidence-first work, bounded units, incremental GitHub sync, and separating explainers from evidence;
- stable division of labor between Web ChatGPT and Codex/Astra.

Do **not** treat ChatGPT Memory as authoritative for:
- Current Phase;
- KEEP / L4 / active candidate / TESTABLE SEED / EXPERIMENT ENTRY;
- the current strongest residual or blocker;
- exact next action;
- temporary source-access failures;
- current standard draft/version claims;
- any scientific verdict that may have changed since the remembered chat.

These are volatile project state and belong in the formal repository.

### Layer B — Context Hub: long-form cross-conversation memory

Use this repository for:
- durable rationale and lessons learned;
- learning notes and beginner explanations;
- historical/killed ideas with explicit historical labels;
- sidequests and non-mainline observations;
- research-workflow lessons;
- compressed conversation context that is useful later but is not current scientific state.

Prefer updating an existing topic file over creating duplicates. Preserve the chain `old view -> new evidence -> changed judgment -> lesson` when that chain matters.

### Layer C — Formal research repositories: authoritative scientific state

Each formal repository owns only its own Phase, KEEP/L4, candidates, seeds, experiments, evidence maturity, current blocker, and next action.

For current-state questions, read the repository's:
1. `AGENTS.md` — stable invariants and routing rules;
2. `HANDOFF.md` — current state dashboard and current-unit pointers;
3. `tasks/NEXT_CODEX_TASK.md` — execution authorization / decision boundary.

Then read only the task-specific files those pointers make relevant. Do not restore current state from old chats, ChatGPT Memory, or this Context Hub.

## 2. Progressive disclosure

Default rule:

> Read the minimum authoritative context needed for the current question, then expand only when a referenced fact, rule, or artifact is needed.

Do not make every new chat read the entire repository, every historical report, or every governance file. Root documents should route the agent to the right material instead of duplicating it.

## 3. Decision boundaries

Within one explicitly authorized bounded unit, the agent may continue without asking after every safe sub-step: search/read public sources, inspect repository files, write the unit artifact, update ledgers, correct mistakes caused by the unit, synchronize `HANDOFF`/`NEXT`, and commit when authorized access is available.

Stop before crossing a scientific decision boundary, including:
- automatically starting a second bounded research unit;
- entering a new Phase when the current gate has not explicitly authorized it;
- creating/promoting a formal candidate, L4, KEEP, TESTABLE SEED or EXPERIMENT ENTRY without the repository's evidence gate;
- designing a solution before the repository allows solution work;
- launching an experiment/acquisition that was not authorized;
- changing permanent methodology because one candidate failed;
- irreversible/external actions not covered by the current task.

## 4. Completion contract

A bounded unit is complete only when:
- its exact question is answered or the exact blocker is recorded;
- relevant evidence/collision/state artifacts are updated;
- the current-state dashboard and execution pointer are synchronized;
- the result is committed when repository write access is available;
- the next decision boundary is explicit.

A first useful source, first draft, or first implementation is not automatically completion.

## 5. Cross-repository firewall

A conclusion, KEEP/L4 state, Phase, candidate, or kill in one formal repository never automatically transfers to another. Cross-track material is a lead until the destination repository records its own evidence and locator.

The Context Hub may link the tracks, but it cannot adjudicate or overwrite their scientific state.

## 6. Public-repository privacy rule

`research-context-hub202609` is public. Do not store passwords, tokens, account identifiers, private contact details, private medical information, confidential communications, unpublished restricted material, or other sensitive personal data here. Keep public research context and non-sensitive workflow/history only.

## 7. Prompt style

For ordinary continuation, prefer short prompts that state the repository and authorization boundary instead of restating the whole method. Example:

> Continue `OWNER/REPO` from the latest GitHub state. Read `AGENTS.md`, `HANDOFF.md`, and `tasks/NEXT_CODEX_TASK.md`; then read only the files needed for the current bounded unit. Complete that unit's completion contract, sync GitHub, and stop at the next scientific decision boundary.

Long prompts are still appropriate when the user intentionally changes the research contract or gives new scientific constraints; they should not be required merely to remind the agent of rules already stored in GitHub.
