---
name: video-background-music-identification
description: Extract, separate, identify, and verify background music from videos or noisy mixed audio. Use when a user asks what song, score, symphony, or instrumental appears in an MP4/MOV/audio recording (e.g. "这个视频的BGM是什么", "识别视频里的背景音乐/配乐/歌曲", "what song is playing in this clip"); when ambience, speech, wind, crowd noise, or sound effects confuse audio fingerprinting; or when a Shazam-style result needs independent waveform verification and an evidence-backed report.
---

# Video Background Music Identification

Build an auditable evidence chain from the supplied media to a verified recording. Treat fingerprint matches as candidates until they explain the requested sound layer and survive local comparison.

## Workflow

0. Check metadata and visual clues first — the cheapest channel, often decisive. Sample video frames for player UI, control-center cards, watermarks, captions, and end credits that name the source video or the music directly. If the source video is identified, check its official description and pinned/top comments for music credits; many creators list licensed tracks there. Continue to audio analysis only when these channels yield nothing.
1. Inspect the source with `ffprobe`. Record duration, codecs, sample rate, channels, file size, and SHA-256.
2. For long media, select a representative 20–90 second interval with `--start` and `--duration`. Run `scripts/prepare_audio.py` to create a lossless source track, a listening MP3, and an HPSS harmonic layer.
3. Listen to the original and harmonic versions. Label audible layers such as music, wind, speech, machinery, and added effects.
4. Choose separation by interference: use HPSS for broad wind or transient noise; run `scripts/separate_vocals.py` when speech or singing dominates. Preserve every derivative label.
5. Inspect a spectrogram when the mixture is ambiguous. Look for sustained horizontal harmonic structures versus broad, irregular noise.
6. Obtain explicit user consent before sending any fingerprint to an external recognition service.
7. Run `scripts/recognize_shazam.py` first on the original track and then, when required, on the best target-music derivative. Never send the source video through this script. When the material allows, recognize several overlapping segments rather than a single window.
8. Treat every match as a hypothesis. Check whether it explains the requested layer, version, vocals, instrumentation, duration, and apparent timestamp.
9. Acquire an authorized or official candidate recording for local verification. Preserve its URL and metadata.
10. Run `scripts/compare_candidate.py` to search the full candidate for the query segment. Limit unusually long candidates with the clip options. Compare the best peak with the background distribution and at least one plausible wrong candidate when possible.
11. Report the conclusion, confidence, exact version, aligned time span, direct official playback link, evidence, rejected candidates, privacy boundary, and limitations.

## Decision Gates

- A raw fingerprint hit may identify ambience or a foreground sound effect. Continue when it fails to explain the requested music.
- Trust only candidates confirmed on two or more adjacent or overlapping segments. One-off hits on a single segment are weak and often noise; a hit repeated on neighboring segments is far more reliable.
- Apply plausibility filters before investing in verification: the candidate's release date must precede the video's publish date, and the artist or channel should match how the video's creator typically licenses music (e.g. Musicbed/Artlist/Epidemic for YouTubers). Reject candidates that fail these checks without running local verification.
- Conflicting hits across denoised variants indicate unstable fingerprints or processing artifacts. Do not vote by frequency.
- Strong HPSS separation can damage attacks, percussion, and reverb. Keep the untouched extraction as evidence.
- A local correlation peak gains meaning from contrast: report the peak, the 99th percentile of other positions, and a wrong-candidate control. Avoid universal pass/fail thresholds.
- Distinguish recording identification from composition identification. State whether the evidence supports the exact master, a cover, a remix, a movement, or only a similar composition.
- Label classical forms carefully. Cinematic strings can sound symphonic without belonging to a numbered symphony.

## Files and Labels

Use stable labels in every handoff:

- `full_audio.wav` / `full_audio.mp3`: untouched audio extraction containing every mixed layer.
- `music_harmonic_hpss.wav` / `.mp3`: harmonic-emphasized analysis derivative.
- `music_no_vocals.wav`: optional Demucs derivative for speech- or vocal-heavy mixtures.
- `manifest.json`: source metadata, hashes, parameters, and file-role annotations.
- Recognition JSON: external candidate evidence.
- Comparison JSON: local alignment evidence.

Never label an external ambience preview as an isolated stem from the source.

## Resources

- Read `references/methods.md` for the signal-processing and evidence model.
- Read `references/prompt-library.md` for reusable blind-test and report prompts.
- Read `references/case-study.md` when an end-to-end worked example is useful.
- Copy `assets/report-template.md` for the final deliverable.

## Dependencies

Require `ffmpeg` and `ffprobe`. Install `scripts/requirements-core.txt`; add `requirements-recognition.txt` for Shazam or `requirements-speech.txt` for Demucs only when needed. Treat `shazamio` as an unofficial client whose behavior can change; document failures and use another authorized recognition service when necessary.

## Privacy and Copyright

- Ask before any external fingerprint submission, even when the API sends a compact signature.
- Keep source media, extracted commercial audio, downloaded reference masters, cookies, and API credentials out of public repositories.
- Keep absolute paths out of shareable JSON. Use `--include-paths` only for private local audits that require them.
- Publish code, parameters, hashes, short factual metadata, and derived measurements. Link to authorized playback pages.
