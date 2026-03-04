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
    "git-flow": {
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

  # Deploy the agent file to the opencode config directory
  agents = {
    git-flow = ./agents/git-flow.md;
  };

  # Register the agent so opencode knows about it
  settings = {
    agent = {
      git-flow = {
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

### `git-flow.md`

Handles all git operations across any project:
- Stages and commits with conventional commit messages
- Creates and manages branches
- Opens pull requests via `gh`
- Two-phase push protocol — always summarises before pushing, requires
  explicit confirmation before `git push`
- Hard deny on `git push --force` to main/master

**Permissions (as shipped):**
| Command | Permission |
|---------|-----------|
| `git status`, `git diff`, `git log`, `git branch` | allow |
| `git add *` | allow |
| `git commit *` | ask |
| `git push *` | deny (requires human to run) |
| `git pull *` | allow |
| `git checkout *`, `git switch *` | allow |
| `git merge *` | ask |
| `gh pr create *` | deny |
| `gh pr *`, `gh issue *` | ask |
| everything else | ask |

Adjust the `permission.bash` map in your `opencode.json` or nix config to
suit your comfort level (e.g. change `git push *` to `ask` once you trust it).
