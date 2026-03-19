---
name: prd-to-issues
description: Converts a PRD (Forgejo issue or local file) into independently executable Forgejo issues organised as vertical slices through integration layers. Invoke with /prd-to-issues <issue-url-or-path>.
context: fork
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# prd-to-issues

Breaks a PRD into a set of Forgejo issues where each issue is a vertical slice
— a thin end-to-end increment that exposes integration unknowns early and can
be merged independently.

## When to use

Invoke with `/prd-to-issues` after a PRD has been approved and you are ready
to begin implementation planning. Requires a PRD in a Forgejo issue or local
file.

## Inputs

- **PRD location** (required): A Forgejo issue URL or a local file path.
  Example: `/prd-to-issues https://forgejo.example.com/org/repo/issues/42`
  Example: `/prd-to-issues docs/prd-feature-x.md`

## Steps

### 0. Verify Forgejo connectivity

Follow the **Forgejo API** procedure in `protocols.md` (steps 1–3) to resolve
`$FORGEJO_URL`, `$FORGEJO_TOKEN`, `$OWNER`, and `$REPO`. Stop if any step
fails — do not attempt issue creation with missing or invalid credentials.

### 1. Locate and fetch the PRD

If given a Forgejo issue URL, parse the issue number from the URL and use the
**Fetch an issue** pattern from `protocols.md`. Extract `.title` and `.body`
from the response.

If given a local file path, read it directly.

### 2. Explore the codebase

Load L0 (`.abstract.md`) and L1 (`.overview.md`). Identify which subsystems
the PRD touches. Read L2 source files only for interfaces or schemas the slices
must implement against.

### 3. Identify vertical slices

A vertical slice is a thin increment that:
- Delivers one user-facing behaviour end-to-end (data → logic → API → UI if applicable)
- Can be merged independently without breaking the main branch
- Exposes integration risk early (prefer slices that cross layer boundaries)
- Has clear, verifiable acceptance criteria

Draft a slice list from the PRD's user stories and acceptance criteria:
- Order slices to expose unknowns as early as possible
- Each slice should be completable in a single focused session
- Mark dependencies between slices if sequencing is required

### 4. Create Forgejo issues

For each slice, create one issue:

Use the **Create an issue** pattern from `protocols.md` for each slice. The
body should follow this structure:

```
## Slice: <title>

**Parent PRD:** <issue URL or file path>

### What this slice delivers
<One paragraph — the specific behaviour this slice adds, end-to-end.>

### Acceptance criteria
- [ ] <Criterion 1>
- [ ] <Criterion 2>

### Files likely touched
<List from .overview.md — be specific.>

### Dependencies
<Other slice issues this depends on, if any. "None" if independent.>
```

Create slices sequentially so each issue is confirmed before the next is started.
If a slice's dependencies are already-created issues, add their URLs to the
body of the dependent slice.

### 5. Report

Print a summary table:

```
Slice #  Title                        Issue URL
───────  ───────────────────────────  ──────────────────────────────────────────
1        <title>                      https://forgejo.example.com/.../issues/43
2        <title>                      https://forgejo.example.com/.../issues/44
```

Note any slices with unresolved ambiguity that may need clarification before
work begins.

## Expected output

One Forgejo issue per vertical slice, with acceptance criteria, file scope, and
dependency links. A summary table printed at the end listing all created issues.
