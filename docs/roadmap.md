# Roadmap

## v1.0 — local-only (this release)

- `sync.py` with `--root` and configurable circles
- 5-circle template
- `contacts-sync` and `contacts-add` skills
- Docs: schema, dunbar philosophy, AI integration

This release is deliberately **dependency-free** (Python stdlib only) and **offline-only**.
You can use it on a flight.

## v1.1 — import hooks (planned, opt-in)

Lightweight "pull initial data" scripts. All require **your own API keys**; the
project will never ship credentials.

- `import_from_vcard.py` — `.vcf` (iOS / macOS / Outlook export) → CSV rows + folder stubs
- `import_from_linkedin.py` — given a profile URL, pull public fields. (Needs a LinkedIn scraping API of your choice; we'll provide the field-mapping logic, not the API client.)
- `import_from_xiaohongshu.py` — same shape, for the 小红书 ecosystem

Design principle: **import is one-way and one-time.** Once a person is in
dunbar, the source of truth is dunbar, not the external service.

## v2.0 — Google Workspace integration (planned)

The big one. Three integration points, all opt-in, all driven by *your* GWS
credentials (no shared backend):

### `pull_contacts()` — Google Contacts → dunbar
- One-way: Google Contacts is the seed, dunbar is where you curate
- Maps Google fields → CSV columns
- New Google contacts you actually want to keep → manual promotion to a circle (not automatic, because Google's contact list is full of noise)

### `push_contacts()` — dunbar → Google Contacts
- One-way the other direction: so your phone's address book auto-has the names + addresses
- Tagged with a `dunbar` Google Contacts group so you can see what came from here

### Calendar / Gmail enrichment
- `get_last_seen(person)` — find the most recent calendar event you both attended → auto-update 上次见面
- `summarize_recent_email(person, days=30)` — pull recent thread subjects with this person, append a one-line "recent threads" digest to the bottom of their `.md`

### Auth model
- Reuse the user's existing `gws` CLI auth (if installed) — zero config
- Fall back to a `.env` with their own OAuth client credentials
- Service-account auth supported but discouraged for personal use

### What we will NOT do
- Ship a hosted backend
- Ship default API credentials
- Auto-promote / auto-demote people between circles based on contact frequency
  (this is exactly the kind of "helpful" decision that breaks the model — a
  human moves people between circles, an agent can suggest, never decide)

## v2.x — community feedback features

Wishlist, prioritized once people use the v1.0 release:

- A `dunbar-cli` thin wrapper so `dunbar sync` works without typing `python3 sync.py`
- An ICS export so birthdays show up in your calendar
- A "yearly review" script (who am I drifting from, who's getting more attention, etc.)
- Multi-language column headers (`schema.json` to define your own header set)

## What's deliberately not on the roadmap

- ❌ A web UI. dunbar is file-first by design — your editor, your file manager, your AI agent are the UI.
- ❌ Cloud sync built-in. Put the repo in a private git remote, or Dropbox, or Syncthing — all work.
- ❌ Multi-user / team mode. Relationships are 1:1; a "team contacts" tool is a different product.
- ❌ Automatic relationship analytics ("strength score" etc.). Inviting AI to grade your friendships is a category error.
