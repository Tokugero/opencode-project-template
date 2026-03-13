# global-config/agents — global opencode agents

These agent files must be copied to your user-level opencode agents directory
**and** registered in your `opencode.json`. They are **not** picked up
automatically just by being in this folder.

---

## Installation

### Manual

```bash
mkdir -p ~/.config/opencode/agents
cp global-config/agents/*.md ~/.config/opencode/agents/
```

Add each agent to `~/.config/opencode/opencode.json` under `agent`:

```json
{
  "agent": {
    "<agent-name>": {
      "mode": "subagent",
      "permission": { "bash": "ask" }
    }
  }
}
```

### Via nix home-manager (recommended)

In your opencode home-manager module (e.g. `home/dev/opencode.nix`):

```nix
programs.opencode = {
  enable = true;

  agents = {
    <agent-name> = ./agents/<agent-name>.md;
  };

  settings = {
    agent = {
      <agent-name> = {
        mode = "subagent";
        permission = {
          bash = "ask";
        };
      };
    };
  };
};
```

---

## Agents in this directory

This directory ships no agents. Add your own project-specific agents here
and follow the installation steps above to deploy them.

**Git workflow** is handled by **agency-agents** — a companion tool deployed
separately. Install agency-agents and deploy it to
`~/.config/opencode/agents/`. See the agency-agents repository for
installation instructions.
