## What it does

`to-tickets` takes a plan, a [spec](https://www.aihero.dev/ai-coding-dictionary/spec), or the conversation you are in, and breaks it into a set of **[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)** on your issue tracker. Each ticket declares its **blocking edges**: the other tickets that have to finish before it can start.

Every ticket is a **tracer bullet**: a narrow but complete path through every layer of the change (schema, API, UI, tests) that can be demoed on its own the moment it lands. That is the constraint that makes it behave differently from the obvious way to split work, which is to cut one layer at a time and integrate at the end. It also sizes each ticket to fit in a single fresh [context window](https://www.aihero.dev/ai-coding-dictionary/context-window), because the thing that will pick the ticket up is a [session](https://www.aihero.dev/ai-coding-dictionary/session) that has never seen your spec.

Each slice also owns an authoritative acceptance checklist. When the source spec has stable identifiers, every identifier maps to one or more ticket criteria so the umbrella promises cannot disappear during decomposition.

## When to reach for it

You invoke this by typing `/to-tickets`. The [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

| Where you are | What to run |
| --- | --- |
| You have a local spec and the build spans several sessions | `/to-tickets`, or `/to-tickets .scratch/<feature>/spec.md` |
| The plan is only in the conversation, never written up | `/to-tickets` reads the thread directly, no spec needed |
| The whole change fits in one context window | [implement](https://aihero.dev/skills-implement), skip the tickets |
| Nothing is decided yet | [grill-with-docs](https://aihero.dev/skills-grill-with-docs), then [to-spec](https://aihero.dev/skills-to-spec) |
| A [wayfinder](https://aihero.dev/skills-wayfinder) map has cleared | [to-spec](https://aihero.dev/skills-to-spec) first, to collapse the map, then `/to-tickets` |

Tickets that `to-tickets` produced are agent-ready by construction. Don't run [triage](https://aihero.dev/skills-triage) over them. Triage is for work that arrived from someone else.

## Prerequisites

`to-tickets` publishes into the Issue Tracker, so [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) must have configured its local markdown implementation and triage-label vocabulary for this repo.

## Tracer bullets, not layers

A **horizontal** slice ships one layer of the change. Nothing works until every layer has landed, and each ticket's acceptance criteria have to reach into work that another ticket owns. A **vertical** slice (the tracer bullet) ships one thin path through all the layers at once, so it is verifiable alone and owns everything it grades.

This is the rule people break most often, and the consequences are well documented. One team ran a 26-ticket stack sliced by layer (corpus, producer, aggregator, selector) and got roughly twenty agent runs per closed ticket, about three quarters of them rework. Their own post-mortem traced every failure class back to the horizontal slicing rather than to the implementations.

Two things happen before anything is published. `to-tickets` looks for prefactoring (the principle "make the change easy, then make the easy change") and orders that work first. Then it presents the breakdown as a numbered list and quizzes you on it: is the granularity right, are the blocking edges real, should anything merge or split. Nothing reaches the tracker until you approve, and that quiz is the place to push back.

## Blocking edges

The edges live as text in one file per ticket under `.scratch/<feature>/issues/<NN>-<slug>.md`, numbered blockers-first. Pass the explicit ticket list or the directory to `ticket-implement`, which resolves the frontier and runs the local queue sequentially. Parallel branches or a fleet remain a separate workflow.

## Acceptance coverage

Every checklist item names an independently verifiable condition owned by that slice. If the source spec carries `AC-N` identifiers, the ticket item names the identifiers it covers. One spec criterion may span several slices, but every identifier must appear before the breakdown is approved.

The two levels have different jobs: [ticket-review](https://aihero.dev/skills-ticket-review) grades one checklist after implementation, while [spec-review](https://aihero.dev/skills-spec-review) grades the authoritative umbrella set after all tickets are complete.

## The wide-refactor exception

One shape breaks the tracer-bullet rule. A **wide refactor** is a single mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so one edit breaks thousands of call sites and no vertical slice can land green.

`to-tickets` sequences that as **expand–contract** instead:

- **Expand**: add the new form beside the old, so nothing breaks.
- **Migrate**: move call sites over in batches sized by blast radius (per package, per directory), one ticket per batch, each blocked by the expand. CI stays green because the old form still exists.
- **Contract**: delete the old form once no caller remains, in a ticket blocked by every migrate batch.

Where even the batches can't stay green alone, they share an integration branch and all block a final integrate-and-verify ticket. Green is promised only there.

## Common questions

**It produced twelve tickets for a three-line change.**
Over-decomposition is the most reported friction on this skill, and it is consistent across practitioners: the [model](https://www.aihero.dev/ai-coding-dictionary/model) defaults to atomic units and loses the grouping that would make them meaningful. The quiz step exists for exactly this: ask it to merge, and it will. The deeper answer is that the tickets have a floor: if the whole change fits in one context window, you don't need this skill at all. Go straight to [implement](https://aihero.dev/skills-implement).

**The tickets came out one per layer: all the schema in one, all the API in another.**
This is the failure the vertical-slice rule is written against, and the skill still produces it sometimes. Catch it at the quiz step by asking one question per ticket: what can I demo when this is done? A ticket with no answer is a horizontal slice. Some people add a "demo path" line to each ticket for this reason, and report it nudges the model toward vertical decomposition.

**Where do the local tickets go? The v1.1 notes said a root-level `tickets.md`.**
They did, and that was a bug: a single shared file also raced when parallel agents wrote to it. Local mode now writes one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, in dependency order, matching the layout the local tracker template already described. The `NN` prefix is a real ticket ID, so `/ticket-implement 03` works instead of retyping a long title. Publication stops rather than overwriting any existing Issue Tracker file at a resolved target path.

**The acceptance criteria graded nothing: some passed before any work was done.**
Every criterion must name an observation that can independently be true or false for that slice. A criterion already true at the base commit, owned by another ticket, or too vague to falsify fails that rule. When the source is a spec, also confirm that its stable identifier appears on the right slice rather than merely somewhere in the ticket set.

**The tickets are published. How do I actually run them?**
`to-tickets` stops at the artifact. Pass the explicit ticket references or their local `issues/` directory to [ticket-implement](https://aihero.dev/skills-ticket-implement). It resolves the dependency frontier, works one ticket per fresh context, and persists each `ticket-review` verdict before advancing. Running ready tickets in parallel still requires separate branches and worktrees outside that invocation.

Before parallel branches or worktrees fork, commit the spec and all ticket files they need on the shared feature branch. Each later ticket Review-result commit changes only its own ticket file, so independent branches can carry and merge their acceptance state without sharing a writable file.

## It's working if

- Every ticket has an answer to "what can I demo when this is done?", and the answer is behaviour, not a layer.
- The list comes back to you numbered, with a "Blocked by" line on each, before anything is published.
- The ticket at the top has no blockers and can be started immediately.
- Nothing in a ticket body is a file path or a line number, except a snippet a prototype produced.
- Each ticket reads like something a fresh session could finish without you in the room.
- Every source acceptance identifier maps to at least one independently verifiable ticket criterion.
- Prefactoring, where it found any, is at the front of the order rather than mixed into feature tickets.

## Where it fits

`to-tickets` is a step in the main build chain:

```txt
grill-with-docs → to-spec → to-tickets → ticket-implement → ticket-review → spec-review
```

Upstream is [to-spec](https://aihero.dev/skills-to-spec), which hands it the settled umbrella criteria; keep both in one unbroken context window. Downstream, [ticket-implement](https://aihero.dev/skills-ticket-implement) accepts one ticket, an explicit list, or the local `issues/` directory, then builds and accepts one slice per fresh context. [spec-review](https://aihero.dev/skills-spec-review) later checks the completed umbrella. When you're unsure which skill or flow fits, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.
