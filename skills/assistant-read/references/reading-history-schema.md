# Reading History Schema

Defines the format for `./data/assistant-read/reading-history.md`.

---

## File Structure

### 1. State Header (YAML front-matter)

```yaml
---
last_read: YYYY-MM-DD
total_reads: N
---
```

| Field | Description |
|---|---|
| `last_read` | ISO date of the most recent read entry (`""` if empty) |
| `total_reads` | Running count of rows in the history table |

### 2. History Table

```markdown
| Title | URL | Theme | Date Read | Keywords |
|---|---|---|---|---|
| Paper title or post snippet... | https://... | llm-agents | 2026-02-25 | agents, reliability |
```

---

## Field Definitions

| Field | Type | Required | Description |
|---|---|---|---|
| `Title` | string | Yes | Title as it appears in the organize file |
| `URL` | string | Yes | Raw URL (not `[link](url)` format) — **primary dedup key** for fast set lookup |
| `Theme` | string | Yes | Slug of the organize file the entry came from (e.g., `llm-agents`) — enables fast write-back lookup |
| `Date Read` | ISO date | Yes | Date the entry was read (YYYY-MM-DD) |
| `Keywords` | string | Yes | Comma-separated keywords from the organize entry — used by recommendation algorithm |

---

## Key Design Decisions

- **Raw URL storage**: URLs are stored as plain strings (not `[link](url)`) to enable fast membership checks without string parsing.
- **Theme slug**: The `Theme` column lets the `add insight` command jump directly to the correct organize file without scanning all themes.
- **Dedup key**: Before appending any entry, check that its URL is not already in the `URL` column. Each URL should appear at most once.

---

## Example File

```markdown
---
last_read: 2026-02-25
total_reads: 2
---

| Title | URL | Theme | Date Read | Keywords |
|---|---|---|---|---|
| Capable but Unreliable: Canonical Path Deviation as a Causal Mechanism of Agent Failure | https://www.alphaxiv.org/abs/2602.19008 | llm-agents | 2026-02-25 | agents, long-horizon tasks, causal inference, reliability |
| Training LLM agents for extremely long-horizon tasks remains an open challenge... | https://x.com/dair_ai/status/2024878225017668091 | llm-agents | 2026-02-25 | LLM agents, long-horizon, reinforcement learning, training |
```
