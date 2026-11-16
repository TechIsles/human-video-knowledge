---
name: mp3-to-srt
description: Convert local MP3, MP4, WAV, M4A, FLAC, OGG, WEBM, and similar audio/video files into one or more SRT subtitle files using local faster-whisper plus optional local Argos Translate. Use when the user asks to transcribe local audio/video, detect the spoken language, extract speech to subtitles, generate .srt captions, or output subtitles in a requested language such as Japanese and Chinese without uploading the source file to a cloud API.
---

# MP3/MP4 to Multilingual SRT

Use the bundled script for local audio/video-to-SRT work. Do not send private audio/video to cloud APIs unless the user explicitly asks for cloud transcription.

## Workflow

1. Confirm the input is a local file path. Reject URLs and remote downloads.
2. Prefer the project venv at `skills/mp3-to-srt/.venv`.
3. Run one-language transcription:

```powershell
.\skills\mp3-to-srt\.venv\Scripts\python.exe .\skills\mp3-to-srt\scripts\transcribe_to_srt.py "C:\path\audio.mp3" --output-language zh
```

4. Run multiple subtitle languages from one MP4:

```powershell
.\skills\mp3-to-srt\.venv\Scripts\python.exe .\skills\mp3-to-srt\scripts\transcribe_to_srt.py "C:\path\video.mp4" --language ja --output-language ja --output-language zh-TW
```

5. Use `--language ja`, `--language zh`, `--language zh-TW`, or another source language when known. Omit it when source-language detection is preferred.
6. Use `--model small` for a balanced default. Use `medium` or `large-v3` only when the user accepts slower processing.
7. For translated outputs, make sure the Argos language packages are installed:

```powershell
.\skills\mp3-to-srt\.venv\Scripts\python.exe .\skills\mp3-to-srt\scripts\install_translation_packages.py --list
```

## Safety

- This skill only handles local files.
- Do not enable URL, YouTube, RSS, or batch downloader features.
- Keep dependencies scoped to this skill venv.
- Do not print full transcripts into chat unless the user asks; report the output file path.
- Translation is local through Argos Translate. If no direct package exists, Argos may use an installed intermediate route such as `ja->en->zh`.
- `zh` means Argos Chinese output. `zh-TW` applies OpenCC `s2tw` conversion after translation for Taiwan-style Traditional Chinese.
