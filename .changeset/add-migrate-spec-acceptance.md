---
"mattpocock-skills": patch
---

Add the in-progress, user-invoked `migrate-spec-acceptance` skill to backfill stable acceptance identifiers and existing ticket mappings for legacy local Markdown specs that already have an authoritative acceptance section. The migration carries forward already valid ticket acceptance into explicitly marked Review-result commits without rerunning reviews or changing spec acceptance checkboxes, and stops instead of converting User Stories or other requirements material into acceptance criteria. `ask-matt` routes this repair step immediately before cumulative spec review.
