---
name: template-update
description: Incorporate an insight, protocol, or convention from another agent session into the opencode-project-template. Extracts the pattern, places it in the right template file, and bumps the version. Invoke with /template-update when you have a conversation excerpt or lesson learned that should be codified into the template.
context: fork
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash
  - Glob
  - Grep
---

# template-update

Codifies an insight from another agent session into the opencode-project-template.
Handles placement, versioning, and changelog in a single pass.

## When to use

Invoke with `/template-update` when you have a conversation excerpt, lesson
learned, or architectural pattern from another project that should be baked
into the template so all future consuming projects inherit it.

## Inputs

The user must provide (inline or as a file path):

- **Conversation excerpt or insight description** — the raw context from the
  other session. Can be a pasted conversation, a summary, or a direct
  statement of what the rule/protocol should be.
- **Target** (optional) — which template file to update. If omitted, the
  skill will determine the best fit.

## Steps

### 1. Read current state

```bash
cat .template-version
```

Read the files that are candidates for the update:
- `template/CLAUDE.md` — orchestration rules, delegation protocols, session behaviour
- `template/protocols.md` — secrets, permission gates, code style
- `global-config/claudecode/CLAUDE.md` — machine-scoped global rules (not per-project)

Determine which file owns the kind of rule being added:
- Agent behaviour / delegation patterns → `template/CLAUDE.md`
- Code style / secrets / permission gates → `template/protocols.md`
- Machine-scoped rules that apply to all projects globally → `global-config/claudecode/CLAUDE.md`

### 2. Extract and phrase the pattern

From the provided context, extract:
- The **trigger condition** (when does this pattern apply?)
- The **steps or rules** (what should the agent do?)
- The **rationale** (why is this better than the default behaviour?)

Phrase it as a second-person instruction to the orchestrator agent ("When X,
do Y because Z"), matching the voice and style of the target file.

### 3. Identify insertion point

Read the target file. Find the section that best fits the new content:
- New delegation protocol → under `## Delegation rules`
- New session behaviour → under `## Session start`
- New permission gate → under `## Permission gates`
- New agent roster entry → under `## Subagent roster`

If no existing section fits, create a new `###` subsection under the most
relevant `##` section.

### 4. Apply the change

Edit the target file to insert the new content. Preserve:
- Existing heading hierarchy
- Markdown list and code block style
- The concise, imperative tone of the surrounding text

### 5. Update CHANGELOG.md

Add a new entry at the top of `CHANGELOG.md` (above the current latest version):

```markdown
## [X.Y.Z] — YYYY-MM-DD

### Added / Changed
- `<target-file>` — one-sentence summary of what was added and why.
```

The new version must be a minor bump (e.g. 1.10.0 → 1.11.0) because adding
a protocol is a backward-compatible feature addition. Only use a patch bump
for pure wording fixes.

### 6. Bump .template-version

```bash
echo "X.Y.Z" > .template-version
```

### 7. Confirm

Print a summary:
- What was added and where
- New version
- One-line CHANGELOG entry

## Expected output

- Target template file updated with the new protocol or rule
- `CHANGELOG.md` has a new entry at the top
- `.template-version` reflects the new version
- A brief summary printed to the user confirming what changed
