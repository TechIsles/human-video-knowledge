---
name: realistic-video-prompting
description: Design and review model-agnostic video prompts that should resemble captured real-world footage, including phone clips, consumer camcorder recordings, analog tape, small-gauge film, action cameras, security cameras, webcams, observational documentary, and found footage. Use only when the user asks for natural capture, physical camera behavior, documentary evidence, or reduced AI polish. Do not trigger for ordinary video generation or polished commercial video requests.
metadata:
  author: HiAPIAI
  license: MIT
  version: 1.2.0
  short-description: Preflight realistic capture prompts
---

# Realistic Video Prompting

Design the clip as a believable recording event before describing its look.

This skill is independently useful for prompt writing and works with any video model. When the user explicitly asks to render, select the matching installed model Skill or API. Seedance 2.0 is one explicit built-in adapter, not the default for every video request.

## Routing

Use this skill for:

- phone, consumer camcorder, analog tape, small-gauge film, action-camera, CCTV, webcam, documentary, or found-footage requests;
- prompts that should feel incidental, observational, unpolished, or physically recorded;
- reviews that specifically ask to reduce synthetic stability, commercial grading, staged acting, or unexplained camera movement.

Do not use it just because the user says “video” or “generate.” Ordinary model requests go directly to that model's executor. Ordinary Seedance tasks go directly to `hiapi-seedance-2-0-video`.

## Operating Modes

### `prompt-only`

Default. Write and preflight the prompt. Do not call any generation API.

### `review-and-render`

Return the reviewed prompt and `awaitingApproval: true`. The first run must not render. Only repeat with `--approved` after explicit user approval.

### `direct-render`

Use only when the current user request already says to generate, render, make, or output the video. Run preflight, then use an explicitly selected executor.

Never infer Seedance from the absence of a target model. Never silently select expensive resolution, storage, or duration options.

## TRACE Capture Framework

Build the prompt through five decisions:

1. **T — Take provenance**
   - Who pressed record?
   - Why were they recording?
   - What device or medium was used?
   - What year or era does the recording belong to?
2. **R — Real-world evidence**
   - Which visible objects prove the place, weather, time, and period?
   - Which details move independently of the subject?
3. **A — Acquisition physics**
   - How does the selected camera respond to motion, light, focus, compression, and handling?
   - Use one coherent capture profile from `references/camera-aesthetics.md`.
4. **C — Continuity and causality**
   - What changes from the first second to the last?
   - How do actions follow from one another?
   - Which small interruption makes the take feel unplanned?
5. **E — Environmental sound**
   - Which visible sources produce each sound?
   - What would this device actually record?

The final prompt still uses seven predictable sections so the automated reviewer can verify it.

## Required Prompt Layout

Use this order:

```text
Character
Location
Visual Style
Camera Style
Timeline
Audio
Goal
```

Chinese equivalents are accepted.

### Character

- Describe only visible, continuity-relevant traits.
- Include clothing material and color, hair, carried objects, and natural posture.
- State that identity and appearance stay consistent.

### Location

- Establish time, place, weather, and period evidence.
- Use a small set of mutually compatible objects instead of a long decorative inventory.
- Include at least one background process: moving shadow, traffic, steam, wind, rain, machinery, or passersby.

### Visual Style

- Describe observable behavior and material response.
- Replace broad quality words with evidence the camera could record.

### Camera Style

- Name the capture device.
- Include multiple device-consistent behaviors affecting framing, focus, exposure, motion, signal, or compression.
- Add at least three explicit exclusions that prevent incompatible polish.

### Timeline

- Use ranges such as `00:00–00:03`.
- Every beat must advance the same take or a clear sequence.
- Include subject action, environmental response, and one plausible recording irregularity.
- End with a credible recording boundary: lowered camera, subject leaving frame, hard stop, fixed empty frame, or signal interruption.

### Audio

- Identify the recording mode: synchronous phone mic, camcorder mic, silent film, muted CCTV, or documentary field audio.
- Connect sounds to visible sources.
- Exclude music, designed effects, or narration unless requested.

### Goal

- One sentence describing what kind of recovered or observed moment the finished clip should resemble.

## Automatic Preflight

Save the prompt and run:

```bash
node scripts/realistic-video-workflow.mjs \
  --prompt-file /absolute/path/to/prompt.txt \
  --mode prompt-only
```

Blocking checks cover:

- all seven sections;
- recorder/device provenance;
- timecoded progression;
- physical capture behavior;
- three or more exclusions;
- executor-specific parameters only when an explicit built-in adapter validates them.

Warnings flag missing closing behavior and vague default-aesthetic terms. Resolve warnings with judgment.

## Model Handoff

Review without rendering:

```bash
node scripts/realistic-video-workflow.mjs \
  --prompt-file /absolute/path/to/prompt.txt \
  --mode review-and-render
```

After explicit approval:

```bash
node scripts/realistic-video-workflow.mjs \
  --prompt-file /absolute/path/to/prompt.txt \
  --mode review-and-render \
  --approved \
  --renderer external \
  --target-model "Veo"
```

When the current request already authorizes rendering:

```bash
node scripts/realistic-video-workflow.mjs \
  --prompt-file /absolute/path/to/prompt.txt \
  --mode direct-render \
  --renderer external \
  --target-model "Kling"
```

`external` returns a model handoff package without creating a task. Immediately validate that package against the target model's current official schema, then invoke the matching installed Skill or API. Model names in examples are routing labels, not hardcoded API identifiers.

For the explicit Seedance 2.0 adapter:

```bash
node scripts/realistic-video-workflow.mjs \
  --prompt-file /absolute/path/to/prompt.txt \
  --mode direct-render \
  --renderer seedance-2.0 \
  --seconds 5 \
  --resolution 720p \
  --ratio 16:9
```

The Seedance adapter imports the installed Seedance skill’s current payload validator before starting its CLI. This repository does not maintain a second `/v1/tasks` client.

See `references/model-routing.md` for generic routing and `references/seedance-handoff.md` for the optional Seedance adapter.

## Delivery

For prompt-only work, return:

- capture provenance;
- selected mode;
- preflight result;
- final prompt;
- adjustable controls: recorder, device, era, environment, recording boundary.

For rendering, return only verified executor facts:

- task id and status;
- local path or remote URL;
- HTTP status and next action on failure.

Do not claim visual adherence unless the returned video was opened and inspected.

## References

- `references/camera-aesthetics.md`
- `references/atmosphere-dictionary.md`
- `references/anti-ai-checklist.md`
- `references/model-routing.md`
- `references/seedance-handoff.md`
