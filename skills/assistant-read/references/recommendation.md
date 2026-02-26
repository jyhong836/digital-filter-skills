# Recommendation Algorithm

Three reading selection modes used by `/assistant-read`. All modes operate on **unread entries** only.

**Definition of "unread"**: An entry is unread if its raw URL (extracted from `[link](url)`) is **not** present in the `URL` column of `./data/assistant-read/reading-history.md`.

---

## Mode 1: Latest

**Goal**: Surface the newest content regardless of personal history.

**Algorithm**:
1. Filter all entries in the candidate theme(s) to unread only.
2. Sort by `Date` field, descending (newest first).
3. Present the top 5.

---

## Mode 2: AI Recommended ★

**Goal**: Continue learning along the trajectory of recent reading history.

**Algorithm**:
1. Load the `Keywords` column from the **last 10 rows** of `reading-history.md` (most recent reads).
2. Build a **keyword union set**: split each row's keywords by `,`, lowercase and trim each token, collect into a single set.
3. For each unread entry:
   - Split its `Keywords` field by `,`, lowercase and trim each token.
   - **Score** = count of keyword tokens that appear in the keyword union set.
4. Sort unread entries by `(score DESC, Date DESC)` — tiebreak by recency.
5. Present the top 5.

**Note**: This mode is marked with ★ in the UI as the default recommendation.

---

## Mode 3: Most Related to Last Read

**Goal**: Go deeper on the topic you just read — depth-first exploration.

**Algorithm**:
1. Get the **most recent row** in `reading-history.md` (top of table, i.e., the last appended entry).
2. Build a **keyword set** from that single row's `Keywords` field (split by `,`, lowercase, trim).
3. For each unread entry:
   - **Score** = count of its keyword tokens that appear in the last-read keyword set.
4. Sort unread entries by `(score DESC, Date DESC)` — tiebreak by recency.
5. Present the top 5.

---

## Display Format

When presenting recommendations, show one numbered list per mode (or all three side-by-side):

```
[1] Latest
    1. <Title> (<Date>)
    2. <Title> (<Date>)
    ...

[2] AI Recommended ★
    1. <Title> — score: N keywords matched
    2. <Title> — score: N keywords matched
    ...

[3] Most Related to Last Read
    1. <Title> — score: N keywords matched
    2. <Title> — score: N keywords matched
    ...
```

The AI Recommended top pick is highlighted with ★.

---

## Edge Cases

| Situation | Behavior |
|---|---|
| Reading history is empty | Mode 2 keyword union is empty → all scores = 0 → fall back to `Date DESC` order |
| Reading history has < 10 rows | Use all available rows for Mode 2 |
| No last-read entry for Mode 3 | Fall back to `Date DESC` order (same as Mode 1) |
| Tie in score across modes 2/3 | Tiebreak by `Date DESC` (newest wins) |
| All unread entries score 0 in Mode 2/3 | Return top 5 by `Date DESC` (graceful fallback) |
