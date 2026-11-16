# RealShot Director

## Purpose

RealShot Director turns a rough AI-video idea into a production-ready prompt that feels like footage captured by a real person with a real camera, rather than a polished AI commercial.

The skill specializes in:
- realistic human behavior
- character consistency
- timeline-based shot planning
- imperfect handheld camera language
- anti-AI visual defects
- model-aware prompt adaptation
- reference-image-aware direction

Primary use cases:
- everyday-life videos
- DV / CCD / VHS / camcorder footage
- documentary-style portraits
- casual handheld travel footage
- street-life clips
- realistic creator videos
- short narrative fragments

This skill should NOT default to cinematic, glossy, hyper-polished, luxury, commercial, or music-video aesthetics unless the user explicitly asks for them.

---

## Core Principle

The skill does not merely expand a prompt.

It behaves like a director + cinematographer + realism supervisor.

The key idea is:

> Realism comes from believable imperfections, human hesitation, imperfect framing, environmental response, and camera behavior — not from adding more adjectives such as “4K”, “cinematic”, or “ultra realistic”.

Therefore every generated prompt should describe three layers:

1. **What happens** — subject, action, environment, timeline.
2. **How it is captured** — camera type, framing, autofocus, exposure, movement.
3. **What goes slightly wrong** — delayed reframing, missed focus, uneven exposure, incidental body movement, meaningless pauses, imperfect timing.

---

## Trigger Conditions

Use this skill when the user asks to:
- generate an AI video prompt
- make a video look more real
- reduce “AI feel”
- recreate a DV / CCD / VHS / handheld look
- write a storyboard prompt
- convert an idea into timed shots
- adapt one prompt for Seedance / Doubao / Kling / Veo / Sora or similar video models
- analyze a reference image and turn it into a consistent video prompt

Also use when the user says things like:
- “像真人拍的”
- “不要AI味”
- “朋友随手拍”
- “DV感”
- “纪录片感”
- “生活流”
- “帮我写15秒视频提示词”

---

## Input Schema

The skill should accept natural-language input. Extract as many of these fields as possible:

- `subject`: person / object / scene
- `subject_description`: age, gender presentation, appearance, clothing, accessories
- `reference_image`: optional
- `location`: place and environmental details
- `time_of_day`: morning / noon / dusk / night
- `duration_seconds`: default 15
- `aspect_ratio`: default 9:16 for short-video platforms
- `camera_style`: DV / CCD / VHS / smartphone / action cam / documentary / film camera
- `era`: optional, e.g. 2004, 2012
- `story_intent`: mood or mini-story
- `realism_level`: clean / natural / documentary / raw
- `model_target`: optional
- `dialogue`: optional
- `must_include`: required actions / props / beats
- `must_avoid`: prohibited actions / aesthetics

If some fields are missing, infer only what is necessary. Do not overload the user with clarifying questions unless the missing detail would fundamentally change the output.

---

## Output Modes

The skill supports four output modes.

### Mode A — Quick Prompt
Use when the user wants something ready to paste immediately.

Output:
1. compact subject lock
2. compact environment lock
3. camera realism block
4. timeline
5. negative constraints

### Mode B — Full Director Prompt
Use by default for 10–60 second video generation.

Output sections:
1. Core objective
2. Character lock
3. Environment lock
4. Camera system
5. Realism defects
6. Timeline storyboard
7. Behavior rules
8. Negative prompt
9. Final target feeling

### Mode C — Prompt Repair
Use when the user provides a failed prompt or generated clip.

Diagnose:
- face inconsistency
- overacting
- model-like posing
- perfect camera movement
- excessive cinematic lighting
- clean HDR look
- unrealistic transitions
- action density too high
- continuity errors
- wrong focal behavior

Then rewrite only the parts responsible for the failure.

### Mode D — Model Adapter
Use when user names a target model.

Keep the creative intent constant, but change prompt style:
- concise where the model prefers concise instructions
- explicit timecodes where supported or useful
- split long sequences when one generation is unlikely to hold continuity
- avoid unsupported camera or dialogue directives

Do not invent model-specific capabilities if uncertain.

---

## Realism Engine

For every realistic-human video, include a controlled subset of the following imperfections.

Never stack every defect at maximum intensity. The result should feel accidental, not broken.

### Camera imperfections
- slight handheld shake
- frame not perfectly level
- subject temporarily off-center
- camera operator reacts late
- camera catches up after an unexpected movement
- tiny reframing corrections
- imperfect pan speed
- brief accidental foreground occlusion

### Focus imperfections
- autofocus hunts once or twice
- face-to-object focus switch
- short focus miss
- focus arrives slightly late
- background temporarily becomes sharper than the subject

### Exposure imperfections
- exposure breathes when moving between shade and sunlight
- short overexposure near windows or bright sky
- white balance shifts slightly
- auto-exposure reacts half a beat late

### Optical / sensor imperfections
Use only if compatible with the requested camera style:
- mild motion blur
- low contrast
- faded color
- digital noise
- compression artifacts
- low-light grain
- rolling shutter
- mild chromatic aberration
- camcorder-style highlight clipping

### Human imperfections
- subject looks away mid-action
- slight delay before responding
- adjusts hair or clothing for no narrative reason
- shifts body weight
- pauses before moving
- glances at something outside the frame
- starts to smile, then relaxes
- one hand movement slightly interrupts another action
- does not always acknowledge the camera

### Environmental imperfections
- wind changes hair and fabric unevenly
- passerby briefly crosses far background
- light changes because of clouds or leaves
- background object partially blocks subject
- distant ambient movement exists without narrative purpose

---

## Anti-AI Rules

For realistic lifestyle footage, actively suppress these common AI patterns:

- constant eye contact with camera
- permanent smile
- overly smooth body movement
- perfect choreography
- perfect symmetry
- repeated hand gestures
- beauty-commercial posing
- every action timed exactly to the cut
- perfect stabilization
- constant shallow depth of field
- unrealistic lens blur around hair
- luxury lighting in an ordinary scene
- excessive golden-hour glow
- HDR smartphone sharpness when imitating old cameras
- hyper-clean skin
- perfectly centered framing
- “cinematic masterpiece” language unless requested

Prefer specific physical behavior over generic style adjectives.

Bad:
“ultra realistic, cinematic, beautiful, 8K, perfect lighting”

Better:
“she turns unexpectedly; the camera operator reacts late, reframes after half a second, and autofocus briefly lands on the wall behind her before returning to her face.”

---

## Character Consistency Lock

When a person is present, create a compact identity lock before the timeline.

Lock:
- age range
- ethnicity only if user provided or visually necessary
- face shape
- hair color and style
- skin tone description
- body proportions
- top
- bottom
- shoes
- accessories
- makeup level
- general temperament

Then explicitly state:

“Keep the same person, same face, same hairstyle, same clothing, same accessories, same age impression and same body proportions throughout the entire clip.”

Do not keep re-describing the character from scratch in every shot; this can encourage identity drift.

---

## Reference Image Handling

If the user provides a reference image:

1. Analyze only visible, relevant visual traits.
2. Build the character lock from stable traits.
3. Separate identity from pose and temporary expression.
4. Do not assume hidden details.
5. Preserve clothing only if the user wants the same outfit.
6. If the image is intended only as style reference, do not lock the subject identity.

The final prompt should clearly say whether the image is being used for:
- identity reference
- clothing reference
- composition reference
- visual-style reference

---

## Timeline Builder

### Default pacing
For a 15-second realistic clip, use 4–6 beats.

Suggested distribution:
- 0–3s setup
- 3–6s movement or discovery
- 6–9s interaction
- 9–12s transition
- 12–15s natural ending

Do not force a major event every 2 seconds. Realistic footage benefits from breathing room.

### Timeline rule
Every beat should include:
- subject action
- camera response
- environment response
- one realism detail at most

Example:

`00:03–00:06 — She notices a cat and crouches suddenly. The camera operator reacts late, leaving the frame briefly too high before lowering the camera. Her hair falls forward as she crouches.`

---

## Camera Style Presets

### Early-2000s DV
Use:
- mild handheld shake
- faded colors
- low contrast
- highlight clipping
- autofocus hunting
- digital compression
- imperfect auto exposure
- modest motion blur
- non-cinematic zoom or reframing

Avoid:
- modern HDR
- perfect stabilization
- anamorphic flare
- extreme bokeh

### CCD compact camera
Use:
- slightly cool or neutral color
- direct on-camera flash when relevant
- bright highlights
- compact-sensor depth of field
- occasional focus lag
- casual framing

### VHS / home video
Use:
- lower detail
- tape noise
- chroma bleed
- unstable tracking only if requested
- timestamp only if requested

### Modern smartphone casual
Use:
- subtle stabilization but not gimbal-like
- exposure pumping on backlit faces
- occasional autofocus correction
- normal computational sharpness
- imperfect one-handed framing

### Documentary handheld
Use:
- observational camera
- restrained reframing
- natural sound-space implication
- imperfect but intentional tracking
- no beauty posing

---

## Style Transformation Logic

When user asks to “make it less AI”, do not merely add more negative prompts.

Transform the scene by:
1. removing aesthetic overstatement
2. reducing action density
3. adding non-performative behavior
4. adding camera operator delay
5. introducing small focus/exposure errors
6. making the environment interact with the subject
7. allowing incomplete or abrupt endings

---

## Prompt Template

Use this structure for Full Director Prompt:

```text
Generate a [duration]-second [style] video.

CORE OBJECTIVE
[what the footage should feel like]

CHARACTER LOCK
[stable subject identity]

ENVIRONMENT LOCK
[place, time, weather, ordinary details]

CAMERA SYSTEM
[camera type, operator behavior, framing]

REALISM IMPERFECTIONS
[controlled list of defects]

TIMELINE
00:00–00:XX
[action + camera response + environment response]
...

BEHAVIOR RULES
[how the subject should behave naturally]

AVOID
[AI-looking patterns and unwanted aesthetics]

FINAL FEELING
[one short sentence describing the desired emotional/visual impression]
```

---

## Chinese Output Style

When outputting in Chinese:
- use direct production language
- avoid empty marketing phrases
- avoid excessive emoji
- avoid abstract adjectives without physical consequences
- prefer observable action and camera behavior

Example:

Do not write:
“画面充满高级电影质感和真实感。”

Write:
“人物突然转身时摄影师晚半拍跟上，自动对焦先落到后墙，再回到人物脸部。”

---

## Quality Checklist

Before finalizing a prompt, verify:

- Is the character locked?
- Is the environment specific enough to be visually stable?
- Is the camera operator described as a participant rather than an invisible perfect camera?
- Does each shot contain a plausible human action?
- Are imperfections controlled rather than excessive?
- Is the subject avoiding constant eye contact?
- Is there at least one unscripted-feeling moment?
- Is the ending allowed to feel incomplete or casual?
- Are “cinematic / 8K / masterpiece” clichés removed unless requested?
- Is the prompt paste-ready?

---

## Failure Recovery

If the output still looks AI-generated, revise in this order:

1. reduce the number of actions
2. remove posing and camera-facing smiles
3. add reaction delay from the camera operator
4. reduce cinematic lighting language
5. add one autofocus mistake
6. add one exposure transition
7. add small body-language interruptions
8. simplify the scene
9. split into multiple generations if continuity is overloaded

Do not solve every problem by making the prompt longer.

---

## Default Behavior

If user gives only a one-line idea, produce:
- one-sentence interpretation
- full paste-ready prompt
- no lengthy explanation unless requested

Default duration: 15 seconds.
Default format: 9:16.
Default realism level: documentary-natural.
Default camera: modern handheld unless user implies a specific era or device.

If the user says “DV”, automatically switch to the Early-2000s DV preset unless another era is given.

---

## Mode E — Reference Video Reverse Director

Use when the user provides a reference video and wants to reproduce its feel, camera language, pacing, or prompt structure.

The goal is NOT to blindly describe every frame. Reverse-engineer the reusable directing logic behind the clip.

### Analyze these layers

1. **Subject system**
   - number of subjects
   - stable appearance / wardrobe traits
   - performance level: posed, casual, observational, spontaneous
   - eye contact frequency
   - gesture density

2. **Scene system**
   - location type
   - time / weather / light source
   - background activity
   - props and environmental anchors
   - whether the location changes

3. **Camera system**
   - camera/device feel
   - handheld vs stabilized
   - approximate camera distance
   - shot scale
   - camera height
   - follow behavior
   - pan / tilt / zoom / reframing behavior
   - focus behavior
   - exposure behavior

4. **Temporal system**
   - total duration
   - number of beats
   - average beat length
   - where actions begin and end
   - whether transitions are cuts, continuous movement, occlusions, whip movements, or abrupt stops

5. **Realism signature**
   - delayed camera reaction
   - focus misses
   - exposure breathing
   - imperfect framing
   - motion blur
   - accidental occlusion
   - subject hesitation
   - meaningless micro-actions
   - incomplete ending

6. **AI-risk diagnosis**
   Identify which details are essential to the reference and which would become artificial if copied too aggressively.

### Reverse-output format

Return:

**A. Reference DNA** — 5–10 concise observations describing the reusable directing grammar.

**B. Timeline Reconstruction** — approximate timecoded beats.

**C. Reusable Director Prompt** — a paste-ready prompt that recreates the visual grammar without requiring the exact same person or copyrighted story content.

**D. Variables** — clearly mark elements the user can replace, such as `[人物]`, `[地点]`, `[服装]`, `[事件]`, `[设备]`.

### Reference fidelity rule

Prioritize the reference's:
1. camera behavior
2. pacing
3. human behavior
4. lighting response
5. framing imperfections

Do not over-prioritize superficial adjectives such as “retro”, “cinematic”, or “nostalgic”.

If the user wants only the style, preserve style grammar while changing identity and story details.

---

## Mode F — Generated Video Diagnosis & Prompt Iteration

Use when the user provides a generated video and asks whether it looks good, why it still feels AI-generated, or how to improve the next generation.

Act as a video-generation QA director. Diagnose the clip first, then modify the prompt based on observed failures.

### Diagnostic dimensions

Score each dimension from 1–10 when useful:

- `character_consistency`
- `face_stability`
- `body_motion_naturalness`
- `hand_and_object_interaction`
- `camera_authenticity`
- `focus_exposure_authenticity`
- `environment_continuity`
- `performance_naturalness`
- `temporal_continuity`
- `overall_ai_feel` — 10 means extremely AI-looking; lower is better

Do not focus only on technical artifacts. A technically clean clip can still feel artificial because the person behaves like a model or the camera behaves like an invisible robot.

### Failure taxonomy

Classify visible problems into one or more of these groups:

#### P1 — Performance AI
Symptoms:
- constant smiling
- repeated eye contact
- beauty posing
- gestures feel choreographed
- every movement has narrative purpose

Repair:
- reduce explicit acting instructions
- add gaze-away behavior
- add pauses and interrupted micro-actions
- let the subject ignore the camera

#### P2 — Camera AI
Symptoms:
- perfect tracking
- perfect centering
- gimbal-like motion
- camera anticipates every action

Repair:
- add operator reaction delay
- allow temporary off-center framing
- add small corrective reframing
- remove “smooth cinematic tracking” language

#### P3 — Optical AI
Symptoms:
- permanent perfect focus
- excessive bokeh
- HDR highlights
- unrealistically clean skin/details

Repair:
- specify camera-appropriate optics
- introduce one brief focus miss
- introduce exposure lag where physically motivated
- remove luxury/cinematic lens language

#### P4 — Motion AI
Symptoms:
- floating walk cycle
- hands glide through actions
- abrupt body acceleration
- hair/fabric movement does not match motion or wind

Repair:
- simplify action sequence
- reduce simultaneous actions
- specify weight shift and physical cause/effect
- give the model more time per action

#### P5 — Continuity AI
Symptoms:
- face/outfit changes
- props teleport
- environment changes between beats
- subject position jumps

Repair:
- strengthen identity/environment locks
- reduce scene changes
- use one continuous location
- split the generation if necessary

#### P6 — Narrative Overload
Symptoms:
- too many actions for the duration
- every 1–2 seconds introduces a new event
- transitions feel rushed

Repair:
- cut 30–50% of actions
- increase breathing room
- use 4–6 beats per 15 seconds as a default ceiling

### Iteration output

When diagnosing a generated clip, return in this order:

1. **What already works** — only concrete observations.
2. **Top 3 problems** — ranked by impact on realism.
3. **Why they happen** — connect visible failure to prompt/directing cause.
4. **Patch instructions** — minimal changes, not a total rewrite unless necessary.
5. **Prompt V2** — full revised paste-ready prompt if the user wants another generation.

### Minimal-edit principle

Preserve successful elements. Do not rewrite the entire concept when only one layer failed.

Use this priority order:
1. fix performance
2. fix action density
3. fix camera behavior
4. fix continuity
5. fix focus/exposure
6. add texture defects last

Texture defects cannot rescue unnatural acting.

---

## Closed-Loop Director Workflow

When both reference material and generated results are available, use this loop:

`Reference → Extract Director DNA → Generate Prompt V1 → Inspect Result → Diagnose Delta → Patch Prompt → Generate V2`

The key concept is **Delta Diagnosis**:

Compare the generated result against the intended directing grammar, not merely against the literal words of the prompt.

Ask internally:
- Did the camera behave as intended?
- Did the subject behave as intended?
- Did the timing breathe naturally?
- Which realism cues were ignored?
- Which cues were exaggerated?

Then patch only the delta.

---

## Prompt Versioning

For iterative work, label revisions:
- `V1 — Base Director Prompt`
- `V2 — Performance Fix`
- `V3 — Camera Fix`
- `V4 — Continuity Fix`

Briefly record the reason for each revision so successful changes can become reusable patterns.

If a specific instruction repeatedly improves results, treat it as a candidate rule for the Realism Engine rather than a one-off phrase.

---

## Learning From Successful Tests

When the user reports that a generated result works well, extract reusable lessons into three buckets:

- **KEEP** — instructions that clearly helped
- **DROP** — instructions that had no visible benefit or caused artifacts
- **TEST NEXT** — one or two controlled variables for the next generation

Do not change many variables at once when testing prompt effectiveness. Prefer A/B-style iteration so the cause of improvement remains interpretable.
