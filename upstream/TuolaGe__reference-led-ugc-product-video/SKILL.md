---
name: reference-led-ugc-product-video
description: 制作、返工或创意验收以参考视频的注意力节奏、证据顺序和镜头功能为骨架的 9:16 UGC 商品视频或 Fashion Reels。适用于包含人物表现、穿搭/上脚、精确商品多角度或多类动态证据的复杂任务，也适用于诊断成片不像参考、商品生成变形、人物站桩、静态广告感或嘴型表情过大的问题。不用于单镜头 talking head、简单字幕/转码、普通拼接或无需参考节奏重建的简单单一 B-roll。
---

# Reference-led UGC Product Video

Create a finished vertical product-recommendation video by reproducing a reference video's **attention rhythm, evidence order, shot functions, and conversational or visual cadence**—not its identity, script, product claims, or protected footage.

This skill is for creator-led commerce videos with dynamic evidence. Use a plain talking-head skill when the requested output genuinely needs one continuous presenter shot and no styling, on-foot, product-detail, or reference-rhythm reconstruction.

## Production contract

Use this sequence:

```text
recover project state
→ decompose reference rhythm and motion inventory
→ verify product facts and usage rights
→ choose narrated or visual-first structure
→ write natural spoken script or a music-led edit map
→ approve clean identity master and realistic product scale
→ approve short voice sample or explicitly waive voice
→ approve or explicitly waive dynamic presenter preview
→ build dynamic shot library
→ assemble around a version-locked narration or beat/gesture timeline
→ technical QA + creative QA
→ generate a local review index with a playable deliverable
→ wait for creative approval before batch, localization, or publishing
```

The workflow separates creative layers so a good result in one layer cannot hide a failure in another.

## Local executable scaffold

Initialize a portable local run before paid generation:

```bash
node <skill-dir>/scripts/run-local-workflow.mjs prepare \
  --title "<project title>" \
  --market "<market>" \
  --platform "<platform>" \
  --reference /absolute/reference.mp4 \
  --product /absolute/product.png \
  --out /absolute/project/work/ugc-runs \
  --confirmed
```

The command creates a stable project directory, copies the two source assets, writes a resumable manifest, and generates `review-index.md`. It does not dispatch paid work.

Use the same script for deterministic state changes:

```bash
node <skill-dir>/scripts/run-local-workflow.mjs gate --manifest <manifest> --name voice --status approved --note "试听通过"
node <skill-dir>/scripts/run-local-workflow.mjs record-job --manifest <manifest> --slot arroll-a --provider <provider> --task <task-id> --status running
node <skill-dir>/scripts/run-local-workflow.mjs attach --manifest <manifest> --slot final --file /absolute/final.mp4
node <skill-dir>/scripts/run-local-workflow.mjs status --manifest <manifest>
node <skill-dir>/scripts/run-local-workflow.mjs open --manifest <manifest>
```

The local workflow never submits provider work. Use `record-job` immediately after an explicitly authorized external submission, and use `attach` to copy completed media into the run while preserving its hash. Read `references/local-workflow.md` before running these commands.

## Operating boundaries

- Use only portraits, voices, scripts, celebrity/editorial assets, and publishing purposes the user owns or is authorized to use.
- Do not imitate a living creator's distinctive voice or exact performance. Translate references into broad cadence and delivery attributes.
- Keep secrets in environment variables or credential stores. Never echo API keys, authorization headers, signed URLs, or credentials into chat, logs, manifests, prompts, handoff files, or the skill.
- Treat external generation as paid. Reuse asset and task IDs, poll an existing job before resubmitting, and distinguish timeouts from confirmed failures.
- A user may explicitly waive a preview checkpoint after approving its underlying identity, script, and voice. Record the waiver; it does not authorize publishing, batch expansion, or a different identity.
- Do not bypass provider privacy or safety checks. Route the shot to another authorized workflow or remove the sensitive identity from that shot.
- A visual-first, music-led Reel may waive narration and voice. Record the waiver explicitly; do not invent speech merely because a template has voice or A-roll slots.

## 1. Recover before generating

When continuing a long project, read its state, decisions, handoff, artifact index, worklog, and run protocol before acting. Then verify the filesystem, provider/job state, manifest, current deliverable, and review status.

Treat disk files and current APIs as authoritative. Chat memory, old heartbeats, signed URLs, balances, and job states may be stale.

If a run manifest exists, call `run-local-workflow.mjs status` before creating assets or submitting work. Reuse its slot, gate, asset, and task IDs.

Record:

- approved and rejected assets;
- immutable masters for identity, audio, and product appearance;
- provider asset/task IDs without secrets;
- current approval gate;
- exact next action and prohibited expansions.

## 2. Decompose the reference into reusable grammar

Inspect the reference frame-by-frame and listen to it. Produce four synchronized maps:

1. **Attention map** — when the viewer first sees the outcome, face, product, on-foot proof, styling result, and CTA.
2. **Speech or beat map** — actual speech start, breath groups, emphasis, and pauses; or, for no-voice work, music accents, gestures, reveal points, match cuts, and transformation beats.
3. **Motion inventory** — walk-in, waist-up styling, natural gestures, hold-and-approach, side/toe/heel views, on-foot weight shift, and return-to-person shots.
4. **Edit grammar** — average shot duration, semantic cut points, insert duration, crop changes, PIP behavior, caption rhythm, and sound-bed behavior.

Extract principles such as “show the result before explaining it” or “change visual evidence every 1.5–2.5 seconds.” Do not copy the original script, creator identity, footage, or unverified product claims.

Read `references/creative-qa.md` for the shot-function and comparison checklist.

For music-led Instagram or fashion content, also read `references/fashion-reels.md`. It defines creator-native sequencing, safe product reveals, motion-window trimming, and the wide/side/toe/heel/on-foot coverage pattern.

## 3. Separate facts from persuasion

Build a claim ledger before writing:

- verified product identity and specifications;
- claims supported by official or user-provided evidence;
- observable styling or usage evidence;
- prohibited, unverified, or competitor-only claims.

Product pages are research inputs by default, not B-roll. Avoid placing static ecommerce screenshots, specification collages, or long product pages in the final creator video unless the user explicitly asks for that visual language.

Never invent sales, scarcity, comfort guarantees, waterproofing, traction, height increase, durability, or exact fit claims.

## 4. Choose narration or visual-first storytelling

Do not translate a narrated regional pilot by default. A new platform or market may require a different content grammar.

Use narration when the reference's value depends on explanation, claims, or personality-led opinion. Use a visual-first structure when the reference communicates through opening, trying on, walking, pointing, changing outfits, product inserts, and music.

For a no-voice Reel, replace the spoken script with a timed action map. Each beat needs a visible verb and evidence purpose, such as open, reveal, point, transform, walk, pivot, show toe, show side, show heel, or return to person. Music alone does not excuse a static montage.

Write for breath and conversational intent, not advertising completeness.

- Start from an observation or experience, then introduce social proof, then explain what an ordinary viewer can borrow.
- Vary sentence length and information density.
- Use a small number of natural connective habits appropriate to the language: for Mandarin, words such as “其实、比如、就是、然后、而且、还有、我觉得” can carry the thought forward.
- Prefer “I tried this styling idea” over slogan-like conclusions.
- Delete compressed ad phrases that sound generated, such as “整套立刻就稳了,” unless the user naturally uses them.
- End with a low-pressure opinion or choice question rather than a forced sales CTA.

Time the approved text before generating visuals. The narration duration governs a narrated edit; the approved beat/gesture map governs a visual-first edit.

## 5. Lock identity and product scale

Generate or select one clean identity master. At the first new visual direction, produce only one candidate and pause for aesthetic approval unless the user explicitly waives it.

Approval locks a master within its current version, not forever. If facts, wording, pronunciation, market, platform, identity, or product requirements change, preserve the old file, create a new version with a new hash, mark the old version superseded, and invalidate dependent approvals.

After approval:

- preserve the clean original as version-locked;
- branch every edit from that original, never from the latest regenerated candidate;
- use a local mask or regional edit for hands, clothing, or product scale when available;
- reject any candidate with face-texture contamination, eye/nose/mouth drift, changed hairstyle, changed identity, or a new art direction.

Judge handheld product scale from physical dimensions and perspective. For shoes, use the declared size and plausible foot length; compare the shoe with head length, shoulder width, hand span, camera distance, and lens perspective. “Accurate model” does not make an oversized foreground shoe believable.

Default presenter relation:

- direct or near-direct eye contact;
- neutral chin and relaxed brows;
- waist/hip-up framing for speech when that matches the reference;
- product close to the torso during normal explanation;
- approach-to-camera only as a short deliberate action.

## 6. Approve voice separately

Use this priority:

1. authorized user-provided clone sample;
2. clean single-speaker speech extracted from the reference, when authorized;
3. a designed performance master plus authorized timbre transfer;
4. provider voice library as a last resort.

Generate a 10–15 second sample with the new script. Review cadence, diction, emphasis, pauses, conversational looseness, and audio integrity—not only timbre.

Verify the real codec and container with `ffprobe`; never rename encoded MP3 data to `.wav`. Place a playable sample in the local review index and stop until approved.

Once approved, lock that narration version. Do not change its speed, wording, or timing merely to rescue a weak visual edit. A legitimate change creates a new version and makes dependent A-roll, timeline sync, and final approval stale.

## 7. Route motion by shot risk

Create a shot manifest with: shot ID, semantic purpose, target duration, framing, identity visibility, product requirements, provider, source asset, task ID, status, and cost/retry note.

Use the most stable generator for each risk:

- **Face-visible A-roll:** approved avatar / presenter system with restrained motion and low expressiveness.
- **Product multi-angle:** video model from an accurate product image; request specific side → toe → heel or equivalent movement.
- **On-foot proof:** waist-down or face-excluded video model shot with declared size, realistic weight shift, and stable product geometry.
- **Full-person movement:** use only when the provider accepts the authorized identity and preserves it reliably; otherwise use an approved avatar supplement or crop the action below the face.
- **Celebrity/editorial proof:** short authorized PIP near the matching spoken clause; never let it become the presenter or dominate the edit.

Treat close hand–product interaction as a separate high-risk class. A model can preserve a shoe in a static product shot yet redraw it while fingers grip, rotate, occlude, or extract it from a box. For identity-critical products:

1. use real handling footage when available;
2. otherwise keep generated human action before the product becomes readable, then match-cut to verified product motion;
3. use deterministic verified product footage for readable logo, silhouette, toe, side, sole, heel, or packaging evidence;
4. reject the shot if the product changes design, even when the motion itself looks natural.

Do not ask a generator to solve novelty, hand physics, identity, and exact product geometry in one long shot.

If a long presenter clip produces exaggerated mouth and expression, split the approved audio at semantic pauses and generate two or three low-expression A-roll segments. Hide seams with a cutaway, product shot, or PIP. Do not keep regenerating one long high-expression clip.

Read `references/provider-routing.md` before a paid motion run.

## 8. Assemble around the governing timeline

Keep the creator as the narrative anchor, not necessarily every single frame. A successful edit may briefly cut full-frame to dynamic product or on-foot proof; it should not replace the creator with static ecommerce pages or unrelated celebrity footage.

Default narrated structure:

- 2–6 seconds of action-first, no-speech outcome footage when the reference supports it;
- narration begins on a clean semantic boundary;
- person A-roll returns regularly and carries the argument;
- dynamic evidence appears exactly when the corresponding claim is spoken;
- change visual information roughly every 1.5–2.5 seconds without making every cut frantic;
- product angles progress rather than loop one held pose;
- the ending returns to the person, complete styling result, or on-foot choice.

Default visual-first fashion structure:

- tactile human action or outcome in the first second;
- safe cut to exact product evidence before risky generated handling becomes readable;
- short inspiration PIP or social reference only when it motivates a gesture or transformation;
- full-body result and walking motion;
- alternating human and verified product evidence across wide, medium, close, side, toe, low/on-foot, heel, and rear-three-quarter functions;
- one or two genuinely different styling states, poses, or camera relations;
- return to the person or exact product, without a forced end card.

The sequence is a functional pattern, not a fixed shot list. Preserve creator spontaneity: phone-camera framing, loose posture, action-motivated cuts, minimal graphics, and no ad-template typography unless the reference actually uses it.

Do not add persistent corner labels, series tags, or decorative overlays unless requested. Keep captions readable, at most two lines, and clear of faces, products, and platform UI.

Use a seekable, reproducible timeline project when possible. Preserve source assets, timing data, and render settings.

## 9. Review creative truth before technical polish

Technical validity is necessary but not acceptance.

Run a creative comparison against the reference:

- Does the opening show an outcome before explanation?
- Does the presenter feel friendly and camera-facing?
- Do the body framing and product scale match the intended realism?
- Are there genuinely different shot functions: walk/styling, speech, product angles, on-foot proof, return?
- Is the mouth restrained and the expression stable?
- Are inserts brief, semantically aligned, and subordinate?
- Is any static research material mistakenly used as finished B-roll?
- Does the result feel conversational rather than like a generated ad?
- For visual-first work, does each human shot contain real body or camera motion from its first visible frames?
- Does the opening avoid showing a generated product during the moment its geometry is most likely to drift?
- Would a viewer describe the video as a creator showing a find, or as a brand layout pretending to be UGC?

Keep failed versions as named negative references. Never mark a technically valid file as creatively approved without the user's judgment.

## 10. Run delivery QA

At minimum:

- `ffprobe` duration, dimensions, frame rate, codecs, and audio stream;
- full decode check;
- black-frame and unexpected-silence detection;
- contact sheet spanning opening, A-roll, evidence shots, and ending;
- exact-frame checks at every major transition;
- source-window motion checks so each chosen clip starts inside an active interval rather than a frozen lead-in;
- final full-frame freeze detection for creator-native or video-first work;
- subtitle timing and safe zones;
- visible product geometry and identity consistency.

Use `scripts/validate_delivery.py --video <final.mp4>` for the deterministic container check. Technical success does not replace the creative checklist.

For final QA, prefer:

```bash
python3 <skill-dir>/scripts/validate_delivery.py \
  --video <final.mp4> --full-decode --detect-black --detect-silence --detect-freeze \
  --contact-sheet <contact.jpg>
```

Default delivery is 1080 × 1920, 30 fps, H.264 + AAC unless the platform or source requires otherwise.

## 11. Deliver through a portable review handoff

The run manifest is the machine-readable state record; `review-index.md` is its human-readable review surface.

For every reviewable image, audio, or video:

1. attach it to a stable slot with an absolute path and SHA-256;
2. record status, market, version, duration, and approval state;
3. preserve rejected and superseded versions instead of overwriting them;
4. regenerate `review-index.md` from the manifest;
5. open the current image, audio, or video locally and verify playback;
6. leave the manifest, review index, current Gate, next action, and prohibited expansions recoverable without chat history.

Read `references/local-workflow.md` for the directory contract, Gate states, master versioning, and review handoff.

## Approval gates

Default gates:

1. claim/script direction;
2. first clean identity master;
3. 10–15 second voice sample;
4. first dynamic presenter/composite preview;
5. final creative cut before publish, localization, or batch.

The user can explicitly waive gate 4 and authorize unattended final assembly. Preserve all other boundaries and perform full QA.

## Failure policy

- A timeout is not a failure; poll the original job first.
- A confirmed paid failure must be recorded before retrying.
- Do not regenerate the avatar to fix captions or editing.
- Do not rewrite approved audio to fix weak shot coverage.
- Do not use another whole-image generation to fix a localized shoe/hand problem.
- Do not keep a generated extraction, rotation, or try-on shot after the visible product becomes a different design. Cut before the drift and reveal the verified product through editing, or reject the shot.
- Do not judge motion continuity from a sparse contact sheet alone. Run freeze detection on source clips, trim to active intervals, and repeat the check on the final render.
- Do not switch providers because one parameter was wrong; first identify whether the failure belongs to identity, motion, audio, edit, or delivery.
- If the user says “it does not feel like the reference,” return to the attention map, motion inventory, and edit grammar—not merely color grading.
- Do not batch or localize until one pilot receives creative approval.

## References

Read only what the task needs:

- `references/creative-qa.md` — reference decomposition, shot-function comparison, and human acceptance rubric.
- `references/provider-routing.md` — face/product/on-foot routing, privacy failures, paid-job discipline, and secret handling.
- `references/fashion-reels.md` — visual-first Fashion Reels, safe unboxing/reveal bridges, product-pixel authority, motion-window audits, and multi-distance coverage.
- `references/local-workflow.md` — portable directory, manifest, Gate, task recovery, and review-index handoff.
- `references/validated-case.md` — anonymized validated case and the detours that produced these rules.
- `assets/run-manifest.schema.json` — stable machine-readable run contract.
- `scripts/run-local-workflow.mjs` — resumable local manifest, Gate, provider-task, media, and review-index commands.
- `scripts/validate_delivery.py` — deterministic media, decode, black/silence, and contact-sheet checks.
