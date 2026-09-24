<p align="center">
  <img src=".github/assets/logo.svg" width="64" alt="Rollcall">
</p>

<h1 align="center">Rollcall</h1>

<p align="center">
  <b>Edit video by talking to it.</b><br>
  A voice agent for macOS that works inside DaVinci Resolve, Premiere Pro and CapCut.
</p>

<p align="center">
  <a href="https://aman123lug.github.io/Rollcall-website/">Website</a> ·
  <a href="docs/ideas/">Ideas</a> ·
  <a href="docs/features/README.md">Features</a> ·
  <a href="#run-the-website">Run locally</a>
</p>

<p align="center">
  <img src=".github/assets/hero.jpg" alt="Rollcall website hero: messy speech flows in, clean editor actions flow out along a timeline" width="100%">
</p>

---

## Say it. It lands on your timeline.

You talk the way you'd talk to an assistant editor. Rollcall cleans up the speech, turns it into a plan of real edits at real timecodes, and applies them through the editor's own scripting API. Every edit is a normal, undoable change in your project.

```text
You say     "Um, at fourteen seconds put a big title, start before you're ready,
             centre it, centre it, and make ready yellow, no, gold."

Rollcall    ✓ 00:14  Title "Start before you're ready." · centred · bold
            ✓        Highlight "ready" · #F5B841
            ↳ dropped: "um", repeated "centre it", "yellow"
```

## What it does

- **Faster than the menu:** a styled caption takes 48 s and 9 clicks by hand, 7 s and one sentence by voice.
- **One prompt, a finished short:** hook, grade, retime, captions and music, exported for Reels, Shorts and TikTok.
- **Talk while it plays:** each command lands at the timecode you said it, with session notes.
- **Your footage stays on your Mac:** speech is transcribed on-device; only command text and timeline metadata leave.

## How it works

```mermaid
flowchart LR
  subgraph Mac["Your Mac"]
    direction LR
    V["Push-to-talk"] --> S["On-device speech"]
    S --> C["Clean-up<br/>fillers · repeats · corrections"]
    E["Editor timeline<br/>normal, undoable edit"]
  end
  C -- "text + timeline metadata" --> M["Agent · Claude"]
  M -- "edit plan (tool calls)" --> E
```

| Layer | Built with | Owns |
|---|---|---|
| **Mac app** | Swift · SwiftUI | Pill UI, push-to-talk hotkey, microphone, on-device speech, editor bridges |
| **Agent** | Go · Anthropic SDK tool runner | Turns the request and timeline state into tool calls: `addTitle`, `setSpeed`, `adjustColor`, … |
| **Editor bridges** | Each editor's scripting API | Resolve (Python/Lua), Premiere (UXP), CapCut (to be confirmed) |

No video or audio ever leaves the Mac.

## Run the website

The site is one static page (`index.html`) with no build step and no dependencies.

```sh
python3 -m http.server 8000     # then open http://localhost:8000
```

Serve it rather than opening the file directly: the microphone demo and the canvas effects need `localhost` or `https`. A push to `main` deploys to GitHub Pages through [`.github/workflows/static.yml`](.github/workflows/static.yml).

```text
index.html          the site: markup, styles, scripts
videos/             demo footage and posters
docs/ideas/         one short file per use case (+ _template.md)
docs/features/      feature list and status
.github/            deploy workflow, README assets
```

**Before launch:** set `WAITLIST_ENDPOINT` in `index.html` (signups aren't stored until then), replace `hello@example.com` in the footer, and swap the placeholder quotes in Stories.

<p align="center"><sub>macOS first · Editor names belong to their owners; Rollcall is not affiliated with them.</sub></p>
