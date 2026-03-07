# Commit message checklist

Use this when drafting a commit message for the current changes.

## Format

```
type(scope): description

[optional body]

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

## Type reference

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `chore` | Build process, dependency updates, tooling |
| `docs` | Documentation only |
| `test` | Adding or correcting tests |
| `ci` | CI/CD pipeline changes |
| `perf` | Performance improvement |
| `revert` | Reverts a previous commit |

## Checklist before committing

- [ ] Subject line under 72 characters
- [ ] Type is one of the types above
- [ ] Scope reflects the subsystem touched (e.g. `api`, `infra`, `auth`)
- [ ] Description explains the *why*, not just the *what*
- [ ] No secrets or credentials in staged files
- [ ] `git diff --staged` reviewed — no unintended files included
