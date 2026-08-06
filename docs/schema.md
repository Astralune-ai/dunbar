# Schema — contacts.csv (21 columns)

The CSV is the source of truth for **every column except 圈层 and 详档** (those two
are derived from filesystem layout by `sync.py`).

Column names default to Chinese — they're short and unambiguous, and changing the
default would break anyone who has hand-edited rows in Numbers/Excel expecting
those headers. If you want English equivalents, or an entirely different column
set, rename them in `dunbar.config.json` — see [Configuration](#configuration)
below. No code edits needed.

## Columns

| # | Name | Type | Required | Notes |
|---|------|------|----------|-------|
| 1 | 昵称 | string | ✅ | The nickname / public name you use for them. Used as the folder name. Must be unique across the whole workspace. |
| 2 | 真实姓名 | string |  | Full legal name. **Leave blank** if you only know a nickname (don't copy 昵称 here — `sync.py` will warn you). |
| 3 | 生日 | `YYYY-MM-DD` or `MM-DD` |  | Both formats are accepted. `MM-DD` is for when you know the day but not the year. |
| 4 | 公司 | string |  | Current employer / company. Only fill if it's actually operational — see [contacts-add §9](../skills/contacts-add/SKILL.md#9--hard-rule-organization--partnership--only-if-its-actually-happening). |
| 5 | 职位 | string |  | Current title. Same "actually operating" rule. |
| 6 | 行业 | string |  | Free-form industry tag (e.g. "Software", "Finance", "Higher Ed"). |
| 7 | 身份 | string |  | Your relationship to them — "partner", "friend", "colleague", "agent". |
| 8 | 介绍人 | string |  | Who introduced you, if anyone. Use the same nickname format as column 1. |
| 9 | 电话 | string |  | Phone number with country code (+61, +1, +86, ...). |
| 10 | 微信 | string |  | WeChat ID. |
| 11 | Email | string |  | Primary email address. |
| 12 | LinkedIn | URL |  | Full URL — `https://www.linkedin.com/in/<vanity>/`. |
| 13 | Discord | string |  | Discord ID or `username#1234`. |
| 14 | 社交媒体 | string |  | Other socials — Twitter / IG / TikTok / 小红书 handle. Multiple → semicolon-separated. |
| 15 | 家庭地址 | string |  | Home address. Can be just a city if that's all you know. |
| 16 | 办公地址 | string |  | Office address. |
| 17 | 圈层 | `核心` / `挚友` / `常联系` / `熟人` / `弱联系` | ✅ (auto) | **Computed by sync.py from folder location.** Don't hand-edit. |
| 18 | 上次见面 | `YYYY-MM-DD` |  | Last meaningful interaction. The summary block prefers this over 上次联系 (legacy name). |
| 19 | 标签 | string |  | Semicolon-separated tags, e.g. `tech;Sydney;sounding-board`. |
| 20 | 详档 | path | ✅ (auto) | **Computed by sync.py.** Path to the person's .md file, relative to the dunbar root. |
| 21 | 备注 | string |  | Free-form notes that don't fit elsewhere. |

## Data contract

```
                    ┌────────────────────────────────┐
                    │ filesystem (circle folders)    │
                    │ owns:  圈层 + 详档 + row exist │
                    └──────────────┬─────────────────┘
                                   │ sync.py reconciles
                    ┌──────────────▼─────────────────┐
                    │ contacts.csv                   │
                    │ owns:  every other column      │
                    └──────────────┬─────────────────┘
                                   │ sync.py rewrites
                    ┌──────────────▼─────────────────┐
                    │ <Person>.md top: CSV-SYNC block│
                    │ everything else: yours         │
                    └────────────────────────────────┘
```

## Field rules to know

- **Dates that Numbers/Excel mangled.** Numbers will silently turn `2026-05-23` into `23/5/2026` (Australian) or `04-15` into `Apr-15`. `sync.py` detects and restores these. You can edit dates in whatever format your spreadsheet app uses — they'll be normalized on the next sync.
- **真实姓名 lint.** If you accidentally fill 真实姓名 with the nickname, `sync.py` will warn you on every run. Blank it out — that field is for "name they go by in formal contexts you don't want to forget", not a duplicate of 昵称.
- **Excel's BOM.** Excel on Windows saves CSVs with a byte-order mark. `sync.py` reads with `utf-8-sig` so it doesn't matter, and writes back in whatever the file already used. Anything else you point at the CSV must do the same — a plain `utf-8` read turns the first column name into `﻿昵称` and every lookup fails.
- **The summary block format.** Declared by the `summary` list in `dunbar.config.json` — see below. With no config file you get the default shape: 圈层 + 上次见面 + identity (真实姓名 / 身份 / 公司 / 职位) + one contact channel + addresses + 标签.

## Configuration

Everything below lives in an optional `dunbar.config.json` in your root. **With no
config file at all, you get the defaults documented above** — the 5 Dunbar circles
and the Chinese column names. Nothing needs configuring to start.

A full example, showing every key. All of them are independent; set only what you
want to change.

```json
{
  "csv": "contacts.csv",
  "noun": "circle",

  "circles": {
    "01_inner": "inner",
    "02_close": "close",
    "03_warm":  "warm",
    "04_known": "known",
    "05_weak":  "weak"
  },

  "columns": {
    "nickname": "昵称",
    "circle":   "圈层",
    "profile":  "详档"
  },

  "dates": { "上次见面": "full", "生日": "monthday" },

  "lint_duplicate_of_nickname": "真实姓名",

  "summary": [
    { "col": "圈层", "bold": true },
    { "col": "上次见面", "fallback": ["上次联系"], "label": "上次见 " },
    { "col": "真实姓名", "skip_if_equals": "nickname" },
    { "join": ["公司", "职位"] },
    { "any": ["Email", "电话", "微信"] },
    { "col": "标签", "label": "标签: " }
  ]
}
```

| Key | Does |
|-----|------|
| `csv` | The CSV filename. Default `contacts.csv` |
| `noun` | What a circle is called in the console output — `circle`, `tier`, `group`. Cosmetic only |
| `circles` | Folder name → the label written into the circle column. Keys must match the folders on disk |
| `columns` | Renames the three structural columns. Every other column is free-form — add, remove or rename them in the CSV header and `sync.py` preserves them untouched |
| `dates` | Which columns get un-mangled after Excel localises them. `"full"` for `YYYY-MM-DD`, `"monthday"` to also accept `MM-DD` |
| `lint_duplicate_of_nickname` | Which column to warn about when it just repeats the key. `null` disables the lint |
| `summary` | The generated block, left to right. Empty values are skipped |

### summary entries

| Entry | Renders |
|-------|---------|
| `{"col": "X"}` | one column |
| `{"col": "X", "fallback": ["Y"]}` | X, or Y when X is empty |
| `{"any": ["A","B","C"]}` | the first one with a value |
| `{"join": ["A","B"]}` | both, space-joined |
| `"label": "ABN "` | a text prefix |
| `"bold": true` | wraps in `**` |
| `"skip_if_equals": "nickname"` | dropped when it only repeats the key |

Only `nickname`, `circle` and `profile` are structural. Everything else — how many
columns there are, what they're called, what language they're in — is yours.

### Worked example: a different domain entirely

The same engine runs an accounting practice's client list — five client tiers
instead of social circles, 23 English columns, `clients.csv` instead of
`contacts.csv` — with no code changes, only this file:

```json
{
  "csv": "clients.csv",
  "noun": "tier",
  "circles": {
    "01_key_clients": "Key",
    "02_active":      "Active",
    "03_seasonal":    "Seasonal",
    "04_prospects":   "Prospect",
    "05_dormant":     "Dormant"
  },
  "columns": { "nickname": "Client", "circle": "Tier", "profile": "Profile" },
  "dates": { "Last Contact": "full", "Next Due": "full" },
  "lint_duplicate_of_nickname": "Legal Name",
  "summary": [
    { "col": "Tier", "bold": true },
    { "col": "Legal Name", "skip_if_equals": "nickname" },
    { "col": "ABN", "label": "ABN " },
    { "any": ["Email", "Phone", "WeChat"] },
    { "col": "Next Due", "label": "⏳ next due " }
  ]
}
```
