---
name: spec-implement
description: "Implement an explicitly referenced local spec through parallel worktree execution, persisted ticket acceptance, cumulative repair, and final full-suite validation."
disable-model-invocation: true
---

# Spec Implement

Implement one local spec through final closeout on the current branch, the **integration branch**. Treat its prepared tickets as a **task graph**: background implementers work across the ready **frontier** in separate branches and worktrees, and one merger integrates and accepts each ticket before it unlocks dependants. This skill owns scheduling and closeout; it does not invoke the user-only `ticket-implement` or `closeout-spec` skills or create a PR.

The model-invoked `ticket-review`, `spec-review`, `code-review`, and `tdd` skills must be available. Review skills own verdicts, acceptance-checkbox updates, and Review-result commits. This skill owns implementation, repairs, and the transition from ticket acceptance to final validation.

## 1. Resolve the spec and task graph

Resolve the current Git worktree root and branch, and record the initial index and worktree state. Keep this worktree as the integration destination and preserve unrelated changes throughout the run. If `docs/agents/issue-tracker.md` is missing or lacks `## Review operations` or `## Commit references`, tell the user to run `/setup-matt-pocock-skills` and stop.

Require one explicit input: either the normalized repo-relative `.scratch/<feature>/spec.md` path or its containing `.scratch/<feature>/` directory. A directory input must contain `spec.md` and the sibling `issues/` directory directly. Reject an `issues/` directory, shorthand name, glob, absolute path, parent escape, path outside `.scratch/`, missing target, or ambiguous target. When the input is absent, report that the explicit spec path or feature directory is required and stop. Do not infer the spec from session context, `HEAD`, commit trailers, branch names, or a workspace scan. Resolve the supplied target and every discovered ticket through the configured Issue Tracker operations without broadening that explicit feature boundary.

Inspect every Markdown file directly inside the spec's sibling `issues/` directory. Exclude files with a configured wayfinder `Type:` value (`research`, `prototype`, `grilling`, or `task`). Every other file is an implementation ticket, including untyped files; require an acceptance checklist on each. Stop if the implementation set is empty. Require the spec's authoritative `## Acceptance Criteria` set with stable identifiers, every identifier mapped by at least one ticket criterion as `(Spec AC-N)`, and no unknown identifier references. Report missing requirements or mappings instead of inventing them.

Keep graph discovery compact: retain each ticket's normalized reference, title, type, blockers, acceptance-checklist presence, and latest verdict. Use the configured Accepted ticket operation on the integration branch, including its persisted review check, rather than trusting a file's passing Verdict alone. Invalid passing persistence stops the run. Resolve every unfinished ticket's `Blocked by` references; each must identify an accepted ticket or a ticket in this graph. Stop on missing blockers, cycles, or an unfinished graph with no ready ticket.

Before any dispatch, ticket repair, or cumulative closeout, read [references/execution-preferences.md](references/execution-preferences.md) and initialize or restore `.scratch/<feature>/spec-implement.json` beside the resolved spec. Persist durable preference and constraint settings before proceeding; keep run-only overrides in the current run state as specified in that reference. This local orchestration file is separate from requirements and acceptance; keep it out of implementation and Review-result commits.

This skill explicitly authorizes the scheduler and delegated owners to choose supported subagent models and reasoning settings for each task without per-launch confirmation, within the effective preference and user constraints. Choose task boundaries, context handoff, and settings together, not from a fixed role or preference lookup. Preserve the same isolation, independent review, acceptance, and validation gates under every preference.

Before dispatching work, read [references/ticket-execution.md](references/ticket-execution.md). If an unfinished ticket has a persisted `Block` on the integration branch, follow **Retry a blocked ticket** first. Keep that ticket's contiguous suffix intact until it is accepted.

## 2. Work the frontier

The frontier contains unfinished, unclaimed tickets whose blockers have valid persisted acceptance on the integration branch. Use numeric filename order to break dispatch ties, then fill available implementation capacity. Keep claims, worker branch and worktree pointers, launch boundaries, source snapshots, and compact results in the current session. Claims are scheduler state, not Issue Tracker status or acceptance. Use commit identities only as internal boundaries, never in ticket files or user reports.

Communicate through **context pointers** to the spec, ticket, execution reference, worktree, and relevant notes. Include the effective preference and user constraints in every delegation or review-skill call; owners pass them onward when spawning their own reviewers. Follow the execution-preferences reference for selection records and serialized persistence. When several tickets need the same exploration, an optional exploration subagent may save shared notes outside the repo; pass the path to implementers rather than duplicating the findings.

1. Claim each ready ticket before dispatch. Take launch snapshots between integration transactions, when every previously integrated ticket is accepted. Create a dedicated branch and worktree from that integration `HEAD`, after confirming its blockers there. Give a fresh background implementer the context pointers, launch boundary, source snapshots, and unrelated-change boundary. It follows **Implementer** in the execution reference and writes only its assigned worktree.

2. As soon as an implementer finishes, release its agent slot and queue its branch for integration. A completed implementation is not accepted. Dispatch a **merger subagent** for one completed ticket at a time, passing the destination `HEAD` as this ticket's integration fixed point and following **Merger** in the execution reference. It is the sole writer to the integration worktree through replay, review, and any Blocking repairs. Other implementers may continue in their own worktrees.

3. After the merger returns, refetch the ticket on the integration branch and confirm acceptance through the configured operation. Only that persisted acceptance releases the claim and unlocks dependants. Retain compact checks, verdicts, findings, and repair-round counts, then refetch the graph and immediately dispatch newly ready tickets without waiting for unrelated running tickets or a whole batch to finish.

Fit concurrency to available agent slots, leaving capacity for integration and the review skills' independent reviewers. If a merger subagent cannot run with its reviewer, the owning session performs the same integration procedure. If isolated implementation is unavailable, process one ticket at a time in the owning worktree: record its fixed point before implementation, then follow **Implement, validate, and commit** and **Review and repair** directly, omitting branch replay. Report that fallback; independent review remains required.

An empty frontier while workers are running or awaiting integration means wait for their results. An unfinished graph with no ready, running, or awaiting-integration ticket is blocked. On a worker or merger stop condition, stop dispatch and integration, ask active workers to pause safely, and preserve their branches and worktrees. Report the unfinished work and its locations. Do not skip the unaccepted ticket or enter closeout.

Once every sibling implementation ticket is accepted and all workers and mergers are quiescent, proceed directly to **Cumulative closeout**, including when all tickets were already accepted at entry. Ticket acceptance is a phase boundary, not completion of this workflow. No separate user invocation of `closeout-spec` is needed.

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

## Cleanup and completion report

After successful closeout, remove only worktrees created by this run whose ticket work is integrated and accepted, whose workers have stopped, and whose remaining files contain no unpreserved work. Verify replayed work by its integrated changes and ticket acceptance, since cherry-pick changes commit identities. Keep any worktree with uncertain or unintegrated work and report its path; never force cleanup. Preserve the integration branch, pre-existing worktrees, and the spec's execution-preferences file. Flush pending selection records at a safe integration boundary, including on an incomplete stop.

Report the normalized spec reference, effective preference and configuration path, accepted and remaining ticket counts, ticket and cumulative repair-round counts, latest persisted verdicts, focused checks, final full-suite outcome, remaining Non-blocking findings, and worktree cleanup or preservation. For an incomplete run, identify the stopped phase, unresolved findings or criteria, preserved work and its locations, and exact input needed to continue. Distinguish implemented branches, integrated ticket acceptance, a passing cumulative review, and completed full validation; do not report commit hashes.
