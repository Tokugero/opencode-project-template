# global-config/claudecode — global Claude Code config

These files are deployed once per workstation to `~/.claude/`. They are
**not** per-project — never copy them into a project repo.

---

## What's here

| File / dir | Deploy to | Purpose |
|-----------|-----------|---------|
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Global instructions loaded into every Claude Code session — git workflow rules, conventional commits, universal safety gates |
| `agents/security-audit.md` | `~/.claude/agents/security-audit.md` | OWASP Top 10 static security analysis. Reads cached OWASP reference docs, auto-detects project tech stack, audits changed files. Pinned to opus. |
| `agents/code-review.md` | `~/.claude/agents/code-review.md` | Language-agnostic code quality review. Checks test coverage, DRY, SOLID, naming, error handling, API design. Pinned to opus. |
| `agents/perf-engineer.md` | `~/.claude/agents/perf-engineer.md` | Dynamic performance analysis in local/dev environments. Measures latency, scaling, caching, cost. Sandboxes itself. Pinned to opus. |
| `security/owasp-*.md` | `~/.claude/security/owasp-*.md` | Cached OWASP Top 10 reference docs (Web, API, LLM). Read by security-audit agent at startup. Update manually when OWASP publishes new versions. |
| `skills/git-flow/` | `~/.claude/skills/git-flow/` | Example skill: git operations. **Optional** — Claude Code has built-in git workflow. Install only if you want to override the default with the two-phase push protocol. |

---

## Installation

### Manual

```bash
# Global instructions (appended, not overwritten)
cat global-config/claudecode/CLAUDE.md >> ~/.claude/CLAUDE.md

# Global agents
mkdir -p ~/.claude/agents ~/.claude/security
cp global-config/claudecode/agents/*.md ~/.claude/agents/
cp global-config/claudecode/security/*.md ~/.claude/security/

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

  agents = {
    security-audit = ./claudecode/agents/security-audit.md;
    code-review = ./claudecode/agents/code-review.md;
    perf-engineer = ./claudecode/agents/perf-engineer.md;
  };

  # Optional: install the git-flow skill
  # skills = {
  #   git-flow = ./claudecode/skills/git-flow;
  # };
};

# OWASP reference docs for security-audit agent
home.file.".claude/security/owasp-web-top10.md".source = ./claudecode/security/owasp-web-top10.md;
home.file.".claude/security/owasp-api-top10.md".source = ./claudecode/security/owasp-api-top10.md;
home.file.".claude/security/owasp-llm-top10.md".source = ./claudecode/security/owasp-llm-top10.md;
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
