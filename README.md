# opencode-project-template

A scaffold for adding LLM agent configuration to an **existing software
project**. It wires an orchestrator/subagent architecture, a knowledge base,
and a cross-system SRE observer into whatever project you already have —
without touching your code, infrastructure, or runtime configuration.

> **What this template is NOT:**
> It does not create a new project. It does not define your application,
> your deployment, or your runtime. It adds `.opencode/` agent files, a
> few documentation stubs, and an `opencode.json` to a project that already
> exists. All project-specific content (code, infra, config) is yours to
> write. Machine-level configuration (global agents, MCP server installs,
> opencode model settings) belongs in your nix config or
> `~/.config/opencode/`, not here.

---

## For humans — quick start

1. **Add to an existing project** (if you have the nix config deployed):
   ```sh
   cd ~/repos/my-existing-project
   opencode   # then run /opencode-init inside the session
   ```
   The `opencode-init` command copies `template/` files from the latest
   template and fills in the placeholders for your project.

2. **Or copy manually:**
   ```sh
   git clone https://github.com/Tokugero/opencode-project-template.git /tmp/opencode-template
   cp -r /tmp/opencode-template/template/. ~/repos/my-existing-project/
   ```
   Then open the project in opencode and ask it to fill in the placeholders.

3. **Install the global git agent once per machine** — see
   [`global-config/agents/README.md`](global-config/agents/README.md).
   This is a one-time machine setup, not a per-project step.

4. **Pin your template version** — after bootstrapping, create `.template-local`
   (gitignored) in the project root with the version from `.template-version`.

---

## Repository layout

```
opencode-project-template/
├── README.md                   ← this file (template repo only — do not copy)
├── CHANGELOG.md                ← template version history (template repo only)
├── .template-version           ← current template semver (template repo only)
├── .gitignore                  ← ignores for the template repo itself
├── global-config/              ← machine-scoped config (deploy once, never copy to projects)
│   └── agents/
│       ├── README.md           ← install instructions + nix wiring
│       └── git-flow.md         ← global git operations agent
└── template/                   ← COPY THIS into your project (like src/)
    ├── AGENTS.md               ← orchestrator dispatch table
    ├── SUMMARY.md              ← subsystem SUMMARY stub (one copy per subsystem)
    ├── opencode.json           ← MCP config + default_agent stub
    ├── .opencode/
    │   └── agents/
    │       ├── project-orchestrator.md ← primary orchestrator agent template
    │       ├── project-sre.md          ← cross-system SRE observer template
    │       └── project-role.md         ← general subagent template
    ├── kb/
    │   └── README.md           ← knowledge base index stub
    └── docs/
        └── sre-todos.md        ← deferred SRE findings stub
```

The `template/` directory is the deliverable. Everything outside it is
template-repo infrastructure (docs, versioning, machine-scoped config).

---

## Template version and upgrades

This repo is versioned with semver. Consuming projects pin the version in
`.template-local` (gitignored, machine-specific). To upgrade:

1. Check your project's `.template-local` for `template-version`.
2. Read `CHANGELOG.md` entries between that version and the target.
3. Apply the **Added** / **Changed** / **Removed** items to your project files.
4. Update `template-version` in `.template-local` and add a row to your `AGENTS.md` Session Log.

---

## What each file is for

| File / dir | Copy to project? | Purpose |
|-----------|-----------------|---------|
| `template/AGENTS.md` | Yes — customise | Orchestrator dispatch table. The LLM reads this every session to understand the project structure and which agent handles what. |
| `template/SUMMARY.md` | Yes — one per subsystem | Subsystem context for subagents. Each major directory in your project should have one. |
| `template/opencode.json` | Yes — customise | MCP server config + `default_agent`. Set `default_agent` to match your orchestrator filename (without `.md`). |
| `template/.opencode/agents/project-orchestrator.md` | Yes — rename + customise | Primary orchestrator agent (`mode: primary`). Wired as `default_agent`. Delegates all work to subagents; never edits files itself. |
| `template/.opencode/agents/project-sre.md` | Yes — rename + customise | Cross-system SRE observer. Reads all subsystem docs, correlates across them, reports findings. Does not make changes. Grows in capability as subsystems are documented. |
| `template/.opencode/agents/project-role.md` | Yes — one per subsystem agent | General subagent template. Owns one subsystem directory; maintains `function_signature.md`; ask-first protocol. |
| `template/kb/README.md` | Yes | Knowledge base index. Agents append SOPs here after non-trivial tasks. |
| `template/docs/sre-todos.md` | Yes | Deferred SRE findings. The SRE agent appends here; you resolve them. |
| `global-config/agents/git-flow.md` | **No — never copy into a project.** Deploy to `~/.config/opencode/agents/` once per machine. | Global git operations agent. Machine-scoped, not project-scoped. Copying it into a project creates a stale duplicate and defeats the purpose of a global agent. |
| `CHANGELOG.md` | No | Template version history. Keep in the template repo only. |
| `.template-version` | No | Records the template's own semver. Use `.template-local` (gitignored) in consuming projects instead. |
| `README.md` | No | This file. The project should have its own README. |

---

## Agent architecture

```
Human
  │
  ▼
Orchestrator  (mode: primary — set as default_agent in opencode.json)
  │  reads AGENTS.md every session
  │
  ├── @<project>-sre      cross-system observer; read-only; grows with the project
  ├── @<project>-<role>   owns one subsystem directory
  ├── @<project>-<role>   owns another subsystem directory
  └── @git-flow           global; handles all git operations across all projects
```

### Orchestrator

The orchestrator (`mode: primary`) is the LLM you talk to directly. It is set
as `default_agent` in `opencode.json` so every session starts with it active.
It reads `AGENTS.md`, identifies which subagent should act, and delegates. It
**never** edits files or runs git commands itself — those are always routed to
a subagent. The `edit` and `write` tools are disabled in its frontmatter to
enforce this.

### SRE agent

The SRE agent is a cross-system observer. It reads every subsystem's
`SUMMARY.md` and `function_signature.md` to build a full picture of the
system, then queries live state to find what is actually happening. It
correlates across subsystems (a symptom in one often has its root cause in
another), recommends fixes, and does **not** make changes. As the project
gains more subsystems with better documentation, the SRE agent's diagnostic
capability grows automatically.

### Role agents

Each major subsystem directory gets its own role agent. The agent:
- Owns exactly one directory
- Reads `SUMMARY.md` and `function_signature.md` before doing anything
- Keeps `function_signature.md` up to date as it works
- Uses an ask-first protocol before executing any action class for the first time
- Writes `in-progress.md` at the start of any multi-step task (gitignored)

### git-flow (global)

Handles commits, branches, and PRs for any project. Deployed once to
`~/.config/opencode/agents/git-flow.md`, not copied into each repo.
The orchestrator routes all git operations here and never runs `git` directly.

Two-phase push: always summarises the proposed push and waits for explicit
human confirmation before executing `git push`.

---
## LLM setup instructions

> **This section is for the LLM adding agent scaffolding to an existing project.**
> If you are an agent running `opencode-init` or applying this template manually,
> read every word of this section before touching any file.
>
> **You are NOT creating a new project.** The project already exists. Your job
> is to add opencode agent configuration to it. Do not create application code,
> infrastructure definitions, deployment configs, or runtime setup. Do not copy
> machine-level config (`global-config/`) into the project. Only the files
> inside `template/` belong in the destination project.

### Step 0 — understand before acting

Before creating or modifying anything:

1. Read `template/AGENTS.md` to understand the orchestrator pattern.
2. Read `template/.opencode/agents/project-orchestrator.md` — primary agent contract.
3. Read `template/.opencode/agents/project-role.md` — subagent contract.
4. Read `template/.opencode/agents/project-sre.md` — SRE observer pattern.
5. Examine the **destination** project directory — directory names, existing
   code, `package.json` / `go.mod` / `Makefile` / `flake.nix`, README if any.
6. Infer: project name, language/stack, major subsystems, likely agent roles.

Do **not** start writing files until you have answered:
- What is the project name? (use the directory name if unclear)
- What are the 2–4 major subsystems or concerns? (e.g. `api`, `frontend`, `infra`)
- What secrets manager is in use, if any?

### Step 1 — files to copy verbatim (no placeholder substitution)

Copy these files from `template/` into the project root as-is:

- `template/kb/README.md` → `kb/README.md`
- `template/docs/sre-todos.md` → `docs/sre-todos.md`

Also append `template/` entries from the template `.gitignore` to any existing
`.gitignore` in the project — do not overwrite.

Then create `.template-local` (gitignored, machine-specific) in the project root:

```yaml
template-version: <value from .template-version in this template repo>
template-path: /absolute/path/to/opencode-project-template
```

Do **not** copy `.template-version` itself — `.template-local` replaces it.

### Step 2 — files to copy and customise

For each file below, copy from `template/` and substitute every `<placeholder>`:

#### `template/AGENTS.md` → `AGENTS.md`

| Placeholder | Replace with |
|------------|-------------|
| `<project-name>` in the title | actual project name |
| `<project>` in agent names | short project slug (e.g. `myapp`) |
| `<role>` in agent names | subsystem name (e.g. `api`, `infra`) |
| `<subsystem>/` paths | actual directory names |
| `<description>` in roster table | one-sentence scope description |
| `<trigger>` in roster table | when to invoke this agent |
| Repository structure tree | actual directory layout of the project |
| Secrets Protocol section | adapt to the project's secrets manager |
| Code Style Guidelines | adapt to the project's language and conventions |
| Session Log first entry | today's date and "Initial setup (v`<version>`)" |

#### `template/SUMMARY.md` — one copy per subsystem directory

Create `<subsystem>/SUMMARY.md` for each major subsystem:

| Placeholder | Replace with |
|------------|-------------|
| `<subsystem>` in the title | directory name |
| "What this subsystem is" body | one paragraph describing the subsystem |
| Component table | actual files/components in the directory |
| Service endpoints table | real URLs or "n/a" |
| Dev workflow commands | real commands for this subsystem |
| Non-negotiables | project-specific rules |

#### `template/opencode.json` → `opencode.json`

Set `default_agent` to match your orchestrator filename (without `.md`).
Remove the `REPLACE_ME` MCP stub or replace it with real MCP servers.
If no MCP servers are needed yet:
```json
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "orchestrator"
}
```

#### `template/.opencode/agents/project-orchestrator.md` → `.opencode/agents/<project>-orchestrator.md`

| Placeholder | Replace with |
|------------|-------------|
| frontmatter `description` | actual project name and scope |
| `<project>` in agent names throughout | short project slug |
| Decision tree items 2+ | one rule per subsystem mapping to its agent |
| Subagents table rows | actual subagent names and scope descriptions |
| `@<project>-sre` references | e.g. `@myapp-sre` |
| Root config files list | actual root config files for this project |

#### `template/.opencode/agents/project-sre.md` → `.opencode/agents/<project>-sre.md`

| Placeholder | Replace with |
|------------|-------------|
| `<project>` in frontmatter description | project slug |
| `<project>` in agent name heading | project slug |

No baked investigation steps needed. The SRE agent discovers how to
investigate by reading each subsystem's `SUMMARY.md` and
`function_signature.md` at runtime. Its capability grows as those docs grow.

#### `template/.opencode/agents/project-role.md` → `.opencode/agents/<project>-<role>.md`

One copy per subsystem agent. Rename to `<project>-<role>.md` (e.g. `myapp-api.md`):

| Placeholder | Replace with |
|------------|-------------|
| frontmatter `description` | actual description of this agent's scope |
| `<project>-<role> agent` heading | e.g. `myapp-api agent` |
| `<subsystem>/` in scope table | actual directory this agent owns |
| `@<project>-sre` references | e.g. `@myapp-sre` |

### Step 3 — create `function_signature.md` stubs

For each subsystem that has a role agent, create
`<subsystem>/function_signature.md` with this stub:

```markdown
# function_signature.md — <subsystem>

_Last updated: YYYY-MM-DD_

## Overview
<!-- One paragraph: what this subsystem as a whole does. -->

## Entries
<!-- The <project>-<role> agent will populate this as it works. -->
```

The role agent fills this in as it explores the subsystem. Do not attempt
to populate it yourself unless you have already read all the files.

### Step 4 — do NOT copy

These items must never be copied into the destination project:

- `global-config/` — **NEVER copy this into a project.** It is machine-scoped.
  The `git-flow.md` agent inside it is deployed once to
  `~/.config/opencode/agents/` (or via nix config) and shared across all
  projects on the machine. Copying it into a project repo creates a stale
  duplicate and defeats the purpose of a global agent.
- `CHANGELOG.md` — template version history, stays in the template repo
- `README.md` (this file) — stays in the template repo
- `.template-version` — use `.template-local` (gitignored) in the project instead

### Step 5 — verify before finishing

After all files are written, confirm:

- [ ] Every `<placeholder>` replaced — search for `<` in all copied files
- [ ] `AGENTS.md` roster table matches the actual `.opencode/agents/` files
- [ ] `opencode.json` has `default_agent` set and no `REPLACE_ME` stubs
- [ ] `default_agent` value matches the orchestrator filename (without `.md`)
- [ ] `.template-local` exists with the correct version and path
- [ ] Session Log in `AGENTS.md` has today's date
- [ ] `global-config/` was NOT copied into the project — if it was, remove it immediately
- [ ] No application code, infra config, or runtime setup was created by this process

Report to the human:
1. Which files were created
2. Which placeholders were inferred vs. which need human input
3. Whether `global-config/agents/git-flow.md` is already deployed on this
   machine (`ls ~/.config/opencode/agents/git-flow.md`) — if not, print the
   install instructions from `global-config/agents/README.md`
