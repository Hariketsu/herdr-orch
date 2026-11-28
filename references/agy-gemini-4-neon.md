# Antigravity (`gemini-4-neon`)

Balanced, engineering-grade worker for parallel feature modules that require
clean abstractions, long-context cross-repo synthesis, and maintainable
multi-file changes. Stronger than execution-only models on module boundaries
and human readability; reserve higher effort tiers for genuinely complex logic.

## Profile

- **Strengths**: maintainable software engineering structure (clean interfaces,
  polymorphic adapters, cohesive modules, readable prose), large-context
  cross-repo synthesis (1M context window across code, docs, and multimodal
  inputs such as diagrams or screenshots), complex multi-file refactoring, and
  low-hallucination structural code review.
- **Failure modes to guard against**:
  - *Leftover test fakes or stubs*: may leave demo/test scaffolding or fake
    branches wired into production paths unless explicitly asked to strip or
    isolate them.
  - *Over-thinking at `xhigh`*: burns excessive hidden thinking tokens and
    latency on `xhigh` with minimal accuracy gain over `high`.
  - *Unanchored visual craft*: without explicit design tokens, reference
    components, or rendered verification, free-form UI layouts can drift in
    typography and spacing.
  - *Long-thread audit fatigue*: over very long sessions, can slip into
    accepting green test summaries instead of checking every integration seam.

## Effort sizing

Select the tier via `--model gemini-4-neon-<effort>`:

- **`medium`**: routine feature modules with clear contracts, scoped refactors,
  and standard documentation or integration updates.
- **`high`** *(recommended default)*: parallel architecture-bearing modules,
  multi-file refactors, cross-repo synthesis, and deep code reviews.
- **`xhigh`**: avoid by default—diminishing returns on cost and latency; use
  only for unusually tangled concurrency, security, or migration boundaries.

## Briefing guardrails

- **Separate production from test scaffolding**: state explicitly where mocks,
  fixtures, or demo harnesses belong and forbid leaving test-only branches in
  production entry points.
- **Give concrete verification commands**: pair Neon with runnable compiler,
  lint, and test suites; for UI changes, point it to existing design tokens and
  require rendered/screenshot verification rather than visual guesswork.
- **Keep sessions scoped**: prefer one focused worktree task per worker rather
  than reusing one long-lived session across unrelated branches.
- **Bound code reviews**: enforce the same two-tier severity split (reproducible
  correctness/safety defects vs. non-blocking advice) so deep reviews stay
  actionable instead of inflating token spend.

## Running `agy` in `herdr-core`

- Launch interactively—never use `-p`/`--print`, so the worker stays watchable
  and steerable inside `hc`. On startup, `agy` briefly reports `blocked, turn 0`
  while signing in (~15–20s); wait until it settles to `idle, turn 0` at the
  `>` prompt before calling `hc send`.
- Pin the model explicitly with `--model gemini-4-neon-<effort>`.
- **Isolated `git worktree` coding/rebasing**: use
  `--dangerously-skip-permissions` to avoid `auto`-mode sandbox-breakout
  prompt stalls (e.g., on `hc send`, worktree `.git` writes, or test runners),
  and always pair it with session-scoped `--settings` `deny` rules (which
  strictly override `--dangerously-skip-permissions` without modifying global
  `settings.json`) plus `--add-dir` for the main `.git` and briefs directories:
  ```bash
  agy --model gemini-4-neon-high \
    --dangerously-skip-permissions \
    --add-dir <main-repo/.git> --add-dir <briefs-dir> \
    --settings '{"permissions":{"deny":["command(git push)","command(gh)","command(rm -rf)"]}}'
  ```
- **Pure read-only review/audit**: `--mode plan` does not enforce read-only
  isolation. Instead, block file edits and writes via session `--settings`
  (`write_file(*)` still permits `read_file`), using `--mode auto` with
  `allow: ["command(hc)"]` when you also want OS terminal sandboxing, or
  `--dangerously-skip-permissions` for zero-prompt shell inspection:
  ```bash
  agy --model gemini-4-neon-high \
    --dangerously-skip-permissions \
    --settings '{"permissions":{"deny":["write_file(*)","command(git commit)","command(git push)","command(gh)"]}}'
  ```
- Note that `--dangerously-skip-permissions` disables the OS terminal sandbox,
  so only use it for coding inside an isolated `git worktree` and always keep
  explicit `deny` guardrails in `--settings`.

## Auditing output

Before accepting a commit or patch from `gemini-4-neon`:

- Confirm `git rev-parse HEAD` and `git status` in the worktree.
- Read the diff specifically for leftover fake/mock branches, unrequested
  compatibility shims, or incomplete integration seams across module
  boundaries.
- Run the full repository verification gate (not only module-level checks) on
  the target environment.
