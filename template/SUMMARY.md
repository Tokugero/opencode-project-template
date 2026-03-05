# SUMMARY.md — <project-name>

**Read this file first.** It gives any agent enough context to orient in the
project without reading all source files. Subsystems each have their own
`<subsystem>/SUMMARY.md` for deeper detail.

---

## What this project is

<!-- One paragraph: what this project does, why it exists, who uses it -->

---

## Repository map

| Subsystem | Path | Role |
|-----------|------|------|
| <!-- fill in --> | `<subsystem>/` | ... |

---

## Subagent quick-reference

| Task type | First agent to invoke |
|-----------|----------------------|
| Runtime error / crash / metrics | `@<project>-sre` |
| Infra / env / secrets / CI | `@<project>-devops` |
| <subsystem> code or behaviour | `@<project>-<role>` |
| Git operations | `@git-flow` |

---

## Service endpoints

| Service | URL | Notes |
|---------|-----|-------|
| <!-- fill in --> | http://... | |

---

## Stack at a glance

| Layer | Technology |
|-------|-----------|
| <!-- fill in --> | ... |

---

## Dev workflow

```sh
# Run tests
# <command>

# Start local stack
# <command>

# Apply a change
# <command>
```

---

## Storage / persistence

<!-- Describe any persistent state: databases, volumes, files, caches -->

---

## Non-negotiables

<!-- Project-wide rules that must always hold -->
- Never commit unencrypted secrets
- Never use floating/latest versions

---

## Subsystem SUMMARY.md files

Each subsystem has its own `SUMMARY.md` with deeper detail. Read the
relevant one before working inside that subsystem:

| Subsystem | File |
|-----------|------|
| <!-- fill in --> | `<subsystem>/SUMMARY.md` |
