---
name: herdr-orch
description: >-
  Orchestrates external coding agents (such as Codex and Antigravity CLI
  workers) through herdr-core with clear delegation boundaries, git worktree
  isolation, model-specific guardrails, and independent verification. Use
  whenever delegating substantial coding, reverse-TDD test authoring,
  large-repo triage, multi-file conflict resolution, or independent code review
  to a worker agent, or whenever the user mentions dispatching Codex, Neon,
  Hydrogen, pairing models, or running a worker via herdr-core.
---

# Herdr Orchestrator

Coordinate external coding workers through the `herdr-core` skill. This skill
covers *when* to delegate, *how* to pick a worker, frame and isolate work, and
*how* to audit results—leaving `hc` CLI mechanics to `herdr-core` itself.

## When to delegate

Every delegation carries fixed overhead: verifying facts, writing a brief,
auditing the diff, and running tests. Delegate only when the work justifies
that overhead, and match task shape to worker tier:

- **Good fits**:
  - *Bounded implementation & rebases*: independent backend/algorithm modules
    with settled interfaces, parallel multi-file modules, and multi-file rebase
    or conflict resolution.
  - *Test-first splits (reverse TDD)*: having a strict worker author red-phase
    tests against a locked contract while the coordinator implements to green
    (or vice versa).
  - *Read-only triage & verification*: large-repo or long-log indexing, static
    cross-repo fact-checking, and independent code review after your own
    implementation.
- **Keep with the coordinator**: small edits (a few dozen lines), open-ended
  product or top-level architecture alignment, UI/visual craft, tight
  front-end/back-end iteration, and all human-facing writing (PR descriptions,
  review replies, issue comments, and user summaries).

## Preparing the brief

Workers see only the brief and the working tree, and execution-focused models
follow instructions literally. If a coordinator passes an unverified
assumption, the worker will faithfully build on the wrong premise.

Verify every file path, symbol name, and repository invariant in code before
writing the brief. Put the brief in a local file and point the worker to it
with a short message, covering:

- **Goal and verified context**: what to achieve and the exact contracts it
  must preserve.
- **Allowed scope**: the specific files or directories the worker may touch, so
  it does not drift into adjacent refactors.
- **Verification commands**: the exact full check commands (including root
  typecheck, lint, and test suites—not just a local subset) the worker must run
  and report before finishing.
- **Escape hatch**: explicit permission to abort cleanly and report `blocked`
  if an upstream rule, guard test, or design conflict alters scope—otherwise
  workers tend to hack around blockers or weaken tests.
- **Delivery and limits**: whether local commits are expected, and what stays
  off-limits (pushing, external write commands, subagents, or editing drafts).

## Workspace and git boundaries

Match isolation to the task so workers cannot corrupt the main working tree or
collide with each other:

- **Read-only review or audit**: run the worker in a read-only sandbox so it
  cannot mutate files.
- **Coding, rebasing, or merge experiments**: create an isolated `git worktree`
  on a dedicated branch (or a throwaway `/tmp/...` worktree for experimental
  merges), and remove temporary worktrees once finished.
- **Local commits**: on a dedicated worktree branch, allow local commits (with
  `-s` if the repo requires DCO) so the coordinator gets an immutable commit
  hash for `git show` review, red/green test comparisons, and remote build
  checks. On a shared main working tree without a worktree, forbid `git commit`
  and review the unstaged diff directly.
- **Coordinator-only authority**: pushing, switching other branches, running
  `gh` write commands, accessing remote servers, and publishing drafts always
  stay with the coordinator.

## Orchestration workflow

1. **Select a model profile** from the table below and read its reference file
   before launching.
2. **Launch and dispatch via `herdr-core`**: start the worker interactively in
   the background with the right sandbox, confirm it is ready at the input
   prompt (dismissing any startup dialog), then send the brief path.
3. **Audit independently**: green tests and a clean `git show --stat` are not
   enough. Never accept a worker's verbal summary or reported hash at face
   value: verify `git rev-parse HEAD` and `git status`, read the actual core
   diff—not just a sample—for scope creep, damaged comments, hardcoded
   test-probe assumptions or temporary placeholders, monolithic procedural
   functions, and leftover test fakes, then run real tests plus repository
   guards yourself (or on the build machine).
4. **Filter review findings**: when using a worker as a reviewer, triage its
   comments before acting—separate real, reproducible defects from pedantic
   overfitting, and summarize the trade-offs for the user.
5. **Cap iterations at two rounds**: if rework or review still does not
   converge after two rounds, stop and escalate to the user with concrete
   options rather than looping.
6. **Clean up**: terminate finished workers and remove any temporary worktrees
   once verified.

## Model profiles

Read only the profile for the model you are dispatching:

- **Codex (`gpt-6.2-sol`)** — static cross-repo fact-checking, reverse-TDD
  red-phase test authoring, multi-file rebases, independent code review, and
  contract-locked protocol/algorithm modules: see
  [references/codex-gpt-6.2-sol.md](references/codex-gpt-6.2-sol.md)
- **Antigravity (`gemini-4-neon`)** — parallel engineering modules requiring
  clean abstractions and long-term maintainability, large-context cross-repo
  refactors, and deep structural reviews: see
  [references/agy-gemini-4-neon.md](references/agy-gemini-4-neon.md)
- **Antigravity (`gemini-4-hydrogen`)** — fast, low-cost execution with a flat
  1M context window for large-repo/log triage, well-patterned boilerplate or
  test batches, and short single-goal fixes: see
  [references/agy-gemini-4-hydrogen.md](references/agy-gemini-4-hydrogen.md)
