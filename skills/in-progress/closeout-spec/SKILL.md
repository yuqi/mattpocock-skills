---
name: closeout-spec
description: "Close a multi-ticket spec whose tickets are already accepted by reviewing, repairing blockers, committing, and repeating until its cumulative verdict can pass."
disable-model-invocation: true
---

# Closeout Spec

Close the implementation of one multi-ticket spec with a review and repair loop. The loop ends only when the current persisted spec review has no Blocking finding and every acceptance criterion is Met.

This skill owns implementation repairs. The model-invoked `spec-review` skill owns independent acceptance, the cumulative verdict, acceptance-checkbox projection, and its Review-result commit. The model-invoked `spec-review`, `code-review`, and `tdd` skills must be available.

## Establish the boundary

Record the initial worktree and index state so unrelated changes remain identifiable throughout the run. Reuse an explicit spec reference supplied by the user; otherwise let `spec-review` discover it from the current committed history.

The Issue Tracker configuration required by `spec-review` must be present, and every sibling implementation ticket must already have valid persisted acceptance. This workflow does not replace `ticket-review`. If review asks for refreshed setup, tell the user to run `/setup-matt-pocock-skills` and stop. If a ticket is not accepted, report its reference and stop at that earlier boundary.

## Review and repair loop

1. Call the Skill tool with "spec-review". Pass the explicit spec reference when one was supplied.

2. Accept a review round only when it formed a cumulative verdict and confirmed persistence of the Review-result commit. Refetch the normalized spec and confirm its latest top-level `## Review` block carries that verdict.

   - `Pass` proceeds to final validation.
   - `Pass with follow-up` proceeds to final validation and leaves every Non-blocking finding visible in the final report.
   - `Block` opens one repair round.
   - A missing verdict, failed axis, stale input, contaminated commit, or unconfirmed persistence stops the loop without implementation edits.

3. Build the repair set from every Blocking finding plus every `Partial` or `Not met` umbrella or ticket criterion in the current review. Preserve its mapping back to the originating axis and criterion. Non-blocking findings remain follow-up work unless the user explicitly adds them to the repair set.

   Repair the implementation against the cited evidence. Do not edit the spec, sibling tickets, their acceptance criteria, or their Review blocks to make the gate pass. When a blocker requires a new product decision, a requirement change, additional authority, or an external-state change, stop and ask for exactly that input.

4. For a behavioral repair at a testable seam, call the Skill tool with "tdd". Run focused checks for every repair before committing. Resolve failures introduced by the repair. Do not run the full test suite during repair rounds.

5. Commit only the in-scope repair changes. Preserve every unrelated index and worktree change, and stop if an overlapping pre-existing change cannot be separated safely. The commit must carry exactly one caller-provided trailer:

   ```text
   Issue-Tracker-Spec: <normalized repo-relative spec reference>
   ```

   The repair commit must not carry an `Issue-Tracker-Review-Result:` trailer and must not change the spec or its sibling ticket files. Verify the committed paths, trailer, and remaining worktree state. A repair round with no substantive in-scope change produces no empty commit and stops as no progress.

   No external commit skill is required. If one is used, supply the exact trailer above and require it to preserve the value without inference or rewriting.

6. Return to step 1. A repeated blocker triggers diagnosis against its new cited evidence before another edit. Continue while a substantive in-scope repair is available; stop when the next attempt would only repeat the same change or expand the approved requirements.

## Final validation

After a persisted `Pass` or `Pass with follow-up`, run the repository's full test suite against the reviewed `HEAD` immediately before declaring closeout complete. This is the only full-suite gate in this workflow.

If it passes, complete the closeout. If it fails because of an in-scope implementation problem, use the failures as the next repair set, follow the repair and commit rules above, then return to `spec-review` before attempting final validation again. Stop and report an unrelated failure or external limitation that prevents trustworthy final validation. Never report success without a passing full-suite result for the final reviewed implementation.

## Completion report

On success, report the normalized spec reference, final verdict, number of repair rounds, focused checks, final full-suite result, and every remaining Non-blocking finding. On an incomplete run, report the current verdict, unresolved Blocking findings or criteria, completed repair rounds, checks run, and the precise reason the loop stopped.
