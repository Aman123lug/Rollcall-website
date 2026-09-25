# 018 · Editing tips library

**Status:** Idea · **Updated:** 2026-09-26

**In one line:** Rollcall looks up a tested recipe for each look and technique ("pookie blush", "velocity edit", "key green screen") instead of improvising from memory.

## Problem
"Make it pookie" or "give it that New York vlog vibe" is a look, not one edit. The model has to guess which grade, grain, text style and cut pattern that means, in each editor, with numbers that actually look right. Guesses vary from run to run, and what worked in one editor version can break in the next.

## How it works
1. **One recipe per look or technique.** Each recipe has:
   - **Name and aliases:** "pookie blush" · "pookie" · "bunny blush" · "pink glowy".
   - **Family:** soft & cute, retro, city, mood, energy (same groups as the Reels page).
   - **What it looks like**, in one line, plus a reference frame.
   - **Steps per editor:** the tool calls with real values, e.g. Resolve: `apply_look("pastel_pink")` → `adjust_color(brightness=1.1, saturation=1.15, warmth=0.3)`.
   - **Tips and don'ts:** "keep skin out of the pink tint", "don't grain under 720p".
   - **Tested on:** editor version and date.
2. **The agent searches the library first.** A look request is matched by name or alias, the recipe's steps become the edit plan, and Claude only fills in the gaps (timecodes, text, which clip).
3. **How-to questions** get answered from the same library, with the steps shown instead of run (see [016](016-intent-and-editing-knowledge.md)).
4. **Recipes can be mixed:** "VHS but pookie" applies both, with the second one's color steps winning.
5. **The library grows from use:** when a user tweaks a recipe by voice and keeps the result, that is saved as a variant they can name ("my pookie").

## The agent looks up tips itself
The library is a tool the agent can call whenever it's unsure how to do what the user asked, like a junior editor checking notes before touching the timeline.

| Tool | Returns |
|---|---|
| `find_recipe(look)` | The best-matching recipe for a named look, or "none" with the closest names |
| `search_tips(question)` | 3–5 short tips for a technique ("speed ramp on the beat", "match two shots' colour") |
| `list_recipes(family)` | Recipe names in a family, to suggest options ("show me retro looks") |

```
User:   "make it look like a y2k digicam but keep it soft"
Agent → find_recipe("y2k digicam")   → steps: LUT, flash vignette, date stamp
Agent → search_tips("soften a harsh look") → lower contrast 10–15%, add bloom, skip sharpening
Agent → apply_look(...) · adjust_color(contrast=0.88) · add_title("SEP 25 2026", VCR)
Agent:  "Digicam look with softer contrast and a date stamp. Want the flash stronger?"
```

- The system prompt says: for any named look or unfamiliar technique, call `find_recipe` or `search_tips` before editing.
- Tips come back short (steps and numbers, not essays) so they don't crowd the context.
- Every answer names the recipe it used, so the user can say "use a different one" or "save this as mine".
- If nothing matches, the agent says so and asks for a reference clip or picks the closest recipe and says which.

## First version
- 20 recipes for Resolve, one per term on the Reels page's aesthetic dictionary.
- Stored as plain files (one YAML per recipe) in the PoC repo, loaded at startup.
- Lookup by name and alias only, no vector search yet.
- A test clip per recipe: run it, screenshot the result, compare with the reference frame.

## Not doing (yet)
- Scraping tutorials into the library. Every recipe is written and tested by us.
- Recipes for editors we don't bridge yet (CapCut, Premiere).

## Open questions
- Who keeps recipes current when an editor updates, and how do we notice a broken one?
- Do we ship our own LUTs for looks Resolve doesn't include (pastel, neon, digicam)?
- Should users be able to share recipes with each other, and how do we review them?
- When no recipe matches, does Claude improvise, or ask the user for a reference clip?
