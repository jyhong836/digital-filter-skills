---
name: organize
description: Process all collected entries from ./data/collect/ into themed, deduplicated markdown indexes in ./data/organize/. Use when the user runs /organize update. Reads all collect source files, classifies each entry into one or more themes, deduplicates by URL (with cross-source arxiv ID matching), and appends new entries to the matching theme files. Creates new theme files when no existing theme fits.
---

# organize

Read all collected data and produce or update structured, themed markdown indexes in `./data/organize/`.

## Usage

```
/organize update
```

---

## Workflow

### Step 1 — Read memory

Read `./memory/MEMORY.md`. Note any organize-related TODOs or state from previous runs.

### Step 2 — Load theme taxonomy

Read `./skills/organize/references/themes.md` for classification rules, signal keywords, and new-theme creation policy.

### Step 3 — Parse collect data

Read each file in `./data/collect/`:
- `./data/collect/x-bookmarks.md`
- `./data/collect/github.md`
- `./data/collect/alphaxiv-lib.md` (if it exists)
- `./data/collect/alphaxiv-rec.md`

For each file:
1. Strip the YAML front-matter (lines between the `---` delimiters).
2. Skip the H1 heading line and the table header rows.
3. Parse each remaining `|`-delimited row into a structured entry: `Title`, `URL`, `Author`, `Date`, `Keywords`, `Notes`.
4. Record the `Source` value from the file's front-matter `source` field.
5. Collect all entries into a single unified list.

If a collect file does not exist, skip it silently.

### Step 4 — Read existing organize files

For each file that exists in `./data/organize/`:
1. Strip the YAML front-matter.
2. Parse each table row to extract raw URLs (unescape from `[link](url)` format).
3. Extract the arxiv ID from any `alphaxiv.org/abs/<id>` or `arxiv.org/abs/<id>` URL.
4. Build two sets per theme file: a **URL set** and an **arxiv ID set**.

Read `./skills/organize/references/schema.md` for the full field definitions and dedup rules.

### Step 5 — Classify and deduplicate

For each entry in the unified list:

1. **Classify** — using signal keywords from `themes.md`, determine which theme file(s) this entry belongs to. An entry may be assigned to multiple themes.

2. **Deduplicate within each target theme** — for each target theme:
   - Extract the raw URL from `[link](url)` syntax.
   - If the raw URL is already in the theme's URL set → skip for this theme.
   - If the URL contains an arxiv ID and that ID is already in the theme's arxiv ID set → skip for this theme.
   - Otherwise mark it as **new** for this theme.

3. **New theme creation** — if an entry matches no existing theme, determine a descriptive slug and create a new theme. See `themes.md` "New Theme Creation" section.

### Step 6 — Write theme files

For each theme that has new entries:

1. If the file `./data/organize/<theme>.md` does not exist:
   - Create it with YAML front-matter (`theme`, `last_organized`, `total_entries: 0`).
   - Add the H1 heading (display name from `schema.md`).
   - Add the table header and separator row.

2. Append new rows at the top of the table (below the header separator), newest first.
   - Field order: `Title | URL | Author | Date | Source | Keywords | Notes | Insights`
   - Leave `Insights` cell empty.
   - Carry over `Keywords` and `Notes` verbatim from the collect entry.

3. Update YAML front-matter: set `last_organized` to today, increment `total_entries`.

If a theme has zero new entries, do not modify its file.

### Step 7 — Report

```
✓ organize update complete

New entries added:
  llm-agents.md                +N  (total: T)
  reasoning-and-knowledge.md   +N  (total: T)
  evaluation-and-benchmarks.md +N  (total: T)
  ai-safety.md                 +N  (total: T)
  tools-and-infra.md           +N  (total: T)
  <new-theme>.md               +N  (total: T)  ← only if created

Total new rows written: <sum>
No new entries: <theme files with 0 new rows>
```

If zero new entries across all themes:
```
No new entries to organize. All collect data is already indexed.
```

---

## Error handling

- If `./data/collect/` contains no files: stop and print "No collect data found. Run /collect first."
- If a collect file has no table rows: skip it and note it in the report.
- If a theme file's YAML front-matter is malformed: warn, attempt to parse table rows anyway, and continue.
- Never silently drop entries — if classification fails, create a new theme rather than skipping.
- Never overwrite existing `Insights` values — only append new rows; never rewrite existing ones.
