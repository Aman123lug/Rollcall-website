# 016 · Understand the request, then answer from real knowledge

**Status:** Idea · **Updated:** 2026-09-25

**In one line:** Rollcall tells an edit from a request for ideas or a how-to question, and answers tips from an up-to-date library, not from memory.

## Problem
"Warm it up" is an edit. "What would make this punchier?" wants ideas. "How do I key green screen?" wants a lesson. Treating them all as commands gives wrong results. Editing knowledge also goes stale with every editor release.

## How it works
1. **Classify each request:** do an edit · suggest ideas · explain how · ask a clarifying question · undo or meta ("what did you change?").
2. **Use context to decide:** playhead, selection, last commands. "Do that again" only makes sense with history.
3. **Edits** go to the edit plan. **Ideas** return 2–3 options to apply by voice. **How-tos** are answered from a retrieval library of curated tips and official editor docs for that editor version, with sources shown.
4. **Log real requests (with consent)** to build the dataset.
5. **Later:** a small on-device model routes intent and handles simple commands (faster, private). Test Apple's on-device model framework in macOS 26 first.

## First version
- Five request types, classified by the main model with structured output.
- A test set of 200–300 realistic requests with the expected type and plan.
- A retrieval library for one editor's docs and tips.

## Not doing (yet)
- Fine-tuning on tutorials. They don't sound like users, and the knowledge in them goes stale. Fine-tune only on logged real requests, and only for routing.

## Open questions
- How much logging will users accept? Opt-in, on-device only?
- Where do curated tips come from, and who keeps them current?
- Does Apple's on-device model route well enough, or do we train our own?
