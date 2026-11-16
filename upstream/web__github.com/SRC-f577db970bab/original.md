---
name: content-digestion
version: 1.4.0
author: Vinson Liang
license: MIT
description: A Claude Skill that digests long-form content (podcasts, videos, articles, transcripts) into structured notes. Two modes — Knowledge Mode (archive into your knowledge base) and Publishing Mode (turn into a long-form read or shareable post). Output language follows the user's conversation language. 把任何长内容（文章、播客、视频文字稿、网页）消化成结构化笔记。两种模式：沉淀模式（存进知识库以后用）/ 发布模式（现在快速读完，或整理成可发布的长文）。输出语言根据用户对话语言自动判断（中文 / 英文 / 任何语言均可）。Triggers when the user provides a URL, pastes a transcript, uploads a file, or asks to digest, summarize, take notes on, or process content. 也适用于「帮我消化这个内容」「整理成笔记」「把这篇文章变成笔记」等场景。
---

# Content Digestion Skill

Digest any long content (article, podcast, video transcript, webpage) into structured notes.

This Skill serves the two most common needs of content creators and knowledge workers: **archiving** (saving content into a knowledge base for later use) and **consuming/publishing** (reading a long piece quickly, or producing a publishable long-form note).

---

## Language Handling (CRITICAL — read this first)

**This Skill produces output in the language the user is conversing in, not the language of the source content.**

Determine the output language using this priority order:

1. **Explicit user instruction** (highest priority): if the user says "output in English" / "用英文输出" / "give me a Chinese summary" / "用中文消化", follow that instruction exactly, regardless of conversation language.

2. **Conversation language**: if no explicit instruction, output in whatever language the user is using to talk to you. If the user writes the request in Chinese, output Chinese notes. If the user writes in English, output English notes.

3. **Source content language** (fallback only): if the conversation has been mixed/ambiguous and you genuinely cannot tell, default to the source content's language.

**Common scenarios:**

| User chats in | Source content is | Output |
|---|---|---|
| English | English | **English** |
| English | Chinese | **English** (translate while preserving meaning) |
| Chinese | English | **Chinese** (translate while preserving meaning) |
| Chinese | Chinese | **Chinese** |

**When translating between languages, preserve:**
- The original speaker's tone and rhythm
- Specific data, names, technical terms (keep names in original script for English/Chinese; provide translation parenthetically on first mention if needed)
- The emotional weight of high-impact quotes — translation should not flatten them

**The methodology of this Skill (frontmatter, wikilinks, quote sandwiches, "preserve by default" principle, integration suggestions) is language-agnostic.** All rules below apply equally to English and Chinese output. Where this document gives examples in one language, the same pattern applies to the other language.

---

## Before digesting: confirm intent

**If the user has not specified a mode**, ask once briefly. Use the language matching the user's conversation language:

**English version of the question:**

> I can help you digest this content. Quick question first:
>
> 1. **Archive it for later** — turn it into structured notes that drop into your knowledge base / Notion / Obsidian / a folder, so you can pull them up anytime
> 2. **Read it now / publish it** — turn it into a long-form note with structure and key points, so you can read this episode in 10 minutes, or share it / post it on your social platforms
>
> If unsure, choose 1.

**Chinese version of the question:**

> 我可以帮你消化这份内容。开始之前问一下：
>
> 1. **沉淀下来，以后再用** —— 整理成结构化笔记存进你的知识库 / Notion / Obsidian / 文件夹，以后随时调用
> 2. **现在就读完 / 发出去** —— 整理成有框架、有重点的长文笔记，你 10 分钟就能读完这期内容，也可以直接分享出去，或者发布到你自己的社交平台
>
> 如果不确定，选 1。

**Skip the question when the user has clearly specified intent.** Trigger keywords (in any language):

**Knowledge Mode triggers** (English): "digest", "archive", "file this", "save for later", "into my knowledge base", "for my notes", "structured notes", "Obsidian-ready", "Notion-ready"
**Knowledge Mode triggers** (Chinese): 「消化」「整理」「入库」「沉淀」「存起来」「做成笔记」「结构化笔记」

**Publishing Mode triggers** (English): "long-form", "publishable", "magazine read", "blog post", "newsletter", "article", "post on twitter / LinkedIn / social", "10 minute read", "summarize as a long-form", "essay-style"
**Publishing Mode triggers** (Chinese): 「写成笔记」「写成长文」「发公众号」「发小红书」「发推特」「发到社交平台」「读完」「总结成可读的文章」

**Both modes**: "do both" / "two versions" / 「两个都要」 / 「两种模式都做一份」

---

## Supported input methods

Process inputs in this priority order:

### 1. Direct URL (article / webpage / blog)

Use the `web_fetch` tool to retrieve the page content, then digest per the rules below.

### 2. Pasted transcript or full article text

The user pastes content directly (podcast transcript, video subtitles, article body) — digest per the rules.

### 3. Uploaded file

The user uploads PDF, TXT, DOCX, SRT, or similar files. Read the content, then digest per the rules.

#### SRT subtitle file parsing

- Use `re.split(r'\n\n+', content)` to split subtitle blocks
- Bilingual SRT (English/Chinese alternating): odd-indexed blocks = English, even-indexed = Chinese
- Strip timestamp lines and sequence numbers when extracting plain text

### 4. YouTube / podcast links

If the environment allows direct YouTube access, use `web_fetch` or similar tools. If direct access fails (many sandboxed environments block YouTube domains), try the following in order:

#### Strategy A: search for third-party transcript hosting pages (preferred)

1. Extract the video ID from the URL
2. Use `web_search` with the video title or ID + "transcript"
3. Look in the results for pages hosting full transcripts. Common sources:
   - **podscripts.co** / **podcasttranscript.ai** — podcast transcript aggregators, generally high quality
   - **youtubetotranscript.com** / **notegpt.io** — YouTube subtitle extractors
   - Blogs or news sites that quoted the video
4. Use `web_fetch` on the transcript URL to retrieve the full text

**Tip**: well-known podcasts (No Priors, Lex Fridman, All-In, Diary of a CEO, etc.) almost always have third-party transcripts. Search with "podcast name + guest name + transcript" for best results.

#### Strategy B: ask the user for help

If Strategy A fails, tell the user (in the user's conversation language):

> I couldn't fetch the transcript automatically. You can provide it this way:
>
> - On the YouTube video page, click "Show transcript" and paste the text here
> - Or get it from youtubetotranscript.com and paste it here
> - Or upload an SRT subtitle file
>
> Once I have the transcript, I'll digest it immediately.

**Core principle**: don't just say "I can't do it" and give up. Try Strategy A first; only ask the user after it fails.

---

## Mode 1: Knowledge Mode 沉淀模式

**Goal**: digest the content into structured, retrievable notes useful to future-you, your AI assistant, or your knowledge base system.

**Key features**:
- YAML frontmatter carrying metadata
- Body uses `[[wikilinks]]` to mark key entities and concepts
- Ends with a "Suggested Wiki Updates / 知识库整合建议" section
- Ends with a "Where to put this note next" guidance section

### Output template

**Section labels (TL;DR, Core arguments, etc.) use the output language naturally.** When outputting in Chinese, use 「TL;DR / 核心论点 / 颠覆认知的点 / 实操要点 / 知识库整合建议」. When outputting in English, use "TL;DR / Core arguments / Counterintuitive points / Action items / Suggested Wiki Updates".

**Example template (English output)**:

```markdown
---
source: "[Original URL or citation]"
type: "podcast | video | article | paper | book"
authors: ["Name 1", "Name 2"]
date_published: "YYYY-MM-DD"
date_ingested: "YYYY-MM-DD"
language: "en | zh | mixed"
summary_for_index: "One-sentence summary for the wiki index file"
entities: ["[[Entity 1]]", "[[Entity 2]]"]
concepts: ["[[Concept 1]]", "[[Concept 2]]"]
concepts_to_create: ["[[Concept 1]]", "[[Concept 2]]"]
tags: [tag1, tag2, tag3]
---

# [Content Title]

## TL;DR

3-5 sentences summarizing what this content is about and why it matters.
Anyone (including future-you) should be able to skim this and decide
"is this worth opening?"

## Core arguments

Organized by theme. Each section uses the three-part pattern:
- Lead-in → quote → interpretation
- Mark key entities and concepts with [[wikilinks]] on first mention
- Bold critical data and conclusions

### I. [Theme 1]

Body text... with [[wikilinks]] on entities and concepts...

> Quote, preserving the speaker's tone
> —— [[Speaker]]

Editor's interpretation...

### II. [Theme 2]

...

## Counterintuitive points

- Unexpected conclusions
- Specific data that supports them

## Action items

Actionable advice prioritized.

---

## 🔗 Suggested Wiki Updates / 知识库整合建议

> This section tells the user how to integrate this digest into their knowledge base.
> Default to **inference** (user usually hasn't provided wiki context).

### Suggested concept pages to create

- `[[Concept 1]]` — brief reason
- `[[Concept 2]]` — brief reason

### Potential cross-references

This content connects strongly with these topics — consider adding backlinks:
- `[[Topic A]]`
- `[[Topic B]]`

### Open questions worth exploring

This content surfaces some open questions worth further research:
1. Question 1
2. Question 2

---

## 📂 Next: where should this note go?

[This section uses a fixed structure — see "Knowledge Mode internal rules → 'Next steps' section structure" below]
```

**Example template (Chinese output)**:

```markdown
---
source: "[原始 URL 或出处]"
type: "podcast | video | article | paper | book"
authors: ["姓名 1", "姓名 2"]
date_published: "YYYY-MM-DD"
date_ingested: "YYYY-MM-DD"
language: "zh | en | mixed"
summary_for_index: "一句话摘要，给 wiki 索引文件用"
entities: ["[[实体 1]]", "[[实体 2]]"]
concepts: ["[[概念 1]]", "[[概念 2]]"]
concepts_to_create: ["[[概念 1]]", "[[概念 2]]"]
tags: [tag1, tag2, tag3]
---

# [内容标题]

## TL;DR

3-5 句话概括这份内容讲了什么、为什么重要。让任何人（包括未来的你）
扫一眼就能判断「要不要点进去看」。

## 核心论点

按主题板块组织。每个板块内：
- 概括 → 引语 → 解读 的三段式
- 关键实体首次出现用 [[双链语法]]
- 关键概念也用双链
- 重要数据和结论用 **加粗** 标记

### 一、[主题板块 1]

正文……用 [[双链]] 标注实体和概念……

> 引语，保留原话精度
> —— [[说话人]]

编者解读……

### 二、[主题板块 2]

…

## 颠覆认知的点

- 反常识结论
- 数据支持（具体数字一定保留）

## 实操要点

按优先级列出可行动的建议。

---

## 🔗 知识库整合建议 / Suggested Wiki Updates

> 这一段告诉用户：这份消化产物如何融入他的知识库。
> 默认基于内容**推测**（用户不一定提供 wiki 上下文）。

### 建议建立的概念页

- `[[概念 1]]` —— 简短理由
- `[[概念 2]]` —— 简短理由

### 可能的交叉引用

这份内容与以下主题强相关，建议在那些页面建立 backlink：
- `[[主题 A]]`
- `[[主题 B]]`

### 延伸阅读问题

这份内容引出了一些值得进一步研究的开放问题：
1. 问题 1
2. 问题 2

---

## 📂 下一步：把这份笔记放到哪里？

[此段落使用固定结构，详见下方「沉淀模式的内部规则 → 「下一步」段落的固定结构」]
```

**Section heading naming convention**:

| English | Chinese |
|---|---|
| TL;DR | TL;DR |
| Core arguments | 核心论点 |
| Counterintuitive points | 颠覆认知的点 |
| Action items | 实操要点 |
| 🔗 Suggested Wiki Updates | 🔗 知识库整合建议 / Suggested Wiki Updates |
| ↳ Suggested concept pages to create | ↳ 建议建立的概念页 |
| ↳ Potential cross-references | ↳ 可能的交叉引用 |
| ↳ Open questions worth exploring | ↳ 延伸阅读问题 |
| 📂 Next: where should this note go? | 📂 下一步：把这份笔记放到哪里？ |

### Knowledge Mode internal rules

**About frontmatter**:
- Fill all fields when possible. If a field can't be determined from the content (e.g., publication date), use `null` or omit it — don't fabricate
- `summary_for_index` is a **one-sentence** summary (under 30 words), specifically for the wiki index file / retrieval system. It's NOT the TL;DR — TL;DR is 3-5 sentences for human reading, while this one-line is for an LLM agent to quickly identify the source in an index
- `entities` are the specific named things in the content: people, organizations, products
- `concepts` are the key concepts, theories, terms appearing in the content
- `concepts_to_create` is a subset of `concepts` — only the ones **worth creating an independent concept page for** (not every concept needs a dedicated page). This field maps to the "writes articles for them" step in Karpathy's LLM Wiki pattern
- `tags` are classification labels, lowercase, snake_case

**About wikilinks**:
- Don't wrap every noun in wikilinks. Only mark entities and concepts **worth being their own page**
- The same entity gets a wikilink **on first mention** in the body; subsequent references can skip
- Wikilink syntax is Obsidian standard, but it also reads as clear visual emphasis in other systems

**About the "Suggested Wiki Updates" section**:
- **If the user provides an existing wiki directory or index**, give specific suggestions for updating existing pages and flagging contradictions based on the real structure
- **If the user has not provided wiki context** (the default case), only generate three subsections: "Suggested concept pages to create", "Potential cross-references", "Open questions worth exploring", and explicitly mark at the top: "This section is based on **inference** from the content"
- Don't pad suggestions for the sake of it. Quality over quantity.

**About the "Next steps" section**:
- This section matters equally for all users. **Do not omit it. Do not abbreviate it.**
- The content is relatively fixed (4 standard destinations + 1 Karpathy advanced section). Don't reinvent it each time
- If the user has explicitly stated which tool they use, you can keep only the matching destination, but **always keep the Karpathy advanced section**

**About suggested filename**:
- At the start of the "Next steps" section, proactively suggest a filename
- Naming convention: `author-topic-source-year.md`, all lowercase, hyphen-separated
- Examples: `sinclair-aging-doac-2024.md`, `karpathy-llm-wiki-gist-2025.md`
- This is the de facto standard in Obsidian / Karpathy-style workflows; LLM agents rely on stable filenames for backlink maintenance
- 例如:`sinclair-aging-doac-2024.md`,`karpathy-llm-wiki-gist-2025.md`
- 这是 Obsidian / Karpathy 风格的事实标准,LLM agent 在维护反向链接时依赖稳定文件名

**"Next steps" section structure** (Knowledge Mode output must include this at the end):

The "Next steps" section uses a fixed structure with two language versions. Use the version matching the output language.

**English version**:

```markdown
---

## 📂 Next: where should this note go?

**Suggested filename**: `[author-topic-source-year].md`

- **If you use Notion**: copy the entire content into a new page. The metadata
  between `---` can be converted to Notion page properties. `[[wikilinks]]`
  need to be manually converted to internal links in Notion.

- **If you use Obsidian**: save this as a `.md` file and drop it into your vault.
  The frontmatter and `[[wikilinks]]` work natively. The Dataview plugin can
  generate indexes from the frontmatter fields.

- **If you just want to save it in Apple Notes / Pocket / a notes app**: copy
  the body content starting from `# Title`, skipping the frontmatter at the top.

- **If you don't have a knowledge base system yet**: save as `.md` or `.txt`
  in a fixed folder (e.g., `~/notes/`). Later, search by filename or feed the
  whole folder to an AI tool to ask questions.

### 🧠 If you're using the Karpathy LLM Wiki workflow

Save this file to `raw/[suggested-filename].md`, then tell your LLM agent:

> Please compile `raw/[filename].md` into `wiki/`:
>
> 1. Add an index entry to `wiki/index.md` using the `summary_for_index` field
>    from the frontmatter
> 2. For each concept in the `concepts_to_create` list, create or update the
>    corresponding concept page under `wiki/concepts/`, integrating relevant
>    passages from this source
> 3. Update backlinks under `wiki/sources/` for this source
> 4. Check for contradictions with existing wiki content; if any, log them in
>    `wiki/log.md`

This prompt works directly with the workflow described in Karpathy's gist —
no additional configuration needed. Your LLM agent (Claude Code / Cursor /
others) will understand it.

> Karpathy's original: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
```

**Chinese version**:

```markdown
---

## 📂 下一步：把这份笔记放到哪里？

**建议文件名**：`[作者-主题-来源-年份].md`

- **如果你用 Notion**：复制全文到一个新页面。`---` 之间的元数据可以转成
  Notion 页面属性。`[[双链]]` 在 Notion 里需要手动转成内部链接。

- **如果你用 Obsidian**：保存为 `.md` 文件丢进 vault。frontmatter 和
  `[[双链]]` 都会原生工作，Dataview 插件可以根据元数据生成索引。

- **如果你只想存在 Apple Notes / 微信收藏 / 备忘录**：复制 `# 标题`
  以下的主体内容，跳过最上面的 frontmatter。

- **如果你还没有知识库系统**：保存为 `.md` 或 `.txt` 文件放到一个固定文件夹
  （比如 `~/notes/`）。以后想回忆任何内容，直接搜文件名，或者把整个文件夹
  喂给 AI 工具问问题。

### 🧠 如果你在用 Karpathy LLM Wiki 工作流

把这份文件保存到 `raw/[建议文件名].md`，然后告诉你的 LLM agent：

> 请把 `raw/[文件名].md` 编译进 `wiki/`：
>
> 1. 在 `wiki/index.md` 添加这份内容的索引条目，使用 frontmatter 里的
>    `summary_for_index` 字段
> 2. 为 `concepts_to_create` 列表里的每个概念，在 `wiki/concepts/` 下创建
>    或更新对应的概念页，把这份内容里的相关段落整合进去
> 3. 更新 `wiki/sources/` 下这份内容的反向链接
> 4. 检查是否和 wiki 中已有内容存在矛盾，如有，在 `wiki/log.md` 标记

这段 prompt 是基于 Karpathy gist 的描述直接可用的——不需要任何额外配置。
你的 LLM agent（Claude Code / Cursor / 其他）会读懂它。

> Karpathy 原文：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
```

---

## Mode 2: Publishing Mode 发布模式

**Self-check before delivering (Chinese output).** Run these in order; each one has cost a real round of revision:

1. Name the through-line in one sentence. Do all headings serve it, and do they repeat its concrete anchor?
2. Is every kept topic explained in full — background, causal chain, who each person is, what each term means?
3. Does any section define its central concept later than its second paragraph?
4. Did any English metaphor or idiom get translated literally?
5. Is there a period sitting where a colon, 「比如」, or 「因为」 belongs?
6. Are there 2–4 blockquotes on the sharpest lines, and no attribution lines under them?
7. Is any paragraph longer than three sentences?
8. Are there three or more parallel items running as prose that should be a bullet list?
9. Count the tidy aphoristic closers and the 「不是 X，是 Y」 constructions. Are they mostly the speaker's, or mostly yours?
10. Any 拆／拆解, any 「国内对照」 sidebar, any editor's voice?

**Goal**: turn long content (a 2-3 hour podcast/video, or a long article) into a long-form note with structure, logic, and human warmth.

**Use cases**:
- You want to read a 2-3 hour piece in 10 minutes (for yourself)
- You want to share it directly on social platforms or with friends (for an audience)

Both use cases share the same output format.

### Article structure

**Title (取标题)**:
- The title is not a neutral description of the topic — it should be a **punchline-style, one-line hook** that lands instantly. Mine it from the sharpest quote or the most counterintuitive idea in the content, not from a generic "X talks about Y" summary.
- Good: 「你没有毛病：对一个不正常的世界，焦虑是正常反应」 / 「扳机很小，子弹是你自己装的」. Weak: 「Gabor Maté 谈创伤与压力」.
- **Offer the user several title directions to choose from, don't just pick one.** Give 3–6 candidates spanning different angles (the core thesis, the most vivid metaphor, the most quotable line), each able to stand alone as a hook. Let the user pick or ask for more variants before finalizing. If the user asks for more, give a longer list grouped by angle rather than a flat list, and say which two or three you'd pick and why.
- **The title should name the piece's subject area, not only its punchline.** A title mined purely from the sharpest abstract line can leave a reader unable to tell what the piece is about. Where the piece sits in a recognizable vertical (marketing, health, investing, engineering), the vertical's keyword should normally appear in the title. Check candidates against this before delivering.
- **Check the title against the body.** If the chosen line comes from a minor thread while 80% of the piece is a walkthrough, either flag the mismatch to the user or add a sentence in the body that earns the title.
- When the title changes, rename the output file to match.

**Opening intro section**:
- **Open with a bold one-line 「本期简介：」 / "In this episode:" lead**, placed directly under the title — one sentence stating who the guest is, where it's from, and the core topic. Format it bold, e.g.:

  > **本期简介：** 创伤治疗专家 Gabor Maté 在播客《The Adversity Advantage》中，谈了为什么这个时代有这么多人焦虑、成瘾、迷失，以及创伤、压力与疗愈该怎么理解。

- After that line, a fuller intro: source citation + speaker/author intro (full name + short reference name to use throughout the piece)
- If the content is part of a series, note which part this covers
- **Do NOT include a "one-sentence recommendation" section** (it feels stiff and breaks reading flow)

**Body**:
- Organized by **theme**, not by timeline (no chronological play-by-play)
- Each theme section has clear sub-points
- Quotes dissolved into the narration (see "Quote handling" below)
- Sections connect through the argument itself, not through announced transitions

**Ending**:
- End on the strongest substantive point of the content itself, then close with a plain `The end.` line
- **Do NOT write a closing 「编者手记」/ editor's reflection.** See "No editorial voice in the article" below.

### Heading hierarchy

**For Chinese output**:
- **Level 1**: Chinese numerals 「一、二、三……」, bold large text, theme-level sections
- **Level 2**: Arabic numerals 「1. 2. 3.」, bold, sub-points within a section
- **Level 3** (occasionally): 「1）2）3）」 for finer breakdowns

**For English output**:
- **Level 1**: Roman numerals "I. II. III." OR descriptive headings ("Part One: ..." / "The Information Theory of Aging" etc.), bold, theme-level sections
- **Level 2**: Arabic numerals "1. 2. 3." or descriptive sub-headings, bold
- **Level 3** (occasionally): "a) b) c)" for finer breakdowns

**Principle**: use heading conventions natural to the output language. Don't force Chinese numerals into English notes or vice versa.

**Every heading must serve the article's through-line.** Before writing headings, name the through-line in one sentence ("how she runs her business with 34 agents"). Then check: does the heading set, read top to bottom, read as seven angles on that one thing — or as seven unrelated topics? If the latter, rewrite.

- **Repeat the through-line's concrete anchor across headings.** 「管着 34 个 agent，为什么她不用「管理」这个词」／「她每天给这 34 个 agent 派活，指令只有三个词」／「34 个 agent 的编制表，和两个人类公司不会设的岗位」. The repeated 「34 个 agent」 is what makes a table of contents legible at a glance.
- **A heading summarizes what the section covers**, concretely enough that a reader scanning only headings knows what they'd get. 「三个词的 prompt」 is too thin; 「她每天给这 34 个 agent 派活，指令只有三个词」 is right.
- A question mark is welcome: 「一个人带 34 个 agent，软件公司还有活路吗」「为什么越来越多年轻人得结直肠癌？」
- Sub-headings follow the same rule one level down, and may state the section's finding when that finding *is* the topic: 「光是把 AI 团队搭起来，你就已经在前 1%」.
- Avoid pure metaphor or suspense that leaves the reader unable to locate a section later ("颜色是一张成绩单", "她自己停掉的那件事")
- Where a top-level heading contradicts the article title (title says 「她管着 34 个 Agent」, section one says she rejects the word 管理), that tension is usable — but flag it in the delivery message so the user chooses deliberately.

### Quote handling (the core technique)

Quotes are what distinguish this kind of note from a generic summary — but in Publishing Mode they are **dissolved into the prose, not displayed in quote blocks**.

- **Default: carry the sharp line inline**, inside the sentence that needs it. Keep the speaker's rhythm and word choice; drop the packaging.
- **Exception: 2–4 blockquotes (`>`) per piece, for the sharpest lines only.** On WeChat a quote block is a visual rest stop and a scannable anchor — a reader skimming headings and quote blocks should still get the spine of the piece. Reserve them for the line that stings, the line that reverses expectation, or a short verbatim burst worth reading slowly.
- **Never add a `—— [[Speaker]]` attribution line** under a quote block. That turns the piece into an interview transcript.
- Never let quote blocks carry ordinary exposition. If the line reads fine as a normal paragraph, it is a normal paragraph.
- Only the **high-impact, emotionally-charged** lines are worth either treatment — not every claim.
- Translation preserves the speaker's tone and rhythm.

**Do this**:

```
直觉既不好也不坏，它是一个生理信号，只传递一条信息：这件事的赌注比你以为的高。
```

**Not this**:

```
Pasricha 收尾的那句话是：

> 你的肠道不是在替你写这个故事。它只是在请你把自己的故事读得更仔细一点。
> —— Trisha Pasricha
```

#### No reporting scaffolding (CRITICAL)

The single most common defect in this kind of draft is narrating **who said what to whom** instead of stating the content. The transcript's turn-taking is a fact about the recording, not information the reader needs.

**Cut these**: 「X 说」「Y 追问」「Z 顺着说」「他的回答是」「她收尾的那句是」「主持人问了一个问题」「接下来才是转折」「她讲了一个例子」「节目最后，她的临别赠言是」 and every equivalent framing sentence.

**Keep attribution only when** it genuinely changes how the claim should be weighed: a contested opinion, a first-person clinical anecdote, a disagreement between two speakers, or a claim whose credibility rests on who is making it.

| Scaffolded (wrong) | Direct (right) |
|---|---|
| Mel 在这里问，需要装袋带去医院吗。Pasricha 的回答是拍照就够了。 | 如果遇到异常颜色，要记得拍照。 |
| Pasricha 说这是好事，所有你没吸收掉的纤维都是送给微生物组的礼物。 | 所有你没吸收掉的纤维都是送给微生物组的礼物。 |
| Mel 追问这是不是美国特有的问题。Pasricha 的回答是全球趋势。 | 这是一个全球趋势。 |

The target register is **knowledge writing** — a knowledgeable person telling you how something works. Not a 记叙文, not meeting minutes, not a play-by-play of a conversation.

If the input is an article rather than a conversation, the same rule applies: state what the piece establishes, don't narrate the author's moves through it.

### Premise before claim (CRITICAL)

The reader of an AI/tech piece is sophisticated but not omniscient. They know the field; they do not necessarily know the specific practice, product, or metric this particular source is built on. **Every load-bearing concept gets its premise stated before the first claim that depends on it.**

Two failure modes, both common:

**1. Using a term as if already introduced.** A section titled 「为什么冷邮件突然不好使了？」 assumes the reader knows what cold email is, who does it, and why anyone cares. State it first, in one sentence: what the practice is, who uses it, what it's for. Then make the claim.

**2. Claims without a subject.** 「回复率一路在掉」 — whose reply rate? To what? Opening a section with a bare metric is the written equivalent of walking into a room mid-sentence. Name the thing, name the metric, then give the direction of movement.

**3. Retrofitting the premise after the payoff.** Supplying missing background is only half the fix; if it lands *after* the sentence it was meant to support, the passage now doubles back and reads worse than before. Watch for this in two places especially:

- **Directly under the 「本期简介」/"In this episode" lead**, which by design states the conclusion first. Everything after that line must move forward. Do not open the next paragraph with a rhetorical question restating what the lead just answered ("X 之后为什么轮到 Y？") — the reader was told the answer one line ago, so the question reads as the writer arguing with themselves.
- **Anywhere a fix was applied in a later editing round.** When adding reasoning to an already-drafted passage, place it before the claim and re-read the whole run of paragraphs, rather than appending it where the note was made.

The forward-moving shape for an intro is: an established fact, the consequence that follows from it, then what this piece therefore covers and how to read it. Attribution ("这是 X 定的调子") rides inside the consequence sentence; parked at the end of its own sentence it reads as an afterthought.

| Missing premise (wrong) | Premise first (right) |
|---|---|
| 回复率一路在掉。 | 冷邮件是海外 B2B 最主流的获客方式之一：拿到一批陌生人的工作邮箱，直接发第一封推销信。这套打法正在失效，最直接的指标是回复率，也就是发出去的邮件里有多少人回，这个数字一路在掉。 |
| 第一个 agent 要干的事就是把这个动作变成一条可以跟进的线索。 | 一个点赞本身没有任何营销价值。你不知道点赞的人是谁，也联系不上他。它要变成一条真正能用的线索，得跨过三道门槛：知道这个人是谁，拿到能联系上他的方式，以及有东西持续去跟进他。 |

The test: read each section opening cold, as someone who skipped everything before it. If the first sentence presumes a fact the piece hasn't supplied, supply it.

### Show the judgment logic, not just the conclusion (CRITICAL)

The audience for this kind of note is people who **do things**. A bare conclusion reads as unearned to them, however correct it is. What makes a piece feel solid is the speaker's reasoning: the dimensions they weighed, the criteria they applied, the alternatives they rejected.

While drafting, mark every sentence that states a judgment, and check the source for the reasoning behind it. If the source gave reasoning, **it is not optional material — it is the most valuable material in the transcript.** Cutting it to save length is the wrong cut.

Specific patterns to preserve:

- **Multi-leg arguments stay split.** If the speaker gives two independent reasons (cost *and* reliability), present them as two, labelled. Merging them into one paragraph turns an argument into an assertion with adjectives.
- **Restore the parallel case.** Speakers often prove a heuristic generalizes by applying it to a second domain ("同一套只追离群值的判断他在投放上也用……"). That second case is what shows it's a method rather than a one-off. Keep it, compressed.
- **Keep the decomposition that licenses the verdict.** "这个岗位已经死了" is only credible next to the five-step breakdown showing every step is now automated.
- **Keep both lenses when the speaker gives two.** If they define a term once by function ("以活儿论") and once by construction ("代码 + 思考循环 + 数据流"), a note that keeps only one has thrown away the part that makes the definition usable.
- **Spell out conversion logic.** "答案在 earned media 的账里" leaves the reader to derive the arithmetic. State it: the same impressions cost money on-platform, so organic impressions get valued at the same rate.

**Where reasoning genuinely wasn't given, do not manufacture it.** State the claim plainly and flag it in the delivery message as an unsupported assertion. Inventing a derivation the speaker never made is the same failure as inventing a fact.

### Conclusion and its support must be adjacent

A verdict in section four whose supporting decomposition sits in section five is a structural defect, even though both are technically present. The reader hits the conclusion holding nothing.

When theme-based reorganization separates a claim from its evidence, **move the evidence to the claim**, then rewrite the later section so it doesn't repeat what moved. Check for this specifically during the cut pass: for each bolded or emphatic judgment, confirm its support is in the same section, ideally the adjacent paragraph.

### Section openings and seams

Theme-based reorganization breaks the source's own connective tissue, so the draft must supply new connective tissue. Every section and sub-section opening does one of two jobs: it either says why this section exists now, or it states the premise the section rests on. A section that opens by simply starting is a seam.

- **A definitional or conceptual section needs its reason for existing.** Dropping 「到底什么算 agent？」 between two build walkthroughs stops the reader cold unless the opening says why the distinction matters for what they're building.
- **Parallel sections need their relationship stated.** Moving from the first approach to the second, name the trade-off between them rather than saying 「这是另一套」.
- **Overview lists must map one-to-one onto the sub-headings that follow.** If the intro promises five steps, the five sub-headings should be recognizably those five steps, in that order, in matching words.
- **Terms are defined where they're first used, not later.** If a term appears in a definition sentence, its own explanation follows immediately — not after an intervening paragraph on something else.
- **Imported paragraphs need a connective clause.** A paragraph relocated from elsewhere in the transcript lands as a non sequitur unless one clause ties it to the argument around it.

### Bold emphasis logic

Use bold to give readers visual anchors:
- Counterintuitive conclusions
- Key data points and statistics
- Core actionable advice
- Particularly important sentences carried inline from the speaker

**Budget: roughly one bolded span per two or three sections of body text.** Bolding a sentence in every paragraph is itself an AI tell — it reads as a machine mechanically issuing one "key point" per unit. When nearly everything is anchored, nothing is. On a draft that has drifted, cut the bold count by half and keep only the lines a reader would quote.

### No editorial voice in the article (CRITICAL)

Publishing Mode output contains **no editor's voice at all**. This overrides any earlier version of this Skill that asked for 编者注 blocks, a closing 编者手记, or marked editorial asides in the body.

**Not allowed anywhere in the file**:
- A closing 「编者手记」/ "Editor's reflection" section
- Inline 「编者注」 blocks or bracketed editor commentary
- Sentences that shift from "what X said" to "my read on this is…"
- Cross-references to the user's other published pieces
- Any value judgment, recommendation, or "what to take away" framing the speaker did not say

The article ends with the content's own strongest point, followed by `The end.`

#### Evidence concerns go in the delivery message, not the article

Evidence discipline still matters — it just lives outside the file. While drafting, keep checking claims against their actual evidence base, and put every concern in the **delivery message** for the user to see:

- **State the boundary of a study's population.** If a finding comes from a specific cohort (one country, one age band, one sex), say so, and flag when the speaker generalizes beyond it. Common case: a result established in **male** subjects spoken about as if it applies to everyone.
- **Flag study-design limits** that the speaker's confident delivery obscures — cross-sectional data presented as causal, a single-lab result, reverse causation left unaddressed.
- **Flag claims you could not verify**, and claims whose only source is the speaker's own telling.
- **Flag where a guideline or consensus statement is being paraphrased loosely**, and give the more accurate reading.
- **Distinguish what the speaker proved from what the speaker asserted.**
- Keep this stance even when you agree with the speaker.

Example delivery note:

> 发布前有几点需要你知道：
> - 所有流行病学数字都是美国样本，中国数据不同
> - 手机与痔疮那项研究是横断面研究、出自她自己实验室，反向因果同样成立
> - 「AGA 不推荐益生菌」的准确含义是证据不足以普遍推荐，不等于无效
> - 测谎仪与小行星那段唯一信源是她本人讲述，未能核实

Then let the user decide whether to cut, soften, or keep the passage. If the user removes something, don't reintroduce it later in the conversation.

### Content selection: cut whole sections, don't polish everything

Publishing Mode is **not** a "preserve as much as possible" mode. A first draft that covers everything is too long; the fix is amputation, not trimming. When in doubt, **cut short rather than cover completely**.

**Priority targets for deletion — remove the whole section, don't shorten it**:
- Sections only loosely connected to the article's main line, however interesting on their own
- Metaphorical attribution / absolution passages (e.g. framing cancer risk as "stacking blocks") — decorative comfort, not information
- The guest's account of their own methodology, career path, or how they came to study something
- Rhetorical question sequences that build drama without adding facts ("如果我们一直搞反了呢？如果是……呢？")
- Host reactions that merely confirm what the guest just said

**Keep**: counterintuitive information, specific data, actionable advice, and the human-color details that make a fact stick.

After cutting a section, **renumber all downstream sections and clean up every reference to the deleted content** in the intro and elsewhere — without being asked.

**Never write a dry "key points summary".** Shorter means fewer topics covered, not thinner coverage of each.

**Cut at the topic level, never at the sentence level.** Once a topic is kept, it gets the full treatment: the background that makes it make sense, every link in the causal chain, who each person is, what each term means, and how it connects to the topic before it. Compressing three reasoning steps into one sentence is the most common way a draft becomes unreadable — the reader stalls at exactly that sentence and cannot say why. When choosing between a longer explanation and a tighter one, choose longer. This is the one place where **宁长勿简** overrides brevity.

Signals that a kept topic has been over-compressed:
- A sentence that only makes sense if you already know the source ("已经解决的那一半靠触发" — a half of *what*?)
- A conclusion presented without the premise that produced it ("有人来推她一把，她本可以做更大的事" — who, why, and why can't she do it herself?)
- A named person, product, or framework introduced without saying what it is and why it appeared here
- A coined concept (「软件工厂」, 「看门狗」) whose definition arrives several paragraphs after its first use

**Definitions and conclusions go at the top of their section**, not at the end. If a section explains a coined concept, define it in plain language in the section's first or second paragraph, then use the example to confirm it. A reader who has to reach paragraph six to learn what the section is about has already left.

### Layout

- **Short paragraphs — 1 to 3 sentences each.** This output is read on a phone in WeChat; a paragraph that runs six lines will be skipped. Break long reasoning into consecutive short paragraphs rather than one dense block.
- **Leave natural room for images.** The user inserts illustrations captioned 「配图由 ChatGPT 制作」 at section transitions and after vivid passages. Don't write walls of text with no breathing point.
- **Use a bullet list whenever three or more genuinely parallel items appear** — a three-tier taxonomy, three requirements, three options. Running them together as prose forces the reader to hold the parallel structure in their head. Everything else stays prose.
- "Magazine long-form" reading feel, not "PowerPoint bullet dump"
- **The em dash 「——」 is welcome for spoken rhythm and for a beat before a punchline** (「活主要不是人在干——」). What is banned is the *translated-English* insertion dash, the one wrapping a subordinate clause mid-sentence. Do not zero out em dashes as a de-AI reflex.
- For Chinese output: insert spaces between Chinese and English characters (e.g., "Sinclair 提出" not "Sinclair提出")
- For English output: standard English typography
- First mention of proper nouns: provide original term parenthetically when crossing languages
- **No horizontal dividers** (Markdown `---` or `<hr>`)
- **Sections connect directly through headings**, no separator decoration

#### Punctuation conventions (Chinese output)

- **Quotation marks: use corner brackets 「」 (and 『』 for nested quotes), never straight double quotes `"` or curly quotes "".** This applies everywhere — body text, blockquotes, headings, the intro. Produce 「」 directly; do not emit straight quotes "to be fixed later."
- Use full-width Chinese punctuation throughout: 。，、；：？！ — not their half-width ASCII counterparts. In particular, a half-width semicolon `;` or comma `,` sitting between two Chinese clauses is a defect; use ；／，.
- Title and section headings follow the same quote convention (「」 not "").
- **A period is a peer-level mark. Do not use it where the next sentence is subordinate to the previous one.** Removing colons and connectives is a common over-correction when stripping translationese; it flattens hierarchy and forces the reader to reconstruct the relationship. Before an example, a reason, or a list, keep the signpost:
  - Example → 「就好像在说：」「比如」「她举的例子是……」「有一次，」
  - Reason → 「因为」「得……才……」「图的就是」
  - List → a colon, or 「放法很简单：」「先问自己三句话：」
  - Wrong: 「听着就是直属经理在派活。Susie 你去那边，Betty 你去这边。」 The reader meets Susie with no idea who she is.
  - Right: 「听着就是直属经理在派活，就好像在说：Susie 你去那边，Betty 你去这边。」
- **Place a term's gloss before its first working use, not after.** Wrong: 「webhook 出过问题，那就学透这一次。webhook 就是两个系统之间传消息的那根管子。」 Right: 「比方说 webhook 出过问题，webhook 就是两个系统之间自动传消息的那根管子。这一次把它弄明白……」


### Output format

- Standard Markdown — paste-ready into any Markdown editor or formatting tool
- If the environment supports it, create a `.md` file in the outputs directory for the user to download

### Cross-language handling 跨语言处理

**When the source content language differs from the output language, follow these rules:**

- Translate naturally — no literal/awkward translations. The note should read as if it were originally written in the output language.
- **Quotes preserve tone and rhythm** — when translating a quote, the translated version should carry the same emotional weight and pacing as the original. Don't flatten Sinclair's "that's not a sick mouse, that's an old mouse" into "Sinclair confirmed the mouse showed signs of aging."
- **Names**: keep proper names in their original script (English names stay in Roman letters even in Chinese notes; Chinese names stay in Hanzi even in English notes).
- **Technical terms**: on first mention, provide the original term parenthetically — e.g., "表观基因组（epigenome）". After first mention, use the natural form for the output language.
- **Gloss sparingly (Publishing Mode).** Explain only two things: (a) English abbreviations that mean nothing in the output language — "GI 门诊" must become "消化内科门诊"; (b) genuine specialist disease names a general reader would stall on, e.g. IBS. Do **not** gloss terms inferable from context (盆底, 憩室, 缺铁性贫血), institution names (Beth Israel), or a foreign drug brand's local equivalent. When unsure, leave it out — over-glossing pads the prose and condescends to the reader.
- **Gloss by tier, not by language.** The question is not "is this word English?" but "would a competent reader of this piece stall here?" Three tiers:
  - *Field-standard terms the audience already owns* — agent, prompt, API, ROI. No gloss. Glossing these is condescending.
  - *Product- or workflow-specific jargon, especially mixed-script strings* — `profile post scraper`, `post reactions`, `post comments`, `catch-all`, `webhook`, `waterfall enrichment`, `CAN-SPAM`, `SDR`, `earned media`. These stall even expert readers, because they name one vendor's specific artifact. Say what it does in the output language first, then give the original term parenthetically once.
  - *Terms the speaker invented or is using idiosyncratically* — 思考循环, 软件工厂. Define at first use, in the same paragraph.
- **Do not add local-market mappings (「国内对照」 sidebars) unless the user asks for them.** They are the editor's voice wearing a label, and in practice they get deleted. If a mapping seems genuinely load-bearing, mention it in the delivery message and let the user decide — do not put it in the draft.
- **Never translate an English metaphor, idiom, or proverb literally.** Translate what it *means*, in an expression the output language actually uses. The failure is invisible to the writer and obvious to the reader.
  - `blow past their ceiling` → 不要写「冲破自己的天花板」（中文的「天花板」现指「顶级/最强」，语义已漂移，读者会卡）→ 「做成自己原本以为做不到的事」「把一个人的上限往上顶一截」
  - `measure twice, cut once` → 不要直译 → 「地基铺一次，后面每个产品都省一次」
  - `the first domino` → 「那个必须先起头的人」
  - Keep an image only when it works without the reference behind it (「西班牙宗教裁判所那种，一盏强光打在脸上」 survives; the reader needs no history).
- **"X is the new Y" is a positional claim, not an identity claim.** Rendering it as 「X 就是新的 Y」 creates a false equation, which will directly contradict the piece if the source elsewhere distinguishes X from Y. Translate the position: 「Y 之后，轮到 X 了」. Watch for the same trap in "the next X", "X eats Y", "X is dead".
- **Attribute to the right speaker.** In multi-speaker sources the host's framing, the guest's claim, and a third party quoted by the guest are three different things. Check who actually said a line before attributing it, especially for the lines punchy enough to become the title. Misattribution is the same class of failure as invented content.
- **Cultural references**: if the source uses a culturally-specific reference the target audience won't know, briefly explain it inline rather than removing it.
- **Quote attribution**: in Publishing Mode there is no attribution line, because there are no quote blocks — see "Quote handling".

---

## Workflow 工作流程

### For conversation / podcast / video content (transcript input):

1. Read the full transcript and understand the overall structure and themes
2. Reorganize by **theme** (not by timeline), split into sections
3. Extract key quotes (Knowledge Mode: preserve original precision; Publishing Mode: pick the high-impact passages)
4. Write editorial transitions and interpretations
5. Mark key elements (Knowledge Mode: wikilinks + bold; Publishing Mode: bold)
6. Write the opening intro (Publishing Mode: no closing reflection — end on content, then `The end.`)
7. Knowledge Mode: generate frontmatter + Suggested Wiki Updates + Next steps
8. Review: any repetition? any human-warmth details missed?
9. **Publishing Mode: cut.** Re-read the draft asking which whole sections can go (see "Content selection"), then renumber and clean up references
10. **Publishing Mode: strip reporting scaffolding.** Search the draft for 「说」「问」「追问」「回答」 and delete every framing sentence that isn't load-bearing
11. **Publishing Mode: read the whole draft in the persona of its actual reader** — a practitioner in the piece's field, not a generalist. Read straight through and mark every place you stall: an undefined term, a claim with no premise, a verdict with no reasoning, a section that starts without saying why it exists, a paragraph that doesn't follow from the one above. Fix the seams; report the ones you chose not to fix and why. **After applying any fix, re-read the surrounding paragraphs in order** — a correct addition placed at the wrong point in the sequence creates a new seam, and repair rounds are where backward-running passages get introduced. **Run this pass before the humanizer pass** — humanizing a draft with broken joints just makes the breaks read more naturally
12. **Publishing Mode: run the humanizer-zh pass** (see below) to strip AI-writing tells before delivery
13. **Publishing Mode: write the delivery message** — evidence concerns and unverified claims (see "Evidence concerns go in the delivery message"), plus 3–6 title candidates for the user to choose from
14. **Rename the conversation to the article's title**

### For article / webpage content:

1. Read the full piece and understand the core arguments
2. Decide whether to reorganize (if original structure is clear, keep it; otherwise reorganize by theme)
3. Extract key paragraphs as quotes
4. Other steps same as above (including the Publishing Mode humanizer pass and aside review)

### Humanizer pass (Publishing Mode, Chinese output)

For Chinese Publishing Mode output, **apply the `humanizer-zh` skill as a standard finishing step**, not only when the user explicitly asks. The goal is to remove AI-writing tells before the piece is delivered. Focus areas that recur in this kind of note:

- **Self-evaluating commentary** that pre-judges for the reader: 「全场最精彩的比喻」「那句最有分量的话」「让人心头一沉的数据」「特别真实的例子」「这是听众最想听的部分」. Cut these — let the quote or fact carry itself.
- **Em-dash overuse (——).** Keep them for spoken rhythm and for a beat before a punchline; convert *translated-English insertion* dashes to commas or periods. Do not drive the count to zero — that is its own tell.
- **Banned word: 拆／拆解.** Use 讲／说／走一遍／过一遍／报一遍／分成两半 instead. Same for the 剖析／梳理／解构 family — they read as knowledge-payment marketing.
- **Tidy aphoristic closers, stacked.** Any single one is fine; four or five in a piece is a machine signature, because real writers don't land a symmetric punchline at the end of every paragraph. Count them and defuse the ones you invented (as opposed to the ones the speaker said).
- **Negation parallelism (「不是 X，是 Y」).** Cheap-looking insight. Count them; keep only the ones that come from the source, rewrite the rest as plain statements.
- **Over-colloquialism that tips into dialect.** 「盘子」「这口」「这码事」「肥」「狠」 read as regional and make the reader wonder where the writer is from. Colloquial means loosening written register, not swapping in a different vocabulary.
- **Filler transitions / logical crutches**: 「接得很自然」「特意」「值得停一下」「由此引出」「这恰恰是」. Replace with plainer phrasing.
- **Half-width punctuation** between Chinese clauses (`;` `,`) → full-width.
- Rule-of-three padding, vague attribution, negation-parallelism, and the other patterns the humanizer-zh skill enumerates.

Add to the focus list: **reporting scaffolding** (see "No reporting scaffolding") and **self-congratulatory glosses on the source** 「她的解释很有说服力」「我很喜欢这个比喻」「这是全场最放松的一段」. Both read as AI padding and both are cut.

Humanizing must not flatten the speaker's own vivid phrasing. Strip *hollow* commentary; keep the concrete image, the number, and the sentence that stings.


---

## What this Skill is NOT for

To avoid wrong expectations, the explicit boundaries:

- **Not a 200-word minimal summary tool**: this Skill's core principle is "preserve by default", especially human-warmth details. If you just need a one-paragraph summary, use a different tool
- **Not a literal translator**: if you need a word-for-word bilingual translation, use a dedicated translation tool
- **Does not judge content quality for you**: this Skill assumes you've already chosen content worth digesting; its job is to digest efficiently, not to filter "is this worth reading"
- **Does not maintain your knowledge base**: the "Suggested Wiki Updates" section is suggestions, not automated wiki maintenance. True automation requires a larger system (see the wiki maintainer role in Karpathy's LLM Wiki pattern)

---

## License

MIT License. Free to use, modify, distribute, including commercially. Just keep the copyright notice.
