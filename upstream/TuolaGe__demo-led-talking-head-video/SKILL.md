---
name: demo-led-talking-head-video
description: Produce, revise, or QA a complete 16:9 creator-style product demo video with a square or circular digital-human presenter over a HyperFrames product story. Use this skill whenever the user asks for a 横版口播、产品演示口播、1:1 数字人画中画、圆形人物视频、presenter-over-demo video, multilingual launch narration, MiniMax-to-剪映换音色 workflow, HeyGen image-to-video, word-highlight captions, branded intro/outro, or a repeatable 16:9 demo-led talking-head production—even when they mention only one stage.
---

# Demo-Led Talking-Head Video

Create a finished 16:9 product story in which the product demonstration is the
main visual and a square or circular digital-human presenter provides trust,
continuity, and creator energy.

This skill owns the **product-demo-plus-presenter** format. Use
`$vertical-talking-head-video` for a 9:16 full-presenter video. Use
`$rachel-digital-human-production` when the task is only the authorized
MiniMax/HeyGen avatar pipeline and does not need the 16:9 product narrative.

The result should feel like a strong YouTube product explanation, not a webinar
slide deck, a screen recording with a face pasted on top, or a corporate ad.

## Production contract

Use this sequence:

```text
brief and claim check
-> four native scripts (zh/en/ja/es)
-> approve the production-language script
-> native pronunciation master
-> authorized personal-timbre replacement in Jianying/CapCut
-> approve final narration
-> approve portrait and presenter shape
-> 12–15 second HeyGen preview
-> approve avatar behavior
-> full 1:1 presenter video
-> gather real product evidence and brand assets
-> storyboard product proof + narrative cards
-> approve storyboard
-> build and animate the 16:9 HyperFrames story
-> transcribe final audio and create word-synchronized captions
-> composite presenter, product story, audio, captions, intro, and outro
-> approve the final Studio preview
-> high-quality render, full-file QA, and named delivery
```

Keep script, performance, timbre, avatar behavior, storyboard, and final
composition as separate approval layers. A later layer should not conceal a
defect in an earlier layer.

## Operating boundaries

- Confirm that the user owns or is authorized to use the portrait, voice,
  script, brand assets, screenshots, and publishing purpose before cloning or
  generating a person.
- Read secrets only from environment variables or connected services. Never
  repeat keys, authorization headers, signed URLs, or private credentials.
- Treat TTS, voice conversion, avatar generation, and cloud rendering as paid
  external actions. Obtain or rely on explicit approval before the first paid
  action and before a paid regeneration.
- Generate a representative 12–15 second avatar preview before the full paid
  avatar unless the user explicitly waives it.
- Reuse stable asset IDs and job IDs. Resume polling after a timeout instead of
  submitting a duplicate paid job.
- Do not imitate a living creator's distinctive voice or performance. Translate
  references such as “MrBeast energy” into broad attributes: fast hook,
  confident projection, varied emphasis, bright affect, and retention-focused
  pacing.
- Verify launch status. Do not turn “coming soon” into “available now.”
- Label simulated or future product behavior as `CONCEPT DEMO`, `PROTOTYPE`, or
  an equivalent visible disclosure.
- Do not bundle personal portraits, voice samples, raw avatars, API keys, signed
  URLs, or private client footage when distributing this skill.

## 1. Set up the project

Create or reuse this layout:

```text
project/
├── inputs/
│   ├── brief.md
│   ├── authorization.md
│   ├── portrait-square.png
│   ├── product-sources.md
│   └── brand/
├── scripts/
│   ├── script-zh.md
│   ├── script-en.md
│   ├── script-ja.md
│   └── script-es.md
├── audio/
│   ├── pronunciation-master-<lang>.wav
│   ├── personal-voice-<lang>.wav
│   ├── transcript.json
│   └── audio_meta.json
├── presenter/
│   ├── preview-15s.mp4
│   └── full-square-raw.mp4
├── hyperframes/
│   ├── BRIEF.md
│   ├── STORYBOARD.md
│   ├── assets/
│   ├── compositions/
│   ├── snapshots/
│   └── index.html
├── work/
│   └── job-state.json
└── outputs/
    └── <brand>-<campaign>-<lang>-demo-led-v01.mp4
```

Use `scripts/init_project.py --root <project> --project <name>` when starting
from scratch. Record source hashes, provider asset IDs, approvals, paths, and
job status in `work/job-state.json`; never store secrets.

## 2. Write four native scripts

Always create Chinese, English, Japanese, and Spanish versions, even when only
one language is being produced. They are four localized performances of the
same argument, not sentence-by-sentence translations.

Read `references/script-playbook.md` before drafting or revising scripts.

The proven arc is:

```text
identity-changing hook
-> familiar old frustration
-> plain-language change
-> visible proof
-> mechanism
-> audience participation
-> larger team or workflow reveal
-> conversational reframe
-> low-friction CTA
```

Write one idea per breath group. Prefer verbs and visible behavior over feature
adjectives. Remove sentences that depend on shouting or exaggerated acting.
Default to 35–50 seconds; shorter is better than padded.

Pause for approval of the production-language script before generating audio.

## 3. Build narration in two layers

The preferred method separates **performance** from **identity**.

### A. Native pronunciation master

Use MiniMax or another API-connected speech tool to generate a native
performance with a provider voice:

- native pronunciation, rhythm, and phrase linking
- bright, confident, smooth creator energy
- warmth and clarity without shrillness or shouting
- close, dry, clean recording
- natural speed that supports lip sync
- no background music, room tone, reverb, or environmental noise

For English, default to contemporary General American with an easy West Coast
feel. The “California” quality should come from warmth, pace, and confidence,
not a caricatured accent.

### B. Authorized personal timbre

Import the approved pronunciation master into Jianying/CapCut. Select the audio,
open the right-side audio controls, choose **换音色**, and apply the user's
previously authorized saved personal voice. Export the transformed audio.

This produces:

```text
native pronunciation and performance
+ authorized personal timbre
= final narration for the presenter
```

Keep the pronunciation master and transformed file separately. Listen to the
entire transformed file for consonant smearing, lost word endings, metallic
artifacts, pitch instability, clipped starts, noise, and loss of native rhythm.

Use an authorized direct MiniMax clone only as a fallback when voice replacement
is unavailable, the language matches the speaker's natural delivery, or the
user approves it after an A/B comparison.

Read `references/audio-and-avatar.md` for prompts, preview design, and avatar QA.

## 4. Approve the portrait and presenter shape

The source portrait should feel like a relaxed creator speaking to their own
camera:

- square canvas, preferably 1024 × 1024 or larger
- clear front or near-front face and visible mouth
- head, shoulders, and upper torso present
- relaxed asymmetry rather than passport-photo posture
- direct or near-direct eye contact
- believable home, studio, or creator environment
- natural light, coherent shadows, and real skin tonal variation
- hair, hands, props, or filters do not cover the mouth

Avoid waxy skin, enlarged eyes, elongated faces, formal centered posture,
impossible shadows, and a frozen hand near the face.

Choose the presenter container according to the composition:

- **rounded square** — default; best for product launch and creator videos
- **circle** — useful when the presenter should feel lighter and less dominant
- **soft portrait card** — use when the presenter needs slightly more torso

The presenter is a supporting layer. On a 1920 × 1080 canvas, start around
320–380 px square. Reserve more space only when facial emotion is a major part
of the story.

## 5. Generate the 1:1 presenter

Use HeyGen or an equivalent image-driven presenter service with the approved
portrait and final personal-voice audio.

Generate a 12–15 second preview containing the hook, a fast phrase, product
names, open vowels, plosives, and one emphasized phrase. Review:

- identity and face shape
- lip-sync timing and mouth amplitude
- teeth, jaw, and lip deformation
- eye contact, blinking, and head movement
- shoulder and upper-body motion
- background stability
- start/end clipping

Conversational mouth motion is preferable to maximum expression. If the mouth
opens too widely, first reduce provider motion/expressiveness, use a cleaner
portrait, or use a more naturally paced audio master.

If the full avatar has almost no arm movement, do not leave a large static torso
in the final composition. Either:

1. regenerate with a supported motion prompt and restrained expressiveness, or
2. crop the final picture-in-picture to head, shoulders, and a small amount of
   upper chest.

The second option is often more natural for a small YouTube-style overlay.
Never fake body motion with aggressive wobble or random camera shake.

After approval, generate the full 1:1 presenter, download it, verify that its
duration matches the final narration, and preserve it as
`presenter/full-square-raw.mp4`.

## 6. Gather product evidence

Use the real product whenever possible:

- clean screenshots or recordings of the current product
- documented navigation and controls
- real comment, edit, history, version, asset, or collaboration states
- official transparent logos and brand tokens
- exact product names and launch status

Separate:

- **documentary proof** — real UI and real behavior
- **authored reconstruction** — a faithful UI needed because clean capture is
  unavailable
- **concept demo** — future, simulated, or multi-agent behavior

Never present reconstruction or concept behavior as an unqualified product
recording.

## 7. Storyboard the 16:9 product story

Use HyperFrames for the 16:9 narrative. Read
`references/hyperframes-story-playbook.md` before creating or revising the
storyboard.

Map narration beats to individual scenes using exact audio timing. Alternate:

- product-proof frames with real UI
- designed narrative frames that simplify an idea
- interaction frames showing cursor, comment, edit, or state change
- one hero frame for the largest workflow or team implication

Do not use full product screenshots for every sentence. The visual rhythm should
follow the argument:

```text
show the product when proof matters
-> use a designed card when an idea needs clarity
-> return to the product when demonstrating action
```

Add a short silent branded opening and a short silent actionable closing:

- Opening: partner logos at equal optical weight and aligned visible centers;
  inspect a rendered frame because transparent padding and asymmetric marks can
  defeat equal CSS boxes. Use one lockup line, no presenter or captions.
- Closing: show the exact next action—search, install, join, open, or try—long
  enough to understand; no presenter or captions.

Pause for storyboard approval before full HTML authoring.

## 8. Build and validate HyperFrames

Use the mandatory `$hyperframes` workflow and the matching domain skills.

- Keep one `BRIEF.md` and one `STORYBOARD.md`.
- Build each scene as a sub-composition.
- Make all animation seek-safe and deterministic.
- Use real narration duration from `ffprobe`, not a rounded estimate.
- Transcribe the final audio with word timestamps.
- Correct transcript words against the approved script without destroying their
  measured timing.
- Drive scene lengths from narration beats, with explicit silent opening and
  closing durations.
- Alternate tracks for overlapping transitions.
- Use a common environmental floor so different product and card scenes feel
  like one film.
- Inspect one midpoint snapshot for every mounted scene.
- Run the animation map and final `hyperframes check`.
- Open the final Studio timeline and wait for user approval before rendering.

Prefer an editorial design system with unequal card sizes, clear hierarchy, and
generous whitespace. Avoid rigid equal-card grids and generic dashboard slop.

## 9. Composite the presenter

The presenter should:

- appear after the branded opening
- remain continuous across narrated scenes
- disappear before the silent closing
- sit where product controls are least important
- keep a consistent margin, border radius, and shadow
- never cover the primary action or active caption line

For a 1920 × 1080 rounded-square presenter, start with:

```text
size: 320–380 px
right margin: 44–64 px
bottom margin: 48–72 px
corner radius: 20–28 px
```

When the raw square avatar shows too much static torso, place it inside an
overflow-hidden wrapper and zoom the inner video around the face:

```css
.presenter-shell {
  width: 340px;
  height: 340px;
  overflow: hidden;
  border-radius: 24px;
}
.presenter {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: 50% 24%;
  transform: scale(1.34);
  transform-origin: 50% 30%;
}
```

Tune these values to the actual portrait; do not crop hair, chin, or shoulders.

## 10. Add word-synchronized captions

Read `references/compositing-and-captions.md` before styling captions.

Use the final narration transcript to build phrase groups with word timing.
Display one short phrase at a time and emphasize the word currently being
spoken.

Default behavior:

- white text on a compact dark translucent strip
- active word uses a high-contrast accent pill
- previously spoken words settle to full white
- upcoming words remain slightly quieter
- active-word motion is a small scale hit, not a bounce
- maximum two lines
- caption band remains left/center and reserves a right-side keep-out zone for
  the presenter

On 1920 × 1080, a reliable starting point is:

```text
caption stage: left 70 px, right presenter width + 150–190 px
bottom band: 0–180 px high
font: Inter or brand sans, 34–46 px, 700–800 weight
background: near-black at 80–88% opacity
active accent: high-contrast lime or brand accent
```

Use seek-safe class changes (`gsap.set`) rather than callbacks that fail during
frame-by-frame render seeking.

## 11. Final review and render

Review the final Studio preview from beginning to end:

- hook lands within the first two narrated seconds
- every product claim has matching visual evidence
- concept demos are labeled
- product text is legible at normal playback
- presenter does not dominate or cover controls
- presenter crop avoids a visibly frozen torso
- mouth amplitude remains natural
- captions match the approved script and active word
- caption groups do not overlap the presenter
- logos use transparent official files and align by visible optical centers
- no black frame appears at transitions
- opening and closing contain no stray presenter or caption
- narration, avatar, captions, and scenes share the same timing origin

After approval, render at high quality:

```text
16:9
1920 × 1080 or higher
H.264 video + AAC audio
30 fps unless the source requires another rate
complete duration within 0.1 seconds of the declared composition
```

Run `scripts/validate_delivery.py` when possible. Inspect a final contact sheet
from the rendered MP4 and verify that the presenter crop contains changing
frames rather than a frozen segment.

Name the file:

```text
<brand>-<campaign>-<lang>-demo-led-vNN.mp4
```

Provide a clickable absolute path to the delivered video.

## Approval checkpoints

Stop for user review at:

1. production-language script
2. pronunciation master when performance is uncertain
3. personal-timbre narration
4. avatar preview
5. visual storyboard
6. final HyperFrames Studio preview
7. before publishing externally

The user may explicitly waive a checkpoint. Record the decision.

## Failure policy

- If only the voice timbre is wrong, return to voice replacement; do not rewrite
  the script or regenerate visuals.
- If pronunciation is non-native, return to the pronunciation master; a direct
  timbre clone will not reliably fix phrasing.
- If the mouth is distorted, return to avatar preview settings or portrait;
  subtitles and cropping cannot repair facial deformation.
- If arms are static but the face is good, use a tighter head-and-shoulders crop
  or regenerate with restrained body motion.
- If product screenshots feel repetitive, replace some with designed narrative
  cards; do not add arbitrary decoration.
- If the visual concept is speculative, label it; do not discard a useful
  concept demonstration merely because it is not documentary footage.
- If captions are wrong, correct transcript text while preserving word timing.
- If a render warns about sparse source-video keyframes, verify the rendered
  presenter region across multiple adjacent frames. Re-encode the source with
  regular keyframes before rerendering if any freeze appears.
- If CLI upgrade verification fails against an existing project, return to the
  last passing pinned version and record why.

## References

Read only what the active stage needs:

- `references/script-playbook.md` — four-language creator writing
- `references/audio-and-avatar.md` — pronunciation, personal timbre, portrait,
  HeyGen, and motion QA
- `references/hyperframes-story-playbook.md` — evidence gathering, storyboard,
  product/card rhythm, animation, and review gates
- `references/compositing-and-captions.md` — presenter placement, crop shapes,
  word-level captions, opening/closing, and final assembly
- `references/qa-and-lessons.md` — failure modes and final export checks

## Validated example

Read `examples/open-design-codex-team-16x9/README.md` when another agent needs a
concrete end-to-end example. It records the accepted script, scene structure,
caption treatment, presenter crop, exact durations, and lessons from the
validated 48.33-second Open Design × Codex Team production.

The example contains reusable, sanitized text records, not copies of the user's
portrait, voice, raw avatar, private media, or machine-local file paths.
