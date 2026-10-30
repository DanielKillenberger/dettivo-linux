# Keep the dictated language when a vocabulary is set

> user (turn 1): "dettivo just started translating stuff to german for some reason why?"
> user (turn 2): "let's test the fix and then create a PR upstream with proper spec"

## Goal & Context

With a `[dictation] vocabulary` set, English dictation on Whisper large-v3-turbo can come back as fluent German, even though the speaker never said a German word. The insertion lands in the focused window, so the user sees a translation they never asked for, and the history item still reports `language: "en"`. Nothing in config, logs or history points at the cause.

Measured on 30 Oct 2026 (0.4.3, large-v3-turbo; `<surname>` stands in for a personal name in the real list) against two retained takes, `c5681bc9-7353-498f-b689-39b0acdbdd76` and `b1f5d075-dd6a-44c5-a34e-1e16910e9bab`:

| Initial prompt | Language | Result |
|---|---|---|
| `Dettivo, Astra, <surname>, Jev, Clef` (what Dettivo sends) | auto | German on both |
| same | `en` | German on both |
| none | auto | English on both |
| none | `en`, `de`, `fr` (`b1f5d075` only) | follows the language |
| `Dettivo` | auto | German on one |
| `<surname>` | auto | English on both |
| `Dettivo, Astra, <surname>, Jev, Clef.` | auto | English on both |

The vocabulary reaches Whisper as a bare comma list with no closing punctuation. That unterminated fragment pulls the decoder off the spoken language, strongly enough to override a forced language. The same words closed with a full stop decode correctly. [paraphrase]
<!-- Source: 10% user / 70% [paraphrase] / 20% [inferred] -->

## Architecture & Data Models

The vocabulary prompt is built from `[dictation] vocabulary` wherever a `RecognizeRequest` is assembled: live dictation, a history rerun, a retried job, a meeting and a file import. Each builds the string itself. The fix gives the speech layer one function that owns the prompt's shape, and every request builder calls it. [inferred]

## API Contracts

`vocabulary_prompt(words) -> Option<String>`, owned by the speech crate beside `RecognizeRequest`: [inferred]

- no words → `None` (no prompt sent, as today);
- words → the words joined with `", "`, closed with `"."`;
- when the last word already ends in `.`, `!` or `?`, no second mark is added (`["Acme Inc."]` → `Acme Inc.`).

## Edge Cases & Constraints

- The prompt stays language-neutral: no English label or sentence around the words, so German and other non-English dictation is not pulled toward English. [inferred]
- Parakeet drops the prompt today; that does not change. [inferred]
- No config key, flag or migration: the stored vocabulary is unchanged, only the string sent to the engine. [inferred]

## Acceptance Criteria

- **R1:** `vocabulary_prompt` returns `None` for an empty vocabulary, joins words with `", "` and ends with a full stop otherwise, and adds no second mark when the last word already ends in `.`, `!` or `?`. Errors: no error surface beyond an empty vocabulary. [inferred]
- **R2:** Every path that sends a vocabulary to the engine (live dictation, rerun, retry, meeting, import) builds the prompt with `vocabulary_prompt`; none joins the vocabulary itself. Errors: no error surface beyond R1. [inferred]
- **R3:** The two retained takes `c5681bc9…` and `b1f5d075…`, run through the Whisper engine with the prompt `vocabulary_prompt` produces for `["Dettivo", "Astra", "<surname>", "Jev", "Clef"]`, transcribe as English. Errors: no error surface beyond R1. [paraphrase]
- **R4:** Across the retained dictation corpus, transcripts with the new prompt flip language no more often than transcripts with no prompt at all, and the bare-list prompt's flips are recorded alongside. The comparison is evidence on the change. Errors: no error surface beyond R3. [inferred]

## Boundaries

- No change to how the language is detected, forced or reported, including the history's `language` label for a take that drifted. [inferred]
- No new vocabulary syntax or user-editable prompt template. [inferred]

## Decision Context

A full stop is the smallest change that fixed both takes and adds no language of its own. `Vocabulary: …` and `Glossary: …` also fixed both, but put English words in front of every prompt, a bias the German speakers Dettivo serves would pay for. A sentence wrapper (`I'm working on …`) has the same cost. Fixing it inside the Whisper engine was rejected: the engine would then rewrite every prompt it is given, and the shape belongs to the code that builds the vocabulary prompt. [inferred]
