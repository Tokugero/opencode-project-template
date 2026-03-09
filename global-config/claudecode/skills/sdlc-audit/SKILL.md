---
name: sdlc-audit
description: >
  Repeatable codebase audit & fix cycle using parallel agent teams.
  Phases: audit, consolidate, fix, validate. All output stays in
  docs/.tmp/ (gitignored, never committed).
argument-hint: "[phase: audit|consolidate|fix|validate|full] [scope: path/to/component,...]"
user-invocable: true
allowed-tools: Bash(*), Read, Grep, Glob, Write(docs/.tmp/*), Task, TeamCreate, TeamDelete, SendMessage, TaskCreate, TaskUpdate, TaskList, TaskGet
---

# SDLC Audit & Fix Cycle

A repeatable cycle that takes a codebase from "unknown state" to "audited,
fixed, and validated" using parallel agent teams with structured output
collation.

Parse the user's arguments first:
- If the first argument is one of `audit`, `consolidate`, `fix`, `validate`,
  or `full`, treat it as the phase selector. Default to `full` if omitted.
- If a `scope` argument is present (comma-separated paths), limit all phases
  to those components only. If omitted, treat the entire repo as in scope.

---

## Before starting any phase

1. Check whether `docs/.tmp` is listed in `.gitignore` at the repo root.
   Run `grep -r 'docs/.tmp' .gitignore` to verify. If it is missing, warn
   the user and run `echo 'docs/.tmp/' >> .gitignore` to add it.
2. Run `mkdir -p docs/.tmp` to ensure the output directory exists.
3. Read `.claude/settings.json`. Verify that background agents will have
   the permissions they need:
   - Review agents (`code-review`, `security-audit`) need at minimum:
     `Read(*)`, `Glob(*)`, `Grep(*)`, `Write(docs/.tmp/*)`.
   - Dev agents (fix sweep, refactor) also need: `Edit(*)`, `Bash(*)`.
   If any required permissions are absent, warn the user before spawning
   agents — background agents cannot prompt for permissions and will silently
   fail without them.

---

## Phase 1: Parallel Audit (read-only)

Run this phase when `phase` is `audit` or `full`.

1. Identify the components to audit. If `scope` was provided, use those paths.
   Otherwise, enumerate the top-level directories or logical components of the
   repo.
2. Create a team via TeamCreate. Use 3 reviewers unless the user specified a
   different count. Name the team `sdlc-audit`.
3. Announce to the user which model and approach each reviewer will use before
   spawning them. Use `code-review` and/or `security-audit` agent types, pinned
   to Opus.
4. Create one TaskCreate entry per component per audit type (e.g., one task for
   `code-review` of `src/api/`, one for `security-audit` of `src/api/`, etc.).
5. Assign tasks to reviewers so each reviewer owns 2-3 components. Use
   TaskUpdate to set the owner for each task.
6. Each reviewer must:
   a. Claim their first task (TaskUpdate status to `in_progress`).
   b. Read the component source thoroughly.
   c. Write all findings to `docs/.tmp/<agent-name>-<component>-<audit-type>.md`.
      Use the repo root as the base for this path.
   d. Mark the task completed (TaskUpdate status to `completed`).
   e. Call TaskList to find the next unclaimed task and repeat until none remain.
7. Wait for all reviewers to report completion. Then send each reviewer a
   shutdown_request via SendMessage and call TeamDelete once all have shut down.

Agent type matters for permissions. If `.claude/settings.json` does not allow
the tools those agent types require, they will stall silently. Verify before
spawning.

---

## Phase 2: Consolidation & Triage

Run this phase when `phase` is `consolidate` or `full`.

Spawn a single consolidation agent (Opus, general-purpose). Pass it the
following instructions:

1. Read every file in `docs/.tmp/` from Phase 1.
2. Produce exactly two output files in `docs/.tmp/`:
   - `consolidated-fix-list.md` — actionable findings grouped by component.
   - `consolidated-exceptions.md` — findings to skip, each with a rationale.
3. Apply the triage rules below to decide which output each finding goes into.
4. Deduplicate cross-source findings: if the same issue appears in both a
   `code-review` and a `security-audit` report, produce one entry in the output
   and annotate it `Source: both`.
5. Cross-reference overlapping file/function findings — note co-location
   opportunities so the fix sweep can handle them together in Phase 3.
6. After writing the two final files, delete all intermediate per-agent report
   files from `docs/.tmp/`. Only `consolidated-fix-list.md` and
   `consolidated-exceptions.md` should remain when done.
7. Every finding from Phase 1 must appear in exactly one of the two output
   files. Nothing may be silently dropped.

### Triage rules

| Category | Output |
|----------|--------|
| Real bugs (wrong IDs, silent failures, broken imports) | FIX |
| DRY violations (duplicated logic across services) | FIX |
| Dead code, dangling functions | FIX |
| Security issues regardless of environment | FIX |
| Large file decomposition | FIX |
| Dev-only QoL (swagger exposure, localhost URLs, debug output) | EXCEPTION |
| Test coverage gaps | EXCEPTION (note for prioritization) |
| Prod-specific hardening handled by infra | EXCEPTION |
| Intentional duplication (documented architectural reason) | EXCEPTION (document why) |

---

## Gate: Confirm before code changes

This gate runs before Phase 3 whenever `phase` is `fix` or `full`, and also
when `fix` is invoked as a standalone phase. It is a hard stop — do NOT
proceed past this point without explicit user approval.

1. Verify that `docs/.tmp/consolidated-fix-list.md` exists. If it does not,
   abort and tell the user to run the `consolidate` phase first.
2. Read `docs/.tmp/consolidated-fix-list.md` and count:
   - Total number of actionable findings.
   - Distinct components (directories/files) affected.
3. Present a concise summary to the user, for example:
   ```
   Fix sweep summary
   -----------------
   Findings to fix     : <N>
   Affected components : <list>
   Branch to create    : sdlc-audit/<YYYY-MM-DD>
   ```
4. Use `AskUserQuestion` to ask explicitly:
   > "The fix phase will modify source code and create commits on a new branch.
   > Are you sure you want to proceed? (yes/no)"
5. If the user answers anything other than an unambiguous "yes", stop
   immediately. Remind them that findings are available in `docs/.tmp/` for
   manual review, specifically `docs/.tmp/consolidated-fix-list.md` and
   `docs/.tmp/consolidated-exceptions.md`.
6. If the user approves, create the audit branch exactly once — do not repeat
   this step per-phase:
   ```
   git checkout -b sdlc-audit/$(date +%Y-%m-%d)
   ```
   Confirm to the user that the branch was created and that all fix commits
   will land on it. All fix phases (Phase 3, Phase 5, Phase 6) operate on
   this branch. Do not switch branches between phases.

---

## Phase 3: Fix Sweep (parallel dev teams)

Run this phase when `phase` is `fix` or `full` (after the gate above has been
passed and the branch has been created).

Verify that `docs/.tmp/consolidated-fix-list.md` exists before proceeding. If
it does not, abort and tell the user to run the `consolidate` phase first.

1. Create a team via TeamCreate. Use 3 dev agents (Sonnet minimum,
   general-purpose) unless the user specified otherwise. Name the team
   `sdlc-fix`.
2. Announce to the user which model and approach each dev agent will use.
3. Read `docs/.tmp/consolidated-fix-list.md` and partition the findings into
   groups. Group by **code path**, not by service boundary. If a fix touches
   the API layer, the database layer, and an MCP handler, all three belong to
   the same group and must go to the same agent. Never split a cross-cutting
   change across agents — they will produce merge conflicts.
4. Create one task per group via TaskCreate and assign to dev agents via
   TaskUpdate.
5. Each dev agent must:
   a. Claim their task (TaskUpdate status to `in_progress`).
   b. Implement all fixes in the group. All commits go on the current branch
      (`sdlc-audit/<date>` created in the gate above) — do not create or
      switch branches.
   c. Commit each fixed issue individually with a conventional commit:
      ```
      fix(scope): description

      Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
      ```
   d. Write a summary of what was fixed to `docs/.tmp/fixed-<agent-name>.md`.
   e. Write any findings that turned out to be non-issues or required exceptions
      to `docs/.tmp/new-exceptions-<agent-name>.md`.
   f. Mark the task completed (TaskUpdate status to `completed`).
   g. Call TaskList to find the next unclaimed task and repeat.
6. Wait for all dev agents to report completion. Send shutdown_request to each
   and call TeamDelete once all have shut down.

---

## Phase 4: Security Validation (read-only)

Run this phase when `phase` is `validate` or `full`.

Verify that `docs/.tmp/fixed-*.md` files exist before proceeding. If they do
not, abort and tell the user to run the `fix` phase first.

1. Create a team via TeamCreate. Use 3 `security-audit` agents (Opus) unless
   the user specified otherwise. Name the team `sdlc-validate`.
2. Announce to the user which model and approach each validator will use.
3. Create one task per component or fix group via TaskCreate. Each validator
   must:
   a. Claim a task (TaskUpdate status to `in_progress`).
   b. Read the fix description from the relevant `docs/.tmp/fixed-*.md` summary.
   c. Read the ACTUAL SOURCE CODE for the fixed component — do not rely only
      on the summary description. Check for bypass vectors the fix may have
      missed.
   d. Read the original security findings from Phase 1 (if they still exist) or
      the `docs/.tmp/consolidated-fix-list.md` for context.
   e. Assess any excepted findings from `docs/.tmp/consolidated-exceptions.md`
      — verify the risk assessment is accurate.
   f. Write a verdict for each finding to `docs/.tmp/sec-val-<scope>.md`.
      Valid verdicts are:
      - `VALIDATED` — fix closes the vulnerability.
      - `INSUFFICIENT` — fix is incomplete; describe the gaps.
      - `RISK ACCEPTED` — finding was excepted and the risk assessment is sound.
   g. Mark the task completed (TaskUpdate status to `completed`).
   h. Call TaskList for the next task and repeat.
4. Wait for all validators to report completion. Send shutdown_request to each
   and call TeamDelete once all have shut down.

"Resolved" in a summary does not mean fixed. Always verify against the actual
source code, not the description.

---

## Phase 5: Fix Insufficient Findings

Run this phase automatically after Phase 4 when `phase` is `full`.

1. Scan all `docs/.tmp/sec-val-*.md` files for any verdict of `INSUFFICIENT`.
2. If none exist, skip this phase and tell the user validation passed cleanly.
3. If any `INSUFFICIENT` verdicts exist, collect them and dispatch targeted fix
   tasks to a small team (1-2 dev agents, Sonnet, general-purpose). Name the
   team `sdlc-remediate`.
4. These are usually small, focused fixes. Apply the same per-fix commit
   discipline from Phase 3. All commits go on the same `sdlc-audit/<date>`
   branch — do not switch branches.
5. After fixes are committed, re-run Phase 4 (security validation) scoped to
   the remediated components only.
6. Repeat until no `INSUFFICIENT` verdicts remain.

---

## Phase 6: Full Refactor (optional)

Only run this phase if the user explicitly requests it by name. Do not run it
as part of `full`.

This phase is for findings in `docs/.tmp/consolidated-exceptions.md` that were
excepted as architectural — large decompositions, intentional duplication to be
resolved, etc.

1. Create one team (Opus, general-purpose, N leads as requested). Name the team
   `sdlc-refactor`.
2. Announce model and approach before spawning.
3. Each lead works sequentially through a phased plan:
   - Phase A: decomposition (split large files, extract modules).
   - Phase B: feature changes (behavior modifications, API redesign).
   - Phase C: miscellaneous fixes (naming, dead code, minor DRY).
4. Run tests after EACH phase. Do not proceed to the next phase if tests fail.
5. Commit after EACH fixed issue using the same conventional commit format.
   All commits go on the same `sdlc-audit/<date>` branch — do not switch
   branches.
6. Breaking changes are acceptable only if there are no external consumers.
   Document this explicitly in the commit message if applicable.

---

## Phase 7: Final Validation

Run this phase at the end of `full`, or when the user requests it explicitly.

1. Run all test suites for every component in scope. Report pass/fail per suite.
2. If any tests fail, stop and report to the user — do not proceed.
3. If applicable, instruct the user to restart services locally and verify
   runtime behavior.
4. Produce a final summary in `docs/.tmp/final-report.md`:
   - List of all fixes applied (from `docs/.tmp/fixed-*.md`).
   - List of all exceptions (from `docs/.tmp/consolidated-exceptions.md` and
     `docs/.tmp/new-exceptions-*.md`).
   - Security validation verdicts (from `docs/.tmp/sec-val-*.md`).
   - Test suite results.
5. Remind the user to review all changes on the audit branch before merging:
   ```
   git diff main...HEAD
   ```
   The audit branch (`sdlc-audit/<date>`) stays checked out. Merging to main
   is a manual step — do not merge automatically.

---

## Collation protocol (all phases)

Follow these rules in every phase without exception:

1. All output stays in `docs/.tmp/` — this directory is gitignored and
   ephemeral. Never write findings to `docs/`, the repo root, or any tracked
   location.
2. Each agent writes to `docs/.tmp/<agent-name>-<output-type>.md`. Use the
   repo root as the base for all paths.
3. The orchestrating skill (this file) concatenates within `docs/.tmp/` when
   collation is needed.
4. The consolidation agent deduplicates and normalizes, then deletes all
   intermediate files — only the final deliverables remain.
5. Never `cd` into `docs/.tmp/`. Always use absolute paths or paths relative
   to the repo root (e.g., `docs/.tmp/foo.md`).
6. Never write, read, or delete any file outside `docs/.tmp/` except when
   reading source code for analysis or writing commits via git.
