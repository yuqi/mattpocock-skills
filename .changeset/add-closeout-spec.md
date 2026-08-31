---
"mattpocock-skills": patch
---

Add the in-progress, user-invoked `closeout-spec` workflow for closing a completed multi-ticket spec. It repeatedly invokes cumulative `spec-review`, repairs every Blocking finding and unmet criterion in scope, validates and commits each repair with the exact spec trailer, and stops only on a persisted `Pass` or `Pass with follow-up`, or when a safe substantive repair is no longer available. Make `spec-review` model-invoked so the new workflow can reuse its independent acceptance gate, and let `ticket-implement` present direct review versus closeout as the two explicit post-queue choices.
