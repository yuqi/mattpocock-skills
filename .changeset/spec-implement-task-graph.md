---
"mattpocock-skills": patch
---

Apply `implement-spec`'s task-graph scheduling to `spec-implement`: run ready tickets concurrently in isolated worktrees, integrate each through one merger, and unlock dependants only after persisted acceptance on the current branch. Keep ticket review ranges linear and preserve the Blocking repair loops and final full-suite gate. Sync the beta listing, Codex metadata, and router without adding PR creation to local closeout.
