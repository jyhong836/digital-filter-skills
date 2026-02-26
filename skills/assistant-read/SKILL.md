---
name: assistant-read
description: Explore organized content, get reading recommendations, and write insights back into organize files. Three sub-commands: /assistant-read (auto-select topic), /assistant-read <topic> (topic-filtered), /assistant-read add insight to <post>: <insight> (write insight back). Stage 3 of the information pipeline.
---

# assistant-read

Explore your organized knowledge, get personalized reading recommendations, and record insights.

## Usage

```
/assistant-read
/assistant-read <topic>
/assistant-read add insight to <post name>: <insight text>
```

---

## Sub-command A: `/assistant-read` (auto-select topic)

Automatically picks the most recently organized theme and presents reading recommendations.

### Step 1 — Read state

Read `./memory/MEMORY.md` for any assistant-read TODOs or in-progress notes.

Read `./data/assistant-read/reading-history.md`:
- Parse YAML front-matter (`last_read`, `total_reads`)
- Build a **read URL set**: collect all values in the `URL` column (raw URLs, no parsing needed)
- If the file does not exist, treat the read URL set as empty and `total_reads` as 0

### Step 2 — Select theme

Read the YAML front-matter of every file in `./data/organize/`:
- Extract `theme` and `last_organized` from each file
- Select the theme with the **most recent `last_organized` date**
- This is the auto-selected theme for this session

### Step 3 — Load unread entries

Read the full table of the selected theme file (`./data/organize/<theme>.md`).

For each row:
- Extract the raw URL from the `[link](url)` format
- If the raw URL is in the read URL set → **already read**, skip
- Otherwise → **unread**, include in candidate list

If the candidate list is empty:
```
Nothing new in `<theme>`. All entries have been read.
Consider running /collect to gather new content, then /organize update.
```
Stop here.

### Step 4 — Load recommendation algorithm

Read `./skills/assistant-read/references/recommendation.md` for the full scoring logic for all three modes.

### Step 5 — Display recommendations

Apply all three modes to the candidate list. Present up to 5 entries per mode in a combined view:

```
Auto-selected theme: <theme display name> (<N> unread entries)

Reading recommendations:

── [1] Latest ────────────────────────────────────
  1. <Title> (<Date>)
  2. <Title> (<Date>)
  ...

── [2] AI Recommended ★ ──────────────────────────
  1. ★ <Title> — <N> keywords matched
  2. <Title> — <N> keywords matched
  ...

── [3] Most Related to Last Read ─────────────────
  1. <Title> — <N> keywords matched  (last read: <last-read title>)
  2. <Title> — <N> keywords matched
  ...

Enter a number to read (e.g., "2-1" for AI Recommended #1, or just "3" for Latest #3):
```

Wait for the user to select an entry.

### Step 6 — Fetch and summarize

Parse the user's selection to identify the chosen entry.

Fetch the entry's raw URL:
- Use WebFetch for standard URLs
- Use `/chrome` (browser automation) if the URL appears paywalled or requires login (e.g., X/Twitter, alphaXiv when behind auth)

Summarize the content in 3–5 bullet points covering:
- Main contribution or argument
- Key findings or takeaways
- Relevance to the theme

If fetch fails (404, paywall, network error):
- Report: "Could not fetch `<URL>`: <error>. Showing metadata only."
- Show the entry's `Notes` and `Keywords` from the organize file as fallback
- Still proceed to log the entry as read

### Step 6.5 — Create note file

After summarizing the paper:
- Derive a slug from the title: lowercase, replace spaces/punctuation with hyphens, truncate to 60 chars
- Create `./data/assistant-read/notes/<slug>.md` with:
  - YAML front-matter: `title`, `url`, `authors`, `date_read` (today), `themes` (list of theme slugs the paper appears in)
  - `# Summary` section: bullet points from the fetch summary
  - `# Discussion` section: empty placeholder

### Step 7 — Update reading history

Append the entry to `./data/assistant-read/reading-history.md`:
- Add a new row at the **bottom** of the table: `| <Title> | <raw URL> | <theme-slug> | <today's date> | <Keywords> | [note](notes/<slug>.md) |`
- Update YAML front-matter: set `last_read` to today, increment `total_reads` by 1

### Step 8 — Prompt for insight or discussion

```
Would you like to:
  [1] Add a quick insight  → /assistant-read add insight to <title>: <insight>
  [2] Explore questions    → I'll suggest questions; answers get saved to the note file
  [3] Skip
```

When the user selects [2] or engages in open discussion:
- Suggest 4–6 tutor-style questions about the paper
- As the user answers or asks Claude to answer, append each Q&A pair to the `# Discussion` section of the note file under a `## Q<N> — <short topic>` heading
- After discussion ends, distill any new generalizable lessons into `./data/assistant-read/experiences.md`:
  - Find the relevant conceptual heading (e.g., "Simulating Patients / Humans") or create a new one
  - Append a bullet with paper citation, method, validation approach, and key limitations

---

## Sub-command B: `/assistant-read <topic>` (topic-filtered)

Searches all organize files for entries matching a topic keyword and recommends from those results.

### Step 1 — Read state

Same as Sub-command A Step 1.

### Step 2 — Search for topic

Read all files in `./data/organize/`.

For each row across all theme files:
- Search for `<topic>` (case-insensitive) in the `Title`, `Keywords`, and `Notes` columns
- Collect all matching rows, tagging each with its source theme slug

Filter to **unread only** (URL not in read URL set).

If zero unread matches:
```
No unread entries for `<topic>`.
All matching entries have been read, or nothing has been collected on this topic yet.
Try /collect to gather new content, then /organize update.
```
Stop here.

### Step 3 — Present mode selection

```
Found <N> unread entries matching "<topic>" across <M> theme(s).

How would you like to select?
  [1] Latest              — newest entry by date
  [2] AI Recommended ★    — best keyword match with your reading history
  [3] Most Related to Last Read — closest to what you last read

Enter mode number:
```

Wait for the user to pick a mode (1, 2, or 3).

### Step 4 — Apply algorithm and display

Read `./skills/assistant-read/references/recommendation.md`.

Apply the selected mode to the matched unread entries. Display the top 5:

```
── [<mode name>] ─────────────────────────────────
  1. ★ <Title> (<theme>) — <score or date>
  2. <Title> (<theme>) — <score or date>
  ...

Enter a number to read:
```

Wait for user selection.

### Step 5 — Fetch, summarize, log, and prompt

Same as Sub-command A Steps 6–8.

---

## Sub-command C: `/assistant-read add insight to <post name>: <insight text>`

Writes a user-provided insight back into the appropriate organize file.

### Step 1 — Parse input

Split the command argument on the **first `:`**:
- Everything before `:` → `<post name>`
- Everything after `:` → `<insight text>` (trim leading/trailing whitespace)

### Step 2 — Find matching entry

Read all files in `./data/organize/`. For each row:
- Check if `<post name>` appears in the `Title` column (case-insensitive substring match)
- Collect all matching rows with their file path and row content

### Step 3 — Handle match count

**0 matches**:
```
No entry found matching "<post name>".
Check the title spelling and retry. Tip: partial titles work (e.g., "Capable but Unreliable").
```
Stop.

**2+ matches**: List candidates with numbers and wait for clarification:
```
Multiple entries match "<post name>":
  1. <Title> (in <theme>.md)
  2. <Title> (in <theme>.md)
  ...
Which entry did you mean? Enter a number:
```
Wait for user response. Then proceed with the chosen entry.

**1 match**: Proceed directly.

### Step 4 — Update Insights cell

Read `./skills/assistant-read/references/reading-history-schema.md` to confirm write-back format.

In the matching theme file (`./data/organize/<theme>.md`):
- Locate the exact row by matching the raw URL (most reliable) or full title
- Read the current `Insights` cell value

If `Insights` is **empty** → set to `<insight text>`

If `Insights` is **non-empty** → append `; <insight text>` (note the semicolon separator)

Write the updated file.

**Never overwrite** an existing insight value — always append.

### Step 5 — Confirm

```
✓ Insight added to "<Title>" in <theme>.md

Insights cell is now:
  <full updated Insights value>
```

---

## Error Handling

| Situation | Action |
|---|---|
| WebFetch fails (paywall, 404) | Report error; still log entry as read; show `Notes` as fallback |
| `/chrome` cannot access URL | Tell user which URL to open in Chrome; still log as read |
| `reading-history.md` missing | Create it on the fly with empty state (`last_read: ""`, `total_reads: 0`) before proceeding |
| All entries in a theme already read | Report "Nothing new in `<theme>`"; suggest `/collect` |
| Insight target is ambiguous | List candidates with numbers; require user clarification before writing |
| Insight target not found | Report error with spelling tip; do not modify any file |
| Organize file front-matter malformed | Warn; attempt to parse table rows anyway; continue |
