# opencode-project-template

A structured starting point for LLM-assisted software projects using
[opencode](https://opencode.ai). It provides an orchestrator/subagent
architecture, a knowledge base, an SRE first-responder pattern, and a global
git operations agent — all wired together so an LLM can navigate and operate
your project safely from the first session.

---

## For humans — quick start

1. **Bootstrap a new project** (if you have the nix config deployed):
   ```sh
   cd ~/repos/my-new-project
   opencode   # then run /opencode-init inside the session
   ```
   The `opencode-init` command pulls the latest template and adapts all
   placeholders to your project automatically.

2. **Or copy manually:**
   ```sh
   git clone https://github.com/Tokugero/opencode-project-template.git ~/.opencode-template
   cp -r ~/.opencode-template/. ~/repos/my-new-project/
   rm -rf ~/repos/my-new-project/.git
   ```
   Then open the project in opencode and ask it to fill in the placeholders.

3. **Install the global git agent once per machine** — see
   [`global-config/agents/README.md`](global-config/agents/README.md).

4. **Pin your template version** — after bootstrapping, note the value in
   `.template-version`. Use `CHANGELOG.md` to diff and upgrade later.

---

## Repository layout

```
opencode-project-template/
├── README.md                   ← this file
├── AGENTS.md                   ← orchestrator dispatch table (copy → project)
├── SUMMARY.md                  ← subsystem SUMMARY template (copy per subsystem)
├── CHANGELOG.md                ← template version history
├── .template-version           ← current template semver
├── opencode.json               ← MCP server config stub (copy → project)
├── .gitignore                  ← ignores in-progress.md and decrypted secrets
├── .opencode/
│   └── agents/
│       ├── project-sre.md      ← SRE first-responder agent template
│       └── project-role.md     ← general subagent template
├── global-config/
│   ├── README.md               ← global vs project-scoped agent explainer
│   └── agents/
│       ├── README.md           ← install instructions + nix wiring
│       └── git-flow.md         ← global git operations agent (deploy once)
├── kb/
│   └── README.md               ← knowledge base index stub
└── docs/
    └── sre-todos.md            ← deferred SRE findings stub
```

---

## Template version and upgrades

This repo is versioned with semver. Consuming projects pin the version in
`.template-version`. To upgrade:

1. Check your project's `.template-version`.
2. Read `CHANGELOG.md` entries between that version and the target.
3. Apply the **Added** / **Changed** / **Removed** items to your project files.
4. Update `.template-version` and add a row to your `AGENTS.md` Session Log.

---

## What each file is for

| File / dir | Copy to project? | Purpose |
|-----------|-----------------|---------|
| `AGENTS.md` | Yes — customise | Orchestrator dispatch table. The LLM reads this every session to understand the project structure and which agent handles what. |
| `SUMMARY.md` | Yes — one per subsystem | Subsystem context for subagents. Each major directory in your project should have one. |
| `opencode.json` | Yes — customise | MCP server config. Add your project's MCP servers here. |
| `.gitignore` | Yes — append to yours | Ignores `in-progress.md` files and decrypted secret artefacts. |
| `.opencode/agents/project-sre.md` | Yes — rename + customise | SRE first-responder agent. Read-only; queries logs and health endpoints; does not make changes. |
| `.opencode/agents/project-role.md` | Yes — one per subsystem agent | General subagent template. Owns one subsystem directory; maintains `function_signature.md`; ask-first protocol. |
| `global-config/agents/git-flow.md` | No — deploy to `~/.config/opencode/agents/` | Global git operations agent. Install once per machine, not per project. |
| `kb/README.md` | Yes | Knowledge base index. Agents append SOPs here after non-trivial tasks. |
| `docs/sre-todos.md` | Yes | Deferred SRE findings. The SRE agent appends here; you resolve them. |
| `CHANGELOG.md` | No | Template version history. Keep in the template repo only. |
| `.template-version` | Yes | Records which template version the project was initialised from. |

---

## Agent architecture

```
Human
  │
  ▼
Orchestrator  (reads AGENTS.md every session)
  │
  ├── @<project>-sre      first stop for any runtime problem; read-only
  ├── @<project>-<role>   owns one subsystem directory
  ├── @<project>-<role>   owns another subsystem directory
  └── @git-flow           global; handles all git operations across all projects
```

### Orchestrator

The orchestrator is the LLM you talk to directly. It reads `AGENTS.md`,
identifies which subagent should act, and delegates. It **never** edits files
or runs git commands itself — those are always routed to a subagent.

### SRE agent

First responder for any running-system problem. It reads logs, queries health
endpoints, and reports findings. It does **not** make changes. The orchestrator
invokes it before touching any code or config for a runtime issue.

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

> **This section is for the LLM initialising a new project from this template.**
> If you are an agent running `opencode-init` or adapting this template, read
> every word of this section before touching any file.

### Step 0 — understand before acting

Before creating or modifying anything:

1. Read `AGENTS.md` in the template to understand the orchestrator pattern.
2. Read `.opencode/agents/project-role.md` to understand the subagent contract.
3. Read `.opencode/agents/project-sre.md` to understand the SRE pattern.
4. Examine the target project directory — directory names, existing code,
   `package.json` / `go.mod` / `Makefile` / `flake.nix`, README if any.
5. Infer: project name, language/stack, major subsystems, likely agent roles.

Do **not** start writing files until you have answered:
- What is the project name? (use the directory name if unclear)
- What are the 2–4 major subsystems or concerns? (e.g. `api`, `frontend`, `infra`)
- What runtime does the SRE agent need to query? (endpoints, log commands)
- What secrets manager is in use, if any?

### Step 1 — files to copy verbatim (no placeholder substitution)

Copy these files as-is into the project root:

- `.gitignore` — append to any existing `.gitignore`, do not overwrite
- `kb/README.md`
- `docs/sre-todos.md`
- `CHANGELOG.md` — only if no existing changelog; otherwise skip
- `.template-version`

### Step 2 — files to copy and customise

For each file below, copy it and substitute every `<placeholder>`:

#### `AGENTS.md`

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

#### `SUMMARY.md` — one copy per subsystem directory

Create `<subsystem>/SUMMARY.md` for each major subsystem:

| Placeholder | Replace with |
|------------|-------------|
| `<subsystem>` in the title | directory name |
| "What this subsystem is" body | one paragraph describing the subsystem |
| Component table | actual files/components in the directory |
| Service endpoints table | real URLs or "n/a" |
| Dev workflow commands | real commands for this subsystem |
| Non-negotiables | project-specific rules |

#### `opencode.json`

Remove the `REPLACE_ME` MCP stub or replace it with real MCP servers.
If no MCP servers are needed yet, use:
```json
{ "$schema": "https://opencode.ai/config.json" }
```

#### `.opencode/agents/project-sre.md`

| Placeholder | Replace with |
|------------|-------------|
| `<project>` in frontmatter description | project slug |
| `<project>` in agent name heading | project slug |
| Delegation table | actual subsystem agents for this project |
| "How to investigate" section | real log commands, health endpoints, metrics tools |

#### `.opencode/agents/project-role.md` — one copy per subsystem agent

Rename to `<project>-<role>.md` (e.g. `myapp-api.md`):

| Placeholder | Replace with |
|------------|-------------|
| frontmatter `description` | actual description of this agent's scope |
| `<project>-<role> agent` heading | e.g. `myapp-api agent` |
| `<subsystem>/` in scope table | actual directory this agent owns |
| Delegation table rows | real subsystem-to-agent mappings |
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

The role agent will fill this in as it explores the subsystem. Do not attempt
to populate it yourself unless you have already read all the files in the
directory.

### Step 4 — do NOT copy

These files stay in the template repo only:

- `global-config/` — deploy these once per machine, not per project
- `CHANGELOG.md` of the template — the project gets its own changelog
- `README.md` (this file) — the project should have its own README

### Step 5 — verify before finishing

After all files are written, confirm:

- [ ] Every `<placeholder>` has been replaced — run a search for `<` in all
      copied files and resolve any remaining angle-bracket tokens
- [ ] `AGENTS.md` roster table matches the actual `.opencode/agents/` files
- [ ] `opencode.json` has no `REPLACE_ME` stubs
- [ ] `.template-version` contains the correct version from this template
- [ ] Session Log in `AGENTS.md` has today's date
- [ ] `global-config/` was NOT copied into the project

Report to the human:
1. Which files were created
2. Which placeholders were inferred vs. which need human input
3. Whether `global-config/agents/git-flow.md` is already deployed on this
   machine (`ls ~/.config/opencode/agents/git-flow.md`) — if not, print the
   install instructions from `global-config/agents/README.md`

