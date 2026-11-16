---
name: edit-video-story
description: Turn supplied video, audio, photos, and creative direction into a polished short-form edit with story structure, selects, captions, motion graphics, color finishing, sound mixing, export, and full QC. Use for application videos, 60–120 second edits, reels, interviews, trailers, explainers, social videos, DaVinci Resolve projects, Remotion compositions, or any request to splice and professionally finish clips.
license: MIT
---

# Edit Video Story

Act as editor, motion designer, colorist, and finishing producer. Preserve source media. Prefer DaVinci Resolve for the primary edit and finish, Remotion for designed motion graphics, and FFmpeg/ffprobe for inspection, transcodes, and objective QC.

Resolve every bundled relative path from the directory containing this `SKILL.md`; never interpret `references/...` or `scripts/...` from the user's project directory.

## Required workflow

1. **Set the delivery contract and route.** Resolve the audience, platform, exact or maximum duration, aspect ratio, resolution, frame rate, caption requirement, music rights, tone, deadline, and desired project/source files. When the user has not fixed an editor, apply the [autonomous production router](references/production-router.md) to choose DaVinci Resolve, Remotion, or another verified route by expected final quality.
2. **Inventory the media.** Run `scripts/media_inventory.py` on all supplied media. Record duration, frame rate, dimensions, orientation, codecs, audio channels, timecode clues, and technical problems before opening an editor.
3. **Protect the originals.** Work from a new project, timeline, and export destination. Never rename, trim, relink, transcode over, or delete source files.
4. **Find the story.** Transcribe dialogue when useful. Mark hook lines, emotional turns, proof, visual variety, clean reactions, and unusable material. Build a paper edit with source file, in/out, role, spoken line, visual, audio treatment, and reason.
5. **Budget time.** Allocate seconds across the story before building the timeline. For a 90-second application video, start with hook 0–8, context 8–25, proof 25–62, reflection 62–78, and close 78–90; adapt to the actual footage and brief.
6. **Build the radio edit.** Make dialogue coherent and emotionally natural before adding b-roll, effects, or music. Remove repetition, filler, and dead air without creating unnatural speech.
7. **Build the picture edit.** Use cutaways to clarify, compress time, hide necessary dialogue edits, and add specificity. Favor motivated straight cuts; use transitions only when they communicate time, place, memory, or a tonal shift.
8. **Design motion and captions.** Establish a small motion system tied to the visual direction. Read [typography casting](references/typography-casting.md); select titles and captions from the available library for the story's voice rather than defaulting to technical typography. Keep them legible, inside platform-safe zones, and timed to meaning. Load a verified Remotion best-practices skill when one is available for Remotion work.
9. **Finish sound and color.** Prioritize dialogue, control noise and room-tone discontinuities, shape music around speech, and normalize for the destination. Use a color-managed workflow, match shots before applying a look, and avoid a blanket LUT when camera color space is unknown.
10. **Review in three gates.** Review the rough cut for story, picture lock for timing and graphics, and final render for technical and perceptual defects. Continue with best judgment when the user asked for autonomous completion; surface a checkpoint when a subjective fork would substantially change the film.
11. **Run final QC.** Read [video-qa.md](references/video-qa.md) and the [zero-defect gate](references/zero-defect-qa.md), run `scripts/video_qc.py` on the exact export, close the defect ledger, and watch the full file from beginning to end with sound.

## Tool routes

- Read [davinci-resolve.md](references/davinci-resolve.md) for Resolve project setup, scripting boundaries, UI finishing, and safe export.
- Read [remotion-and-ffmpeg.md](references/remotion-and-ffmpeg.md) for motion graphics, probing, transcoding, and deterministic checks.
- Read [editorial-workflow.md](references/editorial-workflow.md) for selects, paper edit, pacing, and a 90-second application structure.

## Creative rules

- Make the first seconds specific, not merely fast.
- Cut on thought, emotion, gaze, gesture, or sound—not on arbitrary intervals.
- Let shots breathe when emotion needs space. Compress setup before compressing payoff.
- Use one coherent title system, caption system, transition grammar, and grade.
- Use generated or stock material only when requested or clearly authorized, and track licenses and provenance. Load `humanize-image-generation` when installed for generated stills, illustrations, matte elements, or render plates; otherwise apply provenance and artifact inspection directly. Inspect the exact framed and graded result in the finished sequence, and do not claim an unavailable specialist workflow ran.
- Do not disguise weak footage with excessive effects, speed ramps, zooms, glitch, or cinematic bars.
- Preserve natural skin tone and intelligible dialogue unless the brief explicitly calls for stylization.
- Avoid synthetic voice, face alteration, or misleading compositing unless explicitly requested and appropriate.

## Delivery contract

Return the final master, the editable project or source composition when requested, and any platform-specific derivative clearly labeled. Report duration, resolution, codec, caption treatment, and QC result. Keep proxies, caches, generated frames, and diagnostic files out of the final handoff.

## Resources

- The bundled `scripts/media_inventory.py` and `scripts/video_qc.py` require Python 3.11+ and FFmpeg/ffprobe on `PATH` or supplied explicitly; run each script's `--help` preflight and do not claim a check ran when its executable is unavailable.
- Read [production-router.md](references/production-router.md), [typography-casting.md](references/typography-casting.md), and [zero-defect-qa.md](references/zero-defect-qa.md) for the bundled standalone routing and release gates.
- Read the editorial, Resolve, Remotion/FFmpeg, and video-QA references for a complete application-video edit; load the bundled routing, typography, and zero-defect references at their corresponding workflow steps.
