# global-config — opencode project template

This directory contains configuration that **lives outside any individual
project repo** and must be installed into your user-level opencode config
directory (`~/.config/opencode/` or wherever `$XDG_CONFIG_HOME/opencode/`
resolves on your system).

These are not project-specific agents — they are **global agents** that
operate across all projects. They should be deployed once per user account,
not copied into every repo.

---

## Why global agents exist

Project agents (the ones in `.opencode/agents/`) are scoped to a single repo.
Global agents are registered in your user-level `opencode.json` and are
available in every opencode session regardless of which project you are in.

Typical global agents:
- **git-flow** — handles commits, branches, and PRs across all repos
- **scaffolding agents** — initialize new projects from templates

---

## How to deploy

### Manual

```bash
# Copy agents into the opencode global config directory
cp -r global-config/agents/ ~/.config/opencode/agents/
```

Then register each agent in your `~/.config/opencode/opencode.json`:

```json
{
  "agent": {
    "git-flow": {
      "mode": "subagent",
      "permission": { "bash": "ask" }
    }
  }
}
```

### Via nix (recommended)

If you manage your environment with nix + home-manager, use
`programs.opencode.agents` to deploy agent files and
`programs.opencode.settings.agent` to register them. This ensures every
machine gets the same global agents on rebuild.

See the example in `agents/README.md` for the full nix snippet.

---

## Contents

| Path | Description |
|------|-------------|
| `agents/README.md` | Per-agent installation instructions and nix wiring |
| `agents/git-flow.md` | Global git flow agent — commits, branches, PRs |
