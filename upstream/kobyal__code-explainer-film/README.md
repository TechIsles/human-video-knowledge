# How to make an explainer film from code

A method for producing short, narrated product explainers (two to six minutes) without a
video editor, a microphone, or a designer. Everything is text in a repository: the storyboard,
the scenes, the narration. A script renders the pictures, synthesises the voice, pads each
scene to its narration and joins them into one MP4. Change a sentence, rebuild one scene,
the film is whole again.

It was worked out while making a six-minute explainer for an internal support bot, and it
is written so that a class can reproduce it. The `template/` folder is a complete three-scene
film that builds as-is; start from it rather than from a blank page.

## What you need

| Tool | Why | Install |
|---|---|---|
| Python 3.10+ | runs everything | python.org or your package manager |
| [Manim Community](https://www.manim.community/) | draws and animates the scenes from Python code | `pip install manim` (needs Cairo, Pango, FFmpeg; follow Manim's install page for your OS) |
| [edge-tts](https://github.com/rany2/edge-tts) | text-to-speech, free, many voices and languages, no account | `pip install edge-tts` |
| FFmpeg + ffprobe | muxes voice with video, pads, concatenates | `brew install ffmpeg` / `apt install ffmpeg` |
| A font you like | Manim uses system fonts; the template names one, change it | already on your machine |

Use a virtual environment. The build script finds `manim` and `edge-tts` next to the Python
that runs it, so "run it with the venv's Python" is the only rule.

```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r template/requirements.txt
cd template && python build.py                        # renders out/explainer.mp4
```

The first build takes a few minutes (Manim renders at 720p30). After that you rebuild one
scene at a time in under a minute.

## The pipeline in one picture

```
SCRIPT.md ──► NARR (build.py) ──► edge-tts ──► Title.mp3 ─┐
                                                          ├─► ffmpeg pad + mux ──► Title_final.mp4 ─┐
scenes.py ──► class Title ──────► manim ────► Title.mp4 ──┘                                       ├─► concat ──► out/explainer.mp4
                                              (one pair per scene)                                 ┘
```

Three files carry the film:

- **`SCRIPT.md`**: the storyboard. One block per scene: what is on screen (`V:`) and what is
  said (`N:`). This is where you think. It is the plan, not the source of truth.
- **`build.py`**: the narration dictionary `NARR`, one entry per scene in film order, plus the
  render, narrate, mux and concat steps. The narration text here is what ships.
- **`scenes.py`**: one Manim `Scene` class per `NARR` key, plus a small toolkit (heading, chip,
  bullet list, screenshot on a card, highlight box, caption) so scenes stay consistent.

Everything under `media/` is an intermediate. `out/` holds exactly one file, the film. Keep it
that way: a folder with twelve MP4s in it is a folder nobody knows which file to open.

## The method, step by step

### 1. Write the words first

Start in `SCRIPT.md`. Decide the audience and the three things they should be able to say
afterwards. Then write each scene's narration as spoken sentences, out loud, before drawing
anything. Sentences of ten to twenty words. Numbers spelled the way you would say them
("zero point nine zero", "E C 2"), because the voice reads what you type.

The narration decides the length of every scene. A scene is as long as it takes to say its
words, plus a breath. You never cut a scene short to fit the animation; you write the
animation to fit the words.

Copy the finished narration into `NARR` in `build.py`. Build the film with placeholder
scenes if you like and listen to it. Narration that sounds wrong read aloud is wrong; fix it
now, before a single frame exists.

### 2. Draw each scene to fit its words

One class per scene in `scenes.py`, named exactly as its `NARR` key. Inherit from `Base`, put
the animation in `body()`. Use the toolkit rather than raw Manim where you can:

| Helper | For |
|---|---|
| `head("...")` | scene heading, top of frame |
| `chip("label", "sub")` | a box with a label, for steps and components |
| `bullet([...])` | a left-aligned list |
| `maybe_img("shot.png", ...)` | a screenshot from `assets/`, or a labelled panel if the file is missing |
| `card(img)` | a light mount behind a screenshot or diagram |
| `px_box(img, x0, y0, x1, y1)` | an amber frame over a pixel region of a 1280x720 screenshot |
| `caption("...")` | a line at the bottom |

Three habits that make scenes read well:

- **One idea moves at a time.** Fade things in in the order the narration names them. The
  `Flow` scene in the template moves a highlight from box to box; that is the whole trick.
- **Make the screenshot the subject.** Real screens beat diagrams. Put the screenshot on a
  card, then draw a box around the part you are talking about. The viewer's eye goes where
  the box is.
- **Type large.** 720p on a laptop or a phone is small. Nothing under 18pt, headings at 32.
  The `Text` subclass in the template renders at 4x and scales down, which stops Manim
  splitting words at small sizes.

Rehearse a single scene fast with the draft quality flag:

```bash
FILM_QUALITY=-ql python build.py Flow      # 480p15, seconds instead of minutes
```

### 3. Calibrate the pacing

Animation and narration are produced separately, so they will not be the same length. The
build pads the shorter one (video freezes on its last frame, audio gets silence), so nothing
is ever cut. But a scene whose animation ends ten seconds before the voice does looks frozen.

Each scene has a `PACE` multiplier that stretches only its `self.wait()` calls. Set it so the
animation ends just before the narration. To measure:

```bash
# render one scene at two pace values
FILM_PACE=1 python build.py Flow && ffprobe -v error -show_entries format=duration -of csv=p=0 media/videos/scenes/720p30/Flow.mp4
FILM_PACE=3 python build.py Flow && ffprobe -v error -show_entries format=duration -of csv=p=0 media/videos/scenes/720p30/Flow.mp4
ffprobe -v error -show_entries format=duration -of csv=p=0 media/build/Flow.mp3
```

Video length is linear in pace, `v(P) = A + P·W`, where `A` is the fixed animation time and
`W` the total wait time. From the two measurements solve for `A` and `W`, then set

```
PACE = (audio_seconds - 0.6 - A) / W
```

in the scene class. Rebuild the scene. Do this once per scene after the narration is final;
it takes two minutes and it is the difference between a film and a slideshow.

### 4. Capture the screenshots honestly

- Capture at **1280x720** or a clean multiple, so `px_box` coordinates are pixel coordinates.
- Crop out anything you do not want on a screen forever: account IDs, emails, other people's
  names, notification badges. Do it in the image, not by hoping nobody pauses.
- If the real system cannot be shown, run the real front end against a stub with real-shaped
  data and say so in the narration. Never claim a live session you did not have.
- Keep a note per image of how it was captured and which sentence it supports. When a
  screenshot is replaced, that note tells you which scene to check.

`maybe_img` renders a labelled grey panel when the file is missing, so the film builds before
the assets exist and you can see exactly where each one goes.

### 5. Build, watch, cut

```bash
python build.py            # everything
python build.py Flow       # one scene, then the whole film is re-joined
python build.py --concat   # just re-join, no rendering
```

Watch it end to end with someone who was not involved. Then cut. The scenes that go first
are the ones that prove something rather than show something: terminal output, test
results, tables of metrics, infrastructure evidence. To the viewer they read as the maker
justifying themselves. Show the thing in use, and what the viewer can do with it; leave the
proofs in the documentation. Our six-minute film lost four scenes in review, all of that kind,
and was better for each cut.

## Choosing a voice and a language

`edge-tts --list-voices` prints several hundred voices. Set one with `FILM_VOICE`:

```bash
FILM_VOICE=en-GB-RyanNeural python build.py
FILM_VOICE=he-IL-AvriNeural python build.py      # Hebrew, right-to-left narration works fine
```

Rate `-8%` reads calmer than default for technical material. Neural voices handle acronyms
better when you space the letters ("A W S", "E C 2"). Mixed-language sentences are fine in
the voice; in the picture, right-to-left text needs a font that has the glyphs, named
explicitly in `Text(..., font="...")`.

## Common problems

| Symptom | Cause and fix |
|---|---|
| Words split mid-word in the picture ("co nsole") | Manim glyph quantisation at small sizes. The template's `Text` subclass fixes it; keep using it |
| A scene is missing from the film with no error | The Manim scene raised and left no file. `build.py` prints `!!! MISSING SCENES`; read the Manim traceback above it |
| Scene freezes for seconds at the end | Pace not calibrated. See step 3 |
| Voice cuts off the last word | It does not; the mux adds 0.9 s of tail. If it sounds abrupt, end the sentence with a full stop, the voice pauses on punctuation |
| Fonts fall back to a default | The named font is not installed. Manim prints a warning; pick one from your system's font list |
| Build is slow | Use `FILM_QUALITY=-ql` while iterating; switch to `-qm` for the final. Render one scene at a time |
| Hebrew or Arabic renders left-to-right or as boxes | Name a font with the glyphs (`Arial Hebrew`, `Noto Sans Hebrew`) and check Pango is installed with Manim |

## What a good storyboard looks like

The template's `SCRIPT.md` is the shape. For the real film it came from, here is one scene as
it was planned, so you can see the level of detail that turned out to be useful:

> **Decide** · **V:** three stages left to right: request screen, classify, act. The act
> stage splits into three outcomes with their thresholds; an amber highlight walks the
> narration through them. · **N:** "So how does a message become one of those outcomes?
> A request screen, which asks only whether this is a question at all. Then classification:
> which entry applies, and how sure. Then the action. At zero point nine zero or above,
> answer, verbatim. Between zero point seven zero and zero point nine zero, clarify. Below
> that, silence."

Note that the visual line names what moves and in what order, and the narration names the
same things in the same order. That alignment is most of what makes a scene feel produced.

## License

MIT. Use it for your own films, credit appreciated, not required.
