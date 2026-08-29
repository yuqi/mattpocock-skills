## What it does

`triage` works through local tickets on your project's Issue Tracker, moving each one through a small state machine of **triage roles** (a category role and a state role) and leaving behind either an agent-ready brief, specific questions in Triage Notes, or a `wontfix` ticket with a recorded reason.

It is only for incoming tickets **you didn't create through the planning flow**: raw bug reports and feature requests that someone recorded under `.scratch/<feature>/issues/`. [Tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) that [to-tickets](https://aihero.dev/skills-to-tickets) produced are already agent-ready by construction, and running `triage` over them is wasted work at best.

The second thing that separates it from editing fields by hand: it recommends and waits. It tells you its category and state call with reasoning, plus what it found in the codebase, and applies nothing until you direct it.

## When to reach for it

You invoke this by typing `/triage` and then describing what you want in plain language. The [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own. "Show me anything that needs my attention", "look at the refund ticket", or "move ticket 42 under payments to ready-for-agent".

| What you have | Where to go |
| --- | --- |
| Local ticket files containing raw reports from other people | `/triage` |
| A rough idea of your own, nothing written down | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| A settled conversation to turn into a [spec](https://www.aihero.dev/ai-coding-dictionary/spec) | [to-spec](https://aihero.dev/skills-to-spec) |
| A spec to split into agent-ready tickets | [to-tickets](https://aihero.dev/skills-to-tickets) |
| A confirmed bug that needs a root cause, not classification | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## Prerequisites

`triage` reads and writes local ticket files, so [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) must have generated current `Triage operations` plus the role mapping first. The role names below are **canonical**; setup maps them to the strings recorded in `Category:` and `Status:` fields.

## The state machine

Every triaged item ends up carrying exactly one category role and one state role. Two categories: `bug` (something is broken) and `enhancement` (new feature or improvement). Five states:

| State | Means |
| --- | --- |
| `needs-triage` | You need to evaluate it. Where a ticket without `Status:` normally lands first. |
| `needs-info` | Waiting for more local information. Returns to `needs-triage` when new content follows the latest Triage Notes. |
| `ready-for-agent` | Fully specified, with an agent brief attached. An [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agent can take it. |
| `ready-for-human` | The same brief, plus why this can't be delegated: judgment, external access, manual testing. |
| `wontfix` | Not being actioned, with the reason recorded under Comments. |

That is the whole vocabulary, and the "exactly one state role" invariant is what keeps the queries simple. It is also the most-requested area of the [skill](https://www.aihero.dev/ai-coding-dictionary/skill): users have asked for a sixth state for work that is specified but blocked on another issue, for `deferred` work gated on a future trigger, and for a terminal `implemented` state. None of those has shipped. See the questions below.

`wontfix` splits three ways, and the difference matters because only one of them writes to the knowledge base:

| Why it becomes wontfix | What happens |
| --- | --- |
| Already implemented | A Comments entry pointing at where it already lives. Nothing is written to `.out-of-scope/`, because it is a built feature, not a rejected one. |
| Rejected bug | A polite reason in Comments, with `Status: wontfix`. |
| Rejected enhancement | A file in `.out-of-scope/`, referenced from the reason in Comments, with `Status: wontfix`. |

`.out-of-scope/` is one markdown file per rejected **concept**, not per ticket, written as a short design document rather than a database row: what was rejected, why, and every ticket that has asked for it. `triage` reads the whole directory before it evaluates anything, and matches by concept rather than keyword, so "night theme" matches `dark-mode.md`. When it hits a match it surfaces the old decision and asks whether you still feel the same way, instead of re-litigating the request from scratch.

## Verify before you brief

Before any [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling), `triage` checks that the claim actually holds. For a bug, it reproduces it from the supplied steps. Then it reports which of three things happened: confirmed, with the code path; failed to reproduce; or not enough detail to try, which is itself the strongest `needs-info` signal there is.

It runs two more checks against the codebase in the same pass: **redundancy** (is this already implemented, searched by domain concept rather than by the ticket's wording?) and **prior rejection** (does `.out-of-scope/` already say no?). Both are cheap, and both produce a `wontfix` when they hit.

All of it exists to make one artifact good: the **agent brief**, the top-level section written when a ticket moves to `ready-for-agent`. Once written, the brief is the contract and the original report is context. Briefs are **durable** rather than precise, so they name types, signatures and behavioural contracts, and never file paths or line numbers. A confirmed reproduction makes a far stronger brief than a guess does.

## Common questions

**I ran `/to-spec` and `/to-tickets`, and now those tickets are sitting there untriaged. Do I run `/triage` over them?**
No. They already have `Status: ready-for-agent`, because `to-tickets` creates implementation-ready checklists. `triage` is the on-ramp for local tickets that arrived outside the planning flow; the spec flow is the lane for work you originate. They meet at `ready-for-agent`, not before.

**Is `triage` still relevant now that there's a `to-spec` → `to-tickets` → `ticket-implement` flow?**
Only if you have inbound work. `triage` does a different job from the planning spine: it is the lane for reports other people recorded as local tickets. If every ticket came out of your own planning, you will rarely open it.

**Five states aren't enough: what about blocked, or deferred, or implemented?**
Those states have not shipped. Keep `Status:` honest about the current canonical state, and record blocking, future triggers, or verification context in Triage Notes instead of inventing another state value.

**How is this different from `/diagnosing-bugs`?**
The verification step here is deliberately shallow (enough to answer "is this real, and roughly where does it live"), not to find a root cause. When a bug won't reproduce from the supplied steps in a few minutes, the honest move is `needs-info`, or [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) if you want to chase it now. Neither skill's text currently mentions the other; a user found that seam, and it is still open.

**Can I point it at my whole backlog and let it run?**
You can ask, but the Inventory pass is a compact listing meant for *selection*. Before changing any ticket, triage must read that complete local file, including Comments and existing Triage Notes or Agent Brief. A bulk pass cannot use the inventory summary as its evidence base.

## It's working if

- Every ticket it touches ends with exactly one `Category:` and one `Status:` role, never zero, never conflicting duplicates.
- It gives you a recommendation with reasoning and stops, rather than updating fields and moving on.
- A bug is reproduced from the supplied steps before anything reaches `ready-for-agent`.
- The briefs it writes name types and behaviours, and contain no file paths and no line numbers.
- A request that was rejected six months ago comes back, and it says so and quotes the old reason instead of triaging it fresh.
- Every Triage Notes and Agent Brief section starts with `> *This was generated by AI during triage.*`

## Where it fits

`triage` is an **on-ramp**, not a step in the main chain. The main flow runs from an idea you had (grill, spec, tickets, implement, review), and `triage` is the parallel lane for work that arrived instead. It merges at the same place: a local ticket with `Status: ready-for-agent` and an Agent Brief, which [ticket-implement](https://aihero.dev/skills-ticket-implement) picks up exactly as it would a ticket from [to-tickets](https://aihero.dev/skills-to-tickets), including persisted [ticket-review](https://aihero.dev/skills-ticket-review) acceptance. When a request needs sharpening before it can be briefed, `triage` runs [grilling](https://aihero.dev/skills-grilling) and [domain-modeling](https://aihero.dev/skills-domain-modeling) together, a round of questions at a time, so decisions land in `CONTEXT.md` and the ADRs as they're made. When you're not sure which lane you are in, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.
