---
name: ticket-implement
description: "Implement and accept one local Issue Tracker ticket, or work a local ticket queue sequentially."
disable-model-invocation: true
---

# Ticket Implement

Implement local Issue Tracker tickets with one implementation, commit, and acceptance boundary per ticket.

When the request names multiple ticket references or a local `issues/` directory, read and follow [references/queue.md](./references/queue.md) for the complete queue run. Otherwise follow the single-ticket workflow below.

## Single-ticket workflow

Use the configured Issue Tracker operations to resolve exactly one ticket. If `docs/agents/issue-tracker.md` is missing or does not define both `## Review operations` and `## Commit references`, tell the user to rerun `/setup-matt-pocock-skills` to refresh it.

Resolve the current Git worktree root before opening the ticket. Search only normalized repo-relative references under its `.scratch/` directory. Inspect candidate `Type:` fields before deciding whether an implementation shorthand is ambiguous, and exclude recognized wayfinder decision tickets from that candidate set. An explicit path to a decision ticket still resolves so it can be routed correctly below. Reject a missing, still-ambiguous, absolute, parent-escaping, or non-`.scratch/` reference.

Resolve the sibling `.scratch/<feature>/spec.md` when it exists, applying the same normalized-reference check. Keep the ticket and spec references repo-relative for commit metadata.

Refetch the ticket. If it has a configured wayfinder `Type:` value (`research`, `prototype`, `grilling`, or `task`), report that it is a decision ticket for `/wayfinder` and stop. Otherwise reject a ticket without an acceptance checklist.

Use the configured Accepted ticket operation. If it passes, report that the ticket is already accepted and stop before recording a fixed point or changing implementation. If the file has `Verdict: Pass` or `Verdict: Pass with follow-up` but its persisted ticket review check fails, report invalid persistence and stop rather than skipping or starting implementation. Otherwise confirm every `Blocked by` ticket passes the same Accepted ticket operation. When the ticket's current verdict is not `Block`, record the current `HEAD` as this ticket's fixed point.

For a current `Block`, first require the configured Persisted ticket review check to pass. If the check is missing, fails, or cannot be confirmed, stop with files and commits intact; do not repair, automatically commit or overwrite the Review, or request another review to bypass invalid persistence. Then derive the original ticket fixed point from current history. Starting at `HEAD`, walk first parents backward through the non-empty contiguous suffix whose commits each have exactly one parsed `Issue-Tracker-Ticket:` equal to the selected ticket and, when its sibling spec exists, exactly one matching `Issue-Tracker-Spec:`. Use the first parent of the earliest suffix commit as the fixed point. Stop if `HEAD` does not match, any suffix commit is a merge or has malformed or mixed references, or the same ticket trailer appears earlier outside the suffix. Recompute this boundary from rewritten commits after rebase; do not read a fixed point from the prior Review block.

After deriving that boundary, build the first repair set from every Blocking finding and every `Partial` or `Not met` criterion in the ticket's current Review block. Begin repairing that evidence before returning to review; do not spend an unchanged review call rediscovering the persisted `Block`. Apply the repair-scope and stopping rules below. The repaired implementation then continues through **Implement and verify**.

## Implement and verify

Call the Skill tool with "tdd" where possible, at pre-agreed seams. Run typechecking and focused tests during the initial implementation and each repair round.

Once the selected checks pass, commit every in-scope implementation and test change to the current branch while preserving unrelated worktree changes. Add these caller-provided Git trailers together in one contiguous trailer block:

```text
Issue-Tracker-Ticket: <repo-relative ticket reference>
Issue-Tracker-Spec: <repo-relative sibling spec reference, when it exists>
```

No external commit skill is required. If one is used, supply these exact trailer lines and require it to preserve them without inferring or rewriting their values.

## Review and repair

Before every review, enumerate every commit after the recorded or derived fixed point through current `HEAD`. Stop if the range is empty or contains a merge commit. Parse every message with `git interpret-trailers --parse` and require exactly one `Issue-Tracker-Ticket:` matching the selected ticket on every commit. When the sibling spec exists, require exactly one matching `Issue-Tracker-Spec:` on every commit; otherwise reject any spec trailer. Stop on a missing, duplicated, malformed, mixed, or out-of-contract reference. Use current `HEAD` as the reviewed commit only after the complete range passes.

Call the Skill tool with "ticket-review" using the recorded or derived fixed point and ticket reference. Refetch the ticket and verify the persisted verdict and Review-result commit described by the configured Commit references.

- `Pass` or `Pass with follow-up` completes the ticket. Report any remaining Non-blocking findings.
- A persisted `Block` starts a repair round in the same ticket worker, not a handoff to the user or the next ticket.
- A missing verdict, failed review, stale input, or unconfirmed persistence stops without repair edits, preserving completed implementation and reviews.

Repair every Blocking finding and every `Partial` or `Not met` criterion against the current review's evidence. Keep requirements unchanged and leave checklist and Review-block updates to `ticket-review`; Non-blocking findings remain follow-up work unless the user adds them to scope. Return to **Implement and verify**, commit the repairs with the same reference trailers, then repeat **Review and repair** against the original fixed point so every round covers the initial implementation and all repairs. Rederive the same boundary after rebase using the `Block` history rule above.

Continue repair and review without a numeric round limit while safe, substantive in-scope repairs remain available. A repeated `Block` or one unsuccessful approach is not a reason to stop: investigate the latest evidence and try a different supported repair.

Stop only for an invalid review or commit state, or a demonstrated blocker requiring a product decision, requirement change, additional authority, external-state change, or unsafe interference with unrelated work. Exhaust safe in-scope checks before declaring such a blocker. If investigation establishes that no safe substantive repair can be attempted, stop and explain that evidence instead of inventing changes. Report the unresolved findings, approaches tried, and input needed to proceed. Preserve completed commits and reviews; do not create empty repair commits.

After every ticket under an umbrella spec is accepted, tell the user to choose `/spec-review` for one cumulative verdict or the beta `/closeout-spec` for the review and Blocking-repair loop. Do not invoke either from this workflow.
