---
name: auto-cut
description: Automatic video editing workflow for turning raw footage, scripts, product assets, screen recordings, images, audio, subtitles, or a brief into a polished edited video through a storyboard-first confirmation process. Use when Codex is asked to automate editing, assemble a short video, cut highlights, create social clips, add captions, B-roll, music, voiceover, title cards, overlays, transitions, pacing, or animated packaging. Always organize the edit into a storyboard table first, confirm materials before generation, use packaged templates when possible, use HyperFrames or Remotion for animation/effects, save generated assets back to the matching storyboard/Feishu cells, then cut and QA the final video.
---

# Auto Cut

## Core Rule

Do not start cutting from a vague brief. First turn the request into a shot-by-shot storyboard table, including A-roll, B-roll, template/animation choice, subtitles, cleanup notes, and feedback/status fields. The storyboard table is the source of truth. If a Feishu/Lark document is available and accessible, update that document; otherwise maintain a local mirror and clearly say it still needs syncing.

When the user says to only record, update the skill/storyboard/notes only. Do not generate assets, render, upload, or cut video in that turn.

The uploaded main video defines the master canvas by default. Do not force a 9:16, 1:1, 16:9, blurred-background, crop, contain, or letterbox treatment unless the user explicitly asks for a target delivery ratio. If the source video is 4:3, 16:9, 9:16, square, or any other size, keep that source aspect ratio and framing for the main edit. The default edit only removes dead air, breaths/gasps, repeated words, restarts, and obvious stumbles; otherwise leave the source picture unchanged.

Never distort footage. All video and image scaling must preserve aspect ratio. Do not stretch the main video, A-roll, B-roll, screenshots, logos, or template previews horizontally or vertically. Distortion includes non-uniform scaling, squeezed faces, widened/narrowed UI, duplicated side strips, tiled edge fills, or mismatched split copies of the same frame. If padding is needed after an explicitly confirmed target canvas change, use a neutral background, clean letterbox/pillarbox, or a separately approved background treatment; never use stretched or duplicated source strips as a hidden fill.

B-roll can be user-uploaded material, not only generated template cards. User-uploaded B-roll is treated as approved source material and must keep its original useful composition by default: no stretching, non-uniform scaling, destructive crop, local zoom, edge duplication, speed change, or arbitrary temporal cutting. Use aspect-preserving fit/contain, picture-in-picture, a framed insert, or the full original clip/image unless the storyboard explicitly records the user's approval for a specific crop or time trim.

## Workflow

1. **Segment the narration**
   - Segmentation is not "understanding the video"; it turns long narration into the smallest editable expression units.
   - Every segment must have one clear meaning, map to one visual, last about `2-6s`, and be able to match a template card or B-roll.
   - Use the deterministic segmentation method below and output `segments.json` before building the storyboard.

2. **Build the storyboard table**
   - Include `镜号`, `时间`, `旁白/文案`, `逐字文案/字幕拆解`, `画面设计`, `A-roll`, `B-roll`, `模板/动画`, `字幕动画`, `字幕/屏幕文字`, `清理剪辑`, `转场`, `生成动画/视频确认`, `修改意见`, `备注/待确认`.
   - Resolve source materials before planning. Preserve asset types: images stay images, videos stay videos, screen recordings stay screen recordings.
   - If the user has uploaded B-roll for a shot, record that exact asset in the `B-roll` cell and mark it `confirmed` or `candidate`. Do not replace it with a generated template unless the user asks for a template instead.
   - Record source media dimensions, frame rate, duration, and aspect ratio in `source-manifest.md`. Mark the original main video aspect ratio as the default project canvas.
   - In the storyboard `A-roll` cell, write `保持原始画幅和构图；仅剪气口/重复/口误` unless the user has explicitly requested another framing.
   - Mark each material as `confirmed`, `candidate`, `missing`, or `generate`.
   - If no matching template exists or a mapping is uncertain, ask the user which template/style to use instead of inventing one.

3. **Wait for confirmation**
   - Stop after the storyboard/material table unless the user explicitly says to proceed, such as `确认分镜`, `A/B-roll 已确认`, `可以生成动画`, `可以剪辑`, or an equivalent current-message authorization.
   - Before generating or cutting, re-read the latest `修改意见` or current user instruction.

4. **Generate needed assets**
   - If a shot already has user-uploaded or registered B-roll, do not generate a template asset by default. Generate anyway only when the user explicitly asks.
   - User-uploaded B-roll takes priority over generated cards. Generated templates are fallback visual assets, not replacements for approved user material.
   - Use packaged templates first: `pain_title_card`, `number_impact_card`, `three_step_process_card`, `before_after_card`, `operation_ui_card`, `keyword_highlight_card`, `result_showcase_card`, `quote_summary_card`.
   - Template outputs must use transparent backgrounds by default. Prefer MOV/ProRes alpha for video overlays; use PNG sequence when frame-level alpha is needed; use MP4 only for standalone previews.
   - Put generated assets back into the matching row's `B-roll` cell and set `生成动画/视频确认` to `已并入 B-roll`. If Feishu is not writable, update the local mirror and create a sync table.

5. **Use Remotion / HyperFrames for animation**
   - Use HyperFrames for packaged cards, motion graphics, kinetic captions, lower thirds, and transparent overlay clips. Also invoke `hyperframes:hyperframes` and `hyperframes:hyperframes-cli` when authoring or rendering HyperFrames.
   - Use Remotion for complex React-based timelines, componentized animation systems, advanced transitions, audio-reactive visuals, or when the project already uses Remotion. Also invoke `remotion:remotion-best-practices` before writing Remotion code.
   - Do not hand-roll complex animation in ad hoc image scripts unless it is a small utility for text/overlay generation and the user accepts it.

6. **Cut the video**
   - Tighten A-roll first: remove dead air, long breaths, restarts, filler, and obvious stumbles.
   - Default A-roll cleanup is timing-only: cut breaths/gasps, repeated phrases, false starts, and obvious mistakes. Do not otherwise alter the main picture, scale, crop, reframe, blur-fill, or change aspect ratio.
   - Preserve the original main video canvas exactly by default: output width/height and display aspect ratio should match the uploaded main video unless the user explicitly confirms a different delivery ratio. For example, do not turn a non-9:16 source into 9:16 just because the target is a social clip.
   - Never use non-uniform scaling. In `ffmpeg`, avoid direct `scale=<fixed_width>:<fixed_height>` on footage unless the source already has exactly that ratio. Prefer `scale=-2:<height>`, `scale=<width>:-2`, or `force_original_aspect_ratio=decrease/increase` with explicit, approved pad/crop behavior. Do not create side fills by duplicating, stretching, or offsetting the same footage.
   - Cover awkward cuts with confirmed B-roll, screen recordings, screenshots, or generated transparent overlays.
   - Do not damage user-uploaded material composition. For user-provided B-roll videos/images, preserve the subject, full context, original aspect ratio, and intended framing by default. Do not crop a person/video down to only a head or other unusable fragment. Do not stretch, speed-ramp, punch in, or cut a user-uploaded B-roll clip shorter unless that exact trim is confirmed in the storyboard. Use fit/contain, letterbox/pillarbox, picture-in-picture, a framed insert, or the complete source clip instead.
   - If overlays, cards, captions, or B-roll have a different aspect ratio than the main video, adapt those overlay assets to the source canvas, not the other way around. Transparent overlays should be rendered at the master canvas size whenever practical.
   - Subtitle timing must be derived from the final edited audio timeline, not from estimated segment durations. After A-roll cuts are applied, rebuild or retime subtitle events against the edited audio. Captions must never appear before the spoken audio they represent; if sync is uncertain, bias subtitle start times slightly later rather than earlier.
   - Keep all visible text viewer-facing. Never show row IDs, file names, QA labels, debug badges, workflow notes, or internal status text in the final video.
   - Use `ffmpeg`, existing repo tooling, HyperFrames, or Remotion for deterministic render scripts. Keep the project reproducible.

7. **QA and deliver**
   - Verify media loads, B-roll appears in planned shots, generated overlays are composited correctly, text fits, captions sync, no accidental black/blank frames appear, and audio is intelligible.
   - Verify the output canvas matches the uploaded main video dimensions/aspect ratio unless an explicit target ratio was confirmed. Flag any scale/crop/blur-fill/letterbox introduced without confirmation as a blocking QA failure.
   - Verify no visible distortion: faces, circles, logos, UI screenshots, and room geometry must retain their original proportions. If a frame shows stretched/squeezed people, duplicated side strips, tiled source edges, or split mismatched copies, treat it as a blocking QA failure and rerender with aspect-preserving scaling.
   - Verify subtitle sync on at least the opening, middle, and ending sections after final audio cuts. A subtitle may appear at the same time as speech or slightly after it, but it must not lead the voice. If captions lead the voice, retime and rerender before delivery.
   - Inspect metadata with `ffprobe`: duration, resolution, codec, frame rate, audio.
   - Deliver the final MP4 path, project folder, render command/script, edit summary, cleanup log, QA notes, and any user review risks.

## Narration Segmentation Rules

Use this rule-driven method before storyboard creation. It should be stable and predictable; do not depend on vague model interpretation.

1. **Clean text**
   - Remove extra spaces.
   - Merge repeated punctuation.
   - Normalize Chinese punctuation to `。，！？`.
   - Remove filler particles when they do not carry meaning, such as `啊`, `呢`, `哈`, `这个`.

2. **Hard sentence split**
   - Force split on `。`, `？`, `！`, line breaks, and `；`.

3. **Keyword split**
   - Split again when these structure words appear: `第一步`, `第二步`, `第三步`, `首先`, `然后`, `接下来`, `最后`, `比如`, `例如`, `重点是`, `你会发现`, `我们来看`.

4. **Length control**
   - If a sentence is over `25` Chinese characters, split it.
   - Prefer split points: `，`, `但是`, `并且`, `然后`, `所以`, and natural semantic pauses.
   - If a segment is under `8` Chinese characters, try to merge it with the adjacent segment.

5. **Type tagging**
   - `number`: contains `%`, `元`, `万`, `小时`, `分钟`, `倍`, or digits.
   - `process`: contains `第一步`, `第二步`, `首先`, `然后`, `最后`.
   - `compare`: contains `以前`, `现在`, `原来`, `后来`, `不再`, `而是`, or `从...到...`.
   - `operation`: contains `点击`, `打开`, `输入`, `上传`, `选择`, `生成`, `导出`.
   - `pain`: contains `麻烦`, `痛苦`, `太慢`, `不会`, `卡住`, `浪费时间`.
   - `summary`: contains `所以`, `总结`, `本质`, `核心`, `你会发现`, `重点`.
   - Otherwise use `statement`.

6. **Duration estimate**
   - Estimate as `Chinese character count * 0.25s`.
   - Clamp to a minimum of `2s` and maximum of `6s`.

7. **Merge optimization**
   - Merge consecutive same-type segments when their total duration is under `6s`.
   - Avoid merging across a strong visual/template boundary.

8. **Output format**

```json
[
  {
    "id": "s001",
    "text": "Codex是更适合普通人的工具",
    "type": "statement",
    "duration": 3
  }
]
```

Then map each segment into the storyboard table as a shot or sub-shot.

## Template Tools

Use the preset library at [template_presets.json](templates/template_presets.json). For exact layout/timing rules, read [card-template-specs.md](references/card-template-specs.md) only when changing or validating template behavior.

## Editing Style Templates

When the user asks to reuse a saved reference style, asks for a viral/reference-video editing prompt, or says to use a saved template, read [editing-style-templates.md](references/editing-style-templates.md). Apply the relevant style as storyboard guidance, not as a reason to skip the storyboard-first confirmation workflow. The template named `graph-animation-tutorial-dark` is for dark, high-contrast PPT / graphic-animation tutorial videos with process diagrams, large gold-highlighted keywords, operation B-roll, and step-by-step cards.

Standalone template render:

```bash
python3 scripts/render_cards.py \
  --template three_step_process_card \
  --format mov \
  --output-dir ./card-renders
```

Storyboard-driven template render:

```bash
python3 scripts/render_storyboard_templates.py \
  --storyboard ./storyboard.json \
  --shots S01 S02 S03 \
  --field-overrides ./storyboard-template-fields.json \
  --output-dir ./generated-assets/storyboard-templates
```

Defaults:

- `render_cards.py` renders transparent output unless `--no-transparent` is passed.
- `render_storyboard_templates.py` defaults to transparent MOV.
- `render_storyboard_templates.py` skips existing B-roll unless `--no-skip-existing-broll` is passed.
- Generated sync files: `feishu-cell-updates.json` and `feishu-cell-updates.md`.
- Run these commands from the skill directory. If the skill is installed elsewhere, use that local path; do not assume a fixed user home path.

## Required Local Mirrors

Use these files when Feishu is inaccessible or as working mirrors:

- `storyboard.json` for machine-readable rows.
- `storyboard.md` for reviewable rows.
- `feishu-cell-updates.md` for content that must be pasted/synced into Feishu cells.
- `source-manifest.md` for source media inventory, including original main video dimensions, aspect ratio, frame rate, duration, and whether any target output ratio was explicitly confirmed.
- `edit-plan.md`, `cleanup-log.md`, and `review-notes.md` for final handoff.

## Quality Bar

- Use polished Chinese-capable fonts such as PingFang SC, Source Han Sans, or Noto Sans CJK SC.
- Captions and key text should normally animate for social/tutorial clips unless the user asks for static text.
- Captions must be synchronized to the final edited audio. Never deliver captions that visibly appear before the voice.
- B-roll must support the active line. Avoid decorative or unrelated inserts.
- B-roll may be user-uploaded material. User-uploaded B-roll has priority over generated templates and must not be stretched, destructively cropped, locally zoomed, speed-changed, or arbitrarily cut; use the full clip/image with aspect-preserving placement unless the user explicitly approves a specific trim or crop.
- The main video should keep its uploaded aspect ratio and framing by default. Do not standardize it to 9:16 or any other ratio without explicit user confirmation.
- Never distort video or image assets. Preserve aspect ratio for every source, overlay, logo, screenshot, and template preview. Stretched or duplicated side-strip fills are not acceptable.
- User-uploaded B-roll must keep its original useful composition unless the user approves a specific crop. Never crop a person/interview clip so tightly that only the head remains or context is lost.
- Preserve factual meaning. Do not invent claims, numbers, testimonials, product behavior, or proof assets.
- If a legal, revenue, medical, financial, or brand claim is uncertain, flag it for user review instead of strengthening it.
