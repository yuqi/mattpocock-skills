---
name: migrate-spec-acceptance
description: "Backfill stable acceptance identifiers and ticket mappings for a legacy local Markdown spec without changing requirement semantics."
disable-model-invocation: true
---

# Migrate Spec Acceptance

Repair acceptance traceability for one legacy local Markdown spec whose implementation tickets already exist. This is a requirements-metadata migration: preserve the meaning, wording, order, granularity, completion state, and existing Review verdict of every criterion.

The explicit invocation authorizes edits only to the resolved spec and its sibling implementation tickets, plus one ticket-only migration commit for each modified ticket whose acceptance was valid before migration. Do not implement requirements, run reviews, or commit the spec.

## Resolve the migration scope

Resolve the current Git worktree root first. Read `docs/agents/issue-tracker.md` when present and use its local ticket conventions.

Use an explicit spec reference when the user supplies one. Otherwise walk commits reachable from `HEAD` newest first, parse each message with `git interpret-trailers --parse`, and select the latest unique valid `Issue-Tracker-Spec:` value. If discovery is missing or ambiguous, enumerate repo-relative `.scratch/*/spec.md` candidates and ask the user for one exact reference instead of guessing from the branch name or timestamps.

Normalize the selected reference and require an existing Markdown file inside the current worktree's `.scratch/` directory. Reject absolute references, parent escape, symlink escape, and any other path outside the worktree.

Inspect every Markdown file directly inside the spec's sibling `issues/` directory. Exclude files with a configured wayfinder `Type:` value (`research`, `prototype`, `grilling`, or `task`); every remaining file is an implementation ticket. Stop if the set is empty or any implementation ticket lacks an acceptance checklist.

Before planning changes:

- Record `git status --short` and the current index and worktree diff for every target.
- Read and retain complete byte snapshots of the spec and every implementation ticket.
- Record each file's acceptance-criterion count, wording, order, and checkbox state.
- Evaluate and record whether each ticket currently passes the configured Accepted ticket operation.
- Treat all changes outside the resolved spec and implementation tickets as pre-existing and out of scope.

## Plan the complete migration

Find the spec's one authoritative acceptance set. Prefer `## Acceptance Criteria`; accept one unambiguous localized equivalent such as `## 验收条件`. Stop without writing if the spec has no acceptance set or has multiple plausible sets.

Build the identifier plan in memory:

- A stable identifier has the form `AC-N`, where `N` is a positive integer.
- If no criterion has an identifier, assign `AC-1` onward in current document order.
- If only some criteria have identifiers, preserve every existing identifier and assign unnumbered criteria new identifiers above the current maximum, in document order.
- Stop on duplicate identifiers, malformed attempted identifiers, or one identifier attached to multiple independent criteria.
- Never renumber an existing identifier or reuse a missing number.

Build the complete ticket mapping in memory before editing any file:

- Map every spec identifier to one or more existing ticket checklist items whose full semantics cover it.
- One spec criterion may map to several ticket items or tickets when its delivery is split.
- One ticket item may map to several spec criteria.
- Ticket-only criteria may remain unmapped.
- Preserve valid existing mappings. Stop on a mapping to an unknown spec identifier.
- Do not add, remove, split, merge, reorder, or reword a criterion to make mapping easier.

Every spec identifier must have at least one supported mapping. If any mapping remains ambiguous, show the unresolved identifiers and the smallest necessary questions, then stop with every file unchanged.

## Apply one metadata-only patch

Immediately before writing, refetch the spec and every implementation ticket and compare each one byte-for-byte with its snapshot. Stop on any drift.

Apply the completed migration as one patch:

1. Normalize the authoritative heading to `## Acceptance Criteria` when needed.
2. Insert each stable identifier after the spec criterion's list or checkbox marker, for example:

   ```markdown
   - [ ] AC-1. Existing criterion text
   ```

3. Append mappings to the relevant ticket checklist items, for example:

   ```markdown
   - [x] Existing ticket criterion text. (Spec AC-1) (Spec AC-3)
   ```

Preserve all original criterion text and punctuation before the appended mapping. Preserve checkbox state, `Status`, `Blocked by`, existing Review blocks, comments, and every unrelated byte. Do not modify decision tickets or any path outside the resolved migration scope. A checked ticket criterion remains checked because the mapping adds traceability, not a new acceptance condition.

## Preserve accepted-ticket persistence

Validate the complete metadata-only patch before creating any commit. A modified ticket is eligible to carry its existing acceptance forward only when:

- it passed the configured Accepted ticket operation before migration;
- its criterion wording, order, granularity, and checkbox state are unchanged after removing the appended mapping annotations;
- its `Status`, blockers, Review block, comments, and other content are byte-for-byte unchanged;
- the only additions are valid `(Spec AC-N)` annotations.

For each eligible modified ticket, create one commit that changes only that ticket while preserving every unrelated index and worktree change. Use `git commit --only -- <ticket-reference>` (with the message supplied through `-F -`) so unrelated staged paths cannot enter the commit. Add exactly one matching ticket trailer and one matching sibling spec trailer as one contiguous block:

```text
Issue-Tracker-Ticket: <repo-relative ticket reference>
Issue-Tracker-Spec: <repo-relative spec reference>
```

Use a migration title, not a review title. Verify from the resulting commit object that it changes only the ticket, contains exactly those reference trailers, and leaves the existing verdict, checkbox state, and mappings intact. This reestablishes the configured persisted-ticket check without claiming that another review ran.

Never combine several tickets into one commit. A batch commit would invalidate the ticket-only persistence rule. Leave the spec modification unstaged and uncommitted. Preserve its checkbox state until the later cumulative acceptance result projects coverage and persists the numbered spec together with its own verdict.

If a modified ticket was not Accepted before migration, leave it uncommitted and report that it still requires its normal acceptance workflow. If any non-metadata ticket change is detected, stop before committing that ticket and do not carry its verdict forward.

## Validate and report

Validate all of the following:

- Every spec criterion has exactly one unique stable identifier.
- Every spec identifier appears in at least one implementation-ticket mapping.
- No ticket references an unknown identifier or repeats the same mapping on one checklist item.
- Every file retains its original criterion count, order, wording, and checkbox state after ignoring inserted identifiers, appended mappings, and heading normalization.
- Status fields, blockers, Review blocks, comments, and excluded decision tickets are unchanged.
- Compared with the initial baseline, this migration changed only the resolved spec and mapped implementation tickets.
- Every carried-forward ticket has one verified ticket-only migration commit and still passes the configured Accepted ticket operation.
- The spec remains unstaged and uncommitted, and every unrelated index and worktree change matches the initial baseline.
- `git diff --check` passes for the remaining modified paths.

Report the normalized spec reference, implementation-ticket count, identifier set, modified files, complete `AC-N` to ticket-item mapping, validation results, each ticket migration commit SHA, carried-forward Accepted results, tickets that still require review, and confirmation that the spec and unrelated changes were not staged or committed.
