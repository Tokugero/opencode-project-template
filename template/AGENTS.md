# AGENTS.md — <project-name>

## Purpose

This is the **orchestration layer** for the <project-name> repository.
It describes how the repo is structured, which subagents handle which concerns,
and the safety gates that require human approval.

Subagents working inside a specific subsystem should read their own
`SUMMARY.md` or agent file first. Read this file only when cross-cutting
coordination is needed.

---

## Table of Contents

- [Repository Structure](#repository-structure)
- [Scope](#scope)
- [Subagent Roster](#subagent-roster)
- [Delegation Rules](#delegation-rules)
- [Secrets Protocol](#secrets-protocol)
- [Code Style Guidelines](#code-style-guidelines)
- [Permission Gates](#permission-gates)
- [Session Log](#session-log)

---

## Repository Structure

```
<project>/
├── opencode.json               ← MCP server config + default_agent
├── AGENTS.md                   ← THIS FILE — orchestration dispatch table
├── CHANGELOG.md                ← template version history
├── .template-local             ← gitignored: local template-version + template-path
├── .opencode/
│   └── agents/                 ← project-scoped subagent definitions
│       ├── <project>-orchestrator.md ← primary orchestrator (mode: primary)
│       ├── <project>-sre.md    ← read-only runtime investigation
│       ├── <project>-<role>.md ← per-subsystem agents
│       └── ...
├── kb/                         ← knowledge base: SOPs and runbooks
│   └── README.md               ← KB index table
├── docs/
│   └── sre-todos.md            ← deferred SRE findings
│
└── <subsystem>/                ← one directory per major subsystem
    ├── SUMMARY.md              ← subsystem context for subagents
    └── function_signature.md   ← file/component map; owned by subsystem agent
```

---

## Scope

<!-- Define what is managed in THIS repo vs. project-level overrides -->

- **System-wide LLM/agent configuration** (models, MCP servers, agent
  permissions, opencode settings) is managed centrally (e.g. via your nix
  config). Delegate those changes there — do not duplicate them here.
- **Project-level LLM management** (agent prompts, `AGENTS.md`, `.opencode/`
  overrides) lives in this repo and is what the orchestrator manages here.

---

## Subagent Roster

| Subagent | Agent file | Scope | When to invoke |
|----------|-----------|-------|---------------|
| **<project>-orchestrator** | `.opencode/agents/<project>-orchestrator.md` | Primary agent — delegates all work | Always active (set as `default_agent` in `opencode.json`) |
| **<project>-sre** | `.opencode/agents/<project>-sre.md` | Read-only runtime investigation | Crashes, failures, metrics questions, resource pressure |
| **<project>-<role>** | `.opencode/agents/<project>-<role>.md` | <description> | <trigger> |
| **git-flow** | (global agent) | Git operations | Commits, branches, PRs, git history queries |

### Orchestrator — primary agent

The orchestrator is set as `default_agent` in `opencode.json` (`mode: primary`).
Every session starts with it active. It reads this file, routes each task to the
correct subagent, and **never edits files or runs git commands directly**.

`opencode.json` must contain:
```json
{ "default_agent": "<project>-orchestrator" }
```
The value must match the agent filename without `.md`.

### SRE agent — first responder for all runtime issues

`@<project>-sre` is the **first stop for any running-system problem**. It
queries service endpoints and observability tools. It does NOT make changes.

**Invoke `@<project>-sre` before touching code/config for:**
- Service crashes or restart loops
- Performance degradation
- Any question answerable from logs, metrics, or health endpoints

### git-flow agent — all git operations

`@git-flow` is the **only agent that touches git**. It handles commits,
branching, PRs, and history queries. Always delegate to `@git-flow` for:
- Committing changes after a build or config edit
- Creating or merging branches
- Opening or reviewing pull requests
- Querying git log, diff, or blame

The orchestrator **must never run any git command directly** (`git add`,
`git commit`, `git push`, `git log`, etc.). If `@git-flow` stalls waiting
for human approval, relay the human's "yes/approve" back to `@git-flow`
by resuming the same task session — do not fall back to running git yourself.

---

## Delegation Rules

1. Top-level agent reads this file and identifies which subagent(s) need to act.
2. For runtime issues, delegate to `@<project>-sre` first.
3. Subagents are given a task description and told to read their own agent file.
4. Subagents read `<subsystem>/SUMMARY.md` then `<subsystem>/function_signature.md`
   before touching any files — this is how they navigate without reading all source.
5. Subagents operate only within their own subtree unless the task explicitly
   requires cross-cutting changes.
6. Cross-cutting changes are coordinated by the top-level agent, delegating to
   each subagent in sequence.

### Session start — check for interrupted tasks and template updates

Before doing anything else at the start of a session, run these two checks:

**1. Interrupted tasks:**

```bash
find . -name "in-progress.md" -not -path "./.git/*" 2>/dev/null
```

If any files are found, read each one and surface a resume prompt to the human.

**2. Template sync check:**

```bash
cat .template-local 2>/dev/null
```

If `.template-local` exists, read it and compare `template-version` against
the `CHANGELOG.md` in `template-path`. If the template has a newer version,
present a summary of what changed (from the CHANGELOG) and ask:

> "The opencode-project-template has been updated from vX to vY. Would you like to sync this project now?"

Only proceed with the sync if the human says yes. The sync process is described
in the Helpful Commands section below.

---

## Secrets Protocol

<!-- Adapt to your secrets manager: SOPS+age, Vault, AWS Secrets Manager, etc. -->

| Operation | Command |
|-----------|---------|
| Encrypt | `sops --encrypt --age $(cat ~/.age/public.key) --in-place <file>` |
| Decrypt | `SOPS_AGE_KEY_FILE=~/.age/private.key sops --decrypt <file>` |
| Edit | `SOPS_AGE_KEY_FILE=~/.age/private.key sops <file>` |

**Never commit unencrypted secrets or credentials.**

---

## Code Style Guidelines

<!-- Fill in project-specific style rules -->

- 2-space indentation, no tabs
- Lowercase kebab-case for resource names
- Specific dependency versions — never floating/latest
- Always include resource bounds (memory, CPU, etc.)

---

## Permission Gates — always ask before:

1. Writing or modifying any secret or credential file.
2. Applying changes to a production system.
3. Running destructive commands (`delete`, `drop`, `reset`, `purge`).
4. Any action affecting more than one node/instance simultaneously.
5. Deleting git history or running `git push --force`.

---

## Helpful Commands

```sh
# Syncing from the template
# 1. Read .template-local to get template-path and current template-version
# 2. Read CHANGELOG.md in template-path to identify changes since template-version
# 3. Apply each Added/Changed/Removed item to the relevant files in this project
# 4. Update template-version in .template-local to the new version
# 5. Add a row to the Session Log below
```

### .template-local format

This file is gitignored and machine-specific. Create it once per checkout:

```yaml
template-version: <semver matching .template-version in the template repo>
template-path: /absolute/path/to/opencode-project-template
```

The orchestrator reads this file at session start to detect when the template
has advanced beyond the pinned version and prompts the human to sync.

---

## Session Log

| Date | Summary |
|------|---------|
| YYYY-MM-DD | Initial setup |
