# Changelog — opencode-project-template

All notable changes to this template are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions use [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

When a project adopts this template it pins its version in `.template-local`
(gitignored, machine-specific). To upgrade, diff against the desired version
tag and apply changes manually or with the help of the orchestrator agent.

---

## [1.0.0] — 2026-03-07

### Added
- `template/opencode.json`: `"model": "anthropic/claude-sonnet-4-6"` and
  `"small_model": "anthropic/claude-haiku-4-5"` as project defaults. Establishes
  the three-tier Anthropic model hierarchy as a first-class template convention.
- `template/.opencode/agents/project-orchestrator.md`: `model: anthropic/claude-opus-4-6`
  frontmatter — orchestrators receive the most capable model; subagents inherit
  sonnet from the global default.

### Convention established
- **Orchestrators** → `anthropic/claude-opus-4-6` (pinned in agent frontmatter)
- **General reasoning / subagents** → `anthropic/claude-sonnet-4-6` (global `model`)
- **Lightweight tooling** (title generation, etc.) → `anthropic/claude-haiku-4-5` (global `small_model`)

---

## [0.9.0] — 2026-03-04

### Added
- `template/protocols.md` — extracted from `AGENTS.md`: secrets protocol,
  code style guidelines, permission gates, and helpful commands (including
  the `.template-local` format and template sync procedure). Keeps the
  AGENTS.md system prompt lean by moving stable reference material out of
  the injected context.
- `template/docs/session-log.md` — extracted from `AGENTS.md`: historical
  session log table. Prevents the system prompt from growing unboundedly
  with per-session history entries. AGENTS.md now references this file
  rather than inlining the log.
- `template/SUMMARY.md` promoted to a **project-root-level** orientation
  file (previously it was subsystem-scoped only). The root SUMMARY.md gives
  any agent enough context to orient in the full project — repository map,
  subagent quick-reference, stack at a glance, service endpoints, dev
  workflow, and pointers to per-subsystem SUMMARY.md files.

### Changed
- `template/AGENTS.md`: Secrets Protocol, Code Style Guidelines, Permission
  Gates, Helpful Commands, and Session Log sections removed. Each is now
  referenced by pointer (`protocols.md` or `docs/session-log.md`). File
  reduced from ~219 lines to ~147 lines. Added `SUMMARY.md` and
  `protocols.md` to the repository structure tree.
- `template/AGENTS.md`: Session start section updated — SRE deferred issues
  check added as step 2 (was missing from template). Template sync check
  now references `protocols.md` for the sync procedure rather than inlining it.

---

## [0.8.0] — 2026-03-04

### Added
- `template/` subdirectory: all files intended to be copied into a consuming
  project are now inside `template/`. Everything outside it is template-repo
  infrastructure (README, CHANGELOG, versioning, global-config). This makes
  the boundary between "copy this" and "do not copy this" unambiguous for a
  naive LLM.

### Changed
- `README.md` intro: explicit statement that this template adds agent
  scaffolding to an **existing** project and does not create application code,
  infra config, or runtime setup. Includes a bold "What this template is NOT"
  callout.
- `README.md` quick-start: instructions updated to copy from `template/`
  subdirectory, not the repo root.
- `README.md` repository layout: tree updated to show `template/` as the
  deliverable and everything outside it as template-repo infrastructure.
- `README.md` "What each file is for" table: all paths now prefixed with
  `template/`; `global-config/` row clarified with **NEVER copy** wording.
- `README.md` LLM setup instructions: Step 0 reframed — "you are NOT creating
  a new project"; all copy steps now reference `template/` source paths; SRE
  step notes that no baked investigation steps are needed; Step 4 "do NOT
  copy" list tightened; Step 5 checklist adds "no application code was
  created by this process".
- `template/.opencode/agents/project-sre.md`: rewritten as a cross-system
  observer rather than a baked first-responder. The agent now reads all
  subsystem `SUMMARY.md` and `function_signature.md` files to build a system
  model, then correlates across subsystems to diagnose problems. No
  hardcoded investigation steps — capability grows as subsystem docs grow.
- `template/AGENTS.md`: `global-config/` block removed from the repository
  structure tree (it is not project-scoped).

### Removed
- All top-level copyable files (`AGENTS.md`, `SUMMARY.md`, `opencode.json`,
  `.opencode/agents/`, `kb/`, `docs/`) moved into `template/`. They no longer
  exist at the repo root.

---

## [0.7.0] — 2026-03-04

### Added
- `.opencode/agents/project-orchestrator.md` — primary orchestrator agent
  template (`mode: primary`). Delegates all work to subagents; never edits
  files directly (`edit`/`write` tools disabled). Contains: decision tree for
  routing tasks to the right subagent, subagents table, session-start three-
  step check (interrupted tasks + SRE deferred issues + template sync), and
  permission gates. The write/edit tools are disabled in frontmatter.
- `default_agent` field in `opencode.json` stub, set to `"orchestrator"`.
  Projects rename their orchestrator file accordingly and match the value.
- Orchestrator row added to the Subagent Roster table in `AGENTS.md`, with a
  note explaining `default_agent` and the filename-to-value convention.
- Orchestrator entry added to the repository structure tree in `AGENTS.md`
  and `README.md`.
- `project-orchestrator.md` row added to the What each file is for table in
  `README.md` with copy/rename instructions.
- Step 0 in LLM setup instructions updated to read `project-orchestrator.md`
  before acting.
- Step 2 in LLM setup instructions: `opencode.json` section expanded with
  `default_agent` guidance and placeholder substitution table for
  `project-orchestrator.md`.
- Step 4 (do NOT copy) and Step 5 (verify) updated for the orchestrator.

### Changed
- `README.md` Agent architecture section: orchestrator description expanded to
  explain `mode: primary`, `default_agent`, and the disabled edit/write tools.

---

## [0.6.0] — 2026-03-04

### Added
- `.template-local` concept: a gitignored, machine-specific file that each
  consuming project creates once per checkout. Contains `template-version`
  (the currently-synced version) and `template-path` (absolute local path to
  the template repo). Documented with format and instructions in `AGENTS.md`.
- `Helpful Commands` section in `AGENTS.md` with the syncing workflow
  (5-step procedure) and `.template-local` format specification.
- `.template-local` entry added to `.gitignore`.
- Session-start **template sync check** in `AGENTS.md`: orchestrator reads
  `.template-local`, compares against `CHANGELOG.md` in `template-path`, and
  asks the human before syncing if the template has advanced.
- `Helpful Commands` entry added to the Table of Contents.

### Changed
- `AGENTS.md`: repository structure tree updated to include `.template-local`.
- `AGENTS.md`: `Session start` section renamed to
  "check for interrupted tasks and template updates" and expanded with the
  template sync check procedure.
- `.template-version` replaced by `.template-local` as the canonical
  version-tracking mechanism (`.template-local` includes both version and path,
  and is gitignored so it is machine-specific).

---

## [0.5.0] — 2026-03-03

### Added
- `README.md` — dual-audience documentation: human quick-start (bootstrap,
  manual copy, version pinning) and full LLM setup instructions (5-step
  process: read first, copy verbatim, copy and customise, stub
  `function_signature.md`, verify). Includes repo layout, file-by-file
  copy/skip table, placeholder substitution tables for every template file,
  and agent architecture diagram.

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
