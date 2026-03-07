---
name: git-flow
description: Git operations — stage, commit, branch, PR. Enforces two-phase push and conventional commits. Optional — Claude Code has built-in git workflow; install this skill only to override it with explicit /git-flow invocation.
context: fork
allowed-tools:
  - Bash
  - Read
---

# git-flow

Handles all git operations for the current project with a two-phase push
protocol and conventional commits enforcement.

> **This skill is optional.** Claude Code includes built-in git workflow
> instructions. Install this skill only if you want explicit `/git-flow`
> invocation instead of ambient git behavior.

## When to use

Invoked with `/git-flow` when you want to stage, commit, branch, or open a PR
with explicit confirmation before any push.

## Steps

### 1. Inspect repo state

```bash
git status
git diff --staged
git log --oneline -5
```

Report what is staged, unstaged, and the recent history.

### 2. Stage changes (if needed)

Ask which files to stage. Never `git add -A` without confirmation.

```bash
git add <specific-files>
```

### 3. Commit

Use the checklist in `./checklist.md` to draft the commit message.

```bash
git commit -m "$(cat <<'EOF'
type(scope): description

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

### 4. Phase 1 — present push summary

Before pushing, always print:
- Branch name and commit hash
- Commit message
- Files changed: `git diff --stat HEAD~1`
- Proposed push command and target remote/branch
- Whether a PR would be opened and its draft title

Then stop and ask: **"Ready to push? (or say 'push' to proceed)"**

### 5. Phase 2 — push (only after explicit confirmation)

```bash
git push -u origin <branch>
```

### 6. Pull request (if applicable)

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
- <what changed>

## Test plan
- [ ] <how to verify>

🤖 Generated with Claude Code
EOF
)"
```

## Safety rules

- Never force-push to `main` or `master`
- Never amend commits that have been pushed to remote
- Always confirm before `git reset --hard`, `git branch -D`, or any rebase
- `gh pr create` requires explicit confirmation — never fire automatically
