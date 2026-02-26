---
name: collect
description: Incrementally collect data from Personalized Information Sources (PIS) and append new entries to ./data/collect/. Use when the user runs /collect with one of these sub-commands: x-bookmarks (X/Twitter bookmarks via /chrome), github (GitHub stars via WebFetch), alphaxiv-lib (alphaXiv bookmarked papers via /chrome), alphaxiv-rec (alphaXiv recommended papers via /chrome). Skips already-collected entries to avoid duplicates.
---

# collect

Collect new entries from a Personalized Information Source and append them to `./data/collect/<source>.md`.

## Usage

```
/collect <sub-command>
```

Sub-commands: `x-bookmarks` | `github` | `alphaxiv-lib` | `alphaxiv-rec`

---

## Workflow

### Step 0 - Setup

For different sub-commands, extra setup may be required. See `./skills/collect/references/sources.md` for details.

### Step 1 — Read state

Read `./data/collect/<source>.md` if it exists:
- Parse the YAML front-matter to get `last_id` and `last_collected`
- Build a URL set from all existing table rows (for dedup)
- If the file does not exist, treat state as empty (no prior entries)

### Step 2 — Fetch source

Read `./skills/collect/references/sources.md` for the full extraction guide for the requested sub-command.

Key tool per sub-command:
- `x-bookmarks` → **/chrome** at `x.com/i/bookmarks` (latest 20 posts)
- `github` → **WebFetch** at `https://github.com/<github-username>?tab=stars`
- `alphaxiv-lib` → **/chrome** at `https://www.alphaxiv.org/bookmarks?folder=<alphaxiv-folder-id>` (all records)
- `alphaxiv-rec` → **/chrome** at `https://www.alphaxiv.org/?sort=Recommended` (top 20)

### Step 3 — Deduplicate

Read `./skills/collect/references/schema.md` for the full field definitions and dedup rules.

Filter collected items: keep only those whose URL is **not** in the existing URL set.

### Step 4 — Write output

If new entries exist:
1. If the file does not exist, create it with the state header and table header
2. Append new rows to the table (newest entries at the top, below the header)
3. Update `last_collected` to today's date
4. Update `last_id` to the appropriate cursor value (see sources.md)
5. Update `total_entries` count

If no new entries: report "No new entries found since last collection." Do not modify the file.

### Step 5 — Report

Print a summary:
```
✓ Collected <N> new entries from <source>
  Added: <title1>, <title2>, ...
  Total entries: <total>
  Last collected: <date>
```

---

## Error handling

- If `/chrome` cannot access the page (not logged in, tab not open): stop and tell the user which URL to open in Chrome before retrying.
- If WebFetch returns an error: report the HTTP status and stop.
- Never silently skip errors — always report what went wrong.
