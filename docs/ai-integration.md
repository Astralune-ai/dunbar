# AI integration

The whole point of dunbar's design — CSV as source of truth, filesystem as
structure, plain-markdown profiles, generated summary blocks — is to make
the system **AI-readable without any special integration**. Any LLM with file
access can do useful work against it on day one.

The skills in `skills/` make this even smoother for Claude Code and similar
agent runtimes, but they're optional.

## Claude Code

```bash
# Make sure the environment knows where your dunbar workspace is
echo 'export DUNBAR_ROOT="$HOME/path/to/your/dunbar"' >> ~/.zshrc
source ~/.zshrc

# Install the two skills
mkdir -p ~/.claude/skills
cp -r skills/contacts-sync skills/contacts-add ~/.claude/skills/
```

Then in any Claude Code session:

- "Add Alice to my contacts, she's my partner, lives in Sydney" → triggers `contacts-add`
- "Sync contacts" / "I just moved David to inner circle" → triggers `contacts-sync`
- "Who do I know in Melbourne?" → Claude greps the CSV directly, no skill needed
- "When did I last see Bob?" → Claude reads the CSV, returns the 上次见面 date

## Cursor / Windsurf / Codex

Same idea, no skill system needed. Add this to your project rules / system prompt:

```
You have access to a dunbar relationship workspace at $DUNBAR_ROOT.
- contacts.csv is the source of truth for non-structural fields
- Circles are filesystem directories (01_核心_core through 05_弱联系_weak)
- Per-person markdown profiles live in <circle>/<nickname>/<nickname>.md
- Run python3 $DUNBAR_ROOT/sync.py after any folder change
- See docs/schema.md for the column reference
- See skills/contacts-add/SKILL.md for the strict rules on what NOT to fabricate
```

That's enough context for the model to be useful immediately.

## Plain ChatGPT / Claude.ai (no file access)

Two options:

1. **Paste-and-go.** Paste your `contacts.csv` + the schema doc into the chat. Useful for "give me a query" / "draft an outreach plan" type prompts.
2. **Per-person.** Paste one person's `.md` profile and ask for context-specific help (draft a birthday message, prep for a meeting, etc.).

## What AI is good at, in this system

✅ **Recall.** "Who do I know who works in healthcare?" — a single CSV grep answers it.

✅ **Prep.** "I'm seeing Bob tomorrow, what should I know?" — read the .md, surface the last interaction, suggest things to ask about.

✅ **Triage.** "I haven't logged a 上次见面 for anyone in my inner circle in 60+ days — flag who I should reach out to."

✅ **Capture.** "After my coffee with Carol, jot down what we talked about" — appends to the Interactions section of her .md.

✅ **Onboarding.** `/contacts-add` from a natural-language description.

## What AI should NOT do

❌ **Invent fields.** If you don't know someone's birthday, the AI must not guess. See [contacts-add §8](../skills/contacts-add/SKILL.md#8-dont-).

❌ **Promote speculation to fact.** "Joint venture in discussion" is not "Co-founder". See [contacts-add §9](../skills/contacts-add/SKILL.md#9--hard-rule-organization--partnership--only-if-its-actually-happening).

❌ **Decide circles unilaterally.** Migration between circles is a meaningful act. AI can suggest; the human decides.

❌ **Bypass `sync.py`.** Editing 圈层 or 详档 directly in the CSV will be silently overwritten on the next sync.

## Suggested system-prompt addition (any agent)

```
When the user mentions a person, first check contacts.csv to see if they're in
the dunbar workspace. If yes, also read their per-person .md profile. Use that
context to inform your reply. Never invent facts about a person — if a column
is blank, treat it as "unknown", not as "permission to guess".
```
