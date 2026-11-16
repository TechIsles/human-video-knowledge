---
name: storyboard
description: |
  Use when a video idea, outline, or script needs a printable storyboard: crude sketched frames, six to a landscape page, with the shot, the audio, and the on-screen text under each. Also exports the frames as 1920x1080 PNGs and builds a narrated animatic (Deepgram TTS over the frames) so a cut can be reviewed as a video before anything is shot. Trigger phrases: "storyboard this", "storyboard the video", "make a storyboard grid", "animatic", "narrated storyboard", "turn the script into frames", "/storyboard".
---

# storyboard

Turns a video script into three things, in order: a **beats module** (one Python file, one entry
per story beat, each with a composed sketch), the **printed grid** (an HTML doc from the canonical
template, 3 by 2 panels per landscape page), and optionally **frames and an animatic** (PNG per
panel, then an mp4 with Deepgram voices reading the script over the frames).

The sketches are deliberately crude line art built from a small set of primitives (`scripts/sblib.py`).
That is the point: a storyboard says what is in frame and who is talking, and a stick figure with a
radio does that faster and more honestly than a rendered mock. Do not reach for images, icons, or
fonts to make it prettier.

## Workflow

### 1. Get the script into beats

Start from whatever exists: a script doc with scene tables, an outline, or a conversation. If the
script does not exist yet, write it first (the `video-script` skill covers that) and then storyboard
it; a storyboard of an unwritten script drifts the moment the script is written.

One panel per **shot change**, not per sentence and not per scene. A six minute cut lands at 10 to
14 panels. A montage or a rapid gag run is one panel using `minis()` (four labeled mini frames in a
2 by 2). Split any beat whose shot direction lists several things that would be different pictures.

Each beat carries:

| Field | Holds |
| --- | --- |
| `t` | Timecode string where the beat starts, `0:45` |
| `title` | Three to five words, the name the crew uses for the shot |
| `sketch` | Composed SVG from the primitives (see step 2) |
| `shot` | What the camera does and what is in frame. One or two sentences |
| `audio` | The spoken line, prefixed by who says it: `HOST:`, `VO:`, `RADIO Sam:` |
| `screen` | On-screen text or graphic. `'none'` when there is none |

Keep every text field under about 160 characters. The printed panel has three lines of room per
field at letter landscape; longer text pushes the second row off the page.

### 2. Compose the sketches

Read `reference/primitives.md` for the catalog, then `reference/example-beats.py` for a full
three-cut worked example. Rules that keep the frames legible:

- The canvas is 320 by 180. Keep content inside `x 8..312, y 8..172`. Text below y 172 clips.
- Compose from primitives first; hand-written SVG only for the one thing no primitive draws. When a
  shape recurs across a project (a specific building, a device), write it as a function in the
  beats module, the way the example does with `pipeline()` and `event_card()`.
- Every frame answers "what is the camera looking at". If the answer is a person talking, draw
  the person and a speech bubble with the first few words. If it is a screen, use `terminal()`.
- LEDs and marks carry meaning: red is transmitting, yellow is receiving, `xmark` is a failed
  attempt, `check` is the thing that worked. Say so in the legend if the project uses them differently.
- `caption()` is on-screen text, `tag()` is a shot label (RE-ENACTMENT, ORIGINAL CAPTURE, a date
  stamp). Use `tag` on every re-enacted or archival beat so nobody passes a restage off as the original.
- Facts in bubbles and terminals must match the script. If the script says nine bundles, the
  bubble says nine.

### 3. Write the beats module

`<repo>/tools/storyboard_beats.py` (or `docs/`, wherever the project keeps tooling). It starts with
the skill's scripts directory on `sys.path` and `from sblib import *`, then defines `TITLE`, `LEDE`,
and `CUTS`. `CUTS` is a list of `(letter, cut_name, subtitle, [beat(...), ...])`, one entry per cut.
One video is one cut; alternate versions of the same video are sibling cuts in the same module so
they share composite scenes and print as one document. Optional `LEGEND` overrides the cover legend.

The example header resolves the skill through `~/.claude/skills/storyboard/scripts`, with
`STORYBOARD_SKILL` as an override. Note it in the module docstring: the beats file depends on the
skill, the rendered HTML does not.

### 4. Render, print-check, open

```bash
python3 ~/.claude/skills/storyboard/scripts/render.py tools/storyboard_beats.py --out docs/<name>-storyboard.html
```

Then print it headless before handing it over, because the grid has already failed two ways that
only show in print (a dark-mode viewer printing dark, and Chrome collapsing the grid to two columns):

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu --no-pdf-header-footer \
  --print-to-pdf=/tmp/sb.pdf file://$PWD/docs/<name>-storyboard.html
pdftoppm -png -r 50 /tmp/sb.pdf /tmp/sb-pg   # then look at two pages
```

Expect one cover page plus `ceil(panels / 6)` pages per cut, light background, three columns. Then
open the HTML per the global document rules. The HTML is derived; edit the beats module and re-render.

### 5. Frames and animatic (when asked, or when the script is about to be reviewed as a video)

```bash
python3 ~/.claude/skills/storyboard/scripts/frames.py tools/storyboard_beats.py --outdir <out>/frames
uv run --with pillow --python 3.12 ~/.claude/skills/storyboard/scripts/animatic.py <out>/cut-a.json --frames <out>/frames --out <out>/cut-a.mp4
```

`frames.py` writes one 1920x1080 PNG per panel with a label bar (cut, panel id, title, timecode)
and an `index.json`. `animatic.py` takes a narration JSON: the panel id, the spoken lines as
`[speaker, text]` pairs, a `voices` map from speaker to a Deepgram Flux voice id, and an optional
`hold` in seconds. It synthesizes each line (cached by voice and text next to the output), shows
the panel for as long as its lines run, burns the current line in as a subtitle, and writes the mp4
plus an `.srt` and a `.timing.json`. `--estimate` prints predicted durations with no API calls.

Narration rules: the lines are the script's spoken lines, verbatim, split at speaker changes. Give
each speaker its own voice when the video has more than one (a host and an agent, a caller and a
dispatcher). Silent B-roll beats are a beat with no lines and a `hold`. The animatic runs shorter
than the real edit; treat its runtime as a floor. Put large outputs somewhere gitignored
(`recordings/`, `out/`), since an mp4 per cut is tens of megabytes.

## Where things go

| Thing | Location |
| --- | --- |
| Beats module (source of truth for the frames) | `tools/storyboard_beats.py` in the project |
| Printed grid (derived) | `docs/<name>-storyboard.html` in the project |
| Frames, narration JSON, mp4, srt (derived, large) | a gitignored output directory in the project |
| Primitives and renderers | this skill's `scripts/` |

## Files

This skill lives in its own repo, `~/Developer/storyboard-skill`, symlinked to
`~/.claude/skills/storyboard`. The README there covers install and hand use.

- `scripts/sblib.py`: the primitives, `beat()`, the page renderer, the frame page. Add a primitive
  here only when a third project needs it; project-specific shapes stay in the beats module.
- `scripts/render.py`: beats module to printed grid.
- `scripts/frames.py`: beats module to 1920x1080 PNGs (headless Chrome, `CHROME` overrides the binary).
- `scripts/animatic.py`: frames plus narration JSON to mp4 (Deepgram TTS, Pillow subtitles, ffmpeg).
- `reference/primitives.md`: the catalog with signatures and composition notes.
- `reference/example-beats.py`: three cuts of one project, the worked example.
- `reference/example-narration.json`: the narration JSON for that example's cut A.
- `template/print-ready-html-doc.html`: the doc template `render.py` falls back to when gutils is not on the machine.
