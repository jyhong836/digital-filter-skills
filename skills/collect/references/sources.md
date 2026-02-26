# Collect Sources Reference

Per-source URLs, extraction guidance, and tool selection.

---

## x-bookmarks

**Tool:** `/chrome` (Chrome extension — do NOT use Playwright MCP)
**URL:** `x.com/i/bookmarks`
**Auth required:** Yes — user must be logged in to X in Chrome

**Extraction:**
- Navigate to `x.com/i/bookmarks` using /chrome
- Collect the **latest 20 posts** visible on the page (do not scroll further)
- For each post extract:
  - `text`: first 200 characters of tweet text (strip newlines)
  - `url`: canonical tweet URL (format: `https://x.com/<handle>/status/<id>`)
  - `author`: `@handle` of the tweet author
  - `date`: post date in ISO format (YYYY-MM-DD)
  - `keywords`: infer 2-4 topic keywords from the text
  - `notes`: one-sentence summary
- **Cursor:** use the tweet ID from the oldest collected tweet as `last_id`
- Stop collecting if a tweet's ID is ≤ `last_id` from the state header (incremental)

---

## github

**Tool:** `WebFetch`
**URL:** `https://github.com/<github-username>?tab=stars`
**Auth required:** No — public page

**Setup:** User must provide their GitHub username in `./memory/config.md` as `<github-username>`. The skill will substitute this value when constructing the URL.

**Extraction:**
- Fetch `https://github.com/<github-username>?tab=stars` (paginate via `?page=N` if needed)
- For each starred repo extract:
  - `title`: `owner/repo-name`
  - `url`: `https://github.com/owner/repo`
  - `author`: repo owner handle
  - `date`: date starred (if shown) or date last updated
  - `keywords`: repo topics/tags if available; otherwise infer from description
  - `notes`: repo description (truncate to 150 chars)
- **Cursor:** record the last repo URL seen as `last_id` for incremental runs
- On subsequent runs, stop when reaching the `last_id` repo

---

## alphaxiv-lib

**Tool:** `/chrome` (Chrome extension — do NOT use Playwright MCP)
**URL:** `https://www.alphaxiv.org/bookmarks?folder=<alphaxiv-folder-id>`
**Auth required:** Yes — user must be logged in to alphaXiv in Chrome

**Extraction:**
- Navigate to the bookmarks folder URL using /chrome
- Collect **all visible records** (scroll through the full list)
- For each paper extract:
  - `title`: paper title
  - `url`: alphaXiv paper URL (e.g., `https://www.alphaxiv.org/abs/<arxiv-id>`)
  - `author`: first author (append "et al." if multiple)
  - `date`: submission or bookmark date (ISO format)
  - `keywords`: infer from title/abstract snippet
  - `notes`: abstract snippet (first 150 chars)
- **Cursor:** record `last_id` as the arxiv ID of the most recently added paper
- On subsequent runs, collect all papers not already in the data file (dedup by URL)

---

## alphaxiv-rec

**Tool:** `/chrome` (Chrome extension — do NOT use Playwright MCP)
**URL:** `https://www.alphaxiv.org/?sort=Recommended`
**Auth required:** Yes — user must be logged in for personalized recommendations

**Extraction:**
- Navigate to the URL using /chrome
- Collect the **top 20 recommended papers** visible on the page
- Same field schema as `alphaxiv-lib`
- **Cursor:** `last_collected` date is sufficient (recommendations refresh daily)
- Dedup by URL against existing entries before appending
