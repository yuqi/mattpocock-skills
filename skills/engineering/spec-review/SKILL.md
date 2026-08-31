---
name: spec-review
description: "Review a completed multi-ticket spec at committed HEAD across Standards, Spec, and Integration, then persist its cumulative acceptance verdict. Use for the final umbrella gate or from a review and repair workflow."
---

# Spec Review

Review the completed implementation of one umbrella spec across three independent axes: **Standards**, **Spec**, and **Integration**. The owning session forms one cumulative verdict, then writes and commits one current review block back to the Issue Tracker spec.

Never edit implementation or run executable verification. Inspected tests are evidence, not proof that they ran. A spec is accepted when its latest review Verdict is `Pass` or `Pass with follow-up`; no new global spec status is introduced.

The Issue Tracker configuration and model-invoked `code-review` skill must be available. If `docs/agents/issue-tracker.md` is missing or does not define both `## Review operations` and `## Commit references`, tell the user to rerun `/setup-matt-pocock-skills` to refresh it.

## 1. Resolve the sources and cumulative scope

Resolve the current Git worktree root before opening any requirement source. Normalize every source and commit-trailer reference as a repo-relative `.scratch/` path. Reject absolute paths, `..` escape, and references outside `.scratch/`.

Use an explicit spec reference when the user supplies one. Otherwise walk commits reachable from `HEAD` newest first, parse each message with `git interpret-trailers --parse`, and use the latest valid `Issue-Tracker-Spec:` value. Stop if discovery is missing, ambiguous, malformed, or out of contract.

Inspect every Markdown reference directly inside the spec's sibling `issues/` directory. Exclude files with a configured wayfinder `Type:` value (`research`, `prototype`, `grilling`, or `task`), then treat every remaining file as an implementation ticket. Stop if that set is empty, a reference is outside `.scratch/`, an implementation ticket lacks an acceptance checklist, or any implementation ticket fails the configured Accepted ticket operation. A passing file Verdict with invalid persistence is not accepted. Preserve complete snapshots of the spec and every included ticket for the later drift check. Never exclude an untyped file merely because its checklist is missing.

The spec must contain one authoritative `## Acceptance Criteria` set with stable identifiers. Confirm that every identifier appears in at least one sibling ticket checklist as `(Spec AC-N)` and that no ticket references an unknown identifier. Collect every acceptance criterion from every included implementation ticket, including ticket-only criteria without a spec identifier. Remove prior top-level `## Review` blocks from the spec and tickets before giving requirements to reviewers, but retain their complete original content for persistence checks.

Collect every commit reachable from `HEAD` whose parsed `Issue-Tracker-Spec:` trailer exactly matches the repo-relative spec reference. Stop if none exists. Use the first parent of the earliest matching commit as the cumulative fixed point. If the user supplied a fixed point, require it to resolve to this derived commit rather than replacing the derived boundary.

Inspect every non-merge commit after the cumulative fixed point through `HEAD`. Require exactly one matching `Issue-Tracker-Spec:` trailer on each. Any `Issue-Tracker-Ticket:` trailer must resolve to one of the enumerated sibling tickets. Stop on mixed, missing, duplicated, malformed, or out-of-contract references. This validation is repeated from current commit objects on every run, so a rebase produces a new validated boundary instead of reusing stored SHAs.

Resolve the cumulative committed boundary:

```bash
git rev-parse <fixed-point>
git rev-parse HEAD
git merge-base <fixed-point> <reviewed-commit>
git status --short
```

Construct the exact implementation diff command with one exclusion pathspec for the normalized spec and each enumerated sibling ticket:

```bash
git diff <base-commit> <reviewed-commit> -- . ':(exclude)<spec-reference>' ':(exclude)<ticket-reference>' ...
```

The requirement sources are supplied to reviewers as snapshots, not as implementation evidence. Use this exact filtered command everywhere reviewers are told to inspect the cumulative diff.

Stop if the fixed point is invalid, the committed diff is empty, or the umbrella acceptance set is missing. Exclude staged, unstaged, and untracked implementation. Reviewers read surrounding implementation from Git objects at `<reviewed-commit>`.

Before spawning reviewers, confirm the spec is writable local Markdown and every requirement source still matches its snapshot. Otherwise stop without forming a verdict.

Call the Skill tool with "code-review" to load its repository-standards precedence and Fowler smell baseline as reference, without running its review workflow.

## 2. Run three axes in parallel

Spawn three subagents in parallel, one per axis. Give all reviewers the same pinned commits, exact diff command, umbrella spec, relevant tickets, and dirty implementation-path list. They inspect repository and Git evidence only, run no executable checks, and do not invoke review skills or spawn more subagents.

Give the Standards reviewer the repository standards plus the repository-precedence rules and complete smell baseline loaded from `code-review`, pasted in full. Do not add that Standards-only material to the Spec or Integration reviewer prompts.

### Standards

> Report material documented-standard violations and Fowler smells in the cumulative committed diff. Repository standards override the smell baseline. Classify a finding as Blocking only when a hard rule or demonstrated acceptance risk must be fixed; otherwise classify it as Non-blocking. Cite file, evidence, and impact.

### Spec

> Account for every umbrella acceptance identifier and every acceptance criterion from each included implementation ticket against the current cumulative committed diff. Keep umbrella coverage and per-ticket coverage separate. Identify ticket criteria by normalized ticket reference and exact criterion wording. Mark each Met, Partial, or Not met and map it to committed implementation or test evidence. Partial and Not met are Blocking. Report incorrect or unrequested behavior and contradictions between ticket criteria and the umbrella spec, classifying each finding as Blocking or Non-blocking.

### Integration

> Review the implementation as one system. Trace cross-ticket contracts and relevant end-to-end paths. Report supported risks in state or data transitions, compatibility, migrations, error paths, security boundaries, and integration coverage. Classify each as Blocking or Non-blocking and cite the evidence and impact.

A failed or cancelled axis produces no cumulative verdict and no spec write. Before forming a verdict, validate that the Spec result covers every umbrella identifier and every included ticket criterion exactly once. Missing or duplicated coverage produces no verdict.

## 3. Form the cumulative verdict

Keep findings under their originating axes and apply one gate:

- **Pass**: every umbrella and ticket criterion is Met and all axes have no actionable findings.
- **Pass with follow-up**: every umbrella and ticket criterion is Met, all axes have no Blocking findings, and bounded Non-blocking findings remain.
- **Block**: any umbrella or ticket criterion is Partial or Not met, or any axis has a Blocking finding.

Missing executable-verification results do not downgrade coverage. State their absence in the verification note.

## 4. Replace the current review

Immediately before writing, refetch the spec and every ticket source and compare them byte-for-byte with the complete snapshots supplied to reviewers. If any changed, report stale inputs and do not write.

Replace the spec's latest top-level `## Review` block, or append one when absent. Preserve every unrelated byte.

When the authoritative acceptance set uses task-list checkboxes, reconcile each checkbox from the validated Umbrella coverage before writing the review: check an identifier only when its result is `Met`, and leave or make it unchecked when its result is `Partial` or `Not met`. Preserve every identifier, criterion wording, and criterion order. Do not introduce checkboxes into a spec whose authoritative acceptance set does not already use them. Spec review owns this cumulative checkbox projection; ticket review and metadata migration do not.

```markdown
## Review

Reviewed at: <ISO-8601 timestamp>
Verdict: <Pass | Pass with follow-up | Block>

### Standards
- <Blocking | Non-blocking finding, or None>

### Spec
#### Umbrella coverage
- <AC-N>: <Met | Partial | Not met>, <evidence>

#### Ticket coverage
- <repo-relative ticket reference>
  - <exact criterion>: <Met | Partial | Not met>, <evidence>

#### Findings
- <Blocking | Non-blocking finding, or None>

### Integration
- <Blocking | Non-blocking finding, or None>

### Verification note
- <checks known to have run, or no executable-verification result was available>
```

Do not change the spec's triage `Status` or ticket review blocks. Refetch the spec after writing and verify the new review, the acceptance-checkbox projection, and preservation of unrelated content.

Commit only the normalized spec file while preserving every unrelated index and worktree change. Add these caller-provided trailers as one contiguous block:

```text
Issue-Tracker-Review-Result: spec
Issue-Tracker-Spec: <repo-relative spec reference>
```

No external commit skill is required. If one is used, supply these exact trailer lines and require it to preserve them without inferring or rewriting their values. Verify that the new commit changes only the spec, contains exactly one `Issue-Tracker-Review-Result: spec` trailer, exactly one matching spec trailer and no ticket trailer, and leaves the persisted verdict and projected acceptance checkboxes intact. A failed or contaminated commit means persistence is not confirmed; stop and report it.

Present the runtime result under separate `## Standards`, `## Spec`, and `## Integration` headings. Under each axis, include every Blocking and Non-blocking finding returned for that axis, verbatim or lightly cleaned, with its evidence and impact; write `None` only when that axis has no findings. Do not merge, rerank, suppress, or replace findings merely because another axis determines the cumulative verdict. Under Spec, also report umbrella and ticket coverage totals and list every Partial or Not met criterion; the complete Met evidence matrix remains in the persisted spec and need not be repeated in the runtime result. End with the normalized spec reference, cumulative verdict, verification note, and whether persistence was confirmed. A `Block` result must still report all three axes, not only its decisive finding or commit metadata. Do not persist or report the pinned implementation commit or Review-result commit hash.

When the verdict is `Block`, also report the exact `Issue-Tracker-Spec: <repo-relative spec reference>` trailer that every follow-up implementation commit must preserve before this review is run again.
