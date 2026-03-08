# protocols.md — <project-name>

Operational protocols for agents working in this repository.
Reference this file rather than duplicating rules in AGENTS.md.

---

## Secrets Protocol

<!-- Adapt to your secrets manager: SOPS+age, Vault, AWS Secrets Manager, etc. -->

Never inline a secret value in a shell command. Read from disk at runtime.

| Source | When to use | Example |
|--------|-------------|---------|
| `.env` | Operational config (passwords, DSNs, API keys) | `$(cat .env \| grep KEY \| cut -d= -f2)` |
| `secrets/<name>` | Standalone credential files | `$(cat secrets/<name>)` |

Both `secrets/*` and `.env` are gitignored and never committed.

### Rules for agents

1. **Read from disk, never inline.** Use `$(cat secrets/<name>)` or
   `$(cat .env | grep KEY | cut -d= -f2)`.
2. **Missing secret — two paths:**
   - If safe to generate (e.g. a random token): generate it, write to the
     appropriate file, tell the human what was created.
   - If it must come from the human (e.g. a real API key): stop and ask.
3. **New secret files** follow the pattern `secrets/<service>_<key_name>`,
   all lowercase with underscores.
4. Never add a `!secrets/<file>` gitignore exception for real credential files.

### Secrets operations

| Operation | Command |
|-----------|---------|
| Encrypt | `sops --encrypt --age $(cat ~/.age/public.key) --in-place <file>` |
| Decrypt | `SOPS_AGE_KEY_FILE=~/.age/private.key sops --decrypt <file>` |
| Edit | `SOPS_AGE_KEY_FILE=~/.age/private.key sops <file>` |

**Never commit unencrypted secrets or credentials.**

---

## Code Style Guidelines

<!-- Fill in project-specific style rules -->

- 2-space indentation, no tabs
- Lowercase kebab-case for resource names
- Specific dependency versions — never floating/latest
- Always include resource bounds (memory, CPU, etc.)

---

## Permission Gates — always ask before:

1. Writing or modifying any secret or credential file.
2. Applying changes to a production system.
3. Running destructive commands (`delete`, `drop`, `reset`, `purge`).
4. Any action affecting more than one node/instance simultaneously.
5. Deleting git history or running `git push --force`.

---

## Dev Shell — direnv + Nix

This project uses `direnv` with `nix-direnv` to automatically activate the
correct Nix dev shell when you `cd` into a directory with a `flake.nix`.

**Machine prerequisites** (deployed via NixOS/home-manager, not this repo):
- `direnv` — automatically loads/unloads env vars per directory
- `nix-direnv` — provides the `use flake` directive for direnv
- Shell hook enabled (e.g. `programs.direnv.enable = true` in home-manager)

**Per-checkout setup** (one-time after clone):
```sh
direnv allow                    # project root
direnv allow <subsystem>/       # each subsystem with its own flake.nix
```

Each `.envrc` contains `use flake`, which activates the `flake.nix` in the
same directory. Subsystems with their own `flake.nix` get a dedicated shell
with subsystem-specific dependencies; subsystems without one inherit the
root shell.

This ensures every subprocess — including Claude Code subagents and Task tool
workers — runs inside the correct Nix dev shell with all native libraries
(e.g. `libstdc++.so.6` for numpy) on `LD_LIBRARY_PATH`.

---

## Helpful Commands

```sh
# Syncing from the template
# 1. Read .template-local to get template-path and current template-version
# 2. Read CHANGELOG.md in template-path to identify changes since template-version
# 3. Apply each Added/Changed/Removed item to the relevant files in this project
# 4. Update template-version in .template-local to the new version
# 5. Add a row to docs/session-log.md
```

### .template-local format

This file is gitignored and machine-specific. Create it once per checkout:

```yaml
template-version: <semver matching a version in the template CHANGELOG>
template-path: /absolute/path/to/opencode-project-template
```

The orchestrator reads this file at session start to detect when the template
has advanced beyond the pinned version and prompts the human to sync.
