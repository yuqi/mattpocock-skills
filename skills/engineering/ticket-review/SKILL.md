---
name: ticket-review
description: "Review one implemented ticket against its acceptance criteria and committed diff, then write the latest verdict back to the ticket. Use when one Issue Tracker ticket needs a focused acceptance gate."
---

# Ticket Review

Review one ticket at committed `HEAD` with one reviewer. The reviewer combines repository Standards and ticket Spec coverage at the ticket's own grain, then the owning session writes and commits one current verdict back to the Issue Tracker ticket.

Never edit the implementation or run executable verification. Inspected tests are evidence, not proof that they ran. A ticket is accepted only when its latest review Verdict is `Pass` or `Pass with follow-up` and the configured persisted ticket review check passes; no new global ticket status is introduced.

The Issue Tracker configuration and model-invoked `code-review` skill must be available. If `docs/agents/issue-tracker.md` is missing or does not define both `## Review operations` and `## Commit references`, tell the user to rerun `/setup-matt-pocock-skills` to refresh it.

## 1. Resolve the ticket and committed scope

Obtain the fixed point from before the ticket began and the ticket reference containing its acceptance criteria. Reuse unambiguous inputs already established in the session; ask only when an ambiguity changes the reviewed scope.

Resolve the current Git worktree root before opening the ticket. Use the configured `docs/agents/issue-tracker.md` operations to resolve exactly one normalized repo-relative `.scratch/` reference. Reject a missing, ambiguous, absolute, parent-escaping, non-`.scratch/`, or non-Markdown reference.

Resolve the sibling `.scratch/<feature>/spec.md` when it exists, applying the same normalized-reference check. Keep both references repo-relative for the Review-result commit metadata.

Fetch that normalized ticket reference and keep the complete source snapshot for the later write, but remove any existing top-level `## Review` block from the requirements given to the reviewer.

Resolve and pin the review boundary:

```bash
git rev-parse <fixed-point>
git rev-parse HEAD
git merge-base <fixed-point> <reviewed-commit>
git status --short
```

Construct the exact implementation diff command with the normalized ticket reference excluded. Also exclude the sibling spec reference when it exists:

```bash
git diff <base-commit> <reviewed-commit> -- . ':(exclude)<ticket-reference>' ':(exclude)<sibling-spec-reference>'
```

Omit the sibling-spec exclusion when no sibling spec exists. The ticket and spec are requirement sources supplied separately, not implementation evidence. Use this exact filtered command everywhere the reviewer is told to inspect the diff.

Stop if the fixed point is invalid, the committed diff is empty, or the ticket has no acceptance criteria. Staged, unstaged, and untracked implementation changes are outside the review. When surrounding code is needed, read it from Git objects at `<reviewed-commit>`, not from the worktree.

Before review, refetch the same normalized ticket reference and confirm it is writable and still matches the snapshot already read. Otherwise report the limitation and stop without forming a verdict.

## 2. Run one focused reviewer

Call the Skill tool with "code-review" to load its repository-standards precedence and Fowler smell baseline as reference, without running its review workflow.

Spawn one subagent reviewer. Give it the pinned base and reviewed commits, exact diff command, ticket requirements without the prior review block, repository standards, and the repository-precedence rules plus complete smell baseline loaded from `code-review`, pasted in full. Then give it this brief:

> Account for every ticket acceptance criterion. Mark each Met, Partial, or Not met and cite committed implementation or test evidence. Partial and Not met are Blocking. Also report material repository-standard violations, incorrect or unrequested behavior, and Fowler smells with demonstrated acceptance risk. Classify each additional finding as Blocking or Non-blocking. Read implementation only from the supplied diff or Git objects at the reviewed commit. Do not run tests, builds, linters, typecheckers, benchmarks, migrations, servers, or application code. Do not invoke another review skill or spawn subagents; perform this review directly. Return a compact coverage list followed by findings with classification, file, evidence, and impact.

A failed or cancelled reviewer produces no verdict and no ticket write.

## 3. Form the verdict

Apply one fixed gate:

- **Pass**: every criterion is Met and there are no actionable findings.
- **Pass with follow-up**: every criterion is Met, there are no Blocking findings, and bounded Non-blocking findings remain.
- **Block**: any criterion is Partial or Not met, or any Blocking finding exists.

The absence of executable-verification results is not a finding. Record it explicitly in the verification note.

## 4. Replace the current review

Immediately before writing, refetch the same normalized ticket reference and compare it byte-for-byte with the complete source snapshot. If it changed, the verdict is stale: report the drift and do not write.

Replace the latest top-level `## Review` block in place, from that heading to the next top-level `##` heading. When no review exists, insert the new block before `## Comments`, or at the end when no Comments section exists. Preserve every unrelated byte.

```markdown
## Review

Reviewed at: <ISO-8601 timestamp>
Verdict: <Pass | Pass with follow-up | Block>

### Coverage
- <criterion>: <Met | Partial | Not met>, <evidence>

### Findings
- <Blocking | Non-blocking finding, or None>

### Verification note
- <checks known to have run, or no executable-verification result was available>
```

Reconcile every acceptance checkbox with the completed coverage result: check `Met`, and uncheck `Partial` or `Not met`. Preserve each criterion's wording and order. Do not change the ticket's triage `Status`.

Refetch after the write and verify the new block, checkbox outcome, and preservation of unrelated content.

Commit only the normalized ticket file while preserving every unrelated index and worktree change. Add the caller-provided trailers as one contiguous block:

```text
Issue-Tracker-Review-Result: ticket
Issue-Tracker-Ticket: <repo-relative ticket reference>
Issue-Tracker-Spec: <repo-relative sibling spec reference, when it exists>
```

No external commit skill is required. If one is used, supply these exact trailer lines and require it to preserve them without inferring or rewriting their values. Verify that the new commit changes only the ticket, contains exactly one `Issue-Tracker-Review-Result: ticket` trailer, exactly one matching ticket trailer and the required sibling spec trailer, and leaves the ticket's persisted verdict intact. A failed or contaminated commit means persistence is not confirmed; stop and report it.

Report the normalized ticket reference, verdict, and whether persistence was confirmed. Do not persist or report the pinned implementation commit or Review-result commit hash; current scope is rederived from commit trailers when needed.
