# Collect Data Schema

Defines the data format for all `./data/collect/` output files.

---

## File Structure

Each output file (`./data/collect/<source>.md`) has two parts:

### 1. State Header (YAML front-matter)

```yaml
---
source: <source-name>
last_collected: <ISO date>
last_id: <source-specific cursor>
total_entries: <integer>
---
```

| Field | Description |
|---|---|
| `source` | One of: `x-bookmarks`, `github`, `alphaxiv-lib`, `alphaxiv-rec` |
| `last_collected` | Date of most recent successful collection run (YYYY-MM-DD) |
| `last_id` | Source-specific cursor to enable incremental collection (tweet ID, repo URL, arxiv ID) |
| `total_entries` | Running count of entries in the table (for quick reference) |

### 2. Entry Table

```markdown
# <Source Display Name>

| Title | URL | Author | Date | Keywords | Notes |
|---|---|---|---|---|---|
| Tweet text snippet... | [link](https://x.com/...) | @handle | 2026-02-24 | llm, agent | One-sentence summary |
```

---

## Field Definitions

| Field | Type | Required | Description |
|---|---|---|---|
| `Title` | string | Yes | Tweet text (≤200 chars) / repo name / paper title |
| `URL` | string | Yes | Canonical URL formatted as `[link](url)` — **primary dedup key** (extract raw URL for dedup comparison) |
| `Author` | string | Yes | `@handle` / `owner` / `First Author et al.` |
| `Date` | ISO date | Yes | Post / star / submission date |
| `Keywords` | string | Yes | 2–4 comma-separated topic tags |
| `Notes` | string | No | Short summary or description (≤150 chars) |

---

## Deduplication Rules

1. **Primary key: URL** — Never add two rows with the same URL.
2. Before appending, read all existing `URL` column values into a set.
3. Filter collected items: only append those whose URL is not in the set.
4. After appending, update `total_entries` and `last_collected` in the state header.
5. If zero new entries, report "No new entries found" and do not modify the file.

---

## Example Output File

```markdown
---
source: alphaxiv-lib
last_collected: 2026-02-24
last_id: 2501.12345
total_entries: 3
---

# alphaXiv Library

| Title | URL | Author | Date | Keywords | Notes |
|---|---|---|---|---|---|
| Scaling Laws for Neural Language Models | [link](https://www.alphaxiv.org/abs/2001.08361) | Kaplan et al. | 2020-01-23 | scaling, llm, training | Empirical study of how loss scales with model size, data, and compute. |
| Attention Is All You Need | [link](https://www.alphaxiv.org/abs/1706.03762) | Vaswani et al. | 2017-06-12 | transformer, attention, nlp | Introduces the Transformer architecture based solely on attention mechanisms. |
| LoRA: Low-Rank Adaptation | [link](https://www.alphaxiv.org/abs/2106.09685) | Hu et al. | 2021-06-17 | lora, fine-tuning, efficiency | Low-rank decomposition for efficient LLM fine-tuning without full parameter updates. |
```
