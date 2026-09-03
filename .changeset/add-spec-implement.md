---
"mattpocock-skills": patch
---

Add the model-invoked beta `spec-implement` skill with inline sequential ticket implementation and cumulative closeout workflows. It requires an explicit spec path or containing feature directory whose tickets are already prepared, preserves per-ticket acceptance, repair loops without numeric round limits, focused intermediate validation, and the final full-suite gate without invoking or changing the user-only `ticket-implement` and `closeout-spec` skills. Update the skill router to distinguish this local end-to-end workflow from the concurrent PR-building `implement-spec`.
