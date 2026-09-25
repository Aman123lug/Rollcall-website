# Implementation 01 · Voice edits in DaVinci Resolve, and the path to one-prompt edits

**Status:** POC working · **Updated:** 2026-09-26 · **Code:** `~/Desktop/rollcall-poc`

**In one line:** you type or say an edit, Claude picks a tool, and a Python function makes the edit inside Resolve. This doc covers what works today, what Resolve can't do, and how the same pieces grow into "one prompt, whole video".

## What works today

Tested on **free DaVinci Resolve 21.0.2**, macOS, Python 3.13.

```
mic → Whisper (on the Mac) → Claude (tool runner) → bridge (local socket) → tools.py inside Resolve → timeline
```

The agent has these tools, each a plain Python function in `resolve/tools.py`:
- **Read:** `get_timeline_info`, `list_clips`
- **Titles and markers:** `add_title` (goes on a free track above the video), `add_marker`
- **Clips:** `split_clip`, `delete_clip` (closes the gap by default)
- **Look:** `transform_clip` (zoom, rotate, move, flip, opacity), `adjust_color` (brightness, saturation, warmth), `apply_look` (Kodak or Fuji film-look LUT)

A clip tool acts on the clip under the time you say. Leave out the time and it changes every clip on the track.

## How we talk to Resolve

Resolve only takes edits through its **scripting API** (Python or Lua). We never patch the app and never click the UI.

**Free Resolve blocks scripts running outside the app**, so the edit functions run *inside* Resolve:
1. `scripts/install_bridge.py` puts `rollcall_bridge.py` in Resolve's Scripts folder (one time).
2. The user starts it from **Workspace → Scripts → rollcall_bridge** each time Resolve opens.
3. The bridge listens on `127.0.0.1:50505`. The agent sends it "run `add_title` with these arguments", the bridge runs `tools.py` inside Resolve and sends back the result.
4. The bridge reloads `tools.py` on every call, so code changes apply without restarting it.

**Gotchas we hit, so nobody hits them again:**
- **Scripts menu `.py` files need `#!/usr/bin/env python3` as the first line.** Without it, Resolve runs them with Python 2, which usually isn't installed, and nothing happens.
- **The Console and menu scripts find Python 3 in `/usr/local/bin/python3`** (the python.org install), not our uv venv. The bridge code must be standard library only.
- **`InsertFusionTitleIntoTimeline` always inserts on V1 and pushes everything after it**, cutting into the video. Locking or disabling V1 doesn't help. So `add_title` builds the title in a scratch timeline, turns it into a compound clip, and places that on a free track with `AppendToTimeline`.
- **`AppendToTimeline`'s `endFrame` excludes the last frame.** Frames 0–150 give a 150-frame clip.
- **No blade tool in the API.** `split_clip` deletes the clip and puts back its two halves from the media pool. The clip's transform is kept, but its color grade is lost.
- **Test on copies.** `DuplicateTimeline`, run the change, check with `list_clips`, then delete the copy. Cmd+Z undoes those test steps too, so don't use it to clean up after a test.

## What Resolve's API can't do

- **Transitions**, **speed changes**, **audio volume and fades**, **keyframed animation**, **trimming an existing clip**, **undo**.
- Timeline subtitles only through `CreateSubtitlesFromAudio` (likely Studio only).

## When the API can't do something

There are other ways into Resolve. From most to least reliable:

1. **Scripting API.** Fast, exact, undoable. Covers only what Blackmagic exposes.
2. **Timeline files.** The API can export a timeline as **OTIO / FCPXML / EDL / AAF / DRT** and import one back (`ImportTimelineFromFile`). These formats describe transitions, speed and volume. Our code edits the file and imports it as a *new* timeline. Unknown so far: which features survive the trip.
3. **Fusion inside a clip.** Every clip can have a Fusion composition, and Fusion can keyframe anything: fades, animated zoom, speed (TimeSpeed). Works in place, one clip at a time.
4. **Keystrokes.** The API puts the playhead exactly on a cut, then Python presses Resolve's shortcut (Cmd+T adds a transition, R opens clip speed). Native and undoable, but needs macOS Accessibility permission and Resolve's window in front. Last resort.
5. **Re-render the media.** ffmpeg makes a slowed-down or quieter copy of the file, and we swap it into the timeline. Always works, but it's slow and loses the link to the original.

First picks: **transitions** via keystrokes (Cmd+T) or timeline files · **fades** via Fusion · **speed and volume** via timeline files, with ffmpeg as the backup.

## Running our own code inside Resolve

**Yes, we can write any Python function and run it inside Resolve.** That's what the Console does, what Scripts-menu scripts do, and what our bridge does. It's also how we tested the tools: a temporary function that copied the timeline, tried an edit, reported every track and deleted the copy.

What that code can reach:
- The **whole scripting API** (projects, media pool, timelines, clips, color nodes, render).
- **Fusion scripting** for each clip's composition (nodes, inputs, keyframes).
- **Plain Python:** files, ffmpeg, `osascript` for keystrokes.

What it **can't** do is unlock features the API doesn't have. The Console isn't a back door: `resolve` in the Console is the same object our tools use. Custom code is how we *combine* the doors above (split = delete + append, titles = scratch timeline + compound clip), not a way around them.

**For the agent, this becomes an escape hatch:** a `run_in_resolve(code)` tool, so Claude can write a small function for something no tool covers. Guardrails, because this is arbitrary code on the user's Mac:
- Run it on a timeline **copy** first, and show the before and after.
- Ask the user before running it on the real timeline.
- Log every snippet. Snippets that work and repeat become real tools (the "cache → library" idea in `idea.md`).

## Full scale: one prompt, whole video

**The key change: the agent edits our own timeline document, not Resolve.** Today the agent calls Resolve one step at a time, so it's limited by the API and every try changes the user's real timeline. At full scale:

```
prompt + footage
      ↓
 footage index: transcript, shots, what each shot shows, music beats
      ↓
 agent plans and experiments on our timeline document (OTIO)   ← full freedom, instant, free
      ↓
 compiler: document → Resolve, choosing a route per feature (API, timeline file, Fusion, keys, ffmpeg)
      ↓
 new timeline "Short v1" → agent checks it → fixes → "Short v2"
```

1. **Footage index.** Before editing, we build what the agent needs to know without watching raw video: a transcript with timestamps (Whisper), shot boundaries (`DetectSceneCuts` or a library), a short description of each shot (Claude looking at a few frames), and music beats.
2. **Timeline document.** The agent writes cuts, trims, transitions, speed, volume, titles and color into an **OpenTimelineIO** document. OTIO is an open standard, describes all of these, and Resolve imports it natively. Premiere and Final Cut can use it too, which gives us the "one adapter per editor" design.
3. **Compiler.** Turns the document into the editor. The agent never asks "does the API support crossfades?"; that's the compiler's problem, solved with the doors above.
4. **Never touch the user's work.** Each attempt is a new timeline (v1, v2, …). The user picks.
5. **Check the result.** The agent reads the new timeline back through the API, grabs frames, and checks them against the brief with vision: length, hook in the first 3 seconds, title readable. Then it fixes what's wrong.
6. **Two modes, one system.** Live voice edits stay as today (small, in place, undoable). Whole-video edits use the document, compiler and checking loop. Both share the bridge and `tools.py`.

## Next steps

1. **Timeline file experiment.** On a copy of the demo timeline: export OTIO and FCPXML, add a crossfade, a 50% speed section, a volume drop and a title, import it, and record what survives. This decides route 2 above.
2. **Trim, move, add media, export** as tools (API or delete-and-append).
3. **`run_in_resolve` escape hatch** with the guardrails above.
4. **One-prompt short POC:** footage index for the demo clip → OTIO → import → check loop.

## Open questions

- Which transitions, speed changes and volume settings survive OTIO or FCPXML import into Resolve?
- Do titles, grades and Fusion effects survive the round trip, or do we re-apply them through the API afterwards?
- How good is the agent at judging its own edit from frames? Where do we need the user to pick?
- Cost per video: how many model calls does the check loop need, and what do caching and smaller models save?
- Does free Resolve's bridge (started by hand from the menu) survive real use, or do users need Studio?
