---
title: An unterminated vocabulary list as Whisper's initial prompt flips English dictation to another language
date: "2026-10-30"
track: bug
category: runtime-errors
module: crates/dettivo-speech/src/lib.rs
tags: [whisper, dictation, vocabulary, language]
problem_type: runtime-errors
symptoms: English dictation inserted as fluent German; history records language "en"; text_raw is already German, so polish did not translate
root_cause: The vocabulary reached Whisper as a comma list with no closing punctuation, which pulls large-v3-turbo off the spoken language, strongly enough to override a forced language
resolution_type: fix
last_updated: "2026-10-30"
---

## Problem

On 30 Oct 2026, two English dictations on 0.4.3 came out as German, with `[dictation] language` unset (`auto`) and a five-word vocabulary. `text_raw` was German, so Whisper produced it; the polish pass did not translate. History reports `language: "en"` for both, so the label does not reveal the drift.

## What didn't work

- Forcing `language = "en"`: both takes still decode as German with the bare list. The language parameter itself works: without a prompt, `de`, `fr` and `en` are all obeyed.
- Blaming one word: a German-looking surname on its own did not flip either take, while `Dettivo` alone flipped one.

## Solution

`dettivo_speech::vocabulary_prompt` owns the prompt's shape and closes the list with a full stop, and all five request builders call it. Measurements are in `.flow/evidence/fn-78-keep-the-dictated-language-when-a/prompt-ab.md`.

## Prevention

Do not hand Whisper a raw fragment as an initial prompt. Avoid English labels or sentences around the words too: they fixed the same takes, but pull non-English dictation toward English.

## Test-setup trap

The supervisor tests compile `CARGO_MANIFEST_DIR` into `target_dir()`, so running a second checkout with a shared `CARGO_TARGET_DIR` makes them look for fixtures in the wrong tree and leaves a stale library that hides new functions from later builds. Give each worktree its own target directory when comparing against base.
