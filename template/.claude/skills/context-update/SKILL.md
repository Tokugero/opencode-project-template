---
name: context-update
description: Refresh .abstract.md (L0) and .overview.md (L1) across all subsystems. Diff mode (default) updates only subsystems modified since the last index; regeneration mode rewrites everything. Invoke with /context-update when indexes may be stale, before running the large task planning protocol, or after significant refactors.
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

# context-update

Keeps `.overview.md` (L1) and `.abstract.md` (L0) accurate across all subsystems.
These files are the primary orientation layer for agents — the planning and
execution protocol depends on them being complete and current.

## When to use

Invoke with `/context-update` when:
- About to run the large task planning and execution protocol
- After a significant refactor, file rename, or new subsystem added
- Indexes haven't been updated in a while and you suspect drift
- A domain agent reported a gap between the index and reality

## Inputs

- **Mode** (optional): `diff` (default) or `regen`
  - `diff` — update only subsystems modified since the last index write
  - `regen` — full scrape of every subsystem; use when drift is extensive
    or after large-scale restructuring

## Steps

### 1. Discover subsystems

Find every subsystem that owns a `.overview.md`:

```bash
find . -name ".overview.md" -not -path "./.git/*"
```

Each directory containing one is a subsystem. If none exist, all subsystems
are new — treat every subsystem as requiring full regeneration regardless
of mode.

### 2. Triage dirty subsystems (diff mode only)

For each subsystem directory, find source files modified more recently than
its `.overview.md`:

```bash
find <subsystem>/ -newer <subsystem>/.overview.md \
  -type f -not -path "*/.git/*" -not -name "*.md"
```

If the dirty file list is empty, skip this subsystem — nothing to update.
If `.overview.md` does not exist for a subsystem, treat it as
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
- The current contents of `.overview.md` (diff mode — for patching)

Each agent's job:

1. **AST skeleton extraction (optional, supported languages only):** For
   Python, JS/TS, Rust, Go, Java, C/C++, extract signatures using grep-based
   patterns before reading full files. This reduces tokens consumed:
   - Python: `grep -n "^def \|^class \|^async def "`
   - JS/TS: `grep -n "^export \|^function \|^class \|^const \|^interface \|^type "`
   - Rust: `grep -n "^pub fn \|^pub struct \|^pub enum \|^pub trait \|^fn "`
   - Go: `grep -n "^func \|^type \|^var \|^const "`
   - Java: `grep -n "public \|private \|protected \|class \|interface \|enum "`
   - C/C++: `grep -n "^[a-zA-Z].*(.*)"`
   Fallback to full-file read for unsupported languages or when signatures
   are insufficient.

2. **Produce `.overview.md` (L1)** — merged format:
   - Overview paragraph: what this subsystem as a whole does
   - Component table (one row per file or directory):
     | Path | Type | Purpose | Exports | Deps |
     |------|------|---------|---------|------|
   - Endpoints (if applicable)
   - Dev workflow (if applicable)
   Token budget: ~2 000 tokens.

3. **Produce `.abstract.md` (L0)** — derived from L1:
   - One line per component: path + one-sentence purpose
   - Table format: `| Path | Purpose |`
   Token budget: ~100 tokens.

All agents write directly to their own subsystem's `.overview.md` and
`.abstract.md` — no shared output files, no collision risk.

### 5. Update root context files

After all subsystem agents complete, spawn a single Sonnet consolidation agent
to produce the root `.overview.md` and `.abstract.md`. It reads all updated
subsystem `.overview.md` files and produces:

- **Root `.overview.md`** — project-wide L1: what this project is, repository
  map (all subsystems), subagent quick-reference, service endpoints, stack at
  a glance, dev workflow, storage, non-negotiables. Target: under 150 lines.
- **Root `.abstract.md`** — project-wide L0: one line per subsystem, derived
  from the repository map. Target: under 20 lines.

### 6. Report

Print a completion summary:
- Subsystems updated and file counts
- Whether root context files were rewritten
- Any subsystems skipped (no dirty files)
- Any gaps or ambiguities the agents flagged during scraping

## Expected output

- `.overview.md` and `.abstract.md` updated in every dirty subsystem
- Root `.overview.md` and `.abstract.md` rewritten to reflect current state
- A brief summary printed confirming what changed and what was skipped
