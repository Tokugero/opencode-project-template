---
name: sdlc-review
description: >
  Read-only codebase review using parallel audit agents. Produces
  consolidated findings and exceptions in docs/.tmp/ without making
  any code changes.
argument-hint: "[scope: path/to/component,...] [type: security|code|both]"
user-invocable: true
allowed-tools: Bash(mkdir *), Read, Grep, Glob, Write(docs/.tmp/*), Task, TeamCreate, TeamDelete, SendMessage, TaskCreate, TaskUpdate, TaskList, TaskGet
---

# SDLC Review (Read-Only)

A read-only codebase review that produces consolidated findings and
recommendations without touching source code, creating commits, or making
any changes outside of `docs/.tmp/`.

Parse the user's arguments first:
- If a `scope` argument is present (comma-separated paths), limit the review
  to those components only. If omitted, treat the entire repo as in scope.
- If a `type` argument is present, use it to select which audit agents to
  spawn: `security` (security-audit agents only), `code` (code-review agents
  only), or `both` (default, spawn both types).

---

## Before starting

1. Check whether `docs/.tmp` is listed in `.gitignore` at the repo root.
   Run `grep -r 'docs/.tmp' .gitignore` to verify. If it is missing, warn
   the user and run `echo 'docs/.tmp/' >> .gitignore` to add it.
2. Run `mkdir -p docs/.tmp` to ensure the output directory exists.
3. Read `.claude/settings.json`. Verify that background agents will have
   the permissions they need. Review agents (`code-review`, `security-audit`)
   need at minimum: `Read(*)`, `Glob(*)`, `Grep(*)`, `Write(docs/.tmp/*)`.
   If any required permissions are absent, warn the user before spawning
   agents — background agents cannot prompt for permissions and will silently
   fail without them.

---

## Phase 1: Parallel Audit (read-only)

1. Identify the components to audit. If `scope` was provided, use those paths.
   Otherwise, enumerate the top-level directories or logical components of the
   repo.
2. Create a team via TeamCreate. Use 3 reviewers unless the user specified a
   different count. Name the team `sdlc-review`.
3. Announce to the user which model and approach each reviewer will use before
   spawning them. Spawn agent types according to the `type` argument:
   - `security` — spawn only `security-audit` agents, pinned to Opus.
   - `code` — spawn only `code-review` agents, pinned to Opus.
   - `both` (default) — spawn a mix of `code-review` and `security-audit`
     agents, pinned to Opus.
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

Spawn a single consolidation agent (Opus, general-purpose). Pass it the
following instructions:

1. Read every file in `docs/.tmp/` from Phase 1.
2. Produce exactly two output files in `docs/.tmp/`:
   - `consolidated-fix-list.md` — findings recommended for fixing, grouped
     by component. Label each entry as a **recommendation**, not a required
     action item. Include severity (Critical / High / Medium / Low) and
     enough context for a human to act on it manually.
   - `consolidated-exceptions.md` — findings recommended to skip or accept
     as-is, each with a rationale.
3. Apply the triage rules below to decide which output each finding goes into.
4. Deduplicate cross-source findings: if the same issue appears in both a
   `code-review` and a `security-audit` report, produce one entry in the output
   and annotate it `Source: both`.
5. Cross-reference overlapping file/function findings — note co-location
   opportunities so a human or future `/sdlc-audit fix` run can handle them
   together.
6. After writing the two final files, delete all intermediate per-agent report
   files from `docs/.tmp/`. Only `consolidated-fix-list.md` and
   `consolidated-exceptions.md` should remain when done.
7. Every finding from Phase 1 must appear in exactly one of the two output
   files. Nothing may be silently dropped.

### Triage rules

| Category | Output |
|----------|--------|
| Real bugs (wrong IDs, silent failures, broken imports) | FIX recommendation |
| DRY violations (duplicated logic across services) | FIX recommendation |
| Dead code, dangling functions | FIX recommendation |
| Security issues regardless of environment | FIX recommendation |
| Large file decomposition | FIX recommendation |
| Dev-only QoL (swagger exposure, localhost URLs, debug output) | EXCEPTION |
| Test coverage gaps | EXCEPTION (note for prioritization) |
| Prod-specific hardening handled by infra | EXCEPTION |
| Intentional duplication (documented architectural reason) | EXCEPTION (document why) |

---

## Summary output

After consolidation completes, print a summary to the user:

1. Total findings count across all components.
2. Breakdown: N items recommended for fixing, M items excepted.
3. Top 3 highest-severity findings (by severity level, then component name).
4. Remind the user that all details are in `docs/.tmp/consolidated-fix-list.md`
   and `docs/.tmp/consolidated-exceptions.md`.

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
   reading source code for analysis.

---

## Explicit: no code changes

This skill is read-only. It does not modify source code, create commits, or
make any changes outside of `docs/.tmp/`. To act on the findings, run
`/sdlc-audit fix` or address them manually.
