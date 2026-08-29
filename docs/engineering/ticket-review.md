## What it does

`ticket-review` is the acceptance gate for one implemented Issue Tracker [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket). It pins the committed diff, gives one [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) the ticket checklist and repository standards, then writes and commits Pass, Pass with follow-up, or Block in the ticket's current `## Review` block.

It reviews one slice, not the whole branch or umbrella spec. It never runs tests and never treats inspected test code as proof that verification passed.

## When to reach for it

Type `/ticket-review`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when one completed ticket needs acceptance.

| What needs review | Use |
| --- | --- |
| One ticket and its acceptance checklist | `ticket-review` |
| A broader branch diff | [code-review](https://aihero.dev/skills-code-review) |
| Every ticket under one completed spec | [spec-review](https://aihero.dev/skills-spec-review) |

## Prerequisites

The ticket must use a normalized repo-relative `.scratch/` reference through the Issue Tracker configured by [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills). Its generated contract must include current `Review operations` and `Commit references`; rerun setup to refresh an older file. Absolute paths and `..` escape are rejected. The resolved ticket must be writable local Markdown with an acceptance checklist and committed implementation. The review also needs the fixed point from before that ticket began and the model-invoked [code-review](https://aihero.dev/skills-code-review) skill as its Standards reference.

## One ticket, one gate

The ticket checklist is the boundary. One reviewer accounts for every criterion as Met, Partial, or Not met while also checking the committed diff against repository standards. Partial coverage and hard findings Block; bounded follow-up can remain only when every criterion is Met.

The review is replaceable rather than cumulative. A later run replaces the previous `## Review`, so one ticket has one current verdict instead of a history of competing answers.

## Persistence without another status model

Every completed review reconciles the checklist with its coverage: Met is checked, while Partial and Not met are unchecked. A passing verdict therefore checks every box, while Block may leave a partially accepted checklist. The review does not repurpose the ticket's triage `Status`.

The result is committed separately from the implementation. That commit changes only the ticket file and carries the same `Issue-Tracker-Ticket:` and optional `Issue-Tracker-Spec:` trailers as the implementation it evaluates. Requirement files are excluded from the implementation diff, so a retry reviews code and tests rather than the prior Review block. The commit lets a branch rebase cleanly and carries the verdict when that branch is merged into another worktree.

Later acceptance checks derive persistence from Git rather than trusting the file alone. The newest reachable non-merge commit that changed the ticket must change no other path, carry the exact ticket and optional spec trailers, and contain bytes identical to the current ticket. Only then can a passing Verdict count as completion.

## Common questions

**Why not use `code-review` for every ticket?**

You can, but it reviews Standards and Spec as two broad, separate reports and forms no acceptance verdict. `ticket-review` compresses those questions into one ticket-sized gate and persists the result with the checklist it judged.

**Does Pass mean the tests ran?**

No. The reviewer inspects committed test artifacts and any verification evidence supplied by implementation. The verification note states which checks are known to have run, or says that no executable result was available.

**What if the ticket changes while the reviewer is working?**

The result is discarded. The skill compares the current ticket with the exact source snapshot reviewed and refuses to merge an old verdict into newer requirements.

**Does it commit implementation changes?**

No. It commits only the normalized ticket file after the committed implementation has been reviewed. Unrelated staged, unstaged, and untracked changes remain untouched. A commit helper is optional, but if used it must preserve the exact caller-provided trailers.

## It's working if

- Every checklist item appears once in the coverage result.
- The runtime result reports the pinned implementation SHA without persisting a rebase-stale copy in the ticket.
- The review reads and writes one normalized `.scratch/` ticket reference.
- Every checkbox matches the latest coverage result, including after a regression or partial Block.
- Re-running replaces the old review instead of adding a second current verdict.
- The verification note distinguishes inspected evidence from checks known to have run.
- One Review-result commit changes only the ticket and carries its exact ticket and optional spec trailers.
- A passing file Verdict without matching committed bytes and trailers is not accepted.

## Where it fits

`ticket-review` is the per-slice gate inside [ticket-implement](https://aihero.dev/skills-ticket-implement). [to-tickets](https://aihero.dev/skills-to-tickets) creates the checklist it judges, [code-review](https://aihero.dev/skills-code-review) remains the broader branch review, and [spec-review](https://aihero.dev/skills-spec-review) checks cumulative acceptance after every slice is complete. [ask-matt](https://aihero.dev/skills-ask-matt) routes among those review grains.
