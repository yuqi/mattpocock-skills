---
name: to-spec
description: "Turn the current conversation into a spec and publish it to the project issue tracker: no interview, just synthesis of what you've already discussed."
disable-model-invocation: true
---

This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user; just synthesize what you already know.

The Issue Tracker configuration and triage state mapping should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## Process

1. Resolve the source, then explore the repo to understand the current state of the codebase if you haven't already. If the user supplied a normalized repo-relative `.scratch/` reference, fetch and read the complete local Issue Tracker file before synthesizing.

When that source is a wayfinder `map.md`, read the complete map and collect every Issue Tracker reference in its `## Decisions so far` section. Fetch each referenced decision ticket and require `Status: resolved` plus an `## Answer` section. Stop on a missing, ambiguous, unresolved, or answer-free reference instead of synthesizing an incomplete handoff. Treat the map and those Answers as the authoritative planning source alongside the current conversation; do not copy them back into or otherwise modify the map.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

3. Write the spec using the template below, then publish it through the configured Issue Tracker operations. For a wayfinder map source, publish the spec as its sibling `.scratch/<effort>/spec.md`. Record the mapped `ready-for-agent` value in a `Status:` line near the top of the published spec. No additional triage is needed.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A complete, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

Keep the set complete and non-overlapping. User stories explain who needs what and why; they are source material for acceptance, not a second acceptance list.

## Acceptance Criteria

The one authoritative acceptance set for this spec. Give every criterion a stable identifier:

AC-1. <independently and objectively verifiable condition>

Together the criteria cover every in-scope promise and every applicable error, boundary, compatibility, migration, or security condition. Any statement elsewhere in the spec that can change whether the work is accepted must be represented here.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
