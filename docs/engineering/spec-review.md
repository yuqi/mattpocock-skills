## What it does

`spec-review` is the cumulative acceptance gate for one completed multi-ticket [spec](https://www.aihero.dev/ai-coding-dictionary/spec). It discovers the current local spec and committed boundary from Git trailers, then runs three independent [subagents](https://www.aihero.dev/ai-coding-dictionary/subagent) for Standards, Spec coverage, and Integration before writing and committing one current verdict back to the spec.

It reviews the umbrella, not another ticket. The separate Integration axis exists because individually acceptable slices can still disagree when assembled.

## When to reach for it

You invoke this by typing `/spec-review`, and the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) will not reach for it on its own.

| What needs review | Use |
| --- | --- |
| One implemented ticket | [ticket-review](https://aihero.dev/skills-ticket-review) |
| A broader branch without one umbrella acceptance set | [code-review](https://aihero.dev/skills-code-review) |
| The complete result of a multi-ticket spec | `/spec-review` |

## Prerequisites

The Issue Tracker config must include current `Review operations` and `Commit references`; rerun [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) to refresh an older file. The spec and every sibling ticket must be writable local Markdown inside the current worktree. Every ticket must have a passing current verdict backed by a valid ticket-only Review-result commit with matching bytes and trailers. Implementation commits must preserve their `Issue-Tracker-Spec:` and `Issue-Tracker-Ticket:` trailers. The model-invoked [code-review](https://aihero.dev/skills-code-review) skill supplies the Standards reference.

## Rebase-stable discovery

An explicit spec path wins. Without one, the latest valid `Issue-Tracker-Spec:` trailer in current `HEAD` history identifies the umbrella. The sibling `issues/` directory supplies the complete implementation-ticket set after files carrying a configured wayfinder `Type:` are excluded, and the earliest commit carrying that same spec reference supplies the cumulative base. An untyped file without an acceptance checklist remains an error rather than being silently filtered.

The boundary is derived again from current commit objects on every invocation. Normal rebase can change every SHA without losing the spec path recorded in each commit message. Mixed commits, missing trailers, incomplete ticket acceptance, unknown acceptance identifiers, and references outside `.scratch/` stop before reviewers run.

## Three independent axes

- **Standards** checks the cumulative diff against documented repository rules and material Fowler smells.
- **Spec** accounts for every umbrella acceptance identifier plus every criterion from each included implementation ticket, and reports missing, partial, incorrect, or unrequested behavior.
- **Integration** traces the assembled system across ticket boundaries, including compatibility, state transitions, migrations, error paths, and security boundaries.

The axes stay separate so a clean Standards report cannot hide a missed requirement, and complete ticket coverage cannot hide a broken cross-ticket contract.

## One cumulative verdict

Pass requires every umbrella and ticket criterion to be Met with no actionable findings. Pass with follow-up permits only bounded Non-blocking findings after every criterion is Met. Any Partial or Not met criterion, or any Blocking finding, produces Block. Before the verdict is formed, every umbrella identifier and every ticket criterion must appear exactly once in coverage.

The latest `## Review` block is the acceptance result. When the umbrella acceptance set already uses checkboxes, spec review also projects that result into the checklist: Met criteria are checked, while Partial and Not met criteria are unchecked. It does not alter triage status, ticket reviews, or executable-verification history. Its Review-result commit changes only the spec and carries one matching `Issue-Tracker-Spec:` trailer. Spec and ticket files are excluded from the cumulative implementation diff, so repeated review does not grade its own metadata.

## Common questions

**Do I still need this if every ticket passed `ticket-review`?**

Yes when the tickets form one system. Ticket review proves each slice against its own checklist at implementation time; spec review checks those criteria again against the current cumulative diff, then proves the umbrella promises and cross-ticket contracts. This matters after rebase because no persisted ticket SHA is treated as current evidence.

**Why three reviewers?**

Standards, requirement coverage, and integration failure modes pull attention in different directions. Running them independently prevents one axis from masking another before the fixed verdict gate is applied.

**Does a missing test result automatically Block?**

No. Review is inspection, not execution. Missing executable evidence is stated plainly in the verification note; it becomes a finding only when the spec itself requires evidence that is absent.

**Who checks the spec acceptance checklist?**

`spec-review` does. Ticket completion and ticket review establish slice-level evidence, but only the cumulative umbrella review checks a spec criterion as Met. A metadata migration may add stable identifiers and ticket mappings, but it preserves the existing spec checkbox state until this review runs.

**What happens after a rebase?**

Run `/spec-review` again. It ignores the old stored SHAs for scope discovery and derives the spec and base from the rebased commits' preserved trailers. If the rewritten history dropped or mixed those references, the review stops instead of guessing.

The prior Review-result was committed, so the worktree is clean before the rebase. The next completed review replaces the old block and commits the new verdict with the same spec trailer.

## It's working if

- Every umbrella acceptance identifier appears once in the coverage matrix.
- Every included ticket criterion appears once under its ticket reference.
- Standards, Spec, and Integration findings remain under their own headings.
- The review names the exact reviewed commit and covers the trailer-derived cumulative boundary.
- A no-argument run discovers one local spec, every sibling ticket, and a trailer-validated cumulative boundary.
- A source change during review prevents the result from being written.
- Re-running replaces the prior cumulative review rather than appending another current verdict.
- Existing umbrella checkboxes match the current Met, Partial, and Not met coverage results.
- One Review-result commit changes only the spec and carries its exact spec trailer.

## Where it fits

`spec-review` is the final cumulative gate on the multi-session build path. [to-spec](https://aihero.dev/skills-to-spec) creates the umbrella criteria, [to-tickets](https://aihero.dev/skills-to-tickets) maps them into slice checklists, and [ticket-review](https://aihero.dev/skills-ticket-review) accepts each slice. [code-review](https://aihero.dev/skills-code-review) remains the broad diff report when no umbrella gate applies, and [ask-matt](https://aihero.dev/skills-ask-matt) routes the flow.
