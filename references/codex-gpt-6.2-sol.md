# Codex (`gpt-6.2-sol`)

High-rigor, cost-effective worker for bounded coding, conflict resolution, and
code review. Give it verified facts, tight scope, and right-sized reasoning
effort; audit every line it returns; keep all human-facing writing with the
coordinator.

## Profile

- **Strengths**: deep codebase tracing, multi-file rebase conflict resolution,
  isolated backend/algorithm work, static-contract registration, and catching
  subtle bugs in code review.
- **Failure modes to guard against**:
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
- For a git worktree, grant access to the main repository `.git` directory and
  the briefs directory via `--add-dir`, and omit draft directories it should
  not touch.
- Launch without an initial task argument first so startup dialogs (such as
  `Update available` or folder trust) can be dismissed cleanly before sending
  the brief.

## Auditing output

Before accepting a commit or patch from `gpt-6.2-sol`:

- Confirm the exact commit hash (`git rev-parse HEAD`) and working tree status
  rather than trusting the reported hash.
- Read the full diff for unrequested refactors, unused branches, truncated
  comments, or leaked PR numbers.
- Check commit messages for internal workflow jargon and required sign-offs
  (`Signed-off-by`).
- Run real tests and repository guards on the target environment, and write all
  PR descriptions, review replies, and issue comments yourself.
