---
name: setup-matt-pocock-skills
description: "Configure this repo for the engineering skills: set up its issue tracker, triage state vocabulary, and domain doc layout. Run once before first use of the other engineering skills."
disable-model-invocation: true
---

# Setup Matt Pocock's Skills

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker**: local markdown files under `.scratch/`
- **Triage labels**: the `Status:` values used for the five canonical triage state roles
- **Domain docs**: where `CONTEXT.md` and ADRs live, and the consumer rules for reading them

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `AGENTS.md` and `CLAUDE.md` at the repo root: does either exist? Is there already an `## Agent skills` section in either?
- `CONTEXT.md` and `CONTEXT-MAP.md` at the repo root
- `docs/adr/` and any `src/*/docs/adr/` directories
- `docs/agents/`: does this skill's prior output already exist?
- `.scratch/`: a sign that a local-markdown issue tracker convention is already in use
- Is the `triage` skill installed? (a `triage` skill folder alongside this one, or `triage` in your available skills.) This decides whether Section B runs at all.
- Monorepo signals: a `pnpm-workspace.yaml`, a `workspaces` field in `package.json`, or a populated `packages/*` with its own `src/`. These are present only in a genuinely large multi-package repo; their absence means single-context, which is almost every repo.

### 2. Present findings and ask

Summarise what's present and what's missing. Then take the sections in order. One section, one answer, then the next.

Lead each configurable section with the recommended answer so the user can accept it in a word. Give a one-line explainer only when the choice genuinely branches; skip the section entirely when exploration already settled it (Section B when `triage` isn't installed, Section C when there's no monorepo). Section A is fixed and needs no question.

**Section A: Issue tracker.**

The issue tracker has one implementation: local markdown files under `.scratch/<feature>/`. Tell the user this is what setup will configure, then use [issue-tracker-local.md](./issue-tracker-local.md) for `docs/agents/issue-tracker.md`.

These setup-owned headings are machine-readable contract identifiers:

- In `docs/agents/issue-tracker.md`: `## Review operations`, `## Triage operations`, `## Commit references`, and `## Wayfinding operations`, plus every inline Markdown heading identifier from [issue-tracker-local.md](./issue-tracker-local.md).
- In the chosen instruction file: `## Agent skills`, `### Issue tracker`, `### Domain docs`, and the conditional `### Triage labels`.

Emit every applicable contract heading exactly once in both the draft and written files, preserving its English spelling, capitalization, spacing, and heading level in every locale. Preserve each inline artifact-heading identifier verbatim. Adapt surrounding prose while keeping these headings and the contracts they identify intact.

**Section B: Triage state vocabulary.** Skip this section entirely if the `triage` skill isn't installed (exploration told you), since an uninstalled skill needs no state mapping.

If it is installed, ask exactly one question:

> Do you want to keep the default triage status values? (recommended: **yes**)

The defaults are the five canonical state roles, each local `Status:` value equal to its name: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. On **yes**, write them as-is. Only if the user says no, usually because their local tickets already use other values (e.g. `bug:triage` for `needs-triage`), collect the overrides so `triage` uses the established vocabulary instead of inventing another value.

**Section C: Domain docs.** Default to **single-context** (one `CONTEXT.md` + `docs/adr/` at the repo root). This fits almost every repo; write it without asking.

Offer **multi-context** (a root `CONTEXT-MAP.md` pointing to per-context `CONTEXT.md` files) only when exploration found monorepo signals. Then confirm which layout they want.

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `AGENTS.md` / `CLAUDE.md` is being edited (see step 4 for selection rules)
- The contents of `docs/agents/issue-tracker.md`, `docs/agents/domain.md`, and `docs/agents/triage-labels.md` (the last only when `triage` is installed)

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- If `AGENTS.md` exists, edit it.
- Else if `CLAUDE.md` exists, edit it.
- If neither exists, ask the user which one to create; don't pick for them.

Never create `CLAUDE.md` when `AGENTS.md` already exists (or vice versa); always edit the one that's already there. A `CLAUDE.md` symlink or pointer to `AGENTS.md` keeps the same block available to both harnesses without duplicating it.

If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block:

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the triage state vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout: "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

Include the `### Triage labels` sub-block, and write `docs/agents/triage-labels.md`, only when `triage` is installed and Section B ran. When it isn't, both are omitted.

Then write the docs files using the seed templates in this skill folder as a starting point:

- [issue-tracker-local.md](./issue-tracker-local.md): local-markdown issue tracker
- [triage-labels.md](./triage-labels.md): state-role value mapping (only if `triage` is installed)
- [domain.md](./domain.md): domain doc consumer rules + layout

### 5. Done

Refetch `docs/agents/issue-tracker.md` and the chosen instruction file. Verify that every applicable contract heading listed in step 2 appears exactly once at its specified heading level and that every inline artifact-heading identifier remains verbatim before reporting completion.

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `docs/agents/*.md` directly later; re-run this skill when they need to refresh generated contracts from the current templates or restart setup.
