# Vocabulary prompt shape, measured

Measured 30 Oct 2026 with the 0.4.3 Whisper engine and large-v3-turbo, language `auto`. The real vocabulary has five words; `<surname>` stands in for a personal name. The audio and transcripts are private dictations and are not in this repository.

## Corpus (R4)

Every retained dictation (103 takes) ran through the engine three ways. A take counts as flipped when its transcript reads as a different language from the same take with no prompt, judged by a stopword heuristic and then by reading each disagreement.

| Prompt | Flipped | Takes that flipped |
|---|---|---|
| none | baseline | - |
| `Dettivo, Astra, <surname>, Jev, Clef` (before) | 3 | 2 to German, 1 to Japanese (`- オープン` for "Thank you.") |
| `Dettivo, Astra, <surname>, Jev, Clef.` (after) | 0 | - |

One take is empty and one is a stray mark under all three prompts; both are near silence.

**Limit:** the corpus is English dictation, so it shows no regression for English only. German dictation under the new prompt was not measured.

## The two reported takes (R3)

| Run | `c5681bc9…` | `b1f5d075…` |
|---|---|---|
| 0.4.3 daemon, real dictation (from history) | German | German |
| Dev daemon with this fix, same audio through `DETTIVO_MOCK_MIC`, same vocabulary | English | English |

## Other prompt shapes, same two takes

| Prompt | Language | `c5681bc9…` | `b1f5d075…` |
|---|---|---|---|
| bare list | `en` | German | German |
| `<surname>` | auto | English | English |
| `Dettivo` | auto | English | German |
| `Dettivo, Astra, Jev, Clef` | auto | German | German |
| `Vocabulary: <list>.` | auto | English | English |
| `Glossary: <list>.` | auto | English | English |

## Tests

`cargo test -p dettivo-speech -p dettivo-session -p dettivod --no-fail-fast`: the two new `vocabulary_prompt` tests pass, and so do all but three test binaries. Those three fail for reasons outside this change:

- `dettivo-speech --test supervisor`: timing-sensitive GPU tests under load. A clean checkout of the base commit (`043ce30`) failed 4 of 8 runs at load average near 20, across the same tests.
- `dettivod --test llm_sideload`: `cmake` is not installed on the test machine, so `llama-cpp-sys-2` cannot build.
- `dettivod --test cleanup_dispatch`: the test panics with `native dictation fixtures required` (tiny.en fixture absent).
