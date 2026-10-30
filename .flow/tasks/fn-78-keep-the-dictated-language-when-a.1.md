---
satisfies: [R1, R2, R3, R4]
---
# fn-78-keep-the-dictated-language-when-a.1 Implement Keep the dictated language when a vocabulary is set

## Description
TBD

## Acceptance
Every R-ID in the parent spec's ## Acceptance Criteria is satisfied; judge this task against the spec's criteria directly.

## Done summary
Added `dettivo_speech::vocabulary_prompt`, which joins the vocabulary and closes it with a full stop, and routed all five request builders (dictation, rerun, retry, meetings, imports) through it. The two reported takes now transcribe as English through a dev daemon. Across 103 retained takes the bare list flipped language 3 times and the closed list 0 times. German dictation under the new prompt was not measured: the corpus is English.

stage: impl-review - skipped(config: no review backend set; review-backend returned ASK)

stage: plan-sync - skipped(config: planSync.enabled != true)
## Evidence
- Commits: 4a906c3eeccbf22b772564b9a72a49e9cf414f6c
- Tests: cargo test -p dettivo-speech --lib vocabulary_prompt, cargo test -p dettivo-speech -p dettivo-session -p dettivod --no-fail-fast
- PRs: