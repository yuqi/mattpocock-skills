# Ticket execution

The scheduler owns claims and dispatch. Implementers produce ticket-scoped commits in isolated worktrees. The merger is the sole integration writer and owns the ticket's acceptance loop. Only persisted acceptance on the integration branch advances the task graph.

## Implementer

A Git worktree may lack local requirement or configuration files. Materialize the supplied spec, sibling tickets, and required repository guidance at their same repo-relative paths when needed, preserving their exact bytes and recording them as source inputs. Keep these inputs out of implementation commits. Use the launch commit's implementation and test inputs; if implementation depends on unrelated uncommitted work, report that dependency instead of importing it silently.

Refetch the normalized ticket and spec in the assigned worktree. Confirm the ticket belongs to the selected spec's sibling `issues/` directory, has an acceptance checklist, matches the supplied source snapshot, and has accepted blockers at the launch boundary. Stop on invalid passing persistence. Return without implementation edits if the ticket is already accepted; the scheduler must confirm that result on the integration branch.

Follow **Implement, validate, and commit** below. Return the ticket reference, branch and worktree pointers, checks run, and any stop reason. Leave ticket review, acceptance checkboxes, and Review blocks to integration; a worker's passing checks do not unlock dependants.

## Merger

Take exclusive ownership of the integration worktree until this ticket is accepted or stopped. Require its current `HEAD` to match the integration fixed point supplied by the scheduler, and preserve the recorded unrelated index and worktree changes. Confirm the integration branch and `HEAD` have not moved unexpectedly before every write. Stop on drift or overlap that cannot be separated safely.

Refetch the spec and selected ticket on the integration branch. Require them to match the implementation's source snapshots, recheck blocker acceptance, and verify the worker's launch boundary belongs to the integration history. If the selected ticket is already accepted, return without replaying it. Invalid passing persistence or changed requirements stop integration for reconciliation. Sibling tickets gaining valid acceptance do not by themselves invalidate this ticket's unchanged requirements.

Enumerate the worker commits after its launch boundary. Require a non-empty, merge-free range with exactly one matching ticket trailer and one matching spec trailer on every commit, parsed with `git interpret-trailers --parse`. Require implementation changes, no Review-result markers, and no changes to the spec or sibling ticket files. Reject mixed, missing, duplicated, malformed, or out-of-contract references and uncommitted implementation leftovers.

Integrate the validated ticket commits as one uninterrupted linear range: fast-forward when the destination is still the launch boundary, otherwise cherry-pick only that range onto the pinned integration `HEAD`. Preserve the trailers and both tickets' intended behavior when resolving conflicts. Keep unrelated commits outside the range, and introduce no merge commit. Stop on an empty implementation diff or conflicts requiring changed requirements or additional authority. Leave incomplete Git operations and their work intact when stopping.

The fixed point for review is the integration `HEAD` pinned before replay, not the worker's earlier launch boundary. Recheck the integrated range and run typechecking and focused tests on the combined implementation, resolving in-scope failures with ticket-scoped repair commits. Worker checks alone cannot validate replay against a newer base. Then follow **Review and repair**. Hold exclusive integration ownership through all repair rounds so other tickets cannot enter this ticket's review range.

Return the ticket reference, focused checks, latest verdict, persistence confirmation, repair-round count, Non-blocking findings, and any stop reason.

## Retry a blocked ticket

For a current `Block` on the integration branch, first require the configured Persisted ticket review check to pass. If the check is missing, fails, or cannot be confirmed, stop with files and commits intact; do not repair, automatically commit or overwrite the Review, or request another review to bypass invalid persistence.

Derive the original boundary: walk first parents backward from `HEAD` through the non-empty contiguous suffix whose commits each have exactly one parsed `Issue-Tracker-Ticket:` matching this ticket and exactly one `Issue-Tracker-Spec:` matching this spec. Use the first parent of the earliest suffix commit. Stop if `HEAD` does not match, any suffix commit is a merge or has malformed or mixed references, or the same ticket trailer appears earlier outside the suffix. After rebase, rederive this boundary from current commit objects, not from the prior Review block. Recheck the ticket's blockers before repairing.

Build the first repair set from every Blocking finding and every `Partial` or `Not met` criterion in its current Review block. Begin repairing that evidence before requesting another review. Apply the repair-scope and stopping rules below, then continue through **Implement, validate, and commit** and **Review and repair** with exclusive integration ownership. Do not use an unchanged review call to rediscover the persisted `Block`.

## Implement, validate, and commit

Call the Skill tool with "tdd" where possible, at pre-agreed seams. Run typechecking and focused tests during initial implementation and each repair. Resolve in-scope check failures before committing. Full-suite validation belongs to the final closeout gate, not to ticket rounds.

Commit all in-scope implementation and test changes while preserving unrelated index and worktree changes. Stop if overlapping pre-existing work cannot be separated safely. Keep the spec and sibling ticket files unchanged; review skills own acceptance-checklist and Review-block updates. Implementation and repair commits must carry these exact caller-provided trailers in one contiguous block:

```text
Issue-Tracker-Ticket: <normalized ticket reference>
Issue-Tracker-Spec: <normalized spec reference>
```

No external commit skill is required. If used, require it to preserve those exact values. Implementation commits must not carry an `Issue-Tracker-Review-Result:` marker. Do not make empty repair commits.

## Review and repair

Before every review, enumerate every commit after the ticket's integration fixed point through `HEAD`. Reject an empty range or any merge. Parse each message with `git interpret-trailers --parse`; require exactly one matching ticket trailer and exactly one matching spec trailer on every commit. Stop on missing, duplicated, malformed, mixed, or out-of-contract references.

Call the Skill tool with "ticket-review", passing the original integration fixed point and normalized ticket reference. It runs targeted tests before persisting a passing verdict; do not substitute inspected test code or a worktree-only verdict for its result. Refetch the ticket and verify its persisted verdict and Review-result commit through the configured Commit references.

- `Pass` or `Pass with follow-up`: return accepted and report any Non-blocking findings.
- Persisted `Block`: repair every Blocking finding and every `Partial` or `Not met` criterion against the latest evidence, then repeat implementation, focused validation, commit, and review against the original fixed point. Non-blocking findings remain follow-up work unless the user adds them to scope.
- Missing verdict, failed review, stale input, or unconfirmed persistence: stop without repair edits and preserve completed work.

Continue repair and review without a numeric round limit while safe, substantive in-scope repairs remain available. For reporting, count one complete ticket repair round after a substantive repair, focused validation, a commit, and a persisted review.

A repeated `Block` or one unsuccessful approach is not a reason to stop: investigate the new evidence and try a different supported repair. Stop only for invalid review or commit state, unsafe interference with unrelated work, or a demonstrated need for a product decision, requirement change, additional authority, or external-state change. Exhaust safe in-scope checks first. If no safe substantive repair can be attempted, stop and report the evidence instead of inventing changes. Report the unresolved findings, approaches tried, and input needed to proceed.
