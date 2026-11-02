# Research Memory & Agent Workflow

Updated: 2026-11-02

This document defines how ChatGPT Memory, this Context Hub, engineering project repositories, and formal research repositories divide responsibility. It is intentionally small and stable.

## 1. Memory and project-state layers

### Layer A — ChatGPT Memory: stable personal/work preferences only

Good fits:
- durable research goals and resource constraints;
- preferred explanation style and beginner assumptions;
- stable workflow preferences such as evidence-first work, bounded units, incremental GitHub sync, and separating explainers from evidence;
- stable division of labor between ChatGPT and Codex.

Do **not** treat ChatGPT Memory as authoritative for current project state, exact next actions, temporary blockers, or scientific verdicts.

### Layer B — Context Hub: long-form cross-conversation memory

Use this repository for:
- durable rationale and lessons learned;
- learning notes and beginner explanations;
- historical/killed ideas with explicit historical labels;
- sidequests and non-mainline observations;
- research-workflow lessons;
- compressed conversation context that is useful later but is not current engineering or scientific state.

Prefer updating an existing topic file over creating duplicates. Preserve the chain `old view -> new evidence -> changed judgment -> lesson` when that chain matters.

### Layer C — Project-state repositories: authoritative within their own scope

There are two distinct repository types. They are related, but neither can replace the other.

#### C1 — Engineering project repositories

Example: [`yanye-cmd/d2c-n3-testbed`](https://github.com/yanye-cmd/d2c-n3-testbed).

The engineering repository owns:
- testbed architecture and implementation status;
- environment inventory and validated software versions;
- code, configurations, scripts, and reproducible run manifests;
- engineering task authorization and completion records;
- packet captures, logs, measurements, and engineering result provenance.

For the D2C N3 testbed, start with:
1. `AGENTS.md` — stable engineering and agent rules;
2. `PROJECT_CHARTER.md` — project purpose and scope;
3. `HANDOFF.md` — current engineering state;
4. `tasks/NEXT_CODEX_TASK.md` — current execution authorization.

Then read only task-relevant files. The engineering repository does **not** own KEEP/L4, candidate maturity, novelty, or formal scientific verdicts.

#### C2 — Formal research repositories

Each formal research repository owns only its own scientific Phase, KEEP/L4, candidates, seeds, experiments, evidence maturity, current blocker, and next research action.

For current scientific-state questions, read the repository's:
1. `AGENTS.md` — stable research invariants and routing rules;
2. `HANDOFF.md` — current scientific state and pointers;
3. `tasks/NEXT_CODEX_TASK.md` — research execution authorization / decision boundary.

Then read only the task-specific files those pointers make relevant. Do not restore scientific state from old chats, ChatGPT Memory, or this Context Hub.

## 2. Progressive disclosure

Default rule:

> Read the minimum authoritative context needed for the current question, then expand only when a referenced fact, rule, or artifact is needed.

Do not make every new chat read the entire repository, every historical report, or every governance file. Root documents should route the agent to the right material instead of duplicating it.

## 3. Decision boundaries

Within one explicitly authorized bounded unit, the agent may continue without asking after every safe sub-step: inspect relevant sources/files, execute the authorized task, write the unit artifact, update the appropriate ledgers/state files, correct mistakes caused by the unit, synchronize the handoff and task pointer, and commit when authorized access is available.

Stop before crossing a decision boundary, including:
- automatically starting a second bounded task;
- entering a new research Phase when the current gate has not explicitly authorized it;
- creating/promoting a formal candidate, L4, KEEP, TESTABLE SEED or EXPERIMENT ENTRY without the formal repository's evidence gate;
- designing a solution before the formal research process allows it;
- launching an experiment, installing software, changing host networking, or modifying remote systems when the current engineering task has not authorized it;
- irreversible/external actions not covered by the current task.

A task marked `Status: NONE` is not authorization. A recommendation written in a handoff is not authorization.

## 4. Completion contract

A bounded unit is complete only when:
- its exact question is answered or the exact blocker is recorded;
- relevant engineering or scientific artifacts are updated in the correct repository;
- the current-state dashboard and execution pointer are synchronized;
- the result is committed when repository write access is available and the task permits it;
- limitations, unverified assumptions, and the next decision boundary are explicit.

A first useful source, first draft, first installation, or first implementation is not automatically completion.

## 5. Cross-repository firewall

- Engineering success does not automatically establish a scientific problem, novelty, candidate, or paper result.
- A scientific candidate or verdict does not automatically establish that the engineering testbed implements or reproduces it.
- A conclusion, KEEP/L4 state, Phase, candidate, or kill in one formal research repository never automatically transfers to another.
- The Context Hub may link the repositories and preserve long-term rationale, but it cannot adjudicate or overwrite their current states.

## 6. Public-repository privacy rule

`research-context-hub202609` is public. Do not store passwords, tokens, account identifiers, private contact details, private medical information, confidential communications, unpublished restricted material, or other sensitive personal data here. Keep public research context and non-sensitive workflow/history only.

## 7. Prompt style

For ordinary continuation, prefer short prompts that name the relevant authoritative repository and authorization boundary instead of restating the whole method.

Engineering example:

> Continue `yanye-cmd/d2c-n3-testbed` from the latest GitHub state. Read `AGENTS.md`, `PROJECT_CHARTER.md`, `HANDOFF.md`, and `tasks/NEXT_CODEX_TASK.md`; then read only the files needed for the currently authorized engineering task. If the task status is NONE, explain the state and ask for authorization rather than inventing work.

Formal research example:

> Continue the named formal research repository from its latest GitHub state. Read `AGENTS.md`, `HANDOFF.md`, and `tasks/NEXT_CODEX_TASK.md`; then read only the files needed for the current bounded research unit. Stop at the next scientific decision boundary.
