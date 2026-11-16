---
name: reverse-motion-system
description: Reverse-engineer any motion design video from pixels alone, then recreate it — or your own original piece — through the Higgsfield MCP. Measures beat structure, camera moves, shapes and palette with computer vision, writes a spec.json contract, converts it into a structured JSON video prompt (Google Omni style), and ships the render via Higgsfield. Trigger on - reverse engineer this video, recreate this motion design, measure this reference, clone this animation, build my custom animation from my references.
---

# The Reverse Motion System

Recreate pixel-perfect motion design from any reference video — by MEASURING it, not eyeballing it. Then flip the same system to render something entirely your own.

Every camera move, shape, colour and cut in the final prompt is a calculated value extracted with computer vision. You are never prompting blind.

## Requirements

- Python 3.10+ with `opencv-python-headless` and `numpy`
  (`pip install opencv-python-headless numpy` — use a venv if your system blocks pip)
- The **Higgsfield MCP** connected to Claude Code (`claude mcp add --transport http --scope user higgsfield https://mcp.higgsfield.ai/mcp`)

## The pipeline (run it in order)

### Stage 1 — Measure the reference

```bash
python scripts/measure.py reference.mp4 -o spec.json
```

Computer vision reads every frame and pulls out:
- **Structure**: cuts scored as frame-diff ratio against the local median, fitted black-window and white-flash detectors → the beat list
- **Camera**: Farneback optical flow on lit pixels, solved per frame pair for a similarity transform (scale / rotation / translation) with physical admissibility guards → push-in, pull-back, pan, roll, per-frame values
- **Elements**: connected-component blobs at each beat's midframe → shape class, normalized position and size
- **Palette**: k-means over lit pixels per beat → locked hex values
- **Look**: background floor, grain, bloom notes

`spec.json` is the contract. Later stages read it and never re-derive a number from pixels.

### Stage 2 — Convert the spec into a structured prompt

```bash
python scripts/spec_to_prompt.py spec.json -o prompt.json --style "neon motion graphics on black"
```

Builds a shot-by-shot JSON video prompt: one shot per scene beat, transitions carried from the measured black/flash windows, camera phrased from the measured motion, palette locked to the measured hexes.

### Stage 3 — Render through the Higgsfield MCP

Inside Claude Code, with the Higgsfield MCP connected, ask:

> "Read prompt.json and generate this video on Higgsfield. Use the Google Omni model if it is available in the marketplace, otherwise Seedance 2.0. 9:16, 1080p, honor every shot duration, camera move and palette exactly."

The agent lists the available Higgsfield models, picks the conversational JSON-prompt model, fires the generation, polls the job, and returns the video URL.

### Stage 4 — Your own sauce (the point of the system)

Copying proves the control. Now feed the system YOUR direction instead of a reference:

- Edit `spec.json` directly — change palettes, swap camera moves, retime beats
- Or keep a folder of reference clips (your **inspiration bank**), measure each one, and mix beats across specs into one prompt
- Add your own custom shapes, fonts and camera moves by describing them in the shot descriptions

Then re-run stage 2 + 3. The render follows your numbers with the same precision it followed the reference's.

## Rules for the agent

1. NEVER skip the measurement stage and improvise a prompt — the measured values are the whole point.
2. NEVER re-derive numbers from pixels after `spec.json` is written; edit the spec instead.
3. If the Higgsfield MCP is not connected, stop and give the user the one-line install command above.
4. Verify the model actually available on Higgsfield before claiming it (list models first; prefer Google Omni for JSON prompts, fall back to Seedance 2.0).
5. Aspect ratio 9:16 and 1080p by default unless the user says otherwise.
