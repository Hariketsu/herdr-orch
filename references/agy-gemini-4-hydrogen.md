# Antigravity (`gemini-4-hydrogen`)

Fast, low-cost execution worker with a flat 1M-token context window (half the
token cost of Neon and ~1.5x output speed). Best used as a short-lived worker
for large-repo/log triage and high-throughput, well-patterned execution once
interfaces and plans are already settled.

## Profile

- **Strengths**: high-speed execution at low cost, flat-priced 1M context for
  ingesting large repositories, multi-doc bundles, or long logs into indexed
  summaries, well-patterned batch work (boilerplate, unit-test expansion, i18n
  migration, lint/type cleanup), and quick localized fixes under explicit specs.
- **Failure modes to guard against**:
  - *Premature success ("looks good on paper")*: optimizes for speed and may
    declare a task complete with clean-looking code without actually verifying
    runtime behavior or integration edges.
  - *Long-session context rot*: despite the 1M window, multi-hour or >300K
    token iterative threads degrade—it can forget prior fixes, re-solve closed
    issues, or reintroduce regressions.
  - *Weak autonomous architecture*: breaks down on ambiguous greenfield system
    design or deep multi-service root-cause debugging; keep architecture with
    the coordinator or Neon.
  - *Verbose burn at `xhigh`*: cranking effort to `xhigh` inflates output
    tokens and latency without matching Neon/Codex accuracy, erasing its cost
    advantage.
  - *Edge-case API guessing*: may fabricate obscure CLI flags or niche library
    arguments when relevant docs or type definitions are not in context.

## Effort sizing

Select the tier via `--model gemini-4-hydrogen-<effort>`:

- **`low` / `medium`**: large-repo/log indexing and summarization, mechanical
  formatting, i18n extraction, and repetitive boilerplate.
- **`high`** *(workhorse default)*: scoped bug fixes, unit-test authoring from
  explicit specs, and deterministic multi-file refactors.
- **`xhigh`**: avoid—if a task needs heavier reasoning than Hydrogen `high`,
  dispatch `gemini-4-neon` or `gpt-6.2-sol` instead of pushing Hydrogen to
  `xhigh`.

## Briefing guardrails

- **One short-lived goal per worker**: dispatch for a single bounded task and
  terminate immediately after verification; do not keep a Hydrogen worker alive
  across multiple feature iterations.
- **Require executable proof**: include exact test, build, or grep commands in
  the brief and instruct the worker to report real exit codes and observed
  output—never accept a verbal "done".
- **Supply exact facts and references**: include verified file paths, symbol
  names, and flag syntax in the brief (or tell it to check `--help` / local
  types) so it does not guess obscure APIs.
- **Specify output format for triage**: when using Hydrogen to compress large
  logs or codebases, ask for concise bulleted findings with `path:line`
  citations written to a file rather than long prose.

## Running `agy` in `herdr-core`

- Launch interactively—never use `-p`/`--print`, so the worker stays watchable
  and steerable inside `hc`. On startup, `agy` briefly reports `blocked, turn 0`
  while signing in (~15–20s); wait until it settles to `idle, turn 0` at the
  `>` prompt before calling `hc send`.
- Pin the model explicitly with `--model gemini-4-hydrogen-<effort>`.
- **Isolated `git worktree` coding/batch work**: use
  `--dangerously-skip-permissions` to avoid `auto`-mode sandbox-breakout
  prompt stalls (e.g., on `hc send`, worktree `.git` writes, or test runners),
  and always pair it with session-scoped `--settings` `deny` rules (which
  strictly override `--dangerously-skip-permissions` without modifying global
  `settings.json`) plus `--add-dir` for the main `.git` and briefs directories:
  ```bash
  agy --model gemini-4-hydrogen-high \
    --dangerously-skip-permissions \
    --add-dir <main-repo/.git> --add-dir <briefs-dir> \
    --settings '{"permissions":{"deny":["command(git push)","command(gh)","command(rm -rf)"]}}'
  ```
- **Pure read-only triage/audit**: `--mode plan` does not enforce read-only
  isolation. Block file edits and writes via session `--settings`
  (`write_file(*)` still permits `read_file`), using `--mode auto` with
  `allow: ["command(hc)"]` when you also want OS terminal sandboxing, or
  `--dangerously-skip-permissions` for zero-prompt shell inspection:
  ```bash
  agy --model gemini-4-hydrogen-medium \
    --dangerously-skip-permissions \
    --settings '{"permissions":{"deny":["write_file(*)","command(git commit)","command(git push)","command(gh)"]}}'
  ```
- Note that `--dangerously-skip-permissions` disables the OS terminal sandbox,
  so only use it for coding inside an isolated `git worktree` and always keep
  explicit `deny` guardrails in `--settings`.

## Auditing output

Before accepting a patch or report from `gemini-4-hydrogen`:

- Never trust a "completed" summary without running the real build, typecheck,
  and test suite yourself (or verifying actual runtime behavior).
- For triage reports, spot-check cited `path:line` references against the real
  files before building downstream plans on them.
- For code changes, check for skipped edge cases, hallucinated flags/options,
  and regressions against previously passing tests.
