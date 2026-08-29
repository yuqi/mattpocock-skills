# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Review operations

Used by `ticket-review` and `spec-review`.

- **Current review**: the latest top-level `## Review` block in the ticket or spec. A new completed review replaces it.
- **Review target**: resolve one normalized repo-relative ticket or spec reference under the current Git worktree's `.scratch/` directory. Reject missing, ambiguous, absolute, parent-escaping, or non-`.scratch/` references. Refetch and write the same normalized reference.
- **Persisted ticket review**: find the newest reachable non-merge commit that changes the normalized ticket reference. Reject a missing or ambiguous result. Require that commit to change only that ticket, that its committed ticket bytes equal the current file byte-for-byte, and that it contains exactly one matching `Issue-Tracker-Ticket:` trailer. When a sibling spec exists, require exactly one matching `Issue-Tracker-Spec:` trailer; otherwise reject any spec trailer.
- **Accepted ticket**: the current review has `Verdict: Pass` or `Verdict: Pass with follow-up`, and its persisted ticket review check passes. A passing file Verdict without that commit evidence is not accepted.
- **Ticket completion**: an accepted implementation ticket counts as done when evaluating another ticket's blockers.
- **Checklist projection**: after ticket review, checked items are exactly the criteria marked `Met` in the current review; `Partial` and `Not met` items are unchecked.
- **Write safety**: refetch every requirement source immediately before writing. If any source differs from the reviewed snapshot, reject the stale result instead of overwriting newer requirements.
- **Triage state**: review does not change the existing `Status:` line. Acceptance lives in the current review Verdict.

## Triage operations

Used by `/triage`.

- **Inventory**: scan `.scratch/*/issues/*.md`, excluding wayfinder decision tickets identified by their `Type:` field. An item needs attention when `Status:` is missing, equals the mapped `needs-triage` value, or equals the mapped `needs-info` value and has new local content after the latest Triage Notes.
- **Roles**: record exactly one `Category:` value (`bug` or `enhancement`) and one `Status:` value using the strings mapped by `triage-labels.md`. Replace the existing field value instead of adding another role field.
- **Comments**: append conversation entries under `## Comments`. Every AI-authored triage entry starts with the required triage disclaimer.
- **Triage notes**: replace the current top-level `## Triage Notes` section, or insert it before `## Comments` when absent. Content added after those notes is the local activity signal for a `needs-info` item.
- **Agent brief**: replace the current top-level `## Agent Brief` section, or insert it before `## Comments` when absent. Its acceptance checklist is the contract consumed by `ticket-implement` and `ticket-review`.
- **Wontfix**: set `Status:` to the mapped `wontfix` value and append the reason under `## Comments`. Do not delete the ticket file.
- **Request surface**: this adapter exposes local ticket files only. Skip workflow branches for request surfaces not defined here.
- **Write safety**: refetch the ticket immediately before every write. Reject stale edits instead of overwriting newer local content.

## Commit references

Used by `ticket-implement`, `ticket-review`, and `spec-review`.

- **Ticket-scoped commit**: every implementation, repair, and ticket Review-result commit adds `Issue-Tracker-Ticket: .scratch/<feature-slug>/issues/<NN>-<slug>.md`. When the sibling spec exists, it also adds `Issue-Tracker-Spec: .scratch/<feature-slug>/spec.md`. Keep both lines together in one contiguous trailer block.
- **Spec Review-result commit**: add `Issue-Tracker-Spec: .scratch/<feature-slug>/spec.md`. Commit only the spec file and preserve unrelated index and worktree changes.
- **Review metadata**: exclude the spec and sibling ticket requirement files from implementation diffs. Reviewers receive their current snapshots separately.
- **Reference format**: each value is one normalized repo-relative `.scratch/` path with no surrounding prose. Reject absolute paths, `..` escape, or references outside `.scratch/`.
- **Ticket review range**: before every ticket review, require at least one commit after the ticket fixed point through `HEAD` and reject merge commits in that range. Every commit must contain exactly one `Issue-Tracker-Ticket:` matching the selected ticket. When a sibling spec exists, every commit must also contain exactly one matching `Issue-Tracker-Spec:`; otherwise no spec trailer is allowed. Stop on missing, duplicated, malformed, mixed, or out-of-contract references.
- **Current spec**: an explicit spec reference wins. Otherwise, walk commits reachable from `HEAD` newest first and use the latest valid `Issue-Tracker-Spec:` trailer. Stop on a missing or ambiguous result.
- **Blocked ticket retry**: when the current ticket review is `Block`, require commits with that exact `Issue-Tracker-Ticket:` value to form one non-empty, contiguous first-parent suffix ending at `HEAD`. Use the first parent of the earliest matching commit as the ticket fixed point. Stop if the same ticket appears earlier outside that suffix, or if a suffix commit is a merge, has malformed references, or has a different sibling spec. Derive the suffix again after rebase.
- **Cumulative base**: among commits reachable from `HEAD` whose parsed `Issue-Tracker-Spec:` trailer equals the current spec, use the first parent of the earliest matching commit. Normal rebase rewrites the commit SHA but preserves the trailer, so derive this base again on every review.
- **Scope validation**: every non-merge commit after the cumulative base must carry the same `Issue-Tracker-Spec:` trailer. An `Issue-Tracker-Ticket:` trailer, when present, must resolve to one of the current spec's sibling tickets. Stop on mixed or unreferenced implementation history.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `.scratch/<effort>/map.md` (the Notes / Decisions-so-far / Fog body).
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. A `Type:` line records the ticket type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved`.
- **Blocking**: a `Blocked by: NN, NN` line near the top. A ticket is unblocked when every file it lists is `resolved`.
- **Frontier**: scan `.scratch/<effort>/issues/` for files that are open, unblocked, and unclaimed; first by number wins.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
- **Write safety**: one wayfinder session is the sole writer for a map. Refetch the map and selected child immediately before Claim and compare them with the snapshots already read. After Claim, retain the saved map and claimed child as new snapshots. Refetch both immediately before Resolve and stop if either differs from those post-Claim snapshots. Research subagents return findings to the owning session and do not write Issue Tracker files.
