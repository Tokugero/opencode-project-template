---
name: example-skill
description: One-sentence description of what this skill does and when to invoke it via /example-skill.
context: fork
allowed-tools:
  - Bash
  - Read
# disable-model-invocation: true  # uncomment to prevent automatic invocation
---

# <skill-name>

Brief description of what this skill accomplishes and what problem it solves.

> **Note:** Skills run in an isolated context (`context: fork`). They do not
> share the parent session's conversation history. Pass all required context
> in the task description when invoking.

## When to use

One sentence — what condition triggers this skill. Invoked with `/<skill-name>`.

## Inputs

Describe what information this skill needs to run. The user or orchestrator
should provide these when invoking:

- `<input-1>`: what it is and where to find it
- `<input-2>`: what it is and where to find it

## Steps

```sh
# Step 1: describe what this does
<command>

# Step 2: describe what this does
<command>
```

## Supporting files

Skills can include supporting files in this directory alongside `SKILL.md`:

- `example-resource.md` — shows the pattern; replace with real resources
- Scripts, templates, lookup tables, or reference data the skill needs at runtime

Reference supporting files with relative paths: `./example-resource.md`.

## Expected output

What a successful invocation produces. Be specific so the user knows when the
skill completed correctly.
