# Changelog — opencode-project-template

All notable changes to this template are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions use [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

When a project adopts this template it pins its version in `.template-version`.
To upgrade, diff against the desired version tag and apply changes manually or
with the help of the orchestrator agent.

---

## [0.4.0] — 2026-03-03

### Added
- `global-config/` directory for user-level (non-project) opencode config.
- `global-config/README.md` — explains the distinction between global and
  project-scoped agents and how to deploy global agents manually or via nix.
- `global-config/agents/README.md` — per-agent installation instructions,
  permission table, and nix home-manager wiring snippet.
- `global-config/agents/git-flow.md` — the canonical global git flow agent:
  conventional commits, two-phase push protocol, hard deny on force-push to
  main, `gh` PR support.

### Changed
- `AGENTS.md`: repository structure tree updated to include `global-config/`.

---

## [0.3.0] — 2026-03-03

### Added
- `Scope` section in `AGENTS.md` clarifying what is managed in this repo
  (project-level agent prompts, `.opencode/` overrides) vs. what belongs in a
  central system config (models, MCP servers, global agent permissions).
- `git-flow` row in the Subagent Roster table, marking it as a global agent.
- `git-flow agent` section in `AGENTS.md` with the explicit rule that the
  orchestrator must never run git commands directly — it must relay human
  approval back to `@git-flow` rather than falling back to running git itself.
- `Scope` entry added to the Table of Contents.

### Changed
- `AGENTS.md`: Subagent Roster table and narrative sections updated to include
  git-flow as a first-class documented agent.

---

## [0.2.0] — 2026-03-03

### Added
- `function_signature.md` concept: each subsystem agent is now responsible for
  maintaining a `function_signature.md` file that maps every file and component
  in the subsystem so that agents can navigate without reading all source.
- `CHANGELOG.md` (this file) for tracking template evolution.
- `.template-version` file so consuming projects can pin and diff versions.

### Changed
- `AGENTS.md`: repository structure tree now includes `CHANGELOG.md`,
  `.template-version`, and `function_signature.md` entries. Delegation rules
  updated to require agents to read `function_signature.md` before working.
- `SUMMARY.md`: preamble updated to reference `function_signature.md`; added
  a section explaining its role.
- `.opencode/agents/project-role.md`: frontmatter description and opening
  instruction updated to include `function_signature.md`; full ownership and
  maintenance specification added as a new section.

---

## [0.1.0] — 2026-03-03

### Added
- Initial template structure: `AGENTS.md`, `SUMMARY.md`, `opencode.json`,
  `kb/README.md`, `docs/sre-todos.md`.
- `.opencode/agents/project-role.md` — generic subagent with ask-first
  protocol, interruption recovery (`in-progress.md`), and KB integration.
- `.opencode/agents/project-sre.md` — read-only SRE first-responder agent.

---

## Upgrade guide

To bring a project from one version to the next:

1. Check the project's `.template-version`.
2. Read the changelog entries between that version and the target version.
3. Apply the **Added** / **Changed** / **Removed** items to the project's own
   files, adapting placeholders as appropriate.
4. Update `.template-version` to the new version.
5. Update the project's `AGENTS.md` Session Log with the upgrade date.
