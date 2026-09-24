# 004 · Undo by meaning

**Status:** Idea · **Updated:** 2026-09-25

**In one line:** undo a kind of change, not just the last click: "undo the colour but keep the cuts".

## Problem
Undo is a strict stack. Reverting one type of change means undoing everything after it.

## How it works
1. Every edit is tagged by intent in the session notes (colour, cuts, audio, captions).
2. Say what to undo: "undo the colour changes", "undo everything after the title".
3. Rollcall shows what will be reverted, then applies it as one normal undo step.

## First version
- Undo by category and by "since X".
- Preview before reverting.

## Open questions
- How to handle edits that depend on each other?
- How far back should history go?
