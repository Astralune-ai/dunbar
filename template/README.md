# template/ — start here

This folder is the **working area** for a Dunbar setup. Two ways to use it:

### Option A — fork-and-edit (recommended)
1. Fork this repo as a **template** on GitHub.
2. Clone your fork.
3. Replace the sample people (Alice / Bob / Carol / David / Eve) with your own.
4. `python ../sync.py --root .` to verify.

The root `.gitignore` already protects `contacts.csv` and your person folders
from being committed if you `git push` your fork — but read it once before
trusting it with your real data.

### Option B — use this as scratch / demo only
If you just want to see how sync works, leave the sample data alone and run
`python ../sync.py --root .` from this directory. Move David from `04_熟人_casual`
to `02_挚友_inner` and re-run — you'll see the move detected and CSV patched.

---

## What's in here

```
template/
├── contacts.csv                ← 21-column table; the source of truth for everything
│                                 EXCEPT which circle a person is in and the path to
│                                 their .md file (those are derived from folder layout)
├── 01_核心_core/Alice/Alice.md
├── 02_挚友_inner/Bob/Bob.md
├── 03_常联系_regular/Carol/Carol.md
├── 04_熟人_casual/David/David.md
├── 05_弱联系_weak/Eve/Eve.md
└── _PERSON_TEMPLATE.md         ← skeleton for new people
```

See `../docs/schema.md` for the full column reference, `../docs/dunbar.md` for the
circle philosophy.
