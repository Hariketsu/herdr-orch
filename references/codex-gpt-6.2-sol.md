# Codex (`gpt-6.2-sol`)

High-rigor, cost-effective worker for bounded coding, conflict resolution, and
code review. Give it verified facts, tight scope, and right-sized reasoning
effort; audit every line it returns; keep all human-facing writing with the
coordinator.

## Profile

- **Strengths**: deep codebase tracing, static cross-repo fact-checking,
  reverse-TDD red-phase test authoring (locking edge cases and timing before
  implementation), multi-file rebase conflict resolution, isolated
  protocol/algorithm modules with settled contracts, and catching subtle bugs
  (races, boundary conditions, leaks) in independent code review.
- **Failure modes to guard against**:
  - *Test-probe overfitting & procedural collapse*: tends to hardcode single
    acceptance-probe assumptions or temporary placeholders into schemas and
    production logic, and packs complex state machines into monolithic
    procedural functions and module-level mutable state instead of clean
    abstractions. Avoid delegating greenfield architecture or long-lived
    domain modeling.
  - *Literal obedience*: executes wrong premises without pushing back, because
    it assumes the brief's facts are authoritative.
  - *Rabbit holes*: spots adjacent code and launches unrequested refactors,
    custom validation layers, or unused branches that inflate diffs.
  - *Local optimization*: while fixing a narrow bug, may trim existing comments
    that explain broader behavior or leak PR numbers into code comments.
  - *Defensive / noisy prose*: reports are dense and hard to read; commit
    messages and drafts often leak internal workflow jargon (`coordinator`,
    `red test`, local hashes).
  - *Silent stalls*: may sit quiet for several minutes during context
    compaction or background tool calls—check screen state before assuming it
    is dead.
  - *Long-chain drift*: on open-ended multi-step tasks without checkpoints, it
    can lose the primary goal while optimizing secondary details.

## Effort sizing

Match reasoning effort to risk rather than defaulting to the maximum:

- **`medium`**: routine scoped fixes, guard table updates, standard reviews.
- **`high`**: multi-file rebases, non-trivial backend logic, deeper reviews.
- **`xhigh`**: reserve for money/billing, auth/permissions, external messaging,
  or core persistence/migration boundaries. Max effort on routine tasks causes
  spinning, pedantic loops, and does not reduce hallucination rates.

## Briefing guardrails

- **Pin the file scope**: name the exact files it may edit and state that
  adjacent code must stay untouched.
- **Forbid probe-specific shortcuts**: state clearly that test scenarios are
  examples, not upper bounds—do not hardcode single-probe assumptions or
  temporary caller placeholders into production schemas or control flow, and
  split multi-stage state machines into cohesive helpers rather than one
  monolithic function.
- **Grant the `blocked` exit**: explicitly tell it not to weaken tests or
  guards—if an upstream rule or design conflict changes scope, abort cleanly
  and report `blocked`.
- **Clarify environment limits**: if the local worktree has no `node_modules`
  or network (e.g., partial clone or separate build machine), tell it not to
  waste turns hunting for global toolchains or running full suites locally.
- **Bound code reviews**: enforce two-tier severity—only reproducible defects
  that cause data loss, wrong money/state, privilege bypass, bad external
  sends, or release outages are blocking; everything else is non-blocking
  advice. Filter out overfitting before applying any review suggestion, and cap
  at two rounds.

## Running Codex in a sandbox

- Pass `--no-daemon` so commands execute inside `herdr-core`.
- Use `-s read-only` for pure reviews and audits; use `-s workspace-write -a
  on-request` for coding and rebasing.
- Use `--add-dir` **only** with `-s workspace-write` (for a git worktree, grant
  the main repository `.git` directory and briefs directory, omitting draft
  directories it should not touch). Never pass `--add-dir` with `-s read-only`:
  Codex exits immediately (`effective permissions do not allow additional
  writable roots`), and `-s read-only` can already read external brief paths
  directly.
- Launch without an initial task argument first so startup dialogs (such as
  `Update available` or folder trust) can be dismissed cleanly before sending
  the brief.

## Auditing output

Before accepting a commit or patch from `gpt-6.2-sol`:

- Confirm the exact commit hash (`git rev-parse HEAD`) and working tree status
  rather than trusting the reported hash.
- Read the actual implementation diff (not just `--stat` or file headers) for
  unrequested refactors, unused branches, truncated comments, leaked PR
  numbers, hardcoded test/placeholder assumptions, or monolithic procedural
  functions that will be hard to maintain.
- Check commit messages for internal workflow jargon and required sign-offs
  (`Signed-off-by`).
- Run real tests and repository guards on the target environment (including
  root typechecks/lints that a narrow subset check may miss), and write all
  PR descriptions, review replies, and issue comments yourself.
