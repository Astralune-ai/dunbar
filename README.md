<div align="center">

# dunbar

### *Relationship management your AI can actually use.*

### *给 AI 用的人物关系系统。*

<br>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python: 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Dependencies: stdlib only](https://img.shields.io/badge/Dependencies-stdlib%20only-brightgreen.svg)](sync.py)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-9146FF.svg)](skills/)
[![File-first](https://img.shields.io/badge/File--first-No%20Cloud-orange.svg)](#privacy--隐私)

<br>

**Your phone has 800 contacts. You actually know maybe 150.**

**Your CRM is for customers. Your notes app is for thoughts. Neither is for the people in your life.**

**Your AI agent has no idea who anyone is.**

<br>

### `5 circles · 21 columns · 1 sync command · 0 dependencies · 0 cloud`

<br>

[**🇺🇸 English**](#-english) · [**🇨🇳 中文**](#-中文) · [**Quick start**](#5-minute-quick-start) · [**5 分钟上手**](#5-分钟上手) · [**Why**](#why-dunbar-exists)

</div>

---

# 🇺🇸 English

## Why dunbar exists

Three things break with conventional contact management:

> **1. Your contact list is flat.**
> Your dentist sits next to your best friend, alphabetically. Your AI has no idea which one matters more.

> **2. You have no "last seen" column.**
> So that friend you haven't talked to in 18 months — you still think you're close. You're not.

> **3. Your AI can't see context.**
> "Help me draft a message to Bob." Which Bob? About what? Based on what history? Your agent is flying blind.

dunbar fixes all three with **plain files and one Python script**. No CRM. No cloud. No vendor lock-in. Your data is grep-able, your AI is context-aware, and your relationships finally have a real container.

## What dunbar is

A 5-layer file system for the ~150 people you actually know:

```
your-dunbar/
├── contacts.csv                      ← 21 columns. The source of truth.
│
├── 01_核心_core/<nick>/<nick>.md     ← Core: 3–7 people. Your 3am-call list.
├── 02_挚友_inner/<nick>/<nick>.md    ← Inner: 10–20. Weekly contact.
├── 03_常联系_regular/<nick>/<nick>.md ← Regular: 30–60. Monthly contact.
├── 04_熟人_casual/<nick>/<nick>.md   ← Casual: 50–150. Yearly contact.
└── 05_弱联系_weak/<nick>/<nick>.md   ← Weak: long tail. Recognize-by-face.
```

- **Filesystem = circle assignment.** Move someone to a different circle? `mv` their folder.
- **CSV = facts.** Phone, email, birthday, last-seen. Edit it in Numbers, vim, anywhere.
- **Markdown = context.** Each person has a profile. Top of the file is an auto-generated summary block (so AI sees the facts at a glance). The rest is yours — background, preferences, what you talked about last.
- **`sync.py` reconciles all three.** One command. Stdlib only. No internet required.

## See it work

```console
$ cd template && python3 ../sync.py --root .

📂 FS:  5 people across 5 circles
📋 CSV: 5 rows

✨ New (appended 1 row):
  + Frank (弱联系) → 05_弱联系_weak/Frank/Frank.md
  ↳ only nickname/circle/profile filled — other columns left empty for you

🗓️  Date normalized (1) (un-mangled from Numbers/Excel):
  • Alice.生日: Apr-15 → 04-15

💾 CSV written: contacts.csv

📝 MD summary sync:
  ✨ first-time write: Frank
  🔁 rewrote block:    Bob, Carol
```

That's it. That's the whole workflow.

## 5-minute quick start

### Step 1. Get the repo

```bash
# Use GitHub's "Use this template" button (top of the repo page), or:
git clone git@github.com:Astralune-ai/dunbar.git
cd dunbar
```

### Step 2. Try the bundled template (fictional Alice / Bob / Carol / David / Eve)

```bash
cd template
python3 ../sync.py --root .
```

Expected: `✅ CSV / FS fully in sync.` Look at `01_核心_core/Alice/Alice.md` — see the auto-generated `<!-- CSV-SYNC-BEGIN -->` block at the top.

### Step 3. Move someone, re-sync

```bash
mv 04_熟人_casual/David 02_挚友_inner/David
python3 ../sync.py --root .
```

Expected: `🔄 Updates (1): David: circle 熟人 → 挚友`. The CSV's 圈层 + 详档 columns updated themselves. The summary block at the top of David's `.md` got rewritten.

### Step 4. Make it yours

```bash
cd ..   # back to repo root

# Start a fresh workspace alongside the template (the .gitignore protects root-level data)
cp template/contacts.csv ./contacts.csv
# Trim the 5 sample rows from this new contacts.csv (keep just the header line)

mkdir -p 01_核心_core 02_挚友_inner 03_常联系_regular 04_熟人_casual 05_弱联系_weak

# Add your first person
NICK="Alex"
mkdir -p "01_核心_core/$NICK"
cp template/_PERSON_TEMPLATE.md "01_核心_core/$NICK/$NICK.md"
sed -i '' "s/<Nickname>/$NICK/g" "01_核心_core/$NICK/$NICK.md"   # macOS sed
# (Linux: sed -i "s/<Nickname>/$NICK/g" "01_核心_core/$NICK/$NICK.md")

python3 sync.py
```

Expected: `✨ New (appended 1 row): + Alex (核心) → ...`

### Step 5. Hook up your AI

#### Claude Code (the path of least resistance)

```bash
# Install the three skills
mkdir -p ~/.claude/skills
cp -r skills/contacts-sync skills/contacts-add skills/contacts-event ~/.claude/skills/

# Tell the skills where your workspace lives
echo 'export DUNBAR_ROOT="'"$PWD"'"' >> ~/.zshrc
source ~/.zshrc
```

Now in any session you can say:

- *"Add Alice to my contacts, she's my partner, lives in Sydney"* → triggers `contacts-add`
- *"I just moved David to inner circle, sync"* → triggers `contacts-sync`
- *"Log the mess where Sam's flatmate supposedly extorted money from him"* → triggers `contacts-event`
- *"Who do I know in Melbourne?"* → grep over CSV, no skill needed

#### Cursor / Codex / generic agent

Drop this in your system prompt or project rules:

```
You have access to a dunbar relationship workspace at $DUNBAR_ROOT.
- contacts.csv is the source of truth (21 columns, Chinese headers)
- Circles are filesystem directories (01_核心_core … 05_弱联系_weak)
- Each person has a .md profile at <circle>/<nickname>/<nickname>.md
- Run python3 $DUNBAR_ROOT/sync.py after any folder change
- See docs/schema.md for column reference
- See skills/contacts-add/SKILL.md for the strict "don't fabricate" rules
```

See [docs/ai-integration.md](docs/ai-integration.md) for more.

## Use cases

| You want… | dunbar gives you… |
|---|---|
| **Prep for a meeting** | Read `<their-name>/<their-name>.md`. The summary block on top + your notes below = a 30-second briefing. |
| **"Who do I know who does X?"** | `grep -i "X" contacts.csv`. Or have your AI do it. |
| **Track relationship health** | Sort `contacts.csv` by 上次见面. The bottom rows are the people drifting. |
| **Capture after a conversation** | "Hey AI, log what I just talked about with Bob" → appends to Bob's `## Interactions`. |
| **Move someone to a different circle** | `mv 04_熟人_casual/X 02_挚友_inner/X && python3 sync.py`. Done. |
| **Add a new person from natural language** | `/contacts-add add Alex, my partner, lives in Sydney` |
| **Survive a job change / app change** | Everything is plain text files. Take them with you. |

## Schema cheatsheet

21 columns in `contacts.csv`. Headers are in Chinese — short, unambiguous, stable.

| # | Name | What it is |
|--:|---|---|
| 1 | 昵称 | **Required.** The name you use for them. Used as the folder name. |
| 2 | 真实姓名 | Full legal name. Leave blank if you only know a nickname. |
| 3 | 生日 | `YYYY-MM-DD` or `MM-DD`. |
| 4–7 | 公司 / 职位 / 行业 / 身份 | Current company, title, industry, your-relationship-to-them. |
| 8 | 介绍人 | Who introduced you. |
| 9–14 | 电话 / 微信 / Email / LinkedIn / Discord / 社交媒体 | Channels. |
| 15–16 | 家庭地址 / 办公地址 | Mobile-tap-to-navigate friendly. |
| 17 | 圈层 | **Auto-computed from folder location. Don't hand-edit.** |
| 18 | 上次见面 | Last meaningful interaction date. |
| 19 | 标签 | Semicolon-separated free-form tags. |
| 20 | 详档 | **Auto-computed: path to the .md profile.** |
| 21 | 备注 | Free-form notes. |

Full reference: [docs/schema.md](docs/schema.md). Column names, circle names, the CSV filename and the shape of the generated summary block are all configurable from a single `dunbar.config.json` — no code edits: see [docs/schema.md#configuration](docs/schema.md#configuration).

## What dunbar is NOT

- ❌ **Not a CRM.** No deal pipelines, no opportunity tracking. dunbar is for people in your life, not customers in your funnel.
- ❌ **Not an import sink.** Don't dump 5,000 LinkedIn connections in here. Putting someone in dunbar should be a conscious "yes, this person is in my life" act.
- ❌ **Not multi-user.** Your relationships are yours. A "team contacts" tool is a different product.
- ❌ **Not a web app.** Your editor, your terminal, your AI are the UI.
- ❌ **Not a friendship-grader.** Nothing in here computes a "closeness score". Inviting AI to grade your friendships is a category error.

## Privacy / 隐私

- `contacts.csv` and root-level `01_…` / `02_…` / etc. folders are **gitignored**. Your real data stays local even if you `git push` your fork.
- No telemetry. No cloud. No shared backend. dunbar is files on your disk.
- The template/ people are fictional. Phone numbers use the `+1-555-01xx` reserved-for-fiction prefix.
- If you ever accidentally commit your real `contacts.csv` to a public repo: rotate any leaked credentials immediately, then use [`git-filter-repo`](https://github.com/newren/git-filter-repo) or [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) to scrub git history (a regular `git rm` won't remove it from past commits).

## Roadmap

- **v1.0** (this release) — Local. Stdlib-only. Bilingual.
- **v1.1** — Optional one-time import scripts (vCard, LinkedIn, 小红书). Bring-your-own API key.
- **v2.0** — Google Workspace integration. Google Contacts ↔ dunbar sync; Calendar-driven `上次见面` auto-update; Gmail-driven recent-thread digests appended to per-person profiles.

See [docs/roadmap.md](docs/roadmap.md) for the design sketch — and for what we're deliberately **not** building.

## Skills

| Skill | What it does |
|---|---|
| [`contacts-sync`](skills/contacts-sync/SKILL.md) | Reconcile folders ↔ CSV; rewrite per-person summary blocks. |
| [`contacts-add`](skills/contacts-add/SKILL.md) | Add a new person from natural language. Includes strict "don't fabricate" + "organization = only if it's actually happening" hard rules. |
| [`contacts-event`](skills/contacts-event/SKILL.md) | Log a complex, multi-party, evolving situation around a contact (a dispute, a crisis, a "two sides tell opposite stories" tangle) as a layered event dossier + chronology, kept separate from the clean profile. Five-layer tagging (firsthand / secondhand / hypothesis / commentary / to-confirm) keeps fact and inference from bleeding together. |

All three are plain markdown — drop them in Claude Code (`~/.claude/skills/`), paste them into Cursor project rules, or pipe them into any agent's system prompt.

## Contributing

PRs welcome. High-value contributions:

- **Translations** of `docs/schema.md` / `docs/dunbar.md` to other languages
- **`import_from_*.py` adapters** for v1.1 (vCard / LinkedIn / 小红书) — bring your own API client
- **Better summary block formats** in `build_summary()` — open an issue first to discuss the shape

Keep it dependency-free if possible. Check the [roadmap](docs/roadmap.md) `What's deliberately NOT on the roadmap` list before proposing big features.

## License

[MIT](LICENSE). Do whatever, no warranty.

---

<a id="-中文"></a>

# 🇨🇳 中文

## 为什么要有 dunbar

传统通讯录有三个根本问题：

> **1. 它是平的。**
> 你的牙医和你的挚友按字母顺序紧挨着。AI 不知道谁更重要。

> **2. 它没有"上次见"这一列。**
> 所以 18 个月没见的朋友，你以为你们还熟。你们不熟了。

> **3. AI 看不到上下文。**
> "帮我给 Bob 写条消息。"哪个 Bob？聊什么？上次说到哪了？AI 全是空白。

dunbar 用**纯文本文件 + 一个 Python 脚本**解决这三件事。不是 CRM。不上云。不锁平台。数据可 grep，AI 有上下文，你的人脉关系终于有了一个像样的容器。

## dunbar 是什么

为你**真实认识的 ~150 人**准备的 5 层文件系统：

```
你的-dunbar/
├── contacts.csv                      ← 21 列。事实真源。
│
├── 01_核心_core/<昵称>/<昵称>.md     ← 核心：3–7 人。3 点钟你能打电话的人。
├── 02_挚友_inner/<昵称>/<昵称>.md    ← 挚友：10–20 人。每周联系。
├── 03_常联系_regular/<昵称>/<昵称>.md ← 常联系：30–60 人。每月联系。
├── 04_熟人_casual/<昵称>/<昵称>.md   ← 熟人：50–150 人。每年联系。
└── 05_弱联系_weak/<昵称>/<昵称>.md   ← 弱联系：长尾。脸熟而已。
```

- **文件夹位置 = 圈层归属。** 想把某人挪到别的圈层？`mv` 他的文件夹就行。
- **CSV = 事实。** 电话、邮箱、生日、上次见面。在 Numbers / vim / 任何地方都能改。
- **Markdown = 上下文。** 每人一份长档。文件顶部是机器自动生成的摘要块（AI 一眼看完事实），下面是你自己写的——背景、偏好、上次聊了什么。
- **`sync.py` 把三者保持一致。** 一条命令。零依赖。不需要联网。

## 跑一下看看

```console
$ cd template && python3 ../sync.py --root .

📂 FS:  5 people across 5 circles
📋 CSV: 5 rows

✨ New (appended 1 row):
  + Frank (弱联系) → 05_弱联系_weak/Frank/Frank.md
  ↳ only nickname/circle/profile filled — other columns left empty for you

🗓️  Date normalized (1) (un-mangled from Numbers/Excel):
  • Alice.生日: Apr-15 → 04-15

💾 CSV written: contacts.csv

📝 MD summary sync:
  ✨ first-time write: Frank
  🔁 rewrote block:    Bob, Carol
```

就这。这就是全部工作流。

## 5 分钟上手

### 第 1 步：拿到仓库

```bash
# 在 GitHub 仓库页用 "Use this template" 按钮，或者：
git clone git@github.com:Astralune-ai/dunbar.git
cd dunbar
```

### 第 2 步：跑一下自带的模板（虚构的 Alice / Bob / Carol / David / Eve）

```bash
cd template
python3 ../sync.py --root .
```

应该看到：`✅ CSV / FS fully in sync.` 打开 `01_核心_core/Alice/Alice.md`，看顶部那段自动生成的 `<!-- CSV-SYNC-BEGIN -->` 块。

### 第 3 步：挪一个人，再同步

```bash
mv 04_熟人_casual/David 02_挚友_inner/David
python3 ../sync.py --root .
```

应该看到：`🔄 Updates (1): David: circle 熟人 → 挚友`。CSV 的「圈层」+「详档」列自己更新了，David 长档顶部的摘要块也重写了。

### 第 4 步：开始你自己的工作区

```bash
cd ..   # 回到仓库根目录

# 在根目录建你自己的工作区（.gitignore 已自动保护根目录的数据不被 commit）
cp template/contacts.csv ./contacts.csv
# 把这个新 contacts.csv 里的 5 行示例数据删掉，只保留 header

mkdir -p 01_核心_core 02_挚友_inner 03_常联系_regular 04_熟人_casual 05_弱联系_weak

# 加你的第一个人
NICK="李雷"
mkdir -p "01_核心_core/$NICK"
cp template/_PERSON_TEMPLATE.md "01_核心_core/$NICK/$NICK.md"
sed -i '' "s/<Nickname>/$NICK/g" "01_核心_core/$NICK/$NICK.md"   # macOS sed
# (Linux: sed -i "s/<Nickname>/$NICK/g" "01_核心_core/$NICK/$NICK.md")

python3 sync.py
```

应该看到：`✨ New (appended 1 row): + 李雷 (核心) → ...`

### 第 5 步：接上你的 AI

#### Claude Code（最省事的方式）

```bash
# 安装三个 skill
mkdir -p ~/.claude/skills
cp -r skills/contacts-sync skills/contacts-add skills/contacts-event ~/.claude/skills/

# 告诉 skill 你的工作区在哪
echo 'export DUNBAR_ROOT="'"$PWD"'"' >> ~/.zshrc
source ~/.zshrc
```

之后任何会话里都可以说：

- *"加 Alice 到联系人，她是我女朋友，住悉尼"* → 触发 `contacts-add`
- *"David 挪到挚友圈了，同步一下"* → 触发 `contacts-sync`
- *"把 Sam 跟室友那摊钱的破事记一下"* → 触发 `contacts-event`
- *"我认识的人里有谁在墨尔本？"* → AI 直接 grep CSV，不需要 skill

#### Cursor / Codex / 通用 agent

往 system prompt 或项目规则里塞这段：

```
你可以读用户的 dunbar 关系工作区，位置在 $DUNBAR_ROOT。
- contacts.csv 是事实真源（21 列，中文表头）
- 圈层 = 文件夹（01_核心_core 到 05_弱联系_weak）
- 每个人有 .md 长档在 <圈层>/<昵称>/<昵称>.md
- 用户改了文件夹位置后跑 python3 $DUNBAR_ROOT/sync.py
- 列的详细含义看 docs/schema.md
- skills/contacts-add/SKILL.md 里有"不准编造字段"的硬规则，照做
```

更多细节看 [docs/ai-integration.md](docs/ai-integration.md)。

## 使用场景

| 你想做的事 | dunbar 怎么帮你 |
|---|---|
| **见客户前准备** | 打开 `<对方昵称>/<对方昵称>.md`。顶部摘要 + 下面你的笔记 = 30 秒 briefing。 |
| **"我认识谁是做 X 的？"** | `grep -i "X" contacts.csv`，或者让 AI 替你查。 |
| **盘点关系健康度** | 按「上次见面」排序 `contacts.csv`。底部的几行就是你在疏远的人。 |
| **聊完天后立刻记下** | "AI 帮我把刚跟 Bob 聊的东西记下" → 自动 append 到 Bob 长档的 `## Interactions` 节。 |
| **把某人挪到另一个圈层** | `mv 04_熟人_casual/X 02_挚友_inner/X && python3 sync.py`。完事。 |
| **用自然语言加联系人** | `/contacts-add 加 李雷, 我女朋友, 住悉尼` |
| **跳槽 / 换 app 都不丢人脉** | 纯文本文件。换工作时直接带走。 |

## 数据结构速查

`contacts.csv` 21 列。表头是中文——简短、不歧义、稳定。

| # | 列名 | 含义 |
|--:|---|---|
| 1 | 昵称 | **必填。** 你怎么叫他。文件夹名也用这个。 |
| 2 | 真实姓名 | 全名。只知道昵称就**留空**（不要复制昵称）。 |
| 3 | 生日 | `YYYY-MM-DD` 或 `MM-DD`。 |
| 4–7 | 公司 / 职位 / 行业 / 身份 | 当前公司、职位、行业、他跟你的关系（朋友/同事/伴侣...）。 |
| 8 | 介绍人 | 谁介绍你们认识的。 |
| 9–14 | 电话 / 微信 / Email / LinkedIn / Discord / 社交媒体 | 联系方式。 |
| 15–16 | 家庭地址 / 办公地址 | 手机点击能直接导航。 |
| 17 | 圈层 | **由 sync.py 从文件夹位置算。不要手改。** |
| 18 | 上次见面 | 最近一次有意义的接触日期。 |
| 19 | 标签 | 分号分隔的自由 tag。 |
| 20 | 详档 | **sync.py 自动算的 .md 路径。** |
| 21 | 备注 | 杂项。 |

完整参考：[docs/schema.md](docs/schema.md)。列名、圈层名、CSV 文件名、生成的摘要块长什么样，全部由一个 `dunbar.config.json` 配置，不用改代码：[docs/schema.md#configuration](docs/schema.md#configuration)。

## dunbar 不是什么

- ❌ **不是 CRM。** 不做销售管道、商机跟进。dunbar 是给你生活里的人用的，不是你漏斗里的客户。
- ❌ **不是导入垃圾桶。** 别把 LinkedIn 5000 个连接全塞进来。把人放进 dunbar 应该是"是的这个人在我生活里"的有意识动作。
- ❌ **不做多人协作。** 关系是你自己的，不是团队的。
- ❌ **不是 Web App。** 你的编辑器、终端、AI 就是 UI。
- ❌ **不评分友情。** 这里不算什么"亲密度分数"。让 AI 给你的朋友打分是范畴错误。

## 隐私

- 根目录的 `contacts.csv` 和 `01_…` / `02_…` 等圈层文件夹**默认 gitignore**。即使你 `git push` 你的 fork，真数据也只在本地。
- 没有埋点。不上云。没有共享后端。dunbar 就是你硬盘上的几个文件。
- `template/` 里的人都是虚构的。电话用 `+1-555-01xx`（小说虚构专用前缀）。
- 如果你**不小心**把真实 `contacts.csv` 推到公开仓了：立即吊销泄露的凭据，然后用 [`git-filter-repo`](https://github.com/newren/git-filter-repo) 或 [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) 清 git 历史（普通 `git rm` 只删当前，不删历史）。

## 路线图

- **v1.0**（本版本）— 纯本地。零依赖。中英双语。
- **v1.1** — 一次性导入脚本（vCard / LinkedIn / 小红书），自带 API key。
- **v2.0** — Google Workspace 集成。Google Contacts ↔ dunbar 双向同步；Calendar 自动算「上次见面」；Gmail 最近往来邮件摘要自动 append 到对应长档。

设计草案：[docs/roadmap.md](docs/roadmap.md)。里面也写了我们**故意不做**的东西。

## 协议

[MIT](LICENSE)。随便用，不负责。

---

<div align="center">

<br>

**Stop letting your contact list rot.**
**Stop pretending your AI knows who anyone is.**
**Put the 150 people who matter in a place your tools can see.**

<br>

![dunbar](https://img.shields.io/badge/dunbar-150%20people%2C%20one%20repo-black?style=for-the-badge)

Powered by [**Astralune**](https://github.com/Astralune-ai)

</div>
