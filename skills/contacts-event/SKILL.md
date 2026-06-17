---
name: contacts-event
description: Record and organize a complex, multi-party, evolving situation involving a contact — a dispute, a scam, a crisis, a "two sides tell opposite stories" tangle, anything that takes several conversations to untangle. Produces a layered event dossier + a cause-and-effect chronology stored inside that contact's folder, kept separate from the clean contact profile that contacts-add maintains. The core is five-layer tagging (firsthand fact / secondhand / our hypothesis / commentary / to-confirm) which stops fact and inference from bleeding into each other, so you never act on a secondhand claim you mistook for firsthand. Trigger words: /contacts-event, log this situation, record the dispute, untangle this, event dossier, build a case file for X, the story keeps changing, the two sides don't match, 记录事件, 事件档案, 来龙去脉, 这摊事, 多方说法对不上, 反转, 罗生门. When a known contact gets pulled into a messy / multi-party / contested situation that needs ongoing tracking, use this even if the user doesn't name it — don't cram the analysis and to-dos into the clean contact profile, that's what this is for.
---

# contacts-event — log a complex situation around a contact

User input: `/contacts-event <who + what happened>` (natural language).

## Where's my dunbar root?

Same resolution as `contacts-sync` / `contacts-add`:
1. `$DUNBAR_ROOT` environment variable
2. Current working directory if it has `contacts.csv` and at least one `0N_*` circle folder
3. Ask the user

## What this skill is for (and what it isn't)

`contacts-add` keeps a **clean profile** of a person — basics, how you met, interaction history. It explicitly forbids writing TBDs, risk flags, commentary, or audits of the person into that profile.

But real life produces situations: a contact gets tangled in something **complex, multi-party, with accounts that contradict each other, and still moving** — a dispute, a scam, a crisis, a family emergency. That carries a lot of **inference, open questions, judgments about third parties, and sensitive detail**. None of it belongs in the clean profile, yet it all needs a disciplined home. This skill is that home.

## Two core principles

### Principle 1 — Keep the two artifacts separate
- **Contact profile** (`<nickname>.md`): stays clean, neutral, shareable. Health / legal exposure / judgments about others / to-dos do NOT go here.
- **Event dossier** (`YYYY-MM-DD_<short-event>.md`, same folder): all the analysis, sensitive detail, hypotheses, and to-dos live here. An internal working file, not for outside eyes.

### Principle 2 — Five-layer tagging (the heart of the method)
Tag **every line** in the dossier with how trustworthy it is:

| Tag | Meaning | Use |
|-----|---------|-----|
| 🟦 **Fact / firsthand** | the principal's own firsthand account, or objective records (bank/police records, original message screenshots) | hardest; safe to act on |
| 🟨 **Secondhand** | relayed by a third party / family member | stays 🟨 forever — **never promote to 🟦** |
| 🟧 **Hypothesis** | a logic / assumption you and others constructed | mark it as a guess, not a fact |
| 💬 **Commentary** | the agent's analysis, caution, counter-reading | helps judgment, isn't the verdict |
| ⬜ **To confirm** | a gap you still need to close | the holes to plug before acting |

**Why this is the valuable part**: complex situations blow up when someone treats "what the mother said" as "what the principal said", or "what we guessed" as "what we verified", and then reports to the police / confronts someone on that basis — and it collapses on first contact. Tagging puts whose-version-counts and what's-still-empty out in the open, so a later reversal doesn't catch you having recorded a guess as a fact.

## Flow

1. **Locate the contact**
   ```bash
   NAME="<nickname>"
   grep -ih "^${NAME}," "$DUNBAR_ROOT/contacts.csv"
   ```
   If absent → run `/contacts-add` first to create the person, then come back.

2. **Create / update the dossier** in the contact's folder. Two default artifacts:
   - **Event dossier** `YYYY-MM-DD_<short-event>.md` — five-layer tagging throughout.
   - **Chronology** `<nickname>_chronology_YYYY-MM-DD.md` — narrative + timeline table.

   Keep updating the **same** files as new information arrives across conversations — don't spawn a new file each time.

3. **Optional sub-artifacts** (only when asked):
   - **First-person narrative** — the situation written as a story (for a retrospective, or possible publication; scrub real identities before sharing anything).
   - **Evidence / action checklist** — when there's a dispute or legal angle, a tickable list.

4. **Cross-link**: add one line to the profile's "interactions" section pointing to the dossier (a pointer, no analysis); link `[[nickname]]` back from the dossier.

5. **Evolution discipline** (when updating):
   - **Firsthand beats secondhand.** When the two clash, the firsthand account wins; the secondhand one stays 🟨.
   - **Reversals leave a trail.** When a fact is overturned, mark the old judgment "superseded" but **keep it** — the process trail is how you avoid repeating the mistake.
   - **Status banner.** Maintain a one-line status (latest conclusion / reversal / progress) at the top of the dossier.

## Event dossier template

```markdown
# <nickname> · <short event>

> **Logged**: YYYY-MM-DD ｜ **Status**: <one-line latest>
> **Sources**: <recordings / screenshots / conversations…>
> **Layers**: 🟦 firsthand · 🟨 secondhand · 🟧 hypothesis · 💬 commentary · ⬜ to-confirm
> ⚠️ Internal full record; contains sensitive detail. Any outward-facing version is made separately, with a different scope (see "Outward scope").

## 1. One-liner + current status
## 2. The ask / the dispute
## 3. Established facts (🟦, line by line)
## 4. Each side's account (🟦 firsthand / 🟨 secondhand, side by side — don't merge or reconcile)
## 5. To confirm (⬜, each with "why it matters")
## 6. Our hypothesis vs the other reading (🟧 one version + 💬 the counter-reading, side by side)
## 7. To-dos
## 8. Updates / reversal trail (append by date; mark superseded judgments, don't delete)

## Outward scope
- For your own lawyer / advisor: full and honest.
- For police / outside parties: only true, fact-focused; no false or misleading statements.
```

## Chronology template

```markdown
# <nickname> · chronology

> Layers as above. ★ If there's a reversal, note "the corrected version in §N governs; earlier sections kept as a process trail."

## Narrative: root cause → trigger → course → outcome
(each paragraph tagged with its layer, told as a connected causal story)

## Timeline
| When | What | Layer |
|------|------|-------|

## What's been superseded (on a reversal)
| Old version | Corrected |

## Still open / watch (⬜ + ⚠️)
```

## Don't ❌

- Don't put the analysis / to-dos / third-party judgments / sensitive content into the **clean profile** (that's contacts-add's turf, with its own bans).
- Don't write a 🟨 secondhand claim as a 🟦 fact — however confident the person relaying it is.
- Don't **delete** an overturned judgment on a reversal; mark it superseded and keep it.
- Don't dress up a hypothesis (🟧) as verified fact; and don't record only your version — put the counter-reading next to it (💬).
- Don't invent to-dos; record only what the user actually said.
- For anything legal, label conclusions "directional, not formal legal advice" and don't impersonate a lawyer.

## Example (abstracted, no real PII)

**User input:**
```
/contacts-event log the mess where Sam's flatmate supposedly extorted money from him
```

**The skill does:**
1. grep `contacts.csv` for `^Sam,` → found → use Sam's folder.
2. Create `2026-01-20_flatmate-money-dispute.md` (event dossier) + `Sam_chronology_2026-01-20.md`.
3. Put the family's relayed account (🟨 "the flatmate threatened him at knifepoint") **next to** Sam's own firsthand account (🟦 "there was no threat, I inferred it") — flag the gap, don't merge them.
4. Record "the flatmate is the aggressor" as a hypothesis (🟧), and put the counter-reading next to it (💬 "the flatmate may be an ordinary person who got dragged in").
5. Unknowns — the amount, the recipient's real name, who called the police — go under "to confirm" (⬜).
6. When later conversations establish the truth (a reversal) → update the top status banner, mark the old judgment superseded but keep it, add a "corrected version" section to the chronology.
7. Add one line to Sam's profile interactions pointing to the dossier; `[[Sam]]` back-links from the dossier.
8. If asked, produce a first-person narrative or an evidence checklist.
