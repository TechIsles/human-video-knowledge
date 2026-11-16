---
name: supercut
description: >
  Turn a long video into a short highlight reel. Given a path or URL to a
  video, supercut extracts the audio with ffmpeg, runs local speech-to-text
  with parakeet-mlx (default) or whisper, picks the most informative
  moments based on the transcript, and concatenates them into a single
  cut at the requested length. Invoke with /supercut when the user asks
  to "summarize a video", "make a highlight reel", "trim this recording
  down", or anything similar. Works on .mp4, .mov, .mkv, .webm, .m4v,
  YouTube URLs, and direct video URLs.
---

# Supercut: long video → short highlight reel

You take a long-form video (a workshop, a meeting recording, a lecture, a
demo) and produce a single short MP4 that captures its highlights. You do
this end-to-end, locally, without uploading the video anywhere.

## Output contract

By the end of a successful run, the working directory contains:

- `video.mp4` — the highlight cut, ≤ the requested length
- `transcript.srt` — the full transcript with timestamps
- `INSTRUCTIONS.md` *(optional)* — only if the user explicitly asks for a
  written summary or tutorial; do not generate by default

Print the absolute paths to whatever you produced.

## Pipeline

You will run these phases **in order**. Each phase has its own checkpoint
where you may need to ask the user a question via `AskUserQuestion`.

1. **Resolve input** — accept a local path or a URL; stage the file.
2. **Extract audio** — 16 kHz mono WAV via ffmpeg.
3. **Transcribe** — local ASR (parakeet-mlx default).
4. **Pick highlights** — read the transcript, pick segments that fit the
   target length.
5. **Cut and concatenate** — re-encode each segment, concat with ffmpeg.
6. **Verify and report** — confirm duration, list segments, print paths.

Use `TaskCreate` to register these as five or six tasks at the start so
the user can see progress.

---

## Phase 1 — Resolve input

The user invokes `/supercut <input> [length]`. Be liberal in what you
accept:

- Absolute or relative paths to `.mp4`, `.mov`, `.mkv`, `.webm`, `.m4v`
- `~`-prefixed paths (expand them)
- HTTPS URLs to a video file (`.mp4`, `.mov`, etc.)
- YouTube / Vimeo URLs (use `yt-dlp` if available)

**Decision tree**:

| Input shape | What to do |
|---|---|
| Local path that exists | Use it directly |
| Local path that doesn't exist | Ask the user to confirm the path |
| HTTPS URL ending in a video extension | `curl -L -o input.<ext>` it |
| YouTube/Vimeo/etc. URL | `uvx yt-dlp -f mp4 -o input.mp4 <url>` |
| Anything else | Ask the user for clarification |

After staging the file, run `ffprobe` to capture its duration and codec
info. You will need the duration later for length calculations.

```bash
ffprobe -v error \
  -show_entries format=duration \
  -show_entries stream=codec_type,codec_name,width,height \
  -of json "<input>"
```

If the video is shorter than ~3× the requested supercut length, warn the
user — there isn't much to cut and a supercut may not be valuable.

### What "length" means

If the user passes a length:
- `5m` / `5min` / `300s` / `00:05:00` — parse and use as a hard ceiling
- "around 10 minutes" — treat as a soft ceiling, aim for 90–105% of it

If the user does not pass a length, **decide a sensible default**:

| Source duration | Default supercut length |
|---|---|
| < 10 min | 60–90s (a teaser) |
| 10–30 min | 2–3 min |
| 30–60 min | 4–6 min |
| 60–90 min | 7–10 min |
| 90 min – 3 h | 10–12 min |
| > 3 h | 12–15 min, ask the user |

Tell the user what default you picked and why, **before** you start
cutting. If they want something else, they will say so.

---

## Phase 2 — Extract audio

You only need the audio for transcription, not the original codec.

```bash
ffmpeg -y -i "<input>" -ac 1 -ar 16000 -vn -c:a pcm_s16le audio.wav
```

This produces a 16 kHz mono `s16le` WAV — the format parakeet-mlx and
whisper both consume natively. Do **not** keep the original audio
codec. Re-encoding to PCM is fast and avoids ASR loader bugs.

---

## Phase 3 — Transcribe

### Default: parakeet-mlx (run via `uvx`)

`parakeet-mlx` is NVIDIA's Parakeet TDT model ported to Apple's MLX. It
runs entirely on the local Apple-Silicon GPU/ANE, English only, and is
substantially faster than whisper on Mac. **This is the default**.

Run it through `uvx` — uv handles the ephemeral environment, the
Python version, and caching for you. **Do not manually create a
`.venv/`.** No `pip install`, no activation, no leftover directories
in the user's working tree:

```bash
uvx parakeet-mlx \
  --output-format all \
  --output-template "transcript" \
  audio.wav
```

First run downloads dependencies into uv's global cache (`~/.cache/uv`)
and the Parakeet model weights to `~/.cache/huggingface/`. Subsequent
runs reuse both — wall-clock drops to roughly 1 minute of audio per
second of compute on M-series Macs.

This emits `transcript.srt`, `transcript.vtt`, `transcript.txt`, and
`transcript.json`. The `.json` has per-word timings — useful for
fine-tuning cut points later.

**Why `uvx` over `uv venv` + `uv pip install`:**

| | `uvx` (use this) | manual `.venv/` (avoid) |
|---|---|---|
| Sets up Python | Automatic | You pick the version |
| Where deps live | uv's global cache | The user's working dir |
| Re-runs | Zero install time | Need re-import check |
| Cleanup | None needed | User has to `rm -rf .venv` |
| Offline after first run | Yes | Yes |

If `uvx` is not installed, install uv first — do **not** fall back to
a manual venv:

```bash
command -v uv >/dev/null || curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Fallbacks

You should fall back to whisper if any of these hold:
- The platform is not Apple Silicon (`uname -m` is not `arm64`)
- The audio is non-English (parakeet is English-only)
- `parakeet-mlx` fails to run (e.g. MLX unavailable)
- The user explicitly asks for whisper

```bash
# CLI entry point name differs from package name → use --from
uvx --from openai-whisper whisper audio.wav \
  --model small --output_format srt --output_dir .
mv audio.srt transcript.srt
```

For larger models or non-English audio:

```bash
uvx --from openai-whisper whisper audio.wav \
  --model medium --language fr --output_format srt --output_dir .
```

For multi-speaker workshops or meetings, **ask** whether the user wants
speaker-labeled output. If yes and a Zoom transcript exists, prefer it.
Otherwise, recommend WhisperX (whisper + pyannote) but do not install it
unless the user agrees — pyannote requires a HuggingFace token.

### Use what's already there

Before installing anything, check the working directory. If the user has
already dropped a `transcript.srt`, `*.vtt`, or a Zoom-style transcript
(e.g. one with speaker names and timestamps), use it. **Re-running ASR
when a good transcript already exists is wasteful and produces worse
output than human-curated transcripts.** See `references/transcript-formats.md`
for parsing patterns.

---

## Phase 4 — Pick highlights

This is the only step where your judgment is load-bearing. Do not skip
it. Do not delegate the entire transcript to a substring grep.

### Strategy

1. **Read the full transcript**. Yes, all of it. Use the SRT or VTT, not
   the JSON — timestamps in srt are the canonical form for ffmpeg. If
   the transcript is over ~15k tokens, sample with stride and read the
   chapter structure first, then drill in.
2. **Identify natural sections.** Look for topic shifts, "okay, so…",
   "let me show you…", introductions of new tools, demos beginning,
   questions from the audience, summaries, closings.
3. **Score segments**. Favor:
   - Concrete demos over abstract preamble
   - Specific commands and code over high-level chat
   - The speaker stating an opinion or recommendation directly
   - Clear closings ("the takeaway is…", "if you remember one thing…")
   - Audience Q&A where the question reveals confusion the speaker
     resolves
4. **Pick segments** under the length budget. Aim for **5–12 segments**,
   each **20–120 seconds**, totaling 90–105% of the target length. Many
   short segments beat one long segment.
5. **Trim to clean boundaries**. Each segment should start at the
   beginning of a sentence and end at the end of one. Use the SRT
   timestamps — they are already sentence-aligned. Avoid cutting
   mid-word.
6. **Order by source timestamp** (chronological). Re-ordering is
   confusing for the viewer.

### Useful patterns

- For workshops/tutorials: include the intro pitch (~30s), one or two
  representative demo moments (the user actually running commands), and
  the closing summary.
- For meetings: include the agenda call-out, each decision moment, and
  any explicit action items.
- For lectures: include the thesis statement, two or three illustrative
  examples, and the conclusion.

If the user supplied **focus topics** ("only the ffmpeg part", "skip Q&A",
"emphasize the demo"), use them as a hard filter before scoring.

### Output of this phase

Produce a Markdown table like this and show it to the user. Then proceed
without waiting for approval — they can interrupt you if it's wrong.

```
| # | Start    | End      | Duration | Why                                |
|---|----------|----------|----------|-------------------------------------|
| 1 | 00:02:03 | 00:04:03 |    120s  | Opening pitch + the 3k-line PR pain |
| 2 | 00:27:58 | 00:29:25 |     87s  | `specify init` demo                  |
| … |          |          |          |                                     |
```

---

## Phase 5 — Cut and concatenate

Use `scripts/cut.sh` (bundled with the skill) — it handles the
re-encode/concat pattern correctly. Do **not** try to use `-c copy` for
cuts at arbitrary timestamps; you'll get keyframe-aligned cuts that
miss the start of sentences.

```bash
bash scripts/cut.sh "<input-video>" "<output-video>" \
  "00:02:03-00:04:03" \
  "00:05:25-00:06:20" \
  "00:27:58-00:29:25" \
  "00:32:10-00:33:35" \
  "01:13:45-01:14:28"
```

The script:
1. Re-encodes each segment to a normalized format (1280-wide, H.264
   `veryfast` CRF 23, AAC 128k, 48 kHz) so concatenation is seamless.
2. Writes a concat list and runs `ffmpeg -f concat -c copy`.
3. Probes the output and prints final duration.

If the script is missing or fails, fall back to inline ffmpeg per
`references/ffmpeg-recipes.md`.

---

## Phase 6 — Verify and report

After the cut completes:

```bash
ffprobe -v error -show_entries format=duration -of default=nw=1:nk=1 video.mp4
```

Confirm:
- Duration is within ±10% of the user's target
- Output is playable (run `ffprobe`, look for stream errors)
- Output filesize is reasonable (>1 MB for >1 min of video)

Report to the user with:
- Absolute path to `video.mp4`
- Final duration
- Number of segments
- Path to `transcript.srt` (so they can adjust and re-run)
- Optional: any segments you considered but cut for length

---

## Quick reference

```bash
# Full happy path on a Mac with parakeet-mlx
ffmpeg -y -i input.mp4 -ac 1 -ar 16000 -vn -c:a pcm_s16le audio.wav
uvx parakeet-mlx --output-format all --output-template "transcript" audio.wav
# (now you read transcript.srt and pick segments)
bash scripts/cut.sh input.mp4 video.mp4 "MM:SS-MM:SS" "MM:SS-MM:SS" ...
```

## Anti-patterns

- ❌ Don't skip transcription and pick segments by chapter markers alone
  — you'll miss the most informative moments inside chapters.
- ❌ Don't use `ffmpeg -c copy` for cuts at non-keyframe timestamps — the
  cut will drift to the nearest keyframe and misalign with the SRT.
- ❌ Don't re-order segments. Chronology helps the viewer.
- ❌ Don't generate a written tutorial, summary, or "blog post" unless the
  user asks for one. The output of this skill is a video.
- ❌ Don't upload the video anywhere. Everything stays local.
- ❌ Don't create a `.venv/` in the user's working directory. Use `uvx`
  — it handles the environment globally and leaves no trace.

## References

- `scripts/cut.sh` — re-encode + concat helper (bundled)
- `references/ffmpeg-recipes.md` — alternative ffmpeg invocations
- `references/transcript-formats.md` — parsing SRT/VTT/Zoom transcripts
- `references/highlight-heuristics.md` — extended scoring rubric
