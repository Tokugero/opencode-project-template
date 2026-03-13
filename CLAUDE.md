# CLAUDE.md — opencode-project-template

You are working on the **opencode-project-template** repository — a meta-template
that scaffolds LLM agent configuration into existing projects.

## What this repo is

This is a template-provider, not a normal project. It has three layers:

- `template/` — files that get copied into consuming projects (the deliverable)
- `global-config/` — machine-scoped config deployed once per workstation; never per-project
- Root files — this repo's own docs, versioning, and this file

## Repository structure

```
opencode-project-template/
├── CLAUDE.md                     ← THIS FILE
├── README.md                     ← human quick-start and LLM setup instructions
├── CHANGELOG.md                  ← template version history
├── .template-version             ← current semver
├── docs/
│   └── migration-v2.md           ← 1.x → 2.0.0 upgrade guide
├── global-config/
│   ├── agents/                   ← OpenCode global agents (deploy to ~/.config/opencode/agents/)
│   │   └── README.md
│   └── claudecode/               ← Claude Code global config (deploy to ~/.claude/)
│       ├── README.md             ← install instructions
│       ├── CLAUDE.md             ← global git rules → deploy to ~/.claude/CLAUDE.md
│       └── skills/               ← Claude Code skills (deploy to ~/.claude/skills/)
└── template/                     ← copy this into consuming projects
    ├── AGENTS.md                 ← OpenCode orchestrator dispatch table
    ├── CLAUDE.md                 ← Claude Code orchestrator + dispatch table
    ├── .abstract.md              ← L0 ultra-concise project map (~100 tokens)
    ├── .overview.md              ← L1 subsystem context stub
    ├── protocols.md              ← secrets, permission gates, code style
    ├── opencode.json             ← OpenCode config stub
    ├── .opencode/agents/         ← OpenCode agent templates
    ├── .claude/
    │   ├── settings.json         ← Claude Code project settings stub
    │   ├── agents/               ← Claude Code subagent templates
    │   └── skills/               ← Claude Code skill templates
    ├── kb/README.md              ← knowledge base index stub
    └── docs/sre-todos.md         ← deferred SRE findings stub
```

## Key rules for working on this repo

1. **Keep OpenCode and Claude Code equivalents in sync.** When a template file
   changes, update both `.opencode/` and `.claude/` versions.
2. **Do not modify placeholder substitution steps.** Every `<placeholder>` in
   `template/` must remain intact for the init process.
3. **Do not copy `global-config/` into the template deliverable.** Machine-scoped
   config is never project-scoped.
4. **Version every template change.** Bump `.template-version` and add a
   `CHANGELOG.md` entry for every change to `template/` files.

## Permission gates — always ask before:

- Bumping `.template-version` (requires a CHANGELOG entry first)
- Any change that would break backward compatibility for consuming projects
- Deleting or renaming files in `template/` that are referenced in README.md

## Session start

Check for interrupted tasks:

```bash
find . -name "in-progress.md" -not -path "./.git/*" 2>/dev/null
```
