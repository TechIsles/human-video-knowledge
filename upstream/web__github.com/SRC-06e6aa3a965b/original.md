---
name: travel-xhs-content
description: 小红书旅游内容生成 Skill（GitHub 开源版）。输入景点或城市名，**自动调用 WebSearch 查实时爆款**（无联网时回退到 LLM 知识 + 速查档案），生成 6 种风格的中英文对照文案（系列打卡 / 情绪共鸣 / 反差吸引 / 治愈系 / 探险系 / 极简系）、每个风格 3 张配图的中英文 AI 生图 Prompt（含 1 最佳 + 2 备用），并给出发布时间、互动话术、跨平台适配（小红书 / Instagram / 朋友圈 / 微博 / 抖音 / 视频号）建议。**支持联网 + 离线双模式**，零依赖、零配置、零 API Key，纯 Markdown 提示词工程。

trigger_patterns:
  # 中文核心触发词
  primary_zh: [小红书文案, 小红书攻略, 小红书种草, XHS 文案, 景点社媒, 旅游爆款]
  # 中文场景触发词
  scenario_zh: [城市种草, 景点打卡, 周末游, 3天2夜, 穷游攻略, 毕业旅行, 亲子游, 情侣游]
  # 中文工具触发词
  tool_zh: [AI 生图提示词, prompt 工程, 配图 prompt, 文案生成器, 旅行文案]
  # 英文触发词
  en: [travel content generator, xiaohongshu post, RED app content,
       travel copywriting, AI image prompt, midjourney travel, social media travel]
  # 平台触发词
  platforms: [小红书, RED, Instagram, 朋友圈, 微博, 抖音, 视频号, TikTok, XHS]
  # 风格触发词
  styles: [系列打卡, 情绪共鸣, 反差吸引, 治愈系, 探险系, 极简系]

when_to_use:
  - 用户想为某景点/城市生成小红书图文或视频脚本
  - 用户提供景点图/城市名，需要标题 + 正文 + AI 配图 Prompt + 标签
  - 用户想跨平台同步发布同一份内容
  - 用户想做系列化旅游内容（川西小环线 / 中国城市巡游）
  - 用户希望借鉴爆款规律，生成差异化内容

when_not_to_use:
  - 纯商业广告 / 品牌种草文（非真实体验种草）
  - 与旅游目的地无关的内容（电商 / 代码 / 纯情感文案）
  - 用户只要简短问答（如「九寨沟门票多少」），直接答即可

version: 1.2.0
license: MIT
metadata:
  github:
    repo: travel-xhs-content
    homepage: https://github.com/bethel-mark/travel-xhs-content
  platforms: [xiaohongshu, instagram, wechat, weibo, douyin, shipinhao, tiktok, x-twitter]
  languages: [zh-CN, en]
  style_count: 6
  destination_count: 38
  example_count: 4
  zero_dependencies: true
  web_search_support: true
  auto_fallback: true
---


# 🌸 Travel XHS Content · 小红书旅游内容生成

> 一句话定位：**给一个景点名，输出一份可直接发的小红书 / IG / 朋友圈 / 微博内容方案**。

本 Skill 是 [`travel-social-content`](../travel-social-content)（通用版）的小红书**专精版**：内置小红书爆款规律、标题公式、标签策略、互动话术，并把内容深度优化到小红书**种草感、口语化、视觉冲击**的核心审美。

---

## ✅ 何时调用

- 「帮我做一份 XX 市的小红书方案」「给九寨沟出个种草图文」「稻城亚丁小红书爆款标题」
- 「乐山周末 2 天 1 夜小红书攻略」「帮我写四川美食 + 打卡的小红书」
- 用户提供景点图 / 城市名，要拆出**小红书爆款标题 + 配图 Prompt + 标签组合**
- 用户想**跨平台同步**：同一份内容要发小红书 / IG / 朋友圈 / 微博
- 用户想做**系列化**旅游内容（川西小环线 / 我的中国城市巡游）

## ❌ 何时**不要**调用

- 纯商业广告 / 品牌种草文（非真实体验种草）
- 与旅游目的地无关的内容（电商、代码、纯情感文案）
- 用户只要简短问答（直接答即可，不必出完整方案）

---

## 📥 输入约定

| 字段 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `目的地` | ✅ | — | 城市名 / 景点名（如「稻城亚丁」「宜宾市」） |
| `平台` | ❌ | `小红书` | `小红书 / Instagram / 朋友圈 / 微博 / 抖音 / 视频号 / 全平台` |
| `风格` | ❌ | `情绪共鸣型` | 6 种风格见 [§3](#-3-文案风格库-6-种) |
| `张数` | ❌ | `6` | 每个风格生图 Prompt 数量（3-9） |
| `真实参考` | ❌ | — | 用户提供的参考图 URL 或文字描述（可选） |
| `场景` | ❌ | — | `周末游 / 3天2夜 / 亲子 / 情侣 / 毕业旅行 / 穷游`（可选） |

---

## 🧭 标准执行流程（6 步）

```
① 解析输入 → ② 搜索速查 → ③ 生成文案 → ④ 生图 Prompt → ⑤ 平台适配 → ⑥ 输出 Markdown
```

### Step 1 · 解析输入
提取 `目的地 / 平台 / 风格 / 张数 / 真实参考 / 场景`。
若 `目的地` 在 [`references/destinations.md`](references/destinations.md) 中存在速查档案 → **直接读取缓存**，仅做差异补充；否则进入 Step 2 全量搜索。

### Step 2 · 搜索 4 类信息（**联网 + 自动回退**）

Skill 支持两种运行模式，**优先尝试联网，失败自动回退**：

#### 🌐 模式 A：联网模式（WebSearch / WebFetch）

当 Agent 支持 `WebSearch` / `WebFetch` 工具时调用：

| 信息类型 | 推荐工具 | 搜索词模板 |
|---|---|---|
| 景点门票 / 时间 | WebSearch | `{景点} 门票 开放时间 {当前年}` |
| 爆款规律 | WebSearch | `{目的地} 小红书 爆款 2026` |
| 实时价格 / 攻略 | WebSearch / WebFetch | `{城市} 攻略 {当前年}` |
| 标签热度 | WebSearch | `{目的地} 小红书 热门话题` |
| 实测信息（视频/图文） | WebFetch | 携程/小红书/抖音 URL |

**完整联网策略**：见 [`references/web-search-strategy.md`](references/web-search-strategy.md)

#### 📴 模式 B：离线模式（LLM 内建知识 + 速查档案）

当 Agent **无 WebSearch / WebFetch 工具** 或 **联网失败**（超时/403/网络错误）时自动回退：

1. **首选**：`references/destinations.md` 速查档案（已收录 20+ 景点）
2. **兜底**：宿主 LLM 的预训练知识
3. **补充**：用户提供的参考资料 / 截图

#### ⚙️ 回退触发条件（任一即触发）

```
❌ 触发回退的条件：
  • Agent 工具列表中无 WebSearch / WebFetch
  • WebSearch 调用返回 timeout / 403 / network error
  • WebSearch 返回空结果（搜索引擎无记录）
  • 用户主动指定 web=false
```

#### 🎛️ 用户控制参数

| 参数 | 默认 | 行为 |
|---|---|---|
| `web=auto` | ✅ | 先尝试联网，失败自动回退到离线 |
| `web=true` |   | 仅联网模式（失败时报错） |
| `web=false` |   | 仅离线模式（跳过联网） |

> 💡 **开源原则**：本 Skill 不内置任何 API Key，所有联网都通过宿主 Agent 的 WebSearch 工具完成。零依赖 = 任何 Agent 即装即用，联网能力取决于 Agent 配置。

### Step 3 · 生成 6 种风格文案（中英对照）
依据 [`references/copywriting-formulas.md`](references/copywriting-formulas.md) 的公式，每种风格产出：
- 中文版（口语化、emoji、加粗关键词）
- 英文版（适合 IG / 国外平台）

### Step 4 · 生成 AI 生图 Prompt（中英对照）
依据 [`references/image-prompts-cheatsheet.md`](references/image-prompts-cheatsheet.md)：
- **1 张最佳主图**（封面级，视觉冲击力最强）
- **2 张备用图**（不同机位 / 氛围 / 时间光线）

每个 Prompt 包含：主体描述、构图机位、光线、色调、风格锚词、画质后缀。

### Step 5 · 平台适配
针对目标平台，输出**风格微调建议**：
- 小红书：emoji 丰富 / 标题党 / 标签 15-20 个
- Instagram：英文为主 / hashtag 25-30 个 / 美学优先
- 朋友圈：单图 + 简短文案 / 九宫格统一风格
- 微博：话题词 + 短文案 / 适合互动
- 抖音 / 视频号：3-5 秒镜头脚本 + BGM 建议

详见 [`references/platforms.md`](references/platforms.md)。

### Step 6 · 输出 Markdown 方案
按 [`references/output-template.md`](references/output-template.md) 模板输出完整方案，落地到 `output/<目的地>_<平台>_<日期>.md`。

---

## 🎨 3 文案风格库（6 种）

| # | 风格 | 一句话公式 | 适用场景 |
|---|---|---|---|
| 1 | **系列打卡型** | `第N站·<景点>，<价值点>` | 城市巡游 / 多景点合集 |
| 2 | **情绪共鸣型** | `在<地点>，我终于理解了<金句>` | 单景点深度 / 治愈向 |
| 3 | **反差吸引型** | `以为<刻板印象>，直到<反差事实>` | 破圈 / 引爆话题 |
| 4 | **治愈系** | `<光线>+<静物>+<内心独白>` | 文艺 / 慢节奏 / IG 调性 |
| 5 | **探险系** | `<难度>+<极限>+<成就>` | 户外 / 川西 / 越野 |
| 6 | **极简系** | `<一句话>+<关键词>` | 高级感 / 设计师 / 极简主页 |

每种风格的详细公式 + 示例 + Prompt 关键词见 [`references/copywriting-formulas.md`](references/copywriting-formulas.md)。

---

## 🖼️ 4 生图 Prompt 体系

每个景点 × 每个风格 = 3 张 Prompt（1 主 + 2 备）。公式：

```
[主体描述] + [构图机位] + [光线] + [色调/后期] + [风格锚词] + 8K / photorealistic
```

例（乐山大佛 · 系列打卡型 · 主图）：

```
A magnificent 71-meter Leshan Giant Buddha statue carved into a cliff at the
confluence of three rivers, wide-angle low-angle shot, golden hour backlight
casting long shadows across the cliff face, lush green mountains surrounding,
river mist in foreground, high saturation cinematic color grading, epic scale
landscape photography, photorealistic, 8K UHD.
```

完整 Prompt 工程指南、备用 Prompt 改写技巧见 [`references/image-prompts-cheatsheet.md`](references/image-prompts-cheatsheet.md)。

---

## 📱 5 平台适配速查

| 平台 | 字数 | 标签数 | Emoji | 标题风格 |
|---|---|---|---|---|
| 小红书 | 500-1000 | 15-20 | 大量 | 数字 / 情绪 / 反差 / 悬念 |
| Instagram | 100-300 | 25-30 | 适量 | 简洁 / 关键词 |
| 朋友圈 | 30-100 | 0 | 少 | 短句 / 文艺 |
| 微博 | 80-200 | 3-5 话题 | 适量 | 热点 / 互动 |
| 抖音 / 视频号 | 视频 15-60s | 3-5 | 字幕 | 钩子前 3s |

详细差异与适配模板见 [`references/platforms.md`](references/platforms.md)。

---

## 📁 输出文件结构

方案落到 `output/` 目录，文件名格式：

```
output/
├── 乐山市_小红书_情绪共鸣型_2026-07-20.md       # 单目的地单平台
├── 稻城亚丁_全平台_系列打卡型_2026-07-20.md     # 单目的地多平台
└── 四川巡游_小红书_系列打卡型_2026-07-20.md    # 多目的地
```

---

## 🧪 完整方案 Markdown 结构

参见 [`references/output-template.md`](references/output-template.md)：

```
1. 目的地简述（1 段定位）
2. 5-6 个子景点详细方案（每景点含：文案 / 风格 / 图片 / Prompt / 实用信息表）
3. 6 种风格文案速览
4. 多平台适配（按用户选定平台）
5. 发布建议（封面 / 标题 / 标签 / 时间）
6. 爆款要素拆解
7. 热门趋势分析
8. 参考来源
9. AIGC 标识（合规）
```

---

## 📦 参考包（references/）

| 文件 | 内容 |
|---|---|
| [`destinations.md`](references/destinations.md) | 速查档案：8 大四川景点 + 模板 |
| [`copywriting-formulas.md`](references/copywriting-formulas.md) | 6 种文案风格公式 + 标题套路 + 互动话术 |
| [`image-prompts-cheatsheet.md`](references/image-prompts-cheatsheet.md) | AI 生图 Prompt 工程指南（中英文） |
| [`platforms.md`](references/platforms.md) | 4 平台风格差异 + 适配模板 |
| [`output-template.md`](references/output-template.md) | 完整方案 Markdown 模板 |
| [`banned-words.md`](references/banned-words.md) | 小红书违禁词清单（医疗 / 绝对化 / 功效） |

---

## 🧩 示例输出（examples/）

| 文件 | 内容 |
|---|---|
| [`乐山市_示例.md`](examples/乐山市_示例.md) | 乐山实测：6 景点 + 6 风格 + 18 Prompt |
| [`宜宾市_示例.md`](examples/宜宾市_示例.md) | 宜宾实测：8 模块完整方案 |

---

## 🚀 快速开始

### 在 Codex / Claude Code 中调用

直接对 AI 说：

```
/travel-xhs-content 目的地=稻城亚丁 平台=小红书 风格=情绪共鸣型 张数=6
```

或自然语言：

```
「帮我用 travel-xhs-content skill 生成一份九寨沟的小红书方案」
```

### 在其他 Agent 平台使用

任何能读 SKILL.md + 调 LLM 的 Agent（ChatGPT、Cursor、Cline、Continue 等）都能直接按本文件执行。本 Skill **零依赖、零配置**。

### 作为 Prompt 模板使用

复制 [`SKILL.md`](SKILL.md) + [`references/`](references/) 下任意文件，即可作为系统 Prompt 片段使用。

---

## 🤝 与其它 Skill 的关系

- **上游**：`travel-social-content`（通用版，覆盖多平台 + 复杂场景）→ 拆分出本 Skill 的小红书专精版
- **平行**：`food-social-content`（美食版）、`travel-and-food-skill`（旅游+美食融合版）
- **下游**：本 Skill 输出的 Prompt 可直接喂给任何生图工具（Midjourney / DALL·E / Stable Diffusion / 即梦 / 文心一格）

---

## ✨ Skill 亮点（Why Travel-XHS-Content？）

### 🎯 差异化定位

| 维度 | travel-xhs-content | 通用内容生成 skill |
|---|---|---|
| **平台专精** | 小红书 6 风格深度 | 多平台通用但浅 |
| **文案输出** | 中英双语 5+1 风格 | 单语 1-2 风格 |
| **生图 Prompt** | 每个风格 1 主 + 2 备（共 3 张） | 通常只 1 张 |
| **联网能力** | ✅ WebSearch + 自动回退 | ❌ 仅离线 |
| **速查档案** | 38 个真实景点（含 AI Prompt） | 无 |
| **跨平台** | 5 平台 + 视频脚本 | 3 平台通常 |
| **国际化** | 中英双语 + 英文 references-en/ | 单语 |

### 🚀 7 大核心优势

1. **零依赖**：纯 Markdown 提示词工程，无 Python/Node 运行时
2. **联网智能**：WebSearch 优先 + 4 种失败自动回退，无感切换
3. **速查档案**：38 个真实景点速查（含 AI 主图 Prompt），节省 80% 调研时间
4. **多风格覆盖**：6 种爆款公式（3 主 + 3 备选），覆盖不同账号定位
5. **完整方案**：每份输出含 9 个模块（从文案到发布策略）
6. **跨平台适配**：小红书 / IG / 朋友圈 / 微博 / 抖音 一键转换
7. **国际化**：references-en/ 完整英文版，吸引海外用户

### 💡 独特卖点

- ✅ **可直接发布**：输出文件命名规范（`目的地_平台_风格_日期.md`）
- ✅ **AI Prompt 即用**：所有 Prompt 中英对照，可直接喂给 Midjourney / DALL·E / 即梦
- ✅ **违禁词自查**：内置 banned-words.md 自动规避限流
- ✅ **AIGC 合规**：自动添加标识，符合 2025 法规
- ✅ **MIT License**：自由使用、修改、商用

---

## ⚠️ 使用注意事项

### 🚨 必须遵守

1. **景点信息二次核实**：门票 / 开放时间 / 交通可能随政策变化，发布前**务必查官方**
   - 参考：[故宫博物院](https://www.dpm.org.cn/) / [携程](https://www.ctrip.com/) / 各景区官网
2. **AIGC 标识**：所有发布到小红书/抖音/微博的内容必须标注「AI 生成」，否则违规
3. **不收录品牌词**：避免「第一 / 唯一 / 最」等绝对化用语，触发限流
4. **尊重版权**：图片禁止直接搬运其他博主作品，用 AI Prompt 自创
5. **真实体验**：种草内容应基于真实体验或可靠信息，不要完全虚构

### ⚡ 推荐实践

1. **优先联网**：默认 `web=auto`，可获取 2026 最新爆款规律
2. **多风格测试**：同一景点用 6 种风格各生成 1 份，挑爆款潜力最大的发
3. **微调后再发**：AI 输出建议人工微调 20-30%，增加真实感
4. **多平台同步**：一份内容同步发 5 平台，最大化 ROI
5. **系列化输出**：连续 3-5 篇同一风格内容，触发算法推荐

### ❌ 不推荐做法

1. ❌ 一次生成 10 景点堆在一起发（信息密度太高，算法不推荐）
2. ❌ 直接复制 AI 输出不改（容易被识别为 AI 限流）
3. ❌ 用 AI 图直接当配图（必须二次精修或用 midjourney 等高端工具）
4. ❌ 忽略版权使用他人图片
5. ❌ 标题党 + 内容不符（小红书严打）

### 🔧 故障排查

| 问题 | 原因 | 解决 |
|---|---|---|
| 输出景点信息过时 | 未启用联网（web=false） | 改用 `web=auto` 或 `web=true` |
| AI 生图效果差 | Prompt 缺少机位/光线描述 | 参考 `image-prompts-cheatsheet.md` 重写 |
| 标题不吸引人 | 没用数字/情绪/反差公式 | 参考 `copywriting-formulas.md` §标题公式 |
| 文案被判违规 | 含违禁词 | 对照 `banned-words.md` 改写 |
| 内容空泛 | 目的地不在速查档案 | 让 Agent 用 LLM 知识生成，或补充资料 |

---

## ❓ FAQ

### Q1: 这个 Skill 和其他内容生成 skill 有什么区别？

**A**: 三大独特价值：
1. **小红书专精**：6 种爆款风格 + 5 平台适配，是市面上少数专注小红书的开源 skill
2. **联网能力**：自动调用 WebSearch 查实时爆款，失败静默回退
3. **零依赖**：纯 Markdown，任何 Agent 即装即用

### Q2: 一定要联网吗？无网络环境能用吗？

**A**: 可以！本 Skill 设计为 **联网 + 离线双模式**：
- 联网模式：调用 WebSearch 获取实时数据（更准确）
- 离线模式：使用 LLM 预训练知识 + 速查档案（够用 80% 场景）
- 通过 `web=auto / true / false` 控制

### Q3: 支持哪些生图工具？

**A**: Prompt 适配所有主流生图工具：
- ✅ Midjourney（v6/v7）
- ✅ DALL·E 3
- ✅ Stable Diffusion（含 SDXL / SD3）
- ✅ Adobe Firefly
- ✅ 即梦 / 文心一格 / 通义万相（中文工具）

### Q4: 生成的 AI Prompt 可以商用吗？

**A**: 可以！本 Skill 是 MIT License，但请注意：
- 景点本身可能涉及商标 / 著作权（如「布达拉宫」是建筑名胜，描述 OK；「迪士尼」logo 不可）
- 真实景点照片引用需标注来源或自创
- AI 生成的图片版权依各工具 ToS 而定

### Q5: 如何扩展新景点？

**A**: 三步法：
1. 复制 `references/destinations.md` §自定义目的地模板
2. 填写所有字段（门票 / 季节 / 交通 / AI 主图 Prompt）
3. 提 PR 到项目 GitHub 仓库

### Q6: 6 种风格有什么区别？选哪个？

**A**: 见下表（更详细见 `copywriting-formulas.md`）：

| 风格 | 适合场景 | 涨粉潜力 |
|---|---|---|
| 系列打卡型 | 多景点合集、城市巡游 | ⭐⭐⭐⭐ |
| 情绪共鸣型 | 单景点深度、人文体验 | ⭐⭐⭐⭐⭐ |
| 反差吸引型 | 破圈、引流标题党 | ⭐⭐⭐⭐⭐ |
| 治愈系 | 文艺、IG 调性 | ⭐⭐⭐ |
| 探险系 | 户外、川西、越野 | ⭐⭐⭐⭐ |
| 极简系 | 高级感、设计师主页 | ⭐⭐⭐ |

**建议**：新账号先试「情绪共鸣型」和「反差吸引型」，最容易出爆款。

### Q7: 一份内容能发几个平台？

**A**: 本 Skill 默认输出 5 平台完整适配：
- 小红书（中文版，含 15-20 标签）
- Instagram（英文 caption，含 25-30 hashtag）
- 朋友圈（极简单图 / 九宫格）
- 微博（话题 + 短文案）
- 抖音 / 视频号（60-90s 完整视频脚本）

可全部发，也可挑 1-2 个最合适的发布。

### Q8: 联网搜索会不会很慢？

**A**: 已做性能优化：
- 单个查询 ≤10s 超时
- 整个 Step 2 ≤30s
- 推荐 5 步批量查询，避免多次往返
- 失败立即回退（不阻塞流程）

### Q9: 怎么更新到最新版本？

**A**: 三种方式：
```bash
# 方式 1：git pull（推荐）
cd /Users/a1234/Documents/06-YouTube/travel-xhs-content
git pull origin main

# 方式 2：直接重新克隆
git clone https://github.com/bethel-mark/travel-xhs-content.git

# 方式 3：本地修改后 git push
git add . && git commit -m "update" && git push
```

### Q10: 能在 Cursor / Claude Code / Cline 中用吗？

**A**: 可以！本 Skill 兼容所有支持 SKILL.md 加载的 Agent：
- ✅ Codex（官方）
- ✅ Claude Code
- ✅ Cursor
- ✅ Cline
- ✅ Continue
- ✅ Open WebUI

加载方式：将 SKILL.md 作为系统 Prompt 片段使用。

---

## ⚖️ 合规与版权

- **AIGC 标识**：输出文件末尾必须标注 `本文案由 travel-xhs-content 生成，请遵循《人工智能生成合成内容标识办法》`。
- **真实信息**：景点门票 / 季节 / 交通等可能随政策变化，发布前请用户**二次核实**。
- **原创性**：本 Skill 是 Prompt 编排层，输出的文案与 Prompt 鼓励二次创作与个人化改写。
- **不收录品牌词**：避免"第一 / 唯一 / 最"等绝对化用语，规避小红书限流。

---

## 📜 License

MIT License — 详见 [`LICENSE`](LICENSE)。

## 🙏 致谢

参考了 [`JimLiu/baoyu-skills`](https://github.com/JimLiu/baoyu-skills) 的 SKILL.md 结构、`timandjerry/xiaohongshu-ops` 的运营框架，以及小红书 / 抖音 / 微博平台公开的爆款规律。
