# opencode-project-template

A scaffold for adding LLM agent configuration to an **existing software
project**. It wires an orchestrator/subagent architecture, a knowledge base,
and a cross-system SRE observer into whatever project you already have —
without touching your code, infrastructure, or runtime configuration.

Supports both **OpenCode** and **Claude Code** — the `template/` directory
contains parallel configuration for each tool.

> **What this template is NOT:**
> It does not create a new project. It does not define your application,
> your deployment, or your runtime. It adds agent files, a few documentation
> stubs, and a config file to a project that already exists. All
> project-specific content (code, infra, config) is yours to write.
> Machine-level configuration (global agents, MCP server installs, model
> settings) belongs in your nix config or tool user-config, not here.

---

## For humans — quick start

### Claude Code

1. **Add to an existing project** (copy manually):
   ```sh
   git clone https://github.com/Tokugero/opencode-project-template.git /tmp/opencode-template
   cp -r /tmp/opencode-template/template/. ~/repos/my-existing-project/
   ```
   Then open the project in Claude Code and ask it to fill in the placeholders.

2. **Deploy global config once per machine** — see
   [`global-config/claudecode/README.md`](global-config/claudecode/README.md).
   Installs global git workflow rules and optional skills.

### OpenCode

1. **Add to an existing project** (if you have the nix config deployed):
   ```sh
   cd ~/repos/my-existing-project
   opencode   # then run /opencode-init inside the session
   ```
   The `opencode-init` command copies `template/` files from the latest
   template and fills in the placeholders for your project.

2. **Or copy manually:**
   ```sh
   cp -r /tmp/opencode-template/template/. ~/repos/my-existing-project/
   ```
   Then open the project in opencode and ask it to fill in the placeholders.

3. **Deploy global config once per machine** — see
   [`global-config/agents/README.md`](global-config/agents/README.md) for
   OpenCode agent deployment and the agency-agents companion tool for git workflow.

### Both tools

4. **Pin your template version** — after bootstrapping, create `.template-local`
   (gitignored) in the project root with the version from `.template-version`.

---

## Repository layout

```
opencode-project-template/
├── README.md                   ← this file (template repo only — do not copy)
├── CLAUDE.md                   ← Claude Code instructions for this template repo
├── CHANGELOG.md                ← template version history (template repo only)
├── .template-version           ← current template semver (template repo only)
├── .gitignore                  ← ignores for the template repo itself
├── global-config/              ← machine-scoped config (deploy once, never copy to projects)
│   ├── agents/                 ← OpenCode global agents
│   │   └── README.md           ← install instructions + nix wiring
│   └── claudecode/             ← Claude Code global config
│       ├── README.md           ← install instructions + nix wiring
│       ├── CLAUDE.md           ← global git rules → deploy to ~/.claude/CLAUDE.md
│       └── skills/             ← Claude Code skills (deploy to ~/.claude/skills/)
└── template/                   ← COPY THIS into your project (like src/)
    ├── AGENTS.md               ← OpenCode orchestrator dispatch table
    ├── CLAUDE.md               ← Claude Code orchestrator + dispatch table
    ├── .abstract.md            ← L0 ultra-concise project map (~100 tokens)
    ├── .overview.md            ← L1 subsystem context stub (one copy per subsystem)
    ├── .envrc                  ← direnv config: `use flake` for Nix dev shell activation
    ├── opencode.json           ← OpenCode: MCP config + default_agent stub
    ├── .mcp.json               ← Claude Code: MCP server config (stdio + http examples)
    ├── protocols.md            ← secrets, permission gates, code style
    ├── .opencode/
    │   └── agents/             ← OpenCode agent templates
    │       ├── project-orchestrator.md
    │       ├── project-sre.md
    │       └── project-role.md
    ├── .claude/
    │   ├── settings.json       ← Claude Code: session model + permissions stub
    │   ├── agents/             ← Claude Code subagent templates
    │   │   ├── project-sre.md
    │   │   └── project-role.md
    │   └── skills/             ← Claude Code skill templates
    │       └── example-skill/  ← skill pattern example
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

**Major version upgrades** involve breaking changes and file renames. See the
migration guides in `docs/` for step-by-step instructions:

- [1.x → 2.0.0](docs/migration-v2.md) — `SUMMARY.md` → `.overview.md`,
  `function_signature.md` merged into `.overview.md`, `.abstract.md` added,
  git-flow removed

---

## What each file is for

### Template deliverable (copy into projects)

| File / dir | Tool | Copy to project? | Purpose |
|-----------|------|-----------------|---------|
| `template/AGENTS.md` | OpenCode | Yes — customise | Orchestrator dispatch table read every session. |
| `template/CLAUDE.md` | Claude Code | Yes — customise | Orchestrator instructions + dispatch table (merges AGENTS.md role). |
| `template/.envrc` | Both | Yes — one per flake.nix | direnv config: `use flake` activates the Nix dev shell. Copy to project root and each subsystem that has its own `flake.nix`. Requires `direnv` + `nix-direnv` on the machine. |
| `template/.abstract.md` | Both | Yes — one per project | L0 ultra-concise project map (~100 tokens). Loaded first by every agent. |
| `template/.overview.md` | Both | Yes — one per subsystem | L1 subsystem context: overview, component table, endpoints, dev commands. |
| `template/opencode.json` | OpenCode | Yes — customise | MCP server config + `default_agent`. |
| `template/.mcp.json` | Claude Code | Yes — customise | MCP server config stub; shows both `stdio` and `http` server types. |
| `template/.claude/settings.json` | Claude Code | Yes — customise | Session model + permissions stub. MCP servers go in `.mcp.json`. |
| `template/protocols.md` | Both | Yes | Secrets, permission gates, code style. |
| `template/.opencode/agents/project-orchestrator.md` | OpenCode | Yes — rename + customise | Primary orchestrator (`mode: primary`). Never edits files. |
| `template/.opencode/agents/project-sre.md` | OpenCode | Yes — rename + customise | Cross-system SRE observer. Read-only. |
| `template/.opencode/agents/project-role.md` | OpenCode | Yes — one per subsystem | General subagent. Owns one subsystem. |
| `template/.claude/agents/project-sre.md` | Claude Code | Yes — rename + customise | Cross-system SRE observer. Read-only. |
| `template/.claude/agents/project-role.md` | Claude Code | Yes — one per subsystem | General subagent. Owns one subsystem. |
| `template/.claude/skills/example-skill/` | Claude Code | Optional — copy as pattern | Skill template. Demonstrates skill + supporting-file pattern. |
| `template/kb/README.md` | Both | Yes | Knowledge base index. |
| `template/docs/sre-todos.md` | Both | Yes | Deferred SRE findings. |

### Global / machine-scoped (deploy once, never copy into projects)

| File / dir | Tool | Deploy to | Purpose |
|-----------|------|-----------|---------|
| `global-config/claudecode/CLAUDE.md` | Claude Code | `~/.claude/CLAUDE.md` | Global git workflow rules (two-phase push, conventional commits). |
| `global-config/claudecode/skills/sdlc-audit/` | Claude Code | `~/.claude/skills/sdlc-audit/` | Repeatable audit & fix cycle using parallel agent teams. |
| `global-config/claudecode/skills/sdlc-review/` | Claude Code | `~/.claude/skills/sdlc-review/` | Read-only codebase review. Produces consolidated findings without code changes. |

### Template repo infrastructure (never copy)

| File | Purpose |
|------|---------|
| `CHANGELOG.md` | Template version history |
| `.template-version` | Template's own semver; use `.template-local` in consuming projects |
| `README.md` | This file |
| `CLAUDE.md` | Instructions for working on this template repo itself |

---

## Agent architecture

### OpenCode

```
Human
  │
  ▼
Orchestrator  (mode: primary — set as default_agent in opencode.json)
  │  reads AGENTS.md every session
  │
  ├── @<project>-sre      cross-system observer; read-only
  └── @<project>-<role>   owns one subsystem directory
```

### Claude Code

```
Human
  │
  ▼
Main session  (reads CLAUDE.md every session — IS the orchestrator)
  │
  ├── @<project>-sre      cross-system observer; read-only
  ├── @<project>-<role>   owns one subsystem directory
  └── built-in git        git workflow handled natively
```

### Orchestrator

The orchestrator is the LLM you talk to directly. It reads the dispatch
file (`AGENTS.md` for OpenCode, `CLAUDE.md` for Claude Code), identifies
which subagent should act, and delegates. It **never** edits files or runs
git commands itself.

### SRE agent

The SRE agent is a cross-system observer. It loads `.abstract.md` (L0) for a
quick project map, then reads each subsystem's `.overview.md` (L1) to build a
full picture of the system. It queries live state to find what is actually
happening, correlates across subsystems, recommends fixes, and does **not**
make changes. Its capability grows automatically as subsystem docs improve.

### Role agents

Each major subsystem directory gets its own role agent. The agent:
- Owns exactly one directory
- Loads `.abstract.md` (L0), then `<subsystem>/.overview.md` (L1) before acting
- Keeps `.overview.md` up to date as it works
- Uses an ask-first protocol before executing any action class for the first time
- Writes `in-progress.md` at the start of any multi-step task (gitignored)

---
## LLM setup instructions

> **This section is for the LLM adding agent scaffolding to an existing project.**
> If you are an agent running an init command or applying this template manually,
> read every word of this section before touching any file.
>
> **You are NOT creating a new project.** The project already exists. Your job
> is to add LLM agent configuration to it. Do not create application code,
> infrastructure definitions, deployment configs, or runtime setup. Do not copy
> machine-level config (`global-config/`) into the project. Only the files
> inside `template/` belong in the destination project.

### Step 0 — understand before acting

Before creating or modifying anything:

1. Read `template/CLAUDE.md` (Claude Code) or `template/AGENTS.md` (OpenCode)
   to understand the orchestrator pattern.
2. Read the agent templates in `template/.claude/agents/` or
   `template/.opencode/agents/`.
3. Examine the **destination** project directory — directory names, existing
   code, `package.json` / `go.mod` / `Makefile` / `flake.nix`, README if any.
4. Infer: project name, language/stack, major subsystems, likely agent roles.

Do **not** start writing files until you have answered:
- What is the project name? (use the directory name if unclear)
- What are the 2–4 major subsystems or concerns? (e.g. `api`, `frontend`, `infra`)
- What secrets manager is in use, if any?

### Step 1 — files to copy verbatim (no placeholder substitution)

Copy these files from `template/` into the project root as-is:

- `template/kb/README.md` → `kb/README.md`
- `template/docs/sre-todos.md` → `docs/sre-todos.md`
- `template/protocols.md` → `protocols.md`
- `template/.envrc` → `.envrc` (and one copy per subsystem directory that has
  its own `flake.nix`)

Also append `template/` entries from the template `.gitignore` to any existing
`.gitignore` in the project — do not overwrite.

After first checkout, run `direnv allow` in the project root (and each
subsystem directory with a `.envrc`). This is a one-time security gate.

Then create `.template-local` (gitignored, machine-specific) in the project root:

```yaml
template-version: <value from .template-version in this template repo>
template-path: /absolute/path/to/opencode-project-template
```

Do **not** copy `.template-version` itself — `.template-local` replaces it.

### Step 2 — files to copy and customise

For each file below, copy from `template/` and substitute every `<placeholder>`:

#### Dispatch table

**Claude Code:** `template/CLAUDE.md` → `CLAUDE.md`

| Placeholder | Replace with |
|------------|-------------|
| `<project-name>` in the title | actual project name |
| `<project>` in agent names | short project slug (e.g. `myapp`) |
| `<role>` in agent names | subsystem name (e.g. `api`, `infra`) |
| `<subsystem>/` paths | actual directory names |
| `<description>` in roster table | one-sentence scope description |
| `<trigger>` in roster table | when to invoke this agent |
| Repository structure tree | actual directory layout of the project |

**OpenCode:** `template/AGENTS.md` → `AGENTS.md` (same placeholders)

#### Settings / config

**Claude Code (MCP servers):** `template/.mcp.json` → `.mcp.json`

Replace or remove the placeholder server entries. Two types are shown:
- `stdio` — a locally-installed MCP server binary (e.g. `mcp-server-playwright`)
- `http` — a locally-running HTTP MCP server (e.g. `http://localhost:PORT/mcp/`)

Remove any entries not needed; delete the file entirely if no MCP servers are required.

**Claude Code (settings):** `template/.claude/settings.json` → `.claude/settings.json`

No MCP servers here — those live in `.mcp.json`. No placeholder substitution needed; keep as-is or adjust permissions.

**Do not skip this file.** It pins the main session model to `claude-opus-4-6`. If `.claude/settings.json` is absent, Claude Code falls back to the user's global `~/.claude/settings.json`, which may be set to a different (typically lower-tier) model. The result is the orchestrator running on Sonnet or Haiku instead of Opus — silently, with no warning.

**OpenCode:** `template/opencode.json` → `opencode.json`

Set `default_agent` to match your orchestrator filename (without `.md`).

#### Subsystem context

**Project map:** `template/.abstract.md` → `.abstract.md` (one per project, at the root)

| Placeholder | Replace with |
|------------|-------------|
| `<project-name>` | actual project name |
| Stack line | language, framework, key dependencies |
| Subsystem list | one line per subsystem with directory path |

**Subsystem overviews:** `template/.overview.md` — create one copy per subsystem directory as
`<subsystem>/.overview.md`:

| Placeholder | Replace with |
|------------|-------------|
| `<subsystem>` in the title | directory name |
| Body | one paragraph describing the subsystem |
| Component table | actual files/components with types and dependencies |
| Service endpoints table | real URLs or "n/a" |
| Dev workflow commands | real commands for this subsystem |

#### Subagents

**Claude Code:** `template/.claude/agents/project-sre.md` → `.claude/agents/<project>-sre.md`

| Placeholder | Replace with |
|------------|-------------|
| `<project>` in description | project slug |
| `<project>` in heading | project slug |

**Claude Code:** `template/.claude/agents/project-role.md` → `.claude/agents/<project>-<role>.md`

One copy per subsystem. Rename to `<project>-<role>.md`:

| Placeholder | Replace with |
|------------|-------------|
| frontmatter `description` | actual scope description |
| `<project>-<role> agent` heading | e.g. `myapp-api agent` |
| `<subsystem>/` in scope table | actual directory this agent owns |
| `@<project>-sre` references | e.g. `@myapp-sre` |

**OpenCode:** same placeholders, files in `template/.opencode/agents/`. Also
copy `project-orchestrator.md` → `.opencode/agents/<project>-orchestrator.md`.

#### Skills (Claude Code only, optional)

Copy `template/.claude/skills/example-skill/` as a starting point for any
project-specific skill. Rename the directory and update `SKILL.md` frontmatter.
If no custom skills are needed, do not copy this directory.

### Step 3 — verify context files

Confirm that `.abstract.md` exists at the project root and each subsystem
directory has `.overview.md`. These were created in Step 2. The component
table in each `.overview.md` starts sparse — the role agent fills it in as
it explores the subsystem. Do not attempt to populate it yourself unless
you have already read all the files.

### Step 4 — do NOT copy

These items must never be copied into the destination project:

- `global-config/` — **NEVER copy this into a project.** It is machine-scoped.
  Global config is deployed once per machine to `~/.config/opencode/` or
  `~/.claude/`. Copying it into a project creates a stale duplicate.
- `CHANGELOG.md` — template version history, stays in the template repo
- `README.md` (this file) — stays in the template repo
- `CLAUDE.md` (root) — describes this template repo, not consuming projects
- `.template-version` — use `.template-local` (gitignored) in the project instead

### Step 5 — verify before finishing

After all files are written, confirm:

- [ ] Every `<placeholder>` replaced — search for `<` in all copied files
- [ ] Dispatch table (`CLAUDE.md` or `AGENTS.md`) roster matches actual agent files
- [ ] Settings file has no `REPLACE_ME` stubs (remove or replace MCP entries)
- [ ] `.claude/settings.json` exists and `model` is set to `claude-opus-4-6`
- [ ] `.template-local` exists with the correct version and path
- [ ] `global-config/` was NOT copied into the project
- [ ] No application code, infra config, or runtime setup was created

Report to the human:
1. Which files were created
2. Which placeholders were inferred vs. which need human input
3. Whether global config is deployed on this machine:
   - Claude Code: `ls ~/.claude/CLAUDE.md` — if not, print install instructions
     from `global-config/claudecode/README.md`
   - OpenCode: `ls ~/.config/opencode/agents/` — if no agents installed, point
     to `global-config/agents/README.md` and the agency-agents companion tool
