<div align="center">

# dunbar

**Relationship management your AI can actually use.**

5 Dunbar circles · CSV-as-source-of-truth · Filesystem-as-structure · Markdown-per-person · AI-native

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python: 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Dependencies: stdlib only](https://img.shields.io/badge/Dependencies-stdlib%20only-brightgreen.svg)](sync.py)
[![Made with Claude Code](https://img.shields.io/badge/Made%20with-Claude%20Code-9146FF.svg)](https://claude.com/claude-code)

</div>

---

> **TL;DR.** A contact list isn't a relationship system. dunbar is a 5-layer,
> file-first scaffold for the ~150 people you actually know — with a sync
> script that keeps the CSV, your folder layout, and your per-person notes in
> agreement, and two skills so Claude Code / Cursor / Codex can manage it
> for you.
>
> **TL;DR · 中文。** 通讯录不是关系系统。dunbar 是按邓巴数 5 层切分的"你真实认识的 ~150 人"的本地档案系统：CSV、文件夹结构、每人 markdown 长档自动三方同步，附带两个 skill 让 Claude Code 等 AI agent 帮你管。

## Why

The default tools for tracking people you know are bad:

- **Phone contacts** are flat. Your dentist and your best friend sit next to each other alphabetically.
- **CRMs** are for customers — wrong vocabulary, wrong cadence, wrong defaults.
- **Notion / Airtable databases** drift the moment you stop maintaining them, and your AI can't read them without per-tool integrations.

dunbar is the opposite: **plain text, plain folders, plain CSV.** Any AI agent with file access is useful against it immediately, and your data outlives any tool.

## What it is

```
your-dunbar/
├── contacts.csv                      ← 21 columns; CSV is the source of truth
│
├── 01_核心_core/<nick>/<nick>.md     ← 5 circles, sized for real-life maintenance
├── 02_挚友_inner/<nick>/<nick>.md       (core / inner / regular / casual / weak)
├── 03_常联系_regular/<nick>/<nick>.md
├── 04_熟人_casual/<nick>/<nick>.md
└── 05_弱联系_weak/<nick>/<nick>.md   ← filesystem layout = circle assignment
```

Each person's `.md` file has a **machine-generated summary block** at the top (one-line at-a-glance: circle / last seen / identity / contact / address / tags) and **your hand-written notes** below (background, interactions, what they care about).

`sync.py` keeps all three layers — CSV ↔ folders ↔ summary blocks — in agreement. Move a person folder to a different circle? Run sync. Add a row to the CSV? Run sync. Numbers/Excel mangled your dates? Sync detects and restores.

## Quick start

### Option A — fork as template (recommended)

```bash
# 1. Fork this repo on GitHub (use the "Use this template" button), then:
git clone git@github.com:<you>/<your-fork>.git
cd <your-fork>

# 2. Verify the bundled sample works
cd template
python3 ../sync.py --root .
# → expect: ✅ CSV / FS fully in sync.

# 3. Start your own workspace alongside the template
cd ..
cp template/contacts.csv ./contacts.csv   # 21-column header + 0 rows for you to grow
mkdir -p 01_核心_core 02_挚友_inner 03_常联系_regular 04_熟人_casual 05_弱联系_weak
# Add a person:
mkdir -p 01_核心_core/Alice && echo "# Alice" > 01_核心_core/Alice/Alice.md
python3 sync.py
```

`.gitignore` is preconfigured to **never commit** root-level `contacts.csv` or root-level `01_…` / `02_…` / etc. folders. The `template/` tree (fictional Alice / Bob / Carol / David / Eve) stays tracked so the repo remains usable as a template.

### Option B — already on Claude Code?

Install the two skills:

```bash
mkdir -p ~/.claude/skills
cp -r skills/contacts-sync skills/contacts-add ~/.claude/skills/

# Tell the skills where your workspace is:
echo 'export DUNBAR_ROOT="$HOME/path/to/your/dunbar"' >> ~/.zshrc
source ~/.zshrc
```

Then in any session:

- `add Alice to my contacts, she's my partner, lives in Sydney` → `contacts-add` runs
- `sync contacts` → `contacts-sync` runs
- `who do I know in Melbourne?` → Claude greps the CSV directly

See [docs/ai-integration.md](docs/ai-integration.md) for Cursor / Codex / generic agent setup.

## The 5 circles

| Folder | Label | Who lives here | ~Size |
|---|---|---|---:|
| `01_核心_core` | 核心 | partner, immediate family, your 3am-call people | 3–7 |
| `02_挚友_inner` | 挚友 | best friends, the people you talk to weekly | 10–20 |
| `03_常联系_regular` | 常联系 | regular collaborators, work confidants, extended family | 30–60 |
| `04_熟人_casual` | 熟人 | people you see/message a few times a year | 50–150 |
| `05_弱联系_weak` | 弱联系 | long-tail — old classmates, one-off introductions | 100–500 |

You move people between circles by moving their folder. That's it.

See [docs/dunbar.md](docs/dunbar.md) for the philosophy (why 5, why these sizes, when to promote / demote).

## The data contract (the one thing to internalize)

```
filesystem owns:  圈层 (circle) + 详档 (profile path) + whether a row exists
contacts.csv owns: every other column (phone, email, birthday, last-seen, tags…)
sync.py reconciles, and rewrites the summary block at the top of each .md
```

- Want to put someone in a different circle? `mv` their folder. Don't edit the CSV.
- Want to update their phone number? Edit the CSV. Don't touch the summary block.
- Want to log what you talked about? Write it under `## Interactions` in their `.md`. sync never touches that.

See [docs/schema.md](docs/schema.md) for the full 21-column reference.

## What this is NOT

- ❌ Not a CRM. No pipelines, opportunities, campaigns.
- ❌ Not a sink for "import 5,000 LinkedIn connections". Putting someone in dunbar should be a conscious "yes, this person is in my life" act.
- ❌ Not multi-user. Your relationships are yours.
- ❌ Not a web app. Your editor, your terminal, your AI agent are the UI.

## Privacy

- `contacts.csv` and `01_…/02_…/…/05_…` folders are **gitignored at root** — your real data stays local even if you `git push` your fork.
- No telemetry, no cloud, no shared backend. dunbar is files on your disk.
- The `template/` people (Alice, Bob, Carol, David, Eve) are fictional with `+1-555-01xx` reserved-for-fiction phone numbers.

If you accidentally commit your real `contacts.csv`, **rotate the leaked credentials, then use [`git-filter-repo`](https://github.com/newren/git-filter-repo) or [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) to scrub the history**. (A regular `git rm` won't remove it from past commits.)

## Roadmap

- **v1.0** (this release) — local, dependency-free, English + Chinese
- **v1.1** — optional one-time import scripts (vCard, LinkedIn, 小红书 — BYO API key)
- **v2.0** — Google Workspace integration (Contacts pull/push, Calendar-driven last-seen, Gmail-driven recent-thread digests)

See [docs/roadmap.md](docs/roadmap.md) for details and what's deliberately **not** planned.

## Skills

| Skill | Purpose |
|---|---|
| [`contacts-sync`](skills/contacts-sync/SKILL.md) | Reconcile folders ↔ CSV; rewrite per-person summary blocks |
| [`contacts-add`](skills/contacts-add/SKILL.md) | Add a new person from natural-language input. Includes strict "don't fabricate" + "no speculative organizations" rules. |

Both are plain markdown — they work in Claude Code, copy verbatim into Cursor project rules, and can be pasted into any agent's system prompt.

## Contributing

PRs welcome, especially:

- Translations of `schema.md` / `dunbar.md` column-and-circle vocabulary
- Additional `import_from_*.py` adapters (v1.1)
- Better summary block formats in `build_summary()` — open an issue first to discuss

For new features: keep it dependency-free if possible, and check it against the [roadmap](docs/roadmap.md) `What's deliberately NOT on the roadmap` list.

## License

[MIT](LICENSE) — do whatever, no warranty.

---

<div align="center">

Powered by [**Astralune**](https://github.com/Astralune-ai)

</div>
