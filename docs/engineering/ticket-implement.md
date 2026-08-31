## What it does

`ticket-implement` implements and accepts local Issue Tracker [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket). Point it at one ticket, an explicit ticket list, or a feature's `issues/` directory. Each ticket gets its own fixed point, implementation commit, and committed `ticket-review` verdict.

The defining constraint is the ticket boundary. A multi-ticket queue is still worked one ticket at a time, in dependency order, with a fresh [context](https://www.aihero.dev/ai-coding-dictionary/context) for every ticket. Generic spec or conversation work stays on [implement](https://aihero.dev/skills-implement), whose upstream workflow is unchanged.

## When to reach for it

You invoke this by typing `/ticket-implement`; the agent won't reach for it on its own.

| Your input | Reach for |
| --- | --- |
| One local ticket | `/ticket-implement .scratch/<feature>/issues/01-<slug>.md` |
| An explicit ticket queue | `/ticket-implement .scratch/<feature>/issues/01-<slug>.md .scratch/<feature>/issues/02-<slug>.md` |
| Every ticket for one feature | `/ticket-implement .scratch/<feature>/issues/` |
| A small spec or plan in the conversation | [implement](https://aihero.dev/skills-implement) |
| One committed ticket only needs acceptance | [ticket-review](https://aihero.dev/skills-ticket-review) |
| The whole umbrella needs one cumulative verdict | [spec-review](https://aihero.dev/skills-spec-review) |
| The whole umbrella needs review and Blocking repairs until it can pass | The beta `closeout-spec` workflow |

## Prerequisites

[setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) must have configured the local Markdown Issue Tracker. Its generated contract must include current `Review operations` and `Commit references`; rerun setup to refresh an older file. The ticket needs an acceptance checklist, and every blocker must already be accepted or appear earlier in the resolved queue.

The skill commits to the current branch. It does not create a branch or worktree. Its internal [tdd](https://aihero.dev/skills-tdd) and [ticket-review](https://aihero.dev/skills-ticket-review) calls must be installed; `ticket-review` also needs [code-review](https://aihero.dev/skills-code-review) as its Standards reference.

## One ticket, one boundary

The current `HEAD` is recorded before a ticket's first implementation, then tests and code are committed before `ticket-review` inspects that committed range. The review writes a separate commit containing only the ticket's checklist and current Review block. The run advances only after that commit contains a persisted `Pass` or `Pass with follow-up` verdict. A blocked review leaves both commits intact, then stops.

Every implementation, repair, and ticket Review-result commit carries a repo-relative `Issue-Tracker-Ticket:` trailer and, when the sibling spec exists, an `Issue-Tracker-Spec:` trailer. Review-result commits additionally identify their role with `Issue-Tracker-Review-Result: ticket`. Before review, the skill validates every non-merge commit from the ticket's fixed point through `HEAD`, not only the final message. These references survive normal rebase and let later cumulative review rediscover the current feature boundary from Git history.

That order intentionally differs from generic `implement`. The upstream skill runs `code-review` before its final commit. `ticket-implement` needs a committed diff because `ticket-review` pins both ends of the ticket's acceptance boundary.

## Queue orchestration

The queue is resolved completely before implementation. Explicit references use argument order to break ties between ready tickets; a directory uses numeric filename order. Files carrying a configured wayfinder `Type:` are decision tickets and are excluded from directory queues; explicitly naming one stops and routes it back to `wayfinder`. An untyped implementation ticket without a checklist remains malformed and fails before code changes begin. A ticket is skipped only when its passing Verdict is backed by a current ticket-only commit with `Issue-Tracker-Review-Result: ticket`, exact reference trailers, and matching bytes. Missing or contaminated persistence stops before code changes, as do missing blockers, ambiguous references, cycles, and malformed tickets.

Ready tickets run sequentially in one worktree. Each ticket is handed to a fresh [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent), which completes the single-ticket workflow and returns only its ticket reference, checks, verdict, and persistence result. Commit identities remain internal to the worker. Ending the worker discards its detailed implementation context, so the parent queue does not need to invoke `/compact` between tickets.

## Common questions

**Can I pass `ticket-01 ticket-02 ticket-03` directly?**

Yes. Every reference must resolve unambiguously as a normalized repo-relative path under the current worktree's `.scratch/` directory. Absolute paths and `..` escape are rejected. Decision tickets are filtered before an implementation shorthand is judged ambiguous, while an explicit decision path stops and routes back to `wayfinder`. Dependency edges still control execution order; argument order only decides between tickets that are ready at the same time.

**What happens when one ticket blocks?**

The queue stops. Accepted earlier tickets stay committed and reviewed, while the blocked ticket keeps its commit and latest review for repair. Later tickets do not start.

On retry, the skill walks the contiguous suffix of commits carrying that ticket's `Issue-Tracker-Ticket:` trailer and reuses the parent of the earliest one as the original fixed point. The repair commit gets the same trailers, so `ticket-review` sees the initial implementation and every repair. A normal rebase changes the derived SHA but preserves the boundary. Interleaved or malformed ticket history stops instead of producing a partial review.

**What happens if I pass an already accepted ticket?**

It is reported as already accepted and skipped before implementation begins only when the configured Git persistence check also passes. A Pass left only in the worktree, committed with another path, or no longer matching the committed ticket stops instead. Direct single-ticket and queue invocations use the same rule. New scope belongs in a new ticket rather than silently reopening the accepted one.

**Does it literally run `/compact` after each ticket?**

No. In Codex, each ticket uses a fresh worker context and that worker ends before the next begins. This is a stronger boundary than summarising the same conversation. The parent retains only the compact result needed to schedule the queue.

**Will it run independent tickets in parallel?**

No. This queue owns one worktree, index, and `HEAD`, so workers run sequentially. Parallel work remains a separate branch and worktree workflow. Before forking, commit the shared spec and ticket files on the feature branch. Each worker's Review-result commit changes only its selected ticket, so the result travels with that branch through rebase and merge.

**Do I need a particular commit skill?**

No. `ticket-implement` can commit directly. A commit helper is compatible when it preserves the exact caller-provided `Issue-Tracker-Ticket:` and `Issue-Tracker-Spec:` trailers on every commit it creates without trying to discover or rewrite them.

## It's working if

- The entire input queue is resolved before the first implementation change.
- Each ticket has its own trailer-derived implementation range, including every contiguous repair commit.
- Only tickets with accepted blockers start.
- An already accepted ticket is never implemented again.
- Every completed ticket has a committed `Pass` or `Pass with follow-up` review.
- Every implementation commit contains the verified local ticket and spec trailers that apply to it.
- The parent receives a compact result before the next fresh worker starts.
- Neither `spec-review` nor `closeout-spec` starts automatically after the queue.

## Where it fits

`ticket-implement` is the ticket build and acceptance step in the multi-ticket chain:

```txt
grill-with-docs → to-spec → to-tickets → ticket-implement → ticket-review → spec-review
```

Upstream, [to-tickets](https://aihero.dev/skills-to-tickets) creates the local checklist and blocking edges. Internally, [ticket-review](https://aihero.dev/skills-ticket-review) accepts each committed slice. [implement](https://aihero.dev/skills-implement) remains the generic build path. After the queue, choose [spec-review](https://aihero.dev/skills-spec-review) for one cumulative verdict or the beta `closeout-spec` workflow for repeated review and Blocking repairs. [ask-matt](https://aihero.dev/skills-ask-matt) routes between them.
