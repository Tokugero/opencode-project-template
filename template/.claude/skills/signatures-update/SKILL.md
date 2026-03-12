---
name: signatures-update
description: Refresh function_signatures.md and SUMMARY.md across all subsystems. Diff mode (default) updates only files modified since the last index; regeneration mode rewrites everything. Invoke with /signatures-update when indexes may be stale, before running the large task planning protocol, or after significant refactors.
context: fork
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Task
---

# signatures-update

Keeps `function_signatures.md` and `SUMMARY.md` accurate across all subsystems.
These files are the primary orientation layer for agents — the planning and
execution protocol depends on them being complete and current.

## When to use

Invoke with `/signatures-update` when:
- About to run the large task planning and execution protocol
- After a significant refactor, file rename, or new subsystem added
- Indexes haven't been updated in a while and you suspect drift
- A domain agent reported a gap between the index and reality

## Inputs

- **Mode** (optional): `diff` (default) or `regen`
  - `diff` — update only files modified since the last index write
  - `regen` — full scrape of every subsystem; use when drift is extensive
    or after large-scale restructuring

## Steps

### 1. Discover subsystems

Find every subsystem that owns a `function_signatures.md`:

```bash
find . -name "function_signatures.md" -not -path "./.git/*"
```

Each directory containing one is a subsystem. If none exist, all subsystems
are new — treat every subsystem as requiring full regeneration regardless
of mode.

### 2. Triage dirty subsystems (diff mode only)

For each subsystem directory, find source files modified more recently than
its `function_signatures.md`:

```bash
find <subsystem>/ -newer <subsystem>/function_signatures.md \
  -type f -not -path "*/.git/*" -not -name "*.md"
```

If the dirty file list is empty, skip this subsystem — nothing to update.
If `function_signatures.md` does not exist for a subsystem, treat it as
fully dirty (regenerate that subsystem).

In `regen` mode, skip triage — all subsystems are dirty.

### 3. Print dry-run summary before writing

Before spawning any agents, output a table:

```
Subsystem       Mode    Dirty files
──────────────  ──────  ───────────
<subsystem-a>   diff    4
<subsystem-b>   diff    0 (skip)
<subsystem-c>   regen   all
```

Ask the user to confirm before proceeding if the total dirty file count
is large (>20 files or >3 subsystems in regen mode).

### 4. Spawn parallel Sonnet agents per dirty subsystem

For each subsystem that needs updating, spawn one Sonnet agent (via Task,
model: sonnet). Pass it:
- The subsystem path
- The mode (diff or regen)
- The dirty file list (diff mode) or instruction to read all files (regen)
- The current contents of `function_signatures.md` (diff mode — for patching)

Each agent's job:
- **Diff mode**: read only the dirty files; update or add entries for those
  files in `function_signatures.md`; leave all other entries unchanged.
- **Regen mode**: read all source files in the subsystem; produce a complete
  rewrite of `function_signatures.md`.

`function_signatures.md` format per entry:
```
### <filename>
- **Type**: <module | component | service | schema | config | test | ...>
- **Purpose**: one-sentence description
- **Exports / key symbols**: comma-separated list of functions, classes,
  types, or constants that other files import
- **Dependencies**: files or packages this file imports from
- **Last updated**: YYYY-MM-DD
```

All agents write directly to their own subsystem's `function_signatures.md`
— no shared output files, no collision risk.

### 5. Update SUMMARY.md

After all subsystem agents complete, spawn a single Sonnet agent to rewrite
the project-level `SUMMARY.md`. It reads all updated `function_signatures.md`
files and the existing `SUMMARY.md`, then produces a revised summary that
reflects the current subsystem structure, major components, and
cross-subsystem dependencies.

`SUMMARY.md` should remain concise — a navigator, not a full reference.
Target: under 150 lines. If it grows beyond that, it has drifted into
documentation territory and should be pruned back to orientation facts.

### 6. Report

Print a completion summary:
- Subsystems updated and file counts
- Whether SUMMARY.md was rewritten
- Any subsystems skipped (no dirty files)
- Any gaps or ambiguities the agents flagged during scraping

## Expected output

- `function_signatures.md` updated in every dirty subsystem
- `SUMMARY.md` rewritten to reflect current state
- A brief summary printed confirming what changed and what was skipped
