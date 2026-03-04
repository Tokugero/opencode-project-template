# <subsystem> — SUMMARY.md

Subagent working in this subsystem: read this file and `function_signature.md`
before starting any task.

## What this subsystem is

<!-- One paragraph: what this subsystem does, why it exists, who uses it -->

---

## function_signature.md

`function_signature.md` in this directory is the authoritative map of every
file and component in this subsystem. Consult it to find where a thing lives
before reading source files. The owning subagent keeps it current.

---

## Component table

| Component | Path | Role |
|-----------|------|------|
| <!-- fill in --> | `<subsystem>/...` | ... |

---

## Service endpoints

| Service | URL | Notes |
|---------|-----|-------|
| <!-- fill in --> | http://... | |

---

## Dev workflow

```sh
# <!-- fill in common dev commands -->
# Example:
# Run tests
# <command>

# Apply a change
# <command>

# Restart a service
# <command>
```

---

## Storage / persistence

<!-- Describe any persistent state: databases, PVCs, files, caches -->

---

## Non-negotiables

<!-- Project-specific rules that must always hold in this subsystem -->
- Never commit unencrypted secrets
- Never use floating/latest versions
