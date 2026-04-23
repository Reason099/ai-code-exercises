# Algorithm Deconstruction: Task Priority Sorting

## What this algorithm does

The three functions form a pipeline. `calculate_task_score` assigns a single urgency number to one task. `sort_tasks_by_importance` runs that across a whole list and sorts it. `get_top_priority_tasks` just slices the first N results from that sorted list.

---

## Section-by-section breakdown

### Section 1 — Base priority
A lookup table maps each enum to a weight (LOW=1, MEDIUM=2, HIGH=4, URGENT=6), then multiplies by 10. That multiplication gives future additions room to breathe. Notice the gaps are not linear — LOW→MEDIUM doubles, but HIGH→URGENT only goes 4→6, signalling diminishing returns at the top.

### Section 2 — Due date urgency
A step function with four buckets: overdue (+35), today (+20), within 2 days (+15), within a week (+10), otherwise nothing. These cliff edges mean a task due in 7 days scores identically to one due in 3 days. Whether that's intentional is a good question for your team.

### Section 3 — Status penalty
DONE subtracts 50 — large enough to push a finished task below almost all active ones without actually filtering it out. REVIEW subtracts a softer 15.

### Section 4 — Tag and recency boosts
Two small additive bonuses. The `any()` call short-circuits on the first matching tag, so it's efficient even with long tag lists.

### Section 5 — The Schwartzian transform
Used in `sort_tasks_by_importance`: scores are computed once into a list of `(score, task)` tuples, sorted by the score, then the scores are discarded. This avoids calling `calculate_task_score` O(n log n) times inside the comparator — it's called exactly n times total.

---

## Execution flow

```
get_top_priority_tasks(tasks, limit=5)
  └─ sort_tasks_by_importance(tasks)
        │  [Schwartzian transform: score once, sort, discard]
        └─ for each task:
              calculate_task_score(task)
                ├─ base priority  → weight × 10  →  10–60 pts
                ├─ due date bonus → step buckets  →  0, +10, +15, +20, +35 pts
                ├─ status penalty → DONE −50, REVIEW −15
                ├─ tag boost      → +8 if blocker/critical/urgent
                └─ recency boost  → +5 if updated today
              return score (int)
        sorted(task_scores, reverse=True)[:limit]
```

---

## Concrete example walkthrough

| | Task A | Task B |
|---|---|---|
| Priority | HIGH | URGENT |
| Due date | Tomorrow | Overdue |
| Tags | blocker | none |
| Updated | Today | 3 days ago |
| Status | TODO | DONE |

**Task A scoring:**

| Factor | Calculation | Points |
|---|---|---|
| Base priority | HIGH (4) × 10 | +40 |
| Due date | Due in 1 day | +15 |
| Tag boost | "blocker" matched | +8 |
| Recency boost | Updated today | +5 |
| **Total** | | **68** |

**Task B scoring:**

| Factor | Calculation | Points |
|---|---|---|
| Base priority | URGENT (6) × 10 | +60 |
| Due date | Overdue | +35 |
| Status penalty | DONE | −50 |
| **Total** | | **45** |

**Task A wins (68 vs 45)** despite being lower priority. The DONE penalty overrides URGENT + overdue combined. This is the algorithm saying "completed work shouldn't compete with active work."

---

## Journal documentation

### Main components involved
- `calculate_task_score` — pure function, no side effects, deterministic given the same task state
- `sort_tasks_by_importance` — Schwartzian transform (decorate → sort → undecorate)
- `get_top_priority_tasks` — thin wrapper: sort + slice

### Design patterns discovered
- Schwartzian transform avoids redundant scoring inside the comparator
- DONE penalty acts as a soft-delete floor without actually filtering tasks out
- `any()` short-circuits on first tag match — efficient even with many tags
- The scoring is fully stateless, making it easy to unit test in isolation

### How to explain this to another junior developer
"Imagine each task gets a points score. You earn points for being urgent, for being due soon, and for having critical tags. You lose a big chunk of points if you're already done. The algorithm just totals up the points for every task and sorts them highest to lowest."

---

## Possible improvements

- Replace step-function due date buckets with a continuous decay — for example `max(0, 30 - days * 2)` — to eliminate cliff edges where tasks due 3 days apart score identically
- Make the weights configurable per-user rather than hardcoded constants
- For large lists, use `heapq.nlargest(limit, tasks, key=calculate_task_score)` — O(n log k) instead of O(n log n), much faster when limit is small relative to n

---

## Reflection questions

### How did the AI explanation change your understanding?
The Schwartzian transform was the biggest insight — without knowing the pattern name it just looks like unnecessary tuple wrapping. Understanding that it's a deliberate optimization (score n times, not n log n times) reframes the whole structure.

### What aspects were still difficult after the explanation?
The interaction between the step-function buckets and the base priority weights takes some mental arithmetic to reason about. It's not immediately obvious that an URGENT + DONE task can score lower than a MEDIUM + overdue task without working through the numbers.

### What would you ask your team?
1. Are the cliff edges in the due date buckets intentional? A task due in 7 days scores the same as one due in 3.
2. Why is the DONE penalty a flat −50 rather than filtering DONE tasks out entirely before sorting?
3. Is there a plan to make the weights configurable, or are they considered stable?

---

## Three understanding-check questions

**Question 1 — Edge case:** A task has LOW priority, no due date, no tags, status TODO, last updated 3 days ago. Walk through the code and give the exact final score. What does this tell you about the minimum possible score for any active task?

**Question 2 — Performance:** With 10,000 tasks and `limit=5`, the algorithm still scores all 10,000 before slicing. What Python standard library function could give you the top 5 in O(n log 5) instead of O(n log n)?

**Question 3 — Negative scores:** Can a task's score go negative? Work out a concrete example if so. Does the sort still behave correctly with negative numbers, and is there any case where the output ordering could feel wrong to a user even if the math is correct?
