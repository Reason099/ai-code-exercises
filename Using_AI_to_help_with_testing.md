# Using AI to Help with Testing

---

## Exercise 1.1: Behaviour Analysis

### Step 1: What Do You Think This Function Does?

#### Question 1 — Core Behavior
> In your own words, what does `calculate_task_score` do? What is the *purpose* of the score it returns?

**Answer:**
The function calculates a numeric "importance score" for a task. The higher the score, the more urgent/important the task is. It seems like it's used to rank tasks so the most critical ones float to the top.

---

#### Question 2 — The Inputs
> What does the function expect a `task` object to look like? What fields/attributes does it use, and what types do you think they are?

**Answer:**
The task object needs:

| Field | Type | Notes |
|-------|------|-------|
| `priority` | Enum | One of `LOW`, `MEDIUM`, `HIGH`, `URGENT` |
| `due_date` | datetime | Optional / nullable |
| `status` | Enum | e.g. `DONE`, `REVIEW`, or others |
| `tags` | list of strings | Used to check for special keywords |
| `updated_at` | datetime | When the task was last modified |

---

#### Question 3 — The Output
> Is there a minimum or maximum possible score? Can the score be negative?

**Answer:**
Yes, the score can go negative.

- **Highest possible score:** URGENT priority (60) + overdue (35) + critical tag (8) + recently updated (5) = **108**
- **Lowest possible score:** DONE status (−50) + LOW priority (10) + no due date + no tags + not recently updated = **−40**

---

## Exercise 1.2: Test Planning

### Functions Under Review

- `calculate_task_score` — scores a single task
- `sort_tasks_by_importance` — sorts a list of tasks by score
- `get_top_priority_tasks` — returns top N tasks

---

### Round 1 — Mapping the Relationships

#### Question 1 — Dependencies
> If `calculate_task_score` had a bug, which of the other two functions would be affected?

**Answer:**
Since `sort_tasks_by_importance` calls `calculate_task_score` internally, and `get_top_priority_tasks` calls `sort_tasks_by_importance`, it's a chain. If `calculate_task_score` is broken, both of the other two would also be broken. So the test order should be:

1. `calculate_task_score`
2. `sort_tasks_by_importance`
3. `get_top_priority_tasks`

> ✅ Test from the **bottom of the chain upward.**

---

#### Question 2 — Unit vs Integration
> Which functions get unit tests vs integration tests?

**Answer:**
- A **unit test** tests one function in isolation.
- An **integration test** tests how multiple things work together.

| Function | Test Type | Reason |
|----------|-----------|--------|
| `calculate_task_score` | Unit | Standalone, no dependencies |
| `sort_tasks_by_importance` | Integration | Relies on `calculate_task_score` |
| `get_top_priority_tasks` | Integration | Relies on both functions above |

---

### Round 2 — Thinking About `sort_tasks_by_importance`

#### Question 3 — Sorting Behaviour

**Answer:**

- Empty list → should return an empty list
- Single task list → should return that one task
- Tasks with the same score → order may be unpredictable (no tie-breaking logic in code)
- Highest scored task should always be first

#### Question 4 — Input/Output Contract

**Answer:**
A new sorted list should be returned. The original list should **not** be modified. To test this, keep a copy of the original list before calling the function and compare it afterward.

---

### Round 3 — Thinking About `get_top_priority_tasks`

#### Question 5 — The `limit` Parameter

**Answer:**

| Scenario | Expected Behaviour | Code Behaviour |
|----------|--------------------|----------------|
| `limit=0` | Return empty list | `[:0]` returns `[]` ✅ but may be unintentional |
| `limit=10`, only 3 tasks exist | Return all 3 | Python slicing handles this gracefully ✅ |
| `limit=-1` | Should probably raise an error | `[:-1]` drops the last item ⚠️ — likely a bug |

> ⚠️ **The function does not validate the `limit` parameter.** Negative values silently produce wrong results.

---

## Test Plan Checklist

### 🔴 High Priority — Unit Tests for `calculate_task_score`

- [ ] URGENT priority task scores 60 base points
- [ ] HIGH priority task scores 40 base points
- [ ] MEDIUM priority task scores 20 base points
- [ ] LOW priority task scores 10 base points
- [ ] Unknown/invalid priority defaults to score of 0
- [ ] Overdue task adds 35 points
- [ ] Task due today adds 20 points
- [ ] Task due within 2 days adds 15 points
- [ ] Task due within 7 days adds 10 points
- [ ] Task with no due date adds 0 points
- [ ] DONE status subtracts 50 points
- [ ] REVIEW status subtracts 15 points
- [ ] Tag `"blocker"` adds 8 points
- [ ] Tag `"critical"` adds 8 points
- [ ] Tag `"urgent"` adds 8 points
- [ ] Non-special tags add 0 points
- [ ] Task updated today adds 5 points
- [ ] Task updated yesterday adds 0 points
- [ ] Max possible score: URGENT + overdue + critical tag + recently updated = **108**
- [ ] Min possible score: LOW + DONE + no due date + no tags + not recent = **−40**

---

### 🟡 Medium Priority — Integration Tests for `sort_tasks_by_importance`

- [ ] Returns tasks sorted highest score first
- [ ] Empty list returns empty list
- [ ] Single task list returns that task
- [ ] Original list is not mutated
- [ ] All tasks are present in output (no tasks dropped)
- [ ] Tasks with same score are both included (tie handling)

---

### 🟢 Lower Priority — Integration Tests for `get_top_priority_tasks`

- [ ] Default limit of 5 returns at most 5 tasks
- [ ] `limit=3` returns exactly 3 tasks
- [ ] `limit=10` with only 3 tasks returns all 3 (no crash)
- [ ] `limit=0` returns empty list
- [ ] `limit=-1` — document current (buggy) behaviour or assert it raises an error
- [ ] Returned tasks are in correct priority order

---

## Key Observations & Risks

1. **No input validation on `limit`** — negative values silently return wrong results
2. **Tie-breaking is undefined** — two tasks with equal scores have unpredictable order
3. **`datetime.now()` is called inside the function** — makes tests time-sensitive; consider mocking it
4. **Tag matching is case-sensitive** — `"Blocker"` would not match `"blocker"`
5. **`task.tags` could be `None`** — the `any()` call would raise a `TypeError` if tags is `None` instead of `[]`
