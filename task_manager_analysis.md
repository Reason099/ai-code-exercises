# Project Analysis: Task Manager Application

## Validating Your Understanding

This is a **CLI-based Task Manager application** written in Python. It allows users to create, update, delete, and query tasks with priorities, due dates, tags, and status tracking. Here's a corrected and expanded picture:

| Your Guess | Reality |
|---|---|
| Application purpose | ✅ Correct — it's a task management tool |
| Technologies | Python + `argparse` for CLI, custom storage layer (likely JSON-based) |
| Folder pattern | Modular MVC-like separation: models, storage, manager, CLI |

---

## Project Structure & What Each File Likely Does

From `task_manager.py` alone, we can infer a multi-file structure:

```
project/
├── task_manager.py      # Business logic / "service layer" — orchestrates operations
├── models.py            # Data models: Task, TaskStatus (enum), TaskPriority (enum)
├── storage.py           # Persistence layer — reads/writes tasks.json
├── tasks.json           # Runtime data file (auto-generated, not in source control)
└── main.py / cli.py     # Likely entry point — wires argparse CLI to TaskManager
```

**Key technologies identified:**
- `argparse` — standard library CLI argument parsing
- `datetime` / `timedelta` — date handling (no third-party libs like `arrow` or `pendulum`)
- Custom JSON storage (inferred from `TaskStorage("tasks.json")`)
- Python `Enum` — used for `TaskPriority` and `TaskStatus`

---

## Application Entry Points

The `argparse` import in `task_manager.py` hints at a CLI entry point, but the actual argument parsing likely lives in a separate `main.py` or `cli.py`. Look for something like:

```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(...)
    # subcommands: create, list, update, delete, stats
```

`TaskManager.__init__` is the programmatic entry point — it bootstraps the storage layer immediately.

---

## Feature Deep Dive: Task Creation & Status Updates

### Task Creation Flow

```python
# task_manager.py → create_task()
create_task(title, description, priority_value, due_date_str, tags)
    │
    ├─ 1. Converts priority_value (int) → TaskPriority enum
    ├─ 2. Parses due_date_str ("YYYY-MM-DD") → datetime object
    ├─ 3. Instantiates Task(...) object  ← defined in models.py
    └─ 4. Calls storage.add_task(task)  ← persists to tasks.json
           └─ Returns task_id (likely UUID or incremental int)
```

### Status Update Flow

```python
# task_manager.py → update_task_status()
update_task_status(task_id, new_status_value)
    │
    ├─ Converts int → TaskStatus enum
    ├─ IF status == DONE:
    │       task.mark_as_done()   ← sets completed_at timestamp on the Task object
    │       storage.save()        ← manual save (Task mutated in-place)
    └─ ELSE:
            storage.update_task(task_id, status=new_status)  ← delegated to storage
```

> **Notable design pattern:** There's an intentional split — marking a task `DONE` goes through the `Task` model directly (to capture `completed_at`), while other status changes go straight through the storage layer. This is a subtle but important distinction worth asking your team about.

---

## 5 Questions to Ask Your Team

1. **"What is the `task_id` format?"** — Is it a UUID, an auto-incrementing integer, or something else? This affects how tasks are referenced in the CLI.

2. **"Why does `update_task_status` handle `DONE` differently from other statuses?"** — Understanding this intent will reveal whether `mark_as_done()` has side effects beyond setting `completed_at`.

3. **"Is `storage.save()` called automatically after every mutation, or only manually?"** — There are two patterns in use (`storage.update_task(...)` vs mutate + `storage.save()`), and knowing which is canonical matters for contributing safely.

4. **"Are there plans to replace the JSON storage with a database?"** — The `TaskStorage` abstraction suggests this was designed to be swappable. Knowing the roadmap helps you understand how much to invest in understanding the current storage code.

5. **"Where is the CLI entry point and how are subcommands structured?"** — `argparse` is imported but not used in this file, so understanding the full CLI surface area is essential before making changes.

---

## Exploration Exercise

Try this to verify your mental model:

### Step 1 — Trace a task's full lifecycle in code

Open `models.py` and find the `Task` class. Locate the `mark_as_done()` method and check what fields it sets. Then confirm that `completed_at` is the field used in `get_statistics()` to count `completed_last_week`.

### Step 2 — Verify the storage contract

Open `storage.py` and check whether `add_task()` returns a task ID and whether `update_task()` auto-saves. Compare this to the manual `storage.save()` call in `update_task_status` — does the pattern match or is there an inconsistency?

### Step 3 — Map the CLI to the manager

Find the entry point file and list every CLI subcommand. For each one, trace which `TaskManager` method it calls. You should end up with a complete map like:

```
cli: create   → TaskManager.create_task()
cli: list     → TaskManager.list_tasks()
cli: done     → TaskManager.update_task_status(..., DONE)
...
```

This exercise will either confirm your understanding or reveal gaps — both outcomes are valuable as a new developer on the project.
