# herdr-orch

An orchestration skill built on top of `herdr-core` (`hc`) for pairing a coordinator agent with external coding workers such as Codex.

## Why herdr-orch

Different frontier models have distinct strengths and failure modes. A strong coordinator excels at understanding fuzzy intent, planning, and writing human-facing prose, while an execution-focused worker like Codex (`gpt-6.2-sol`) excels at deep codebase tracing, multi-file rebases, and strict code review—yet struggles with human communication, scope creep, and blind obedience to flawed premises.

`herdr-orch` captures the practical discipline needed to run that pairing reliably without duplicating `herdr-core`'s own CLI documentation:

- **Clear delegation threshold**: delegate bounded backend/algorithm modules, complex conflict resolution, and independent reviews; keep small edits, UI craft, and human-facing writing with the coordinator.
- **Verified briefs & escape hatches**: verify code facts before briefing, pin file scope to prevent rabbit-hole refactors, and explicitly allow workers to abort and report `blocked` on scope/guard conflicts.
- **Workspace & sandbox isolation**: read-only sandboxes for pure audits; isolated `git worktree` branches for coding, rebasing, and merge experiments.
- **Independent verification & circuit breaker**: audit real diffs and run real tests instead of trusting verbal summaries; cap rework/review loops at two rounds.
- **Modular model profiles**: one-level-deep reference cards under `references/` so adding or updating a model never requires rewriting the core workflow.

## Structure

```text
herdr-orch/
├── SKILL.md                           # Core orchestration workflow and routing
├── references/
│   └── codex-gpt-6.2-sol.md           # Model profile for Codex (gpt-6.2-sol)
└── README.md
```

## Prerequisites

- `herdr-core` (`hc`) installed along with its `herdr-core` skill.
- One or more supported worker CLIs (currently documented: OpenAI Codex with `gpt-6.2-sol`).

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
