---
name: spec-implement
description: "Implement an explicitly referenced local spec end to end: work its prepared ticket queue sequentially, persist ticket acceptance, then repair cumulative review findings and pass final full-suite validation. Use when the user provides the spec file or its containing feature directory and wants implementation through closeout, not a concurrent PR build."
---

# Spec Implement

Implement one local spec through final closeout on the current branch. This skill contains its own queue and closeout workflows; it does not invoke the user-only `ticket-implement` or `closeout-spec` skills. It does not create branches, worktrees for implementation, or PRs.

The model-invoked `ticket-review`, `spec-review`, `code-review`, and `tdd` skills must be available. Review skills own verdicts, acceptance-checkbox updates, and Review-result commits. This skill owns implementation, repairs, and the transition from ticket acceptance to final validation.

## 1. Resolve the spec and queue

Resolve the current Git worktree root and record the initial index and worktree state. Preserve unrelated changes throughout the run. If `docs/agents/issue-tracker.md` is missing or lacks `## Review operations` or `## Commit references`, tell the user to run `/setup-matt-pocock-skills` and stop.

Require one explicit input: either the normalized repo-relative `.scratch/<feature>/spec.md` path or its containing `.scratch/<feature>/` directory. A directory input must contain `spec.md` and the sibling `issues/` directory directly. Reject an `issues/` directory, shorthand name, glob, absolute path, parent escape, path outside `.scratch/`, missing target, or ambiguous target. When the input is absent, report that the explicit spec path or feature directory is required and stop. Do not infer the spec from session context, `HEAD`, commit trailers, branch names, or a workspace scan. Resolve the supplied target and every discovered ticket through the configured Issue Tracker operations without broadening that explicit feature boundary.

Inspect every Markdown file directly inside the spec's sibling `issues/` directory. Exclude files with a configured wayfinder `Type:` value (`research`, `prototype`, `grilling`, or `task`). Every other file is an implementation ticket, including untyped files; require an acceptance checklist on each. Stop if the implementation set is empty. Require the spec's authoritative `## Acceptance Criteria` set with stable identifiers, every identifier mapped by at least one ticket criterion as `(Spec AC-N)`, and no unknown identifier references. Report missing requirements or mappings instead of inventing them.

Keep queue discovery compact: retain each ticket's normalized reference, title, type, blockers, acceptance-checklist presence, and latest verdict. Use the configured Accepted ticket operation, including its persisted review check, rather than trusting a file's passing Verdict alone. Invalid passing persistence stops the run. Resolve every unfinished ticket's `Blocked by` references; each must identify an accepted ticket or a ticket in this queue. Stop on missing blockers, cycles, or an unfinished queue with no ready ticket.

## 2. Work the frontier

Choose the first unfinished ticket whose blockers are accepted, using numeric filename order among ready tickets. Run only one ticket worker at a time in this worktree.

Dispatch each ready ticket to a fresh subagent. Give it the normalized ticket and spec references, the initial unrelated-change boundary, and the complete **Single-ticket worker** section below. It refetches the ticket itself and invokes `ticket-review` with its nested reviewer. It returns only the ticket reference, checks run, latest verdict, persistence confirmation, repair-round count, remaining findings, and any stop reason, not commit hashes. If nested subagents are unavailable, execute that section directly in the owning session.

After each worker, refetch its ticket and confirm acceptance through the configured operation. Only valid persisted acceptance advances the frontier. Preserve compact results and round counts for reporting, then refetch the queue and blockers before selecting another ticket. A worker that stops unaccepted stops the whole run; do not skip it or enter closeout.

Once every sibling implementation ticket is accepted, proceed directly to **Cumulative closeout**, including when all tickets were already accepted at entry. Ticket completion is a phase boundary, not completion of this workflow. No separate user invocation of `closeout-spec` is needed.

## Single-ticket worker

### Establish the ticket boundary

Refetch the ticket and confirm it is an implementation ticket with an acceptance checklist in the selected spec's sibling `issues/` directory. Recheck persisted acceptance and blockers. Return without implementation edits if it is already accepted; stop on invalid passing persistence or an unaccepted blocker.

When its current verdict is not `Block`, record the current `HEAD` as the ticket's fixed point. For a current `Block`, first require the configured Persisted ticket review check to pass. If the check is missing, fails, or cannot be confirmed, stop with files and commits intact; do not repair, automatically commit or overwrite the Review, or request another review to bypass invalid persistence. Then derive the original boundary: walk first parents backward from `HEAD` through the non-empty contiguous suffix whose commits each have exactly one parsed `Issue-Tracker-Ticket:` matching this ticket and exactly one `Issue-Tracker-Spec:` matching this spec. Use the first parent of the earliest suffix commit. Stop if `HEAD` does not match, any suffix commit is a merge or has malformed or mixed references, or the same ticket trailer appears earlier outside the suffix. After rebase, rederive this boundary from current commit objects, not from the prior Review block.

After deriving a blocked ticket's boundary, build the first repair set from every Blocking finding and every `Partial` or `Not met` criterion in its current Review block. Begin repairing that evidence before requesting another review. Apply the repair-scope and stopping rules below, then continue through **Implement, validate, and commit**. Do not use an unchanged review call to rediscover the persisted `Block`.

### Implement, validate, and commit

Call the Skill tool with "tdd" where possible, at pre-agreed seams. Run typechecking and focused tests during initial implementation and each repair. Resolve in-scope check failures before committing. Full-suite validation belongs to the final closeout gate, not to ticket rounds.

Commit all in-scope implementation and test changes while preserving unrelated index and worktree changes. Stop if overlapping pre-existing work cannot be separated safely. Keep acceptance criteria unchanged and leave checklist and Review-block updates to the review skills. Implementation and repair commits must carry these exact caller-provided trailers in one contiguous block:

```text
Issue-Tracker-Ticket: <normalized ticket reference>
Issue-Tracker-Spec: <normalized spec reference>
```

No external commit skill is required. If used, require it to preserve those exact values. Implementation commits must not carry an `Issue-Tracker-Review-Result:` marker. Do not make empty repair commits.

### Review and repair

Before every review, enumerate every commit after the ticket's fixed point through `HEAD`. Reject an empty range or any merge. Parse each message with `git interpret-trailers --parse`; require exactly one matching ticket trailer and exactly one matching spec trailer on every commit. Stop on missing, duplicated, malformed, mixed, or out-of-contract references.

Call the Skill tool with "ticket-review", passing the original fixed point and normalized ticket reference. It runs targeted tests before persisting a passing verdict; do not substitute inspected test code or a worktree-only verdict for its result. Refetch the ticket and verify its persisted verdict and Review-result commit through the configured Commit references.

- `Pass` or `Pass with follow-up`: return accepted and report any Non-blocking findings.
- Persisted `Block`: repair every Blocking finding and every `Partial` or `Not met` criterion against the latest evidence, then repeat implementation, focused validation, commit, and review against the original fixed point. Non-blocking findings remain follow-up work unless the user adds them to scope.
- Missing verdict, failed review, stale input, or unconfirmed persistence: stop without repair edits and preserve completed work.

Continue repair and review without a numeric round limit while safe, substantive in-scope repairs remain available. For reporting, count one complete ticket repair round after a substantive repair, focused validation, a commit, and a persisted review.

A repeated `Block` or one unsuccessful approach is not a reason to stop: investigate the new evidence and try a different supported repair. Stop only for invalid review or commit state, unsafe interference with unrelated work, or a demonstrated need for a product decision, requirement change, additional authority, or external-state change. Exhaust safe in-scope checks first. If no safe substantive repair can be attempted, stop and report the evidence instead of inventing changes. Report the unresolved findings, approaches tried, and input needed to proceed.

## 3. Cumulative closeout

Refetch every sibling implementation ticket and require valid persisted acceptance before entering this phase. Keep the same normalized spec reference. This phase reviews the cumulative implementation, including previously accepted tickets and every later spec-level repair.

1. Call the Skill tool with "spec-review", passing the explicit spec reference. It performs lightweight independent review without executable verification. Refetch the spec and confirm that the current verdict and Review-result commit were persisted successfully.

   - `Pass` or `Pass with follow-up`: proceed to **Final full-suite validation**, retaining all Non-blocking findings for the report.
   - Persisted `Block`: build the repair set from every Blocking finding plus every `Partial` or `Not met` umbrella or ticket criterion, preserving the originating axis and criterion mapping.
   - Missing verdict, failed axis, stale input, contaminated commit, or unconfirmed persistence: stop without implementation edits.

2. Repair the implementation against the cited evidence. Keep the spec, sibling tickets, acceptance criteria, and Review blocks unchanged; reviewers own their updates. Non-blocking findings stay out of scope unless the user includes them. Stop and request the precise input when progress requires a product decision, changed requirements, additional authority, or an external change.

3. For a behavioral repair at a testable seam, call the Skill tool with "tdd". Run focused checks before committing and resolve failures introduced by the repair. Do not run the full test suite during repair rounds.

4. Commit only in-scope repair changes, preserving unrelated index and worktree changes. Stop on inseparable overlap or when no substantive repair is available. The commit must carry exactly one caller-provided spec trailer:

   ```text
   Issue-Tracker-Spec: <normalized spec reference>
   ```

   Spec-level repair commits must not carry ticket or Review-result trailers and must not change the spec or sibling ticket files. Verify committed paths, trailers, and remaining worktree state. A commit helper must preserve the exact supplied trailer; no empty repair commit is allowed.

5. Return to step 1. Diagnose repeated blockers against their new evidence before editing. Continue while a substantive in-scope repair is available; stop when the next attempt would only repeat the same change or expand approved requirements.

Count one complete cumulative repair round after a substantive repair, focused validation, a repair commit, and the following persisted spec review have all completed. Carry this count across a final-validation failure and its return to the closeout loop.

## 4. Final full-suite validation

After a persisted cumulative `Pass` or `Pass with follow-up`, run the repository's full test suite against the final reviewed implementation immediately before declaring completion. Use a checkout whose implementation and test inputs match the reviewed version, isolating unrelated dirty changes when needed. Before reporting success, confirm that the reviewed implementation and passing result still apply to current `HEAD`. A previously stored passing spec verdict does not bypass this gate on resume.

- Full suite passes: complete the spec and report the result.
- Failure caused by an in-scope implementation problem: use it as the next repair set, follow cumulative closeout steps 2 through 5, then attempt final validation again only after another persisted passing spec review.
- Unrelated failure, external limitation, or inability to complete trustworthy validation: stop and report incomplete closeout.

This is the only full-suite gate in the workflow. It can run again after a repair changes the implementation; ticket acceptance or a passing review alone is never proof of final completion.

## Completion report

Report the normalized spec reference, accepted and remaining ticket counts, ticket and cumulative repair-round counts, latest persisted verdicts, focused checks, final full-suite outcome, and remaining Non-blocking findings. For an incomplete run, identify the stopped phase, unresolved findings or criteria, preserved work, and exact input needed to continue. Distinguish an accepted queue, a passing cumulative review, and completed full validation; do not report commit hashes.
