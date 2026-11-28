# herdr-orch

English | [简体中文](README.zh-CN.md)

An orchestration skill built on top of `herdr-core` (`hc`) for pairing a coordinator agent with external coding workers across complementary model tiers (Codex and Antigravity CLI workers).

## Why herdr-orch

Different models have distinct strengths, cost profiles, and failure modes:

- **Codex (`gpt-6.2-sol`)** excels at static cross-repo fact-checking, reverse-TDD red-phase test authoring, multi-file rebases, and rigorous bug review—yet struggles with human communication, scope creep, and overfitting code structure to a single test probe.
- **Antigravity (`gemini-4-neon`)** excels at maintainable module abstractions, long-context cross-repo synthesis, and parallel engineering implementation—yet can leave test fakes behind or burn tokens on `xhigh`.
- **Antigravity (`gemini-4-hydrogen`)** offers half the cost and ~1.5x speed with a flat 1M context window for large-repo/log triage and well-patterned batch work—yet declares success too early without executable checks and degrades over long multi-turn sessions.

`herdr-orch` captures the practical discipline needed to route work across these profiles reliably without duplicating `herdr-core`'s own CLI documentation:

- **Clear delegation threshold & model routing**: delegate bounded modules, reverse-TDD test authoring, large-repo triage, complex conflict resolution, and independent reviews to the right tier; keep small edits, top-level alignment, UI craft, and human-facing writing with the coordinator.
- **Verified briefs & escape hatches**: verify code facts before briefing, specify full verification commands, pin file scope to prevent rabbit-hole refactors, and explicitly allow workers to abort and report `blocked` on scope/guard conflicts.
- **Workspace & sandbox isolation**: read-only sandboxes for pure audits; isolated `git worktree` branches for coding, rebasing, and merge experiments.
- **Independent verification & circuit breaker**: read real core diffs for both scope creep and engineering maintainability (not just `--stat` and green tests), run full verification gates yourself, and cap rework/review loops at two rounds.
- **Modular model profiles**: one-level-deep reference cards under `references/` so adding or updating a model never requires rewriting the core workflow.

## Structure

```text
herdr-orch/
├── SKILL.md                           # Core orchestration workflow and routing
├── references/
│   ├── codex-gpt-6.2-sol.md           # Model profile for Codex (gpt-6.2-sol)
│   ├── agy-gemini-4-neon.md           # Model profile for Antigravity (gemini-4-neon)
│   └── agy-gemini-4-hydrogen.md       # Model profile for Antigravity (gemini-4-hydrogen)
├── README.md
└── README.zh-CN.md
```

## Prerequisites

- `herdr-core` (`hc`) installed along with its `herdr-core` skill.
- One or more supported worker CLIs (currently documented: OpenAI Codex with `gpt-6.2-sol`, and Antigravity CLI `agy` with `gemini-4-neon` and `gemini-4-hydrogen`).

## Installation

```bash
npx skills add Hariketsu/herdr-orch
```

Or install from a local checkout:

```bash
npx skills add /path/to/herdr-orch
```

## Extending with new models

Add a new flat reference file under `references/<agent>-<model>.md` covering:

- Profile (strengths and observed failure modes)
- Effort sizing
- Briefing guardrails
- Sandbox / CLI specifics
- Coordinator audit checklist

Then add a single routing line under `## Model profiles` in `SKILL.md`.
