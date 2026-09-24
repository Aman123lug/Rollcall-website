# 003 · Check it before export

**Status:** Idea · **Updated:** 2026-09-25

**In one line:** one command runs a quality pass on the edit and fixes what it safely can.

## Problem
Small mistakes slip into exports: flash frames, black gaps, clipped audio, caption typos, wrong loudness.

## How it works
1. Say "check it" before exporting.
2. Rollcall scans the timeline for common problems.
3. Safe fixes are applied (e.g. loudness to −14 LUFS); the rest are listed with timecodes.
4. Click a problem to jump to it.

## First version
- Flash frames, black gaps, audio clipping, loudness target per platform.
- Caption spelling and safe-zone checks.

## Open questions
- Which fixes are safe to apply without asking?
- Per-platform presets (YouTube, TikTok, broadcast)?
