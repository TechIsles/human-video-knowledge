---
name: video-understanding
description: Analyze video files and screen recordings by turning them into timestamped evidence bundles with metadata, sampled frames, scene-change keyframes, sudden audio volume-change events, optional ASR speech transcription with automatic language detection and caption files, optional OCR, optional transcript input, event JSONL, and optional long-video chaptering with chunk extraction plus parallel-understanding plans. Use when an agent needs to summarize or understand videos (.mp4, .mov, .m4v, .webm, .avi, .mkv), locate moments by timestamp, extract tutorial or workflow steps, compare scenes, inspect UI screen recordings, detect audio/visual transitions, generate captions, analyze long videos in chunks, or answer questions grounded in video content.
---

# Video Understanding

## Core Workflow

Convert video into a compact evidence bundle before answering nontrivial questions. Avoid trying to reason from the filename or a single arbitrary frame.

1. Identify the user's goal: summary, timestamp search, workflow extraction, visual QA, UI-screen-recording analysis, or comparison.
2. Run the bundled extractor script to create metadata, frames, scene-change keyframes, optional audio volume-change events, ASR captions, OCR, and `events.jsonl`.
3. Read `metadata.json`, `timeline.md`, and relevant `events.jsonl` slices. For detailed event semantics, read `references/evidence_schema.md`.
4. Inspect selected frames with available image/vision tools when visual details matter. Prefer scene-change frames for narrative shifts and sampled frames for coverage.
5. Answer with timestamped evidence. Use `HH:MM:SS` or `MM:SS`, mention uncertainty when inferring actions from pixels, and distinguish observed evidence from interpretation.

## Evidence Extraction

Run the script from this skill folder:

```bash
python3 scripts/video_evidence.py /path/to/video.mp4 --output /tmp/video-evidence --sample-interval 5 --scene-threshold 0.35 --ocr auto
```

Use shorter intervals for precise UI workflows:

```bash
python3 scripts/video_evidence.py /path/to/screen-recording.mov --output /tmp/ui-evidence --sample-interval 2 --scene-threshold 0.25 --ocr auto
```

If the user provides an external transcript, pass it through so it is merged into the evidence:

```bash
python3 scripts/video_evidence.py /path/to/video.mp4 --output /tmp/video-evidence --transcript /path/to/transcript.txt
```

If downstream ASR or audio inspection is needed, extract a mono 16 kHz WAV:

```bash
python3 scripts/video_evidence.py /path/to/video.mp4 --output /tmp/video-evidence --extract-audio
```

To detect sudden volume changes, add audio analysis:

```bash
python3 scripts/video_evidence.py /path/to/video.mp4 --output /tmp/video-evidence --analyze-audio --audio-window 0.5 --audio-change-db 8
```

To transcribe speech and generate captions with automatic language detection:

```bash
python3 scripts/video_evidence.py /path/to/video.mp4 --output /tmp/video-evidence --asr auto --asr-language auto --caption-format both
```

Configure ASR when needed:

```bash
python3 scripts/video_evidence.py /path/to/video.mp4 --output /tmp/video-evidence --asr whisper --asr-model base --asr-language zh --caption-format srt
```

The extractor requires `ffmpeg` and `ffprobe`. OCR is optional and uses `tesseract` when available. If OCR is unavailable, continue with frames and metadata, and inspect frames visually.
ASR is optional. `--asr auto` uses `whisper` when installed, otherwise `whisper-cli` when installed. Whisper may download or use a local model cache on first run.

## Long Videos

For long videos, first create a lightweight global index, then split only as deeply as the user's goal requires.

Use chaptering when the video is too long for one careful pass, when the user asks for detailed understanding, or when parallel workers would make review faster:

```bash
python3 scripts/video_evidence.py /path/to/long-video.mp4 --output /tmp/video-index --sample-interval 10 --scene-threshold 0.30 --analyze-audio --asr auto --chapterize event --chunk-seconds 600 --chunk-overlap 5 --parallel-plan
```

Add `--extract-chunks` when the next step should process physical clips:

```bash
python3 scripts/video_evidence.py /path/to/long-video.mp4 --output /tmp/video-index --sample-interval 10 --scene-threshold 0.30 --analyze-audio --asr auto --chapterize event --chunk-seconds 600 --chunk-overlap 5 --extract-chunks --parallel-plan
```

Choose chunk settings by task:

- Broad summary: `--chunk-seconds 600` to `900`, `--sample-interval 10` to `20`, ASR on when speech matters.
- Tutorial or UI workflow: `--chunk-seconds 120` to `300`, `--sample-interval 2` to `5`, `--ocr auto`.
- Highlights or edit analysis: `--chapterize event`, `--analyze-audio`, and a smaller `--chunk-snap-window` when cuts are frequent.
- Exact captioning: prioritize `--asr auto --asr-language auto --caption-format both`; frames can stay sparse.

`--chapterize fixed` cuts by duration. `--chapterize event` starts from fixed targets, then snaps boundaries to nearby scene changes, sudden audio changes, or transcript segment ends. `--chunk-overlap` expands physical clips around each chapter so ASR and visual reasoning keep local context; during merge, prefer evidence inside the chapter's own start/end window.

For parallel understanding:

1. Run the index pass with `--chapterize ... --parallel-plan`; add `--extract-chunks` when separate workers need clip files.
2. Read `chapters.json` and `parallel_plan.md`.
3. Run each chapter command independently, or assign chapters to parallel workers with the chunk file and the original user question.
4. Merge `chapter_runs/*/timeline.md`, `events.jsonl`, transcripts, and selected frames into chapter summaries.
5. Produce the final answer from the merged timeline, citing timestamps from the original video and calling out cross-chapter transitions.

## Output Files

- `metadata.json`: duration, dimensions, fps, stream information, extraction settings.
- `events.jsonl`: timestamped records for metadata, sampled frames, scene-change frames, audio volume changes, ASR/caption outputs, OCR text, and transcript segments.
- `chapters.json`: chapter windows, clip windows, event counts, and chunk paths when long-video chaptering is enabled.
- `parallel_plan.md`: per-chapter extraction commands and merge guidance when `--parallel-plan` is used.
- `audio_levels.json`: per-window dBFS measurements when `--analyze-audio` is used.
- `transcript.txt`: timestamped ASR transcript when `--asr` is used.
- `captions.srt` / `captions.vtt`: caption files when ASR and caption output are enabled.
- `timeline.md`: human-readable timeline with frame links and OCR snippets.
- `frames/`: JPEG frame assets.
- `chunks/`: optional physical chapter clips when `--extract-chunks` is used.
- `audio.wav`: optional extracted audio when requested.

## Screen Recording Guidance

Treat screen recordings like partial Record & Replay evidence. A normal video does not contain real click, keyboard, or Accessibility-tree events unless they are visibly rendered or separately logged.

- Infer clicks only when the cursor, UI response, or transcript makes the action clear.
- Prefer exact visible text from OCR or frame inspection over guessed labels.
- For tutorial extraction, turn visible UI states into stable semantic steps such as "open Export Settings" rather than coordinates.
- For debugging or app QA, cite the frame timestamp and visible symptom.

## Answering Rules

- Ground every important claim in a timestamp, frame, OCR text, transcript text, or metadata.
- Do not over-sample full-resolution frames unless the user needs frame-accurate inspection.
- Do not copy sensitive on-screen text into the answer unless the user explicitly asks and it is necessary.
- When evidence is sparse, say what signal is missing, such as audio, cursor visibility, or readable UI text.
- For long videos, summarize by chapters first, parallelize chunk review when helpful, then drill into requested windows.

## Resources

- `scripts/video_evidence.py`: create the evidence bundle from a local video file.
- `references/evidence_schema.md`: event schema and interpretation notes.
