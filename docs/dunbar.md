# Why "Dunbar"

Robin Dunbar's number — roughly 150 — is the cognitive ceiling on people you
can maintain stable social relationships with at one time. Inside that 150
he proposed concentric layers of intimacy:

| Layer | ~Size | What it means |
|-------|------:|---------------|
| Loved ones (support clique) | 5 | The handful of people you'd call in a 3am crisis |
| Sympathy group | 15 | People whose death would devastate you |
| Affinity group | 50 | People you'd invite to a big personal event |
| Active network | 150 | People you can hold a real conversation with off the top of your head |
| Acquaintances | 500 | People whose face you'd recognize |
| Recognizable faces | 1500 | Outer limit of names + faces |

`dunbar` collapses this into **5 working circles** sized for actual relationship
maintenance, not anthropology:

```
01_核心_core      —  partners, immediate family, support clique          (~3–7)
02_挚友_inner     —  best friends, the people you talk to weekly         (~10–20)
03_常联系_regular —  regular collaborators, work confidants, family       (~30–60)
04_熟人_casual    —  people you see/message a few times a year            (~50–150)
05_弱联系_weak    —  long-tail — old classmates, one-off introductions   (~100–500)
```

The exact numbers don't matter. The **discipline** does:

## Why 5 circles, not a flat list

A flat contact list creates two failure modes:

1. **Mode 1 — guilt.** You see a name from 2019, feel like you "should" reach out, and either burn energy doing it or burn energy avoiding it. With circles, weak ties are *supposed* to be quiet — they're not failures.
2. **Mode 2 — flatness.** You spend the same attention on the 50th person as the 5th. The 5th notices the dilution.

Circles let the cadence match the layer. **You don't owe a wedding invite to your acquaintance group.** You don't owe a quarterly check-in to your weak-ties group. You *do* owe your inner circle real attention, and the system makes that legible.

## Why move people between circles

People migrate. A colleague becomes a friend; a friend moves to a different city and you fall out of regular contact. The whole point of `sync.py` being filesystem-driven is that **moving a person to a different circle is a `mv` command** — you don't have to think about which column in a database to update.

> **When to move someone up a circle:** They've been doing the work of an inner-circle relationship — initiating, showing up, being present in your harder moments. Promote them.

> **When to move someone down:** You haven't really talked in 12+ months and neither of you has been the one to break it. That's a `04 → 05` move, not a moral failure.

## Why a profile per person, not just a CSV row

The CSV holds **facts** (phone, email, last-seen date). The `.md` holds **context** (how you know them, what they care about, what you talked about last). Both are needed.

The auto-generated summary block at the top of each `.md` is the bridge: when you open the file, you see the CSV facts at a glance, then your hand-written notes below. When you re-edit the CSV (say, update their phone number), the next sync re-writes the summary — your notes are untouched.

## What dunbar deliberately is NOT

- **Not a CRM.** No deal pipelines, no opportunity tracking, no campaigns. If you're managing customers, use a CRM. dunbar is for people you actually know.
- **Not a contact-import sink.** No "import 5,000 LinkedIn connections". The first time you put someone in dunbar should be a conscious "yes, this person is in my life" act.
- **Not a social graph.** It's your view of your relationships, not a network analysis tool.
- **Not multi-user / collaborative.** This is yours. Relationships are 1:1.

## Further reading

- Robin Dunbar's [original 1992 paper](https://www.sciencedirect.com/science/article/abs/pii/004724849290081J)
- [How Many Friends Does One Person Need?](https://www.amazon.com/dp/0571253423) (Dunbar, 2010) — accessible book version
