# Global Claude Code instructions

These rules apply to every Claude Code session on this machine.

---

## Git workflow

### Commit messages

Use conventional commits format: `type(scope): description`

- Focus on the "why" not the "what"
- Keep the subject line under 72 characters
- Common types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `ci`

### Two-phase push protocol

**Phase 1 — always present a summary before pushing:**

After staging and committing, show:
- Branch name and commit hash
- Commit message
- Files changed (`git diff --stat HEAD~1`)
- Proposed push command and target remote/branch
- Whether a PR would be opened and its draft title

Then stop and ask: "Ready to push? (or say 'push' to proceed)"

**Phase 2 — only proceed with explicit confirmation:**

If the user included "push" or "go ahead and push" in their original request,
treat that as pre-authorization and skip the prompt — but still print the
summary before executing.

### Safety rules — always enforced

- Never force-push to `main` or `master`
- Never amend commits that have been pushed to remote
- Always run `git status` and `git diff --staged` before committing to confirm scope
- Always confirm before destructive operations (`reset --hard`, `branch -D`, etc.)

---

## Permission gates — always ask before:

1. Writing or modifying any secret or credential file
2. Applying changes to a production system
3. Running destructive commands (`delete`, `drop`, `reset`, `purge`)
4. Any action affecting more than one node/instance simultaneously
5. Deleting git history or running `git push --force`
