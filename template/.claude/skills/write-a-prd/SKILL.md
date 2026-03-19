---
name: write-a-prd
description: Transforms a conversation or idea into a Product Requirements Document (PRD) with user stories, then submits it as a Forgejo issue. Invoke with /write-a-prd when ready to formalise a feature or initiative.
context: fork
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - Edit
---

# write-a-prd

Converts a raw idea or conversation into a structured PRD, grounded in the
actual codebase, then submits it as a Forgejo issue for tracking.

## When to use

Invoke with `/write-a-prd` when:
- A feature or initiative needs formal requirements before work begins
- You have a rough idea and want to produce something actionable
- Stakeholders need a written spec to review or approve

## Inputs

- **Idea or context** (required): A description of what you want to build.
  Can be vague — the interview will sharpen it.
- **Forgejo repo** (optional): Defaults to the remote inferred from `git remote get-url origin`.

## Steps

### 0. Verify Forgejo connectivity

Follow the **Forgejo API** procedure in `protocols.md` (steps 1–3) to resolve
`$FORGEJO_URL`, `$FORGEJO_TOKEN`, `$OWNER`, and `$REPO`. Stop if any step
fails — do not attempt issue creation with missing or invalid credentials.

### 1. Gather a detailed description

Ask the user to describe the feature or initiative in their own words. Capture:
- The problem being solved
- Who benefits and how
- Any constraints or non-goals already known

### 2. Explore the codebase

Load L0 (`.abstract.md`) and L1 (`.overview.md`) to understand the project
structure. For areas directly relevant to the feature, read L2 source files.
Verify or correct any claims the user made about how the system currently works.

### 3. Conduct a design interview

Run a focused interview (see `/grill-me` approach) to resolve ambiguities:
- What are the acceptance criteria?
- What user stories cover this feature?
- Are there edge cases, error states, or rollback scenarios to handle?
- Does this touch existing APIs, data models, or auth flows?

Stop when all open questions are resolved.

### 4. Sketch major modules

Identify which parts of the codebase will change and how:
- New files or directories
- Modified interfaces or schemas
- New dependencies
- Migration or rollout considerations

### 5. Draft the PRD

Produce a structured document:

```markdown
# PRD: <Feature Name>

## Problem
<What problem does this solve? Who is affected?>

## Goals
- <Measurable goal 1>
- <Measurable goal 2>

## Non-goals
- <What this does NOT include>

## User stories
- As a <role>, I want to <action> so that <benefit>.

## Acceptance criteria
- [ ] <Criterion 1>
- [ ] <Criterion 2>

## Technical scope
<Which subsystems change and how. Reference files from .overview.md.>

## Open questions
- <Any unresolved decisions — should be empty before submission>
```

### 6. Submit as a Forgejo issue

Use the **Create an issue** pattern from the Forgejo API section of
`protocols.md`, with `title = "PRD: <Feature Name>"` and `body = <prd content>`.
Print the returned `.html_url` when done.

## Expected output

A Forgejo issue containing the full PRD, with all acceptance criteria defined
and open questions resolved. The issue URL is printed at the end.
