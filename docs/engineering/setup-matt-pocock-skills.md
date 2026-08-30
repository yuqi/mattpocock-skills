## What it does

`setup-matt-pocock-skills` configures three conventions for one repo: the local markdown issue tracker, the triage state vocabulary, and the domain-doc layout. It records them as markdown files under `docs/agents/`.

The **Issue Tracker** interface stays in place, but it has one shipped implementation: markdown files under `.scratch/`. The generated contract also defines triage and review operations plus the repo-relative ticket and spec commit references used for implementation scope, committed Review results, and accepted-ticket persistence checks. Review-result commits carry an explicit role trailer that acceptance validates, while commit identities remain internal to the workflow. Downstream skills still read `docs/agents/issue-tracker.md`, so no provider choice is added to each caller.

It is a prompt-driven skill, not a deterministic script. It reads your existing `AGENTS.md` or `CLAUDE.md`, your existing domain docs, and any prior local tracker files, then waits for you to confirm before writing anything.

## When to reach for it

You invoke this by typing `/setup-matt-pocock-skills`; the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own. It is deliberately marked non-invokable, so no other skill can fire it for you.

Reach for it once per repo, before the first use of any other engineering skill. If [triage](https://aihero.dev/skills-triage), [to-spec](https://aihero.dev/skills-to-spec), [to-tickets](https://aihero.dev/skills-to-tickets) or [wayfinder](https://aihero.dev/skills-wayfinder) start guessing where your issues go, or use status values your local tickets do not recognise, they have not been set up here yet. A repo already halfway through a project is a fine place to run it; the skill reads what is already there and no earlier work is wasted.

## Prerequisites

It writes into the repo you run it in:

| It writes | Where |
| --- | --- |
| `issue-tracker.md` | `docs/agents/` |
| `domain.md` | `docs/agents/` |
| `triage-labels.md` | `docs/agents/`, only when the `triage` skill is installed |
| An `## Agent skills` block | `AGENTS.md` when present, otherwise the existing `CLAUDE.md` |

All of it is committed markdown. There is no user-level or global mode: the config lives in the repo, so every repo gets its own copy.

## The setup decisions

It leads each section with the recommended answer, and skips whatever exploration already settled. Most runs are two confirmations and done.

| Decision | What it proposes | When it actually asks |
| --- | --- | --- |
| **Issue tracker** | local markdown under `.scratch/<feature>/` | never: this implementation is fixed |
| **Triage state values** | keep the five canonical names (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) | only if the `triage` skill is installed |
| **Domain docs** | single-context: one `CONTEXT.md` plus `docs/adr/` at the root | only if it spots monorepo signals, and then it offers a multi-context `CONTEXT-MAP.md` |

The issue tracker is no longer a setup choice. Setup always writes the local template, which stores specs and tickets beneath `.scratch/<feature>/` and needs no remote CLI.

## Common questions

**Do I have to use GitHub?**

No. This distribution keeps the Issue Tracker interface but ships only the local markdown implementation under `.scratch/`. GitHub, GitLab, and custom provider templates are not part of setup.

**Do I need to re-run it after updating the skills?**

Re-run it when you want to refresh the generated files from the current templates or restart the setup. There is no tracker-switching mode because local markdown is the only implementation.

**Which instruction file does it update?**

It prefers `AGENTS.md`, the cross-agent instruction file Codex reads. If only `CLAUDE.md` exists, it edits that file instead so Claude-only repos keep working. If both harnesses share the repo, keep `AGENTS.md` canonical and make `CLAUDE.md` a symlink or pointer to it. If neither file exists, the skill asks which one to create rather than guessing your harness.

**It didn't create my triage states.**

There is nothing external to create. `docs/agents/triage-labels.md` maps the five canonical state roles to the values recorded in local ticket `Status:` fields. If you keep the canonical names, the identity mapping is already complete.

**Can I configure the other skills' behaviour here ([grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) cadence, question format, tone)?**

No. It configures three things: tracker, triage states, doc layout. There have been direct requests to make it the home for per-user preferences, and the standing answer is that skills stay opinionated: *"Config is death."* Preferences belong in your `AGENTS.md`, or in `CLAUDE.md` for a Claude-only repo, as plain instructions that every skill already reads.

**Can I keep the config in `~/.claude` instead of committing it to every repo?**

Not today. There is an open request for exactly this from someone running the skills across many repos, and no user-level mode exists. Every repo carries its own `docs/agents/`.

**Isn't it strange to have a skill that configures the other skills?**

One long-standing complaint says yes, in these words: *"having a skill to set up the other skill does not feel right to me: that means the LLM is configuring its own skills."* The trade is real and acknowledged: the alternative to a setup step is duplicating tracker instructions into every skill that touches issues. The output is inspectable, editable markdown, which is the mitigation: you can read every file it wrote and change it by hand, and day-to-day tweaks are exactly that, not another run.

## It's working if

- `docs/agents/issue-tracker.md` and `docs/agents/domain.md` exist, plus `triage-labels.md` if `triage` is installed.
- An `## Agent skills` section appears in the instruction file your harness actually reads, with a one-line summary pointing at each of those files.
- `docs/agents/issue-tracker.md` points at `.scratch/<feature>/`, and the triage strings match the values used in local issue files.
- `docs/agents/issue-tracker.md` defines `## Triage operations`, `## Review operations`, and `## Commit references`, including the `Issue-Tracker-Ticket:`, `Issue-Tracker-Spec:`, and `Issue-Tracker-Review-Result:` trailer format.
- Afterwards, `/to-tickets` publishes without asking you where tickets live, and `/triage` updates the configured local role fields and sections.
- Nothing in the skill files themselves changed. If setup edited a `SKILL.md`, something went wrong.

## Where it fits

`setup-matt-pocock-skills` is the **run-once setup** for the engineering flow, the precondition everything else assumes rather than a step in the chain. Its neighbours are its readers: [triage](https://aihero.dev/skills-triage), which uses the state-role mapping written here; [to-spec](https://aihero.dev/skills-to-spec) and [to-tickets](https://aihero.dev/skills-to-tickets), which publish into the tracker named here; and [wayfinder](https://aihero.dev/skills-wayfinder), which reads the "Wayfinding operations" section of the same tracker file to know how maps and child [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) are stored. The domain-doc layout it records is the one [domain-modeling](https://aihero.dev/skills-domain-modeling) fills in later: it creates `CONTEXT.md` and ADRs lazily, when a term or decision actually gets resolved, so an empty repo after setup is the expected state. For which skill to reach for next, [ask-matt](https://aihero.dev/skills-ask-matt) routes the whole set.
