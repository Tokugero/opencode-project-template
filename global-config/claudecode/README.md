# global-config/claudecode — global Claude Code config

These files are deployed once per workstation to `~/.claude/`. They are
**not** per-project — never copy them into a project repo.

---

## What's here

| File / dir | Deploy to | Purpose |
|-----------|-----------|---------|
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Global instructions loaded into every Claude Code session — git workflow rules, conventional commits, universal safety gates |
| `skills/git-flow/` | `~/.claude/skills/git-flow/` | Example skill: git operations. **Optional** — Claude Code has built-in git workflow. Install only if you want to override the default with the two-phase push protocol. |

---

## Installation

### Manual

```bash
# Global instructions (appended, not overwritten)
cat global-config/claudecode/CLAUDE.md >> ~/.claude/CLAUDE.md

# Skills (optional)
mkdir -p ~/.claude/skills
cp -r global-config/claudecode/skills/git-flow ~/.claude/skills/
```

### Via nix home-manager (recommended)

In your Claude Code home-manager module (e.g. `home/dev/claudecode.nix`):

```nix
programs.claude-code = {
  enable = true;

  memory.text = builtins.readFile ./claudecode/CLAUDE.md;

  # Optional: install the git-flow skill
  # skills = {
  #   git-flow = ./claudecode/skills/git-flow;
  # };
};
```

---

## Design notes

### Why no git-flow agent?

OpenCode's `git-flow` agent is replaced by **two things** in Claude Code:

1. **Built-in git workflow** — Claude Code natively understands git and follows
   `includeGitInstructions` conventions without a dedicated agent.
2. **`CLAUDE.md` rules** — The two-phase push protocol and conventional commits
   format from the original agent are captured as global rules in
   `global-config/claudecode/CLAUDE.md`, loaded into every session.

The `skills/git-flow/` skill is provided as an **example of the skill pattern**
and as an optional override if you want explicit `/git-flow` invocation
instead of ambient behavior.

### Skills vs. commands

- **Command** (`.claude/commands/name.md`) — single file, invoked by `/name`
- **Skill** (`.claude/skills/name/SKILL.md`) — directory with supporting files
  (scripts, templates, checklists); same invocation pattern

Use a skill when the supporting files meaningfully add capability (lookup
tables, scripts, step-by-step checklists). Use a command for simple prompts.
