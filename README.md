# chat-to-content

> 把对话转文章和贴图 · Turn your AI chat sessions into publishable articles and share cards

![License](https://img.shields.io/badge/license-Apache--2.0-blue)
![Python](https://img.shields.io/badge/python-3.7%2B-3776AB)

一个把「跟 AI 聊完就散」变成「聊完即可发布」的技能：读取当前会话的真实对话记录，整理成一篇能直接发的文章，和一组竖版分享卡片。

---

## 它解决什么问题

跟 AI 把一件事聊透了、方案跑通了、报错也解决了，然后关掉窗口——下次遇到同类问题重新想一遍，想分享又重新组织一遍。同一个动作，重复无数次。

这个技能把那个动作固定下来：**读原始记录 → 提炼骨架 → 出文章和贴图**。

关键在第一步。它不依赖模型的「记忆」，而是直接读本地的会话记录文件。素材是磁盘上的事实，不是回忆。

## 特性

- **只认真实对话**。所有内容必须能在原始记录里找到出处；找不到就不写，也不替你编。
- **前置检查**。没有可读的历史对话、或有效内容过薄，直接停下并说明原因，不硬凑一篇稿子。
- **两种产物，可拆可选**。只要文章、只要贴图、或者两样都要，一句话指定。
- **贴图用 HTML 渲染，不用文生图**。中文永远不会糊、不会缺笔画，1080×1440 出图，2 倍高清。
- **6 种卡片版式 × 5 套配色**。内容多自动拆卡，溢出自动等比缩放。
- **几乎零依赖**。截图直接调系统自带的 Edge / Chrome 无头模式，不需要装浏览器驱动；只有 JPG 输出与长图合并才需要 Pillow。

## 快速开始

### 1. 安装

技能格式为 `SKILL.md` + YAML frontmatter，放进客户端的技能目录即可。

```bash
git clone https://github.com/weikion/chat-to-content.git \
  ~/.workbuddy/skills/chat-to-content
```

没有技能机制的客户端也能用——直接调用 `scripts/` 下的两个脚本，见「命令行用法」。

### 2. 一句话触发

在任意已经聊出实质内容的会话里说：

```
总结本次对话，生成文章和贴图
```

其他触发词：`总结会话` / `转成文章` / `生成贴图` / `生成分享卡片`。

要其中一种，说清楚就行：

| 你说 | 得到 |
| --- | --- |
| 「只要文章」「写篇稿子就行」 | 一份 Markdown 文章 |
| 「只要贴图」「生成分享卡片」 | 一组竖版卡片 |
| 「都要」（默认） | 文章 + 卡片 |

### 3. 拿到产物

```
outputs/2026-10-26_会话总结_ai-video-workflow/
├── _source/transcript.md                    # 从会话记录抽取的干净素材
├── 2026-10-26_ai-video-workflow_article.md  # 文章
├── cards/
│   ├── card-01.png   封面
│   ├── card-02.png   要点
│   ├── card-03.png   金句
│   ├── card-04.png   经验清单
│   └── card-05.png   结论
└── cards.json                               # 卡片数据，可复现
```

## 产物长什么样

**文章**遵循固定骨架：主标题 + 2-3 个不同风格的备选、场景化开篇、带观点的小标题、短句清单式总结、提问式结尾。口语化短段落，不堆术语。

**贴图**默认 5 张一套（也可出 3 张精简版或单张金句卡）：

| 序号 | 类型 | 承担什么 |
| --- | --- | --- |
| 01 | `cover` | 结果前置，勾住人 |
| 02 | `points` | 这次到底解决了什么 |
| 03 | `quote` | 关键转折的那句话 |
| 04 | `checklist` | 可复用经验清单 |
| 05 | `conclusion` | 结论 + 互动触点 |

## 命令行用法

### 抽取会话记录

```bash
# 前置检查：当前会话是否已有可总结的历史对话
python scripts/extract_session.py --check

# 抽取成干净素材（过滤工具调用、思维链、系统注入内容）
python scripts/extract_session.py --out-dir ./outputs/_source

# 找不到会话时，列出最近的可用会话
python scripts/extract_session.py --list 10

# 指定某个会话文件
python scripts/extract_session.py --session-file "/path/to/session.jsonl" --out-dir ./outputs/_source
```

`--check` 的判定门槛：用户发言 ≥ 1 条、助手回复 ≥ 1 条、有效正文 ≥ 200 字，三者齐了才返回 `ok: true`，否则返回具体原因。

### 渲染卡片

```bash
# 标准渲染（1080×1440，2 倍高清）
python scripts/render_cards.py --data cards.json --out-dir ./cards --theme coral --scale 2

# 小红书 / 视频号竖屏
python scripts/render_cards.py --data cards.json --out-dir ./cards --size 1080x1920 --theme ink

# 输出 JPG，并额外拼一张长图（需 Pillow）
python scripts/render_cards.py --data cards.json --out-dir ./cards --format jpg --merge
```

### 卡片数据格式

```json
{
  "theme": "coral",
  "brand": "你的署名",
  "size": "1080x1440",
  "cards": [
    {"type": "cover", "kicker": "AI 工作流", "title": "把一次 AI 对话\n变成能发的文章", "subtitle": "一句话，对话变稿子"},
    {"type": "points", "heading": "这次解决了什么", "items": [{"t": "短标题", "d": "一句说明"}]},
    {"type": "quote", "text": "真正省时间的不是 AI 帮你写，[[而是这套流程不用你再想一遍]]", "who": "—— 本次会话"},
    {"type": "checklist", "heading": "可复用的经验", "items": ["先确认素材源在哪，再谈自动化"]},
    {"type": "conclusion", "heading": "写在最后", "body": "一段话收口。", "highlight": "[[一句结论]]", "tags": ["#AI工作流"]}
  ]
}
```

文字字段支持极简富文本：`**加粗**`、`[[高亮]]`、`\n` 换行。

### 可选参数

| 参数 | 取值 | 说明 |
| --- | --- | --- |
| `--theme` | `coral` `ink` `tech` `mint` `sun` | 配色，默认珊瑚粉 |
| `--size` | `1080x1440` / `1080x1920` | 画布比例，默认 3:4 |
| `--scale` | 整数 | 输出倍率，2 = 2 倍高清 |
| `--format` | `png` / `jpg` | JPG 需 Pillow |
| `--merge` | 开关 | 额外输出一张纵向拼接长图 |
| `--brand` | 文本 | 卡片底部署名 |
| `--keep-html` | 开关 | 保留中间 HTML，便于手动微调版式 |

## 工作原理

```
会话记录 (.jsonl)
   │
   ▼
extract_session.py ──► transcript.md        过滤工具调用 / 思维链 / 系统注入，只留真实问答
   │
   ▼
五维提炼                                     核心问题 · 关键转折 · 结论方法 · 踩坑 · 可复用经验
   │
   ├──────────────► article.md               标题（含 2-3 备选）· 开篇 · 分节正文 · 划重点 · 互动
   │
   └──► cards.json ──► render_cards.py ──► card-01.png ...
                        HTML 模板 + 无头浏览器截图
```

几个关键设计：

1. **素材来自文件，不来自记忆**。模型上下文被压缩后仍能读到完整原始对话。
2. **指不回原话的内容一律删掉**。这条规则挡住了所有「为了让故事更顺而编一点」的冲动。
3. **贴图不用文生图**。中文在文生图里一定会糊、会缺笔画；HTML 渲染的字与屏幕上完全一致。
4. **版式是代码**。改配色只是改几个 CSS 变量，改比例只是一个参数。

## 文件结构

```
chat-to-content/
├── SKILL.md                      # 技能主文档：触发规则、前置检查、工作流、命名规范
├── references/
│   ├── extraction.md             # 五维提炼框架、脱敏规则、真实性校验清单
│   ├── article.md                # 文章结构、标题公式、语感要求
│   └── cards.md                  # 贴图排版规范、卡片字段、色板、套系设计
├── scripts/
│   ├── extract_session.py        # 会话记录定位与抽取
│   └── render_cards.py           # 卡片 JSON → PNG / JPG
├── assets/
│   └── card_template.html        # 卡片版式模板（内置溢出自动缩放）
├── README.md
└── LICENSE
```

## 环境要求

| 项目 | 要求 |
| --- | --- |
| Python | 3.7 及以上，只用标准库 |
| 系统 | Windows / macOS / Linux |
| 截图 | 系统装有 Chrome、Edge 或 Chromium；找不到时自动回退 Playwright |
| Pillow | 可选，仅 `--format jpg` 与 `--merge` 需要 |

Playwright 回退方案需要额外安装：

```bash
pip install playwright && playwright install chromium
```

## 常见问题

**Q：`--check` 返回 `ok: false`，但明明聊了很多。**
工作区目录被移动过导致 slug 对不上。用 `--list 10` 找到最近的会话，再用 `--session-file` 指定。

**Q：贴图上的字有点糊。**
检查是否用了 `--scale 1`（默认 2 倍）。另外不要改用文生图模型生成卡片，中文排版不可控。

**Q：卡片内容被裁切了。**
单卡条目过多。脚本会自动等比缩小，缩到 0.55 倍以下就说明该拆卡了，而不是继续压字号。

**Q：可以换成自己的品牌和配色吗？**
可以。`--brand` 改署名，`--theme` 换配色，或直接改 `scripts/render_cards.py` 里的 `THEMES` 字典新增一套。

**Q：适配哪些客户端？**
所有支持 `SKILL.md` + YAML frontmatter 的客户端。会话记录路径按需在 `extract_session.py` 的 `PROJECTS_ROOT` 处调整。

## 许可证

[Apache License 2.0](LICENSE)



