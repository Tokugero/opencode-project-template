# AGENTS.md — <project-name>

## Purpose

This is the **orchestration layer** for the <project-name> repository.
It describes how the repo is structured, which subagents handle which concerns,
and the safety gates that require human approval.

Subagents working inside a specific subsystem should read their own
`.abstract.md` (L0) and `.overview.md` (L1), or agent file first. Read this
file only when cross-cutting coordination is needed.

> **Protocols and history are in separate files to keep this prompt lean:**
> - Secrets, permission gates, code style, helpful commands → `protocols.md`
> - Session history → `docs/session-log.md`

---

## Table of Contents

- [Repository Structure](#repository-structure)
- [Scope](#scope)
- [Subagent Roster](#subagent-roster)
- [Delegation Rules](#delegation-rules)

---

## Repository Structure

```
<project>/
├── opencode.json               <- MCP server config + default_agent
├── AGENTS.md                   <- THIS FILE — orchestration dispatch table
├── .abstract.md                <- project-wide context summary (read this first)
├── .overview.md                <- full file/component map (orient before diving in)
├── protocols.md                <- secrets, permission gates, code style, helpful commands
├── CHANGELOG.md                <- template version history
├── .template-local             <- gitignored: local template-version + template-path
├── .opencode/
│   └── agents/                 <- project-scoped subagent definitions
│       ├── <project>-orchestrator.md  <- primary orchestrator (mode: primary)
│       ├── <project>-sre.md    <- read-only runtime investigation
│       ├── <project>-<role>.md <- per-subsystem agents
│       └── ...
├── kb/                         <- knowledge base: SOPs and runbooks
│   └── README.md               <- KB index table
├── docs/
│   ├── session-log.md          <- historical session notes (not loaded into prompt)
│   └── sre-todos.md            <- deferred SRE findings
│
└── <subsystem>/                <- one directory per major subsystem
    ├── .abstract.md            <- subsystem context summary (L0)
    └── .overview.md            <- file/component map (L1); owned by subsystem agent
```

---

## Scope

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

---

## Delegation Rules

1. Top-level agent reads this file and identifies which subagent(s) need to act.
2. For runtime issues, delegate to `@<project>-sre` first.
3. Subagents are given a task description and told to read their own agent file.
4. Subagents follow L0/L1/L2 protocol: read `.abstract.md` (scan), `.overview.md`
   (orient), source files only as needed — this is how they navigate without reading all source.
5. Subagents operate only within their own subtree unless the task explicitly
   requires cross-cutting changes.
6. Cross-cutting changes are coordinated by the top-level agent, delegating to
   each subagent in sequence.

### Session start — three checks before any task

**1. Interrupted tasks:**

```bash
find . -name "in-progress.md" -not -path "./.git/*" 2>/dev/null
```

If any files are found, read each and surface a resume prompt to the human.

**2. SRE deferred issues:**

```bash
grep -A7 "Status: open" docs/sre-todos.md
```

If open entries exist, summarise them before proceeding.

**3. Template sync check:**

```bash
cat .template-local 2>/dev/null
```

If `.template-local` exists, compare `template-version` against the
`CHANGELOG.md` in `template-path`. If newer, present a summary and ask:
> "The opencode-project-template has been updated from vX to vY. Would you like to sync?"

Only proceed with the sync if the human says yes.
See `protocols.md` → Helpful Commands for the sync procedure.
