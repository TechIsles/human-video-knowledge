---
name: interview
description: |
  Turn existing spoken-source transcripts into source-anchored video structures without rewriting quoted speech. Use for two post-recording workflows: (1) reconstructing long, fragmented interviews into人物片 or other long-form finished-video structures, and (2) mining lectures, livestreams, talks, Q&A, or relatively continuous speech for roughly 1-3 minute clips. Trigger for 采访稿整理, 人物片结构, 成片命题, 原文锚定结构稿, 长采访重构, 讲座切片, 直播切片, 找切片选题, or spoken-video transcript editing. Do not use for pre-interview question design or generic prose polishing.
---

# Interview

Turn existing spoken material into a structurally effective, publishable video plan while keeping every quoted source block exact and traceable.

This skill starts after spoken material exists. Use `interview-prep-question-design` before recording. Do not turn this skill into generic transcript polishing or article rewriting.

## Non-Negotiable Constraints

Treat source integrity and finished-video structure as co-equal hard constraints.

For every quoted source block:

- Preserve the original characters and word order exactly.
- Do not replace, add, delete, polish, paraphrase, or silently repair wording.
- Do not correct ASR errors inside the quote. Flag suspected errors separately.
- Only merge broken line breaks when every source character remains unchanged. If you do, state: `以下原文仅合并断行，不改字、不润色。`
- Keep a source anchor: original timecode, SRT index, paragraph number, heading, question label, or searchable quote keyword.

You may select, trim at block boundaries, group, and reorder source blocks. Represent an internal cut as separate anchored blocks; never hide it inside one apparently continuous quote.

When source order and finished-video order differ, show the finished-video order and retain the original anchor on every block. Do not create a causal relationship, chronology, attitude, or conclusion that the speaker did not express.

Keep all titles, structural notes, subtitles, narration, and editor instructions outside quoted source blocks.

## Route the Task

Follow the user's explicit request first. Otherwise infer one mode from the material and target output.

### Long-Form Reconstruction

Choose this mode for multi-hour interviews, fragmented answers,人物片, cross-scene assembly, or requests for a finished long-form structure. The material may require wide reordering across questions, scenes, and time periods.

Load:

- `references/editorial-method.md`
- `references/longform-reconstruction.md`

Default first delivery: 1-3 candidate directions with evidence, structural outline, risk, and approximate viable length. After the user selects a direction, produce the full source-anchored assembly draft. If the user asks for the draft immediately, skip the direction-selection ceremony.

### Clip Mining

Choose this mode for lectures, livestreams, talks, courses, Q&A, or requests for roughly 1-3 minute publishable clips. Treat a clip as a local reconstruction, not necessarily a continuous excerpt.

Load:

- `references/editorial-method.md`
- `references/clip-mining.md`

Default delivery: about 10 ranked clip directions, each with its thesis, why it works, source range, estimated duration, and representative exact source blocks. When requested, assemble any direction into a complete source-anchored clip draft.

Do not impose a fixed source-window length. A three-minute clip may draw from ten or more minutes of nearby material, and its blocks may be reordered when the finished-video logic improves.

### Ambiguous Requests

Ask only when the choice changes the deliverable and cannot be inferred. Use one short question: `你要重构一条完整长片，还是从素材里寻找短切片？`

Do not load both mode references unless the task genuinely contains both deliverables.

## Shared Editorial Workflow

1. Read the source, available anchors, target platform, target duration, and delivery request.
2. Route to long-form reconstruction or clip mining.
3. Map source blocks by theme, structural function, strength, and anchor without rewriting them.
4. Identify the question or thesis the finished video should answer.
5. Select only the source blocks that advance that thesis.
6. Reorder blocks for audience comprehension and structural progression rather than preserving source chronology by default.
7. Produce the mode-specific deliverable.
8. Run the quality gates before delivery.

If platform, duration, or output format is missing, infer a reasonable default and state it briefly. Do not delay useful work for low-risk missing details.

## Narration and Gap Policy

Keep narration off by default.

When the source cannot support a coherent transition:

1. Identify the exact material gap.
2. Prefer a different source block, a visible edit, a subtitle card, or a follow-up question.
3. Suggest narration only when the user explicitly permits or requests it.
4. If narration is requested, separate its location, function, and draft wording from all source quotes.

Never use narration to make the speaker appear to have reached a conclusion absent from the source.

## Source Anchors

Use the strongest anchor the material provides:

- SRT: original index and timecode.
- Time-coded transcript: original start/end time.
- Feishu or document transcript: source heading plus paragraph or searchable keyword.
- Plain text: stable temporary paragraph numbers plus searchable keyword.
- Segmented notes: question title or segment label plus keyword.

Do not invent timecodes. If temporary paragraph numbers are added, label them as temporary navigation aids.

## Quality Gates

Before delivery, verify:

- **Exactness:** every quoted block remains character-for-character source text apart from merged line breaks.
- **Traceability:** every block has a usable source anchor.
- **Structure:** every block has a reason to exist and advances the thesis.
- **Progression:** the assembled order creates understanding rather than information accumulation.
- **Context integrity:** trimming or reordering does not distort meaning or invent causality.
- **Mode fit:** long-form work gets directional structure; clip work gets concise ranked candidates and usable source text.
- **Separation:** source quotes, editorial notes, subtitles, and narration are visibly distinct.

If a gate fails, expose the gap instead of silently repairing the quote.
