---
name: contacts-add
description: Add a new person to a dunbar relationship workspace from natural-language input. Resolves fields, dedupes against the CSV, picks a circle (or asks), creates the person folder + stub .md, appends a CSV row, runs sync. Strict rules — never fabricates fields, never writes "organization / partnership" facts until they're actually formed. Trigger words: /contacts-add, add contact, new contact, 加联系人, 加人, 新增联系人, 入档.
---

# contacts-add — add a new person to your dunbar workspace

User input: `/contacts-add $ARGUMENTS` where `$ARGUMENTS` is a natural-language description of one person.

## Where's my dunbar root?

Find it in this order, same as the `contacts-sync` skill:
1. `$DUNBAR_ROOT` environment variable
2. Current working directory if it has `contacts.csv` and at least one `0N_*` circle folder
3. Ask the user

## 1. Dedupe first

```bash
NAME="<nickname>"
grep -ih "^${NAME}," "$DUNBAR_ROOT/contacts.csv"
```

If a row is found → tell the user "X already exists in your contacts — update instead of add?" and wait for their call before proceeding.

## 2. Parse the input (only what's stated, never inferred)

Extract from `$ARGUMENTS` (leave blank if not stated):

| Field | Rule |
|-------|------|
| 昵称 | Required. Identify from the input (the noun phrase after "add", "new", etc.) |
| 真实姓名 | Only if the user said the full name. If only a nickname is given, **leave blank** |
| 公司 / 职位 / 行业 | Only if explicitly stated |
| 身份 | Whatever role the user described ("agent", "friend", "classmate") |
| 介绍人 | Only if the user said "introduced by X" |
| 电话 / 微信 / Email / Discord | Extract verbatim |
| LinkedIn | Full URL (`linkedin.com/in/<vanity>/`) |
| 社交媒体 | Handle or link for Twitter / IG / TikTok / 小红书 / etc. |
| 家庭地址 | Only if user said "lives in X" / "home is in X" |
| 办公地址 | Only if user said "works in X" / "office is in X" |
| 圈层 | Only if user named one; otherwise see §3 |
| 标签 | Keywords the user mentioned, semicolon-separated |

**Never fabricate** any field. If the user didn't say it, it stays blank. (This is the single most-violated rule when LLMs try to be "helpful" — resist.)

## 3. Pick a circle

- User named one → use it
- Default to **熟人** (`04_熟人_casual`) when in doubt — the safe fallback
- If context strongly hints (e.g. "my partner" → 核心; "we'll be working together regularly" → 常联系), suggest one and confirm
- When unsure, ask one short question rather than guess

## 4. Create the folder + stub MD

```bash
mkdir -p "$DUNBAR_ROOT/<NN_circle>/<nickname>/"
```

Write `<nickname>.md`, following the template (**do NOT** include `<!-- CSV-SYNC-BEGIN -->` block — sync.py adds it):

```markdown
# <nickname>

## Basics
- Public name: <nickname>
- (other fields: only what the user provided; leave omitted ones out — don't list "TBD" placeholders)

## How we know each other
<paraphrase of the relationship origin the user described — one short paragraph>
_(empty if the user didn't say)_

## Interactions
_(empty — fill in over time)_

## Shared items / follow-ups
_(empty — fill in over time)_
```

If the person is a creator / public figure / has notable online presence, you may add `## Content notes` and `## Public metrics` sections.

## 5. Append a CSV row

Use Python (not awk — quoted fields with commas will break):

```python
import csv, os
p = os.path.join(os.environ["DUNBAR_ROOT"], "contacts.csv")
with open(p, encoding="utf-8") as f:
    rows = list(csv.reader(f))
header = rows[0]
new_row = [""] * len(header)
new_row[header.index("昵称")] = "<nickname>"
# ... fill only the columns the user provided ...
rows.append(new_row)
with open(p, "w", encoding="utf-8", newline="") as f:
    csv.writer(f).writerows(rows)
```

**Leave 圈层 and 详档 blank in this step** — sync.py owns those (filesystem is the source of truth).

## 6. Run sync

```bash
python3 "$DUNBAR_ROOT/sync.py" --root "$DUNBAR_ROOT"
```

This will:
- Detect the new folder → fill in 圈层 + 详档
- Write the `<!-- CSV-SYNC-BEGIN -->` summary block at the top of the .md

## 7. Report

```
✅ Added <nickname> to contacts.csv
  Circle: <X>  ·  Profile: <circle>/<nickname>/<nickname>.md
  Filled: <comma-separated list of non-empty columns>

Empty (fill in later if you want):
  - <list of unfilled columns — informational, not nagging>
```

## 8. Don't ❌

- Don't invent any field value (introducer, circle, industry, birthday, etc.)
- Don't write "TBD", "to be confirmed", "🔴 risk", "verify this" sections in the MD body
- Don't write audit-style follow-ups about the person (e.g. "[ ] check whether their credentials are real")
- Don't bypass sync.py to hand-edit 圈层 / 详档 in the CSV
- Don't add the same person twice (step 1 dedupe is mandatory)
- When 真实姓名 == 昵称, leave 真实姓名 blank (don't copy)

## 9. ⛔ Hard rule: "Organization / partnership = only if it's actually happening"

Any description of an **organization, company, partnership, project, role, or title** can ONLY be written into the contacts profile **when that organization is actually operating and that partnership is actually under way**.

| State | Can it go in the contact's profile (公司 / 职位 / 身份 / 标签 / 备注)? |
|---|---|
| Registered company / signed contract / work in progress | ✅ yes |
| Salary paid / project delivered / money received | ✅ yes |
| "We're planning to set up" / MOU draft / "we intend to" | ❌ **no** |
| Prep meetings / framework conversations / on-hold / pending Asher's confirmation | ❌ **no** |
| Other-party self-claim with no actual output | ❌ **no** |

Violating this is the single biggest failure mode of contact-management LLMs. Even if the person introduces themselves as "CEO of X" — if X is a paper company and they've shipped nothing, **in the contact record they are just the person, not "CEO of X"**.

✅ Always safe to record:
- The person themselves (education, languages, address, visa, personality, what they self-describe as their background)
- Things that have actually happened between you (meeting dates, conversations)
- The person's standalone identity ("UNSW MEd student" is a fact about them regardless of any business deal)

❌ Don't record:
- "CEO of X Co" — unless X is a registered entity and they are a legal officer
- "Co-founder of Y project" — unless Y has actually started shipping
- "<your> partner / collaborator" — unless the partnership is concrete (signed / paid / actively executing)
- Speculative comp splits, unsigned terms, hypothetical roles

Project-prep artifacts (MOUs, org charts, external PDFs, draft frameworks) belong in a project workspace, not a contacts profile. Once the project actually exists, come back and fill in the 身份 / 公司 / 职位 columns.

## 10. Example

**User input:**
```
/contacts-add add Alice, my partner, lives in Sydney, alice@example.com
```

**The skill does:**
1. grep `contacts.csv` for `^Alice,` → not found → continue
2. Parse: 昵称=Alice, 身份=partner, 家庭地址=Sydney, Email=alice@example.com
3. Circle → "partner" strongly hints 核心 → confirm with user, then proceed
4. `mkdir -p $DUNBAR_ROOT/01_核心_core/Alice/`
5. Write `Alice.md` with Basics + How-we-know-each-other (empty, since user didn't elaborate) + empty Interactions + empty Shared
6. Append CSV row (21 cols, fill 昵称 / 身份 / 家庭地址 / Email, rest blank)
7. Run sync.py → fills in 圈层=核心, 详档=01_核心_core/Alice/Alice.md, prepends summary block
8. Report
