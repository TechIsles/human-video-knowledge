---
name: universal-video-editing-skill
description: 面向多场景的自动视频剪辑执行型 Skill。用于口播或访谈去停顿与字幕、长视频转 Shorts/Reels/小红书短视频、多素材 Vlog 或蒙太奇、教程与产品演示剪辑，以及视频/音频/图片混合编排、横竖屏重构、BGM 混音和跨平台导出。支持“自动、审片、手动”三种模式；当用户说“自动剪辑、剪口播、长转短、做 Vlog、生成短视频、加字幕、竖屏重构、edit video、make shorts、create a montage”等请求时使用。
---

# 通用自动视频剪辑

把内容判断交给 Agent，把探测、计划校验、渲染和验收交给确定性脚本。默认本地处理，永不修改原始素材。

## 硬性规则

1. 先运行 `python scripts/uve.py doctor`，再处理素材。
2. 所有产物写入素材目录旁的独立 `edit-output/`；不得覆盖或改写源文件。
3. 默认使用 `review` 模式：先生成计划和本地审片页，得到明确确认后才能渲染。
4. 只有用户明确选择 `auto` 时才跳过审片；`manual` 只执行用户给定的时间码和参数。
5. 覆盖已有成片、调用云 API、下载外部素材或使用版权状态不明的资源前必须确认。
6. API Key 只能从环境变量读取；不得写入计划、日志、命令输出或仓库。
7. 每次渲染后用 `ffprobe` 检查时长、分辨率、音视频流，并抽取首帧、中帧、尾帧检查。

## 标准流程

从 Skill 根目录执行下列命令。若 Agent 的当前目录不是 Skill 根目录，先解析 `SKILL.md` 所在目录，再使用脚本的绝对路径。

### 1. 诊断与建项

```bash
python scripts/uve.py doctor
python scripts/uve.py init --source "/path/to/media" --preset talking-head --mode review
```

可用预设：`talking-head`、`long-to-short`、`vlog`、`tutorial`、`social`。选择规则见 [references/workflows.md](references/workflows.md)。

### 2. 素材分析

```bash
python scripts/uve.py inventory --project "/path/to/media/edit-output"
python scripts/uve.py analyze --project "/path/to/media/edit-output"
python scripts/uve.py transcribe --project "/path/to/media/edit-output" --provider local
```

`analyze` 生成缩略图和静音区间；`transcribe` 默认使用本地 faster-whisper，缺失时回退到 Whisper CLI。需要视觉语义时，优先由当前 Agent 查看缩略图；只有用户同意联网后才使用 `vision` 命令。云适配规则见 [references/providers.md](references/providers.md)。

### 3. 内容决策

读取：

- `analysis/inventory.json`
- `analysis/transcripts/index.json`（存在时）
- `analysis/silence.json`
- `analysis/thumbnails/`

依据场景规则挑选片段。不要仅凭音量删除内容；保留情绪停顿、笑声、反应镜头和叙事所需上下文。长转短时按“开场钩子、独立完整、情绪强度、信息密度、结尾回报”评分。

### 4. 建立并校验剪辑计划

```bash
python scripts/uve.py draft-plan --project "/path/to/media/edit-output"
python scripts/uve.py validate --project "/path/to/media/edit-output"
```

编辑 `plans/edit-plan.json`，不要绕过其 Schema。字段说明见 [references/plan-schema.md](references/plan-schema.md)。长转短需要为每个候选片段分别建立计划文件。

### 5. 审片

```bash
python scripts/uve.py preview --project "/path/to/media/edit-output"
```

在 `review` 模式下打开 `preview/index.html`，让用户确认或下载修改后的计划。用户未确认时不得渲染。`auto` 模式仍需先运行 `validate`。

### 6. 渲染与验收

```bash
python scripts/uve.py render --project "/path/to/media/edit-output"
python scripts/uve.py verify --project "/path/to/media/edit-output"
```

若目标文件已存在，脚本会拒绝覆盖；只有用户明确同意后才传 `--overwrite`。失败时保留工作目录和日志，修正后最多重试三次，不要反复进行同一无效渲染。

## 场景分流

- 口播/访谈：转录优先，去除假开场、重复口误和过长无意义停顿；保留自然呼吸。
- 长转短：先评分再选段；每条短视频必须脱离原视频也能理解，默认 9:16。
- Vlog/蒙太奇：画面优先，交替远中近景，避免连续相似镜头；音乐节拍只作为辅助。
- 教程/产品演示：语义顺序和可读性优先，不为追求快节奏切断操作步骤。
- 社媒：前三秒建立钩子，使用安全区、动态字幕和平台导出预设，但不得承诺“爆款”。

## 资源导航

- [references/workflows.md](references/workflows.md)：五类场景的选择、评分和验收规则。
- [references/plan-schema.md](references/plan-schema.md)：剪辑计划字段和示例。
- [references/providers.md](references/providers.md)：本地、视觉、转录和 TTS 适配器。
- [references/installation.md](references/installation.md)：Codex、Claude Code、Gemini CLI、OpenClaw 安装方式。
- [references/safety.md](references/safety.md)：源文件、密钥、版权和覆盖规则。
- [references/english-guide.md](references/english-guide.md)：English quick guide for non-Chinese users.
