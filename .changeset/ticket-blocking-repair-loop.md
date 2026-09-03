---
"mattpocock-skills": patch
---

Let `ticket-implement` repair persisted Blocking findings and unmet criteria, validate and commit each repair, and repeat `ticket-review` against the original ticket boundary. Repair rounds use focused checks; `ticket-review` requires passing targeted tests before writing `Pass` or `Pass with follow-up`, and records failed or unavailable verification as `Block`. Queue workers continue without a numeric round limit while safe, substantive in-scope repairs remain available; safety, authority, invalid-state, or no-progress blockers stop the run with completed work intact. Keep `spec-review` free of executable verification, and make `closeout-spec` run the full test suite only against the final reviewed implementation before reporting successful closeout.
