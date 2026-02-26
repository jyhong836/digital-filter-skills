# Organize Data Schema

Defines the data format for all `./data/organize/` output files.

---

## File Structure

Each output file (`./data/organize/<theme>.md`) has two parts:

### 1. State Header (YAML front-matter)

```yaml
---
theme: <theme-slug>
last_organized: <ISO date>
total_entries: <integer>
---
```

| Field | Description |
|---|---|
| `theme` | Filename slug matching the theme (e.g., `llm-agents`, `ai-safety`) |
| `last_organized` | Date of most recent `/organize update` run that modified this file (YYYY-MM-DD) |
| `total_entries` | Running count of rows in the table (for quick reference) |

### 2. Entry Table

```markdown
# <Theme Display Name>

| Title | URL | Author | Date | Source | Keywords | Notes | Insights |
|---|---|---|---|---|---|---|---|
| Paper title or tweet snippet... | [link](https://...) | Author et al. | 2026-02-24 | alphaxiv-rec | llm, agent | One-sentence summary. | |
```

---

## Field Definitions

| Field | Type | Required | Description |
|---|---|---|---|
| `Title` | string | Yes | Paper title / tweet text (≤200 chars) / repo `owner/name` |
| `URL` | string | Yes | Canonical URL formatted as `[link](url)` — **primary dedup key** |
| `Author` | string | Yes | `@handle` for tweets / `First Author et al.` for papers / `owner` for repos |
| `Date` | ISO date | Yes | Post / submission / star date (YYYY-MM-DD) |
| `Source` | string | Yes | One of: `x-bookmarks`, `github`, `alphaxiv-lib`, `alphaxiv-rec` |
| `Keywords` | string | Yes | 2–4 comma-separated topic tags (carry over from collect; refine if needed) |
| `Notes` | string | No | Summary or abstract snippet (≤150 chars; carry over from collect) |
| `Insights` | string | No | Initially empty; filled later by `/assistant-read` |

---

## Deduplication Rules

1. **Primary key: raw URL** — extract the raw URL from `[link](url)` syntax for comparison. Two rows with identical raw URLs must never coexist in the same theme file.

2. **Cross-source arxiv dedup** — `https://www.alphaxiv.org/abs/<id>` and `https://arxiv.org/abs/<id>` with the same `<id>` are the same paper. Extract the arxiv ID from the URL path and compare IDs. Keep the entry already present; do not add a second row.

3. **Cross-file dedup is intentional** — the same entry MAY appear in multiple theme files (multi-theme assignment). This is expected and correct.

4. **Build a per-file URL set before appending** — read all existing `URL` column values in a theme file into a set, normalizing to raw URL strings. Only append rows whose normalized URL is absent from that set.

5. **Arxiv ID normalization** — extract ID from both `alphaxiv.org/abs/<id>` and `arxiv.org/abs/<id>` patterns using the segment after `/abs/`. Treat IDs as equal regardless of which domain hosts them.

6. **Never overwrite Insights** — when a theme file is updated, only new rows are appended. Existing rows (including any filled `Insights` values) are never modified.

---

## Theme Display Names

Map from filename slug to H1 heading used in the file:

| File | H1 Heading |
|---|---|
| `llm-agents.md` | `LLM Agents` |
| `reasoning-and-knowledge.md` | `Reasoning and Knowledge` |
| `evaluation-and-benchmarks.md` | `Evaluation and Benchmarks` |
| `ai-safety.md` | `AI Safety` |
| `tools-and-infra.md` | `Tools and Infrastructure` |
| *(new theme)* | Derive from slug: replace hyphens with spaces, title-case each word |

---

## Example Output File

```markdown
---
theme: llm-agents
last_organized: 2026-02-24
total_entries: 4
---

# LLM Agents

| Title | URL | Author | Date | Source | Keywords | Notes | Insights |
|---|---|---|---|---|---|---|---|
| Training LLM agents for extremely long-horizon tasks remains an open challenge... | [link](https://x.com/dair_ai/status/2024878225017668091) | @dair_ai | 2026-02-20 | x-bookmarks | LLM agents, long-horizon, reinforcement learning, training | New work addressing long-horizon task training challenges for LLM agents. | |
| Capable but Unreliable: Canonical Path Deviation as a Causal Mechanism of Agent Failure | [link](https://www.alphaxiv.org/abs/2602.19008) | Lee | 2026-02-21 | alphaxiv-rec | agents, long-horizon tasks, causal inference, reliability | Stochastic drift from optimal solution path causes LLM agent failures. | |
| New research from Google DeepMind. What if LLMs could discover entirely new multi-agent learning al | [link](https://x.com/omarsar0/status/2026044154040742150) | @omarsar0 | 2026-02-23 | x-bookmarks | multi-agent, LLM, research, DeepMind | Google DeepMind explores LLMs discovering new multi-agent learning algorithms autonomously. | |
| muratcankoylan/Agent-Skills-for-Context-Engineering | [link](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) | muratcankoylan | 2026-02-24 | github | agent, context engineering, multi-agent, skills | Comprehensive skill collection for context engineering and multi-agent systems. | |
```
