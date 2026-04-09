# Task Manager CLI

![Python](https://img.shields.io/badge/Python-3.11+-blue) ![License](https://img.shields.io/badge/License-MIT-green) ![Dependencies](https://img.shields.io/badge/Dependencies-none-teal)

A lightweight command-line task management tool built with pure Python. Create, track, and organise tasks directly from your terminal — no external dependencies required.

---

## Contents

1. [Features](#features)
2. [Installation](#installation)
3. [Usage](#usage)
4. [Configuration](#configuration)
5. [Project Structure](#project-structure)
6. [Running Tests](#running-tests)
7. [Troubleshooting](#troubleshooting)
8. [Contributing](#contributing)
9. [License](#license)

---

## Features

- **Task creation** — title, description, priority, due date, and tags
- **Filtering & listing** — filter by status, priority, or overdue date
- **Status workflow** — `todo` → `in_progress` → `review` → `done`
- **Tag management** — add and remove tags per task
- **Statistics** — overview of task counts by status
- **Zero dependencies** — ships with Python standard library only

---

## Installation

### Prerequisites

- Python 3.11 or higher
- No additional packages required

### Quick Start

```bash
git clone <repository-url>
cd task-manager
python cli.py --help
```

---

## Usage

### Create a task

```bash
python cli.py create "Task Title" \
  --description "Details here" \
  --priority 2 \
  --due "2024-01-31" \
  --tags "tag1,tag2"
```

### List tasks

```bash
python cli.py list                      # all tasks
python cli.py list --status todo        # filter by status
python cli.py list --priority 3         # filter by priority
python cli.py list --overdue            # overdue only
```

### Update a task

```bash
python cli.py update-status <id> in_progress
python cli.py update-priority <id> 4
python cli.py update-due-date <id> "2024-02-15"
```

### Manage tags

```bash
python cli.py add-tag <id> "urgent"
python cli.py remove-tag <id> "urgent"
```

### Details & stats

```bash
python cli.py show <id>
python cli.py stats
```

---

## Configuration

| Option | Values | Notes |
|--------|--------|-------|
| `--priority` | 1 – 4 | 1 = LOW, 2 = MEDIUM, 3 = HIGH, 4 = URGENT |
| `--status` | `todo`, `in_progress`, `review`, `done` | Default: `todo` on creation |
| `--due` | `YYYY-MM-DD` | Optional due date |
| `--tags` | Comma-separated string | e.g. `"bug,frontend"` |

---

## Project Structure

```
task-manager/
├── cli.py          # Entry point — all CLI commands
├── tests/
│   └── ...         # unittest test suite
└── README.md
```

---

## Running Tests

```bash
# Basic output
python -m unittest discover tests

# Verbose output
python -m unittest discover -v tests

# Single test file
python -m unittest tests.test_filename
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `python: command not found` | Use `python3 cli.py` or check your PATH |
| Wrong Python version | Run `python --version` — must be 3.11+ |
| Invalid date format | Use `YYYY-MM-DD`, e.g. `2024-01-31` |
| Unknown task ID | Run `python cli.py list` to find valid IDs |
| `Invalid status` error | Use exactly: `todo`, `in_progress`, `review`, `done` |

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

Please ensure all tests pass before submitting a PR.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
