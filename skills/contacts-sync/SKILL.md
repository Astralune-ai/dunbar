---
name: contacts-sync
description: Sync circle folders ↔ contacts.csv for a dunbar relationship workspace. Run this after you drag a person's folder to a different circle (e.g. 05_弱联系_weak/X → 04_熟人_casual/X) or after creating a new person folder. It detects moves / new / orphan states and rewrites the summary block at the top of each person's .md profile. Trigger words: /contacts-sync, sync contacts, 同步联系人, 同步contacts, 圈层同步.
---

# contacts-sync — reconcile folders ↔ contacts.csv

Backed by `sync.py` in the dunbar root.

## Where's my dunbar root?

This skill works on whatever the user has set as their dunbar workspace. Look in this order:

1. `$DUNBAR_ROOT` environment variable
2. The current working directory if it contains `contacts.csv` AND at least one `0N_*` circle folder
3. Otherwise — ask the user where their dunbar workspace is

## User input parsing

`$ARGUMENTS` containing `dry`, `--dry-run`, or `preview` → run as a dry-run.
Otherwise → apply the changes.

## Execute

```bash
# Apply (default)
python3 "$DUNBAR_ROOT/sync.py" --root "$DUNBAR_ROOT"

# Or, from inside the workspace:
cd "$DUNBAR_ROOT" && python3 sync.py

# Dry-run preview
python3 "$DUNBAR_ROOT/sync.py" --root "$DUNBAR_ROOT" --dry-run
```

## Output meaning

| Marker | Means | Script behavior |
|--------|-------|-----------------|
| 🔄 Updates | A person's folder changed circle | Update CSV's 圈层 + 详档 columns |
| ✨ New | Folder exists in FS, no row in CSV | Append a new row (nickname + circle + profile only; other columns left blank) |
| ⚠️ Orphans | CSV row exists, folder missing | Warn only — does NOT auto-delete |
| 🗓️ Date normalized | Numbers/Excel mangled a date | Restored to ISO format |
| ⚠️ Lint | 真实姓名 == 昵称 | Suggest blanking 真实姓名 until full name is known |
| ✅ in sync | No diff | CSV untouched |

## When to run

- **After dragging a person folder to a different circle** — main use case
- **After creating a new person folder** (`mkdir <circle>/<nickname>/` + writing the .md)
- Periodic sanity check
- When the AI notices the user is doing relationship-management actions (e.g. adding several contacts in a row), proactively offer to run

## Data contract (important)

| Field | Source of truth | How to change |
|-------|-----------------|---------------|
| **圈层** | Filesystem (which circle directory the folder is in) | Move the folder + sync |
| **详档** | Filesystem (actual .md path) | sync computes it |
| **Row existence** | Filesystem (folder presence) | `mkdir` → sync appends; deletion is manual |
| **Other 19 columns** (phone/Email/tags/birthday/…) | The CSV itself | Edit `contacts.csv` directly (Numbers, your editor, a spreadsheet UI). sync never touches these. |

## Don't

- ❌ Don't bypass sync to hand-edit the 圈层 or 详档 columns — the next sync will overwrite them
- ❌ Don't `mv` just the `.md` file without moving the whole person folder — sync only looks at folder location
- ❌ Don't create person folders outside the circle directories — sync won't see them
