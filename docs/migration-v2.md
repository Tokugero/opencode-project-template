# Migration guide — 1.x → 2.0.0

This guide walks through upgrading a project from any 1.x template version to
2.0.0. See [CHANGELOG [2.0.0]](../CHANGELOG.md) for the full change list.

Estimated time: ~15 minutes for a project with 2–4 subsystems.

---

## What changed

| Category | Before (1.x) | After (2.0.0) |
|----------|-------------|--------------|
| Subsystem context | `<subsystem>/SUMMARY.md` | `<subsystem>/.overview.md` |
| Component index | `<subsystem>/function_signature.md` | Merged into `.overview.md` component table |
| Project map | _(none)_ | `.abstract.md` at project root |
| Context-update skill | `signatures-update` | `context-update` |
| OpenCode git agent | `git-flow` (global) | agency-agents (separate install) |

---

## Step-by-step checklist

### 1 — Rename SUMMARY.md files → .overview.md

For each subsystem directory that has a `SUMMARY.md`:

```bash
# Run from your project root
find . -name "SUMMARY.md" -not -path "./.git/*" -not -path "./node_modules/*"
```

For each result, rename and merge:

- [ ] Rename `<subsystem>/SUMMARY.md` → `<subsystem>/.overview.md`
- [ ] If a `<subsystem>/function_signature.md` exists alongside it, paste its
  content into the **Component table** section of `.overview.md`, then delete
  `function_signature.md`

### 2 — Create .abstract.md at project root

Create one `.abstract.md` in the project root. This is the L0 context file
(~100 tokens) loaded first by every agent.

Copy `template/.abstract.md` from the 2.0.0 template and fill in the
placeholders:

```markdown
# .abstract.md — <project-name>

Stack: <language/framework, key dependencies>
Subsystems: <subsystem1/>, <subsystem2/>, …
Orchestrator reads: CLAUDE.md (Claude Code) / AGENTS.md (OpenCode)
```

- [ ] `.abstract.md` created at project root

### 3 — Update agent files to L0/L1/L2 protocol

Each agent file in `.claude/agents/` and `.opencode/agents/` needs a context
loading section. Replace any reference to `SUMMARY.md` or
`function_signature.md` with the tiered protocol:

```markdown
## Context loading

1. **L0** — read `.abstract.md` (project root) for project map
2. **L1** — read `<subsystem>/.overview.md` for your subsystem
3. **L2** — read specific source files only when L1 is insufficient
```

Also add an **Anti-patterns** section if the template version you're upgrading
from doesn't have one. Copy from `template/.claude/agents/project-role.md`.

- [ ] `project-sre` agent updated (reads all `.overview.md` files instead of `SUMMARY.md` files)
- [ ] Each `project-<role>` agent updated (reads own `<subsystem>/.overview.md`)
- [ ] Anti-patterns section added to each agent

### 4 — Update dispatch tables (CLAUDE.md / AGENTS.md)

In your project's `CLAUDE.md` (Claude Code) and/or `AGENTS.md` (OpenCode):

- [ ] Repo structure tree: replace `SUMMARY.md` with `.abstract.md` and `.overview.md`
- [ ] Remove any `function_signature.md` entries from the tree
- [ ] Planning protocol references: update any mention of `SUMMARY.md` or
  `function_signature.md` to `.abstract.md` / `.overview.md`
- [ ] Remove `@git-flow` from the subagent roster (if present)

### 5 — Rename context-update skill (if installed)

If your project has `.claude/skills/signatures-update/`:

```bash
mv .claude/skills/signatures-update .claude/skills/context-update
```

Update the `SKILL.md` frontmatter `name:` field to `context-update`.

- [ ] Skill directory renamed (or skipped if not installed)

### 6 — Remove git-flow references

Search for remaining `git-flow` references:

```bash
grep -r "git-flow" . --include="*.md" --include="*.json" -l \
  --exclude-dir=.git --exclude-dir=node_modules
```

For each file found:
- Remove `@git-flow` from subagent rosters
- Remove git-flow from architecture diagrams
- Remove any `git-flow:` block from `opencode.json` agent registration

- [ ] No `git-flow` references remain in project files

### 7 — Update .gitignore (optional)

The 2.0.0 template's `.gitignore` includes `.abstract.md` is **not** gitignored
(it should be committed). If you have a blanket ignore for dotfiles, verify
`.abstract.md` and `**/.overview.md` are tracked.

- [ ] Context files are committed, not ignored

### 8 — Update .template-local

Update the version in your project's `.template-local`:

```yaml
template-version: 2.0.0
template-path: /absolute/path/to/opencode-project-template
```

- [ ] `.template-local` updated to `2.0.0`

---

## Quick search to verify completion

Run these after finishing to confirm no stale references remain:

```bash
# Should return nothing
grep -r "SUMMARY\.md\|function_signature" . --include="*.md" -l \
  --exclude-dir=.git --exclude-dir=node_modules

# Should return nothing
grep -r "git-flow\|signatures-update" . --include="*.md" --include="*.json" -l \
  --exclude-dir=.git --exclude-dir=node_modules
```
