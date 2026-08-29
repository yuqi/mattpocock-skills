# Ticket queue

Resolve and complete a local Issue Tracker queue sequentially. The single-ticket workflow remains authoritative for each ticket.

## Resolve the queue

Resolve every ticket before changing implementation. Search only normalized repo-relative references under the current Git worktree's `.scratch/` directory. Reject absolute paths, `..` escape, and references outside `.scratch/`.

Keep queue discovery compact. Retain only each file's reference, title, configured `Type:` value when present, blockers, acceptance-checklist presence, and latest verdict. The selected worker refetches the full ticket before implementation.

Accept either input shape:

- **Explicit references**: resolve every argument as one ticket, filtering recognized wayfinder decision candidates before treating an implementation shorthand as ambiguous. An explicit path to a file with a configured wayfinder `Type:` still resolves, then stops and routes to `/wayfinder`. Preserve argument order as the tie-breaker between simultaneously ready tickets.
- **Issues directory**: inspect every Markdown file directly inside the directory and exclude files with a configured wayfinder `Type:` value (`research`, `prototype`, `grilling`, or `task`). Use numeric filename order as the tie-breaker for the remaining implementation tickets.

Stop on a missing, ambiguous, duplicate, out-of-contract, or acceptance-checklist-free implementation reference. A file without a recognized wayfinder `Type:` is not silently excluded merely because its checklist is missing. Read every ticket's latest review and `Blocked by` edges, and evaluate completion through the configured Accepted ticket operation. A passing file Verdict whose persisted ticket review check fails stops the queue before implementation. Resolve blockers only through normalized `.scratch/` references. Validate that every unfinished ticket's blockers resolve either to accepted tickets or to tickets in the queue. Skip only tickets whose Accepted ticket check passes. If every ticket is accepted, report the completed queue and stop. Otherwise stop before implementation when a blocker is missing, the graph has a cycle, or no unfinished ticket is ready.

## Work the frontier

Choose the first ready ticket by the queue's tie-breaker. Never run two ticket workers concurrently in one worktree.

Keep one orchestrator and dispatch every ready ticket to a fresh subagent, one at a time. Give the worker only that ticket and the complete single-ticket workflow. The worker runs `ticket-review` and its reviewer as a nested subagent, then returns only the ticket reference, fixed point, reviewed commit, Review-result commit, checks run, review verdict, and whether persistence was confirmed. Ending that worker provides the context boundary before the next ticket.

After every worker, refetch its ticket and continue only when the persisted verdict is `Pass` or `Pass with follow-up` and its Review-result commit is confirmed; otherwise stop with completed commits and reviews intact. Retain only the worker's compact result, then recompute the frontier from the ticket files before dispatching the next worker.

If nested subagents are unavailable, let the worker stop after its implementation commit and return the ticket reference, fixed point, committed `HEAD`, and checks run. The orchestrator then calls the Skill tool with "ticket-review" using the returned fixed point and ticket reference.

Do not invoke `spec-review`; it remains the user's separate cumulative gate after the queue is complete.
