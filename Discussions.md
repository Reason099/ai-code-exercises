**Exercise Part 1: Understanding Project Structure**

**Initial Exploration & Self-Assessment**

#### **1. Directory Structure**

```
use-cases/task-manager/
├── python/
│   ├── app.py
│   ├── cli.py
│   ├── models.py
│   └── storage.py
├── javascript/
├── java/
```

---

#### **2. Initial Understanding (My Guess)**

**How it’s organised:**

It looks like the project is split into different parts based on what each file does:

* `models.py` → defines the main data structures (like Task, status, priority)
* `storage.py` → handles saving and loading data (JSON file)
* `app.py` → contains the main logic of the program
* `cli.py` → handles user input from the command line

So overall it seems pretty modular and easy to follow.

---

**Technologies used:**

From what I can see, it uses:

* Python 3
* JSON for saving data
* argparse for command line input
* Standard libraries like `datetime`, `uuid`, `enum`, `json`, and `os`

---

**Main components (my understanding):**

* A Task object with fields like id, title, description, priority, status, etc.
* Enums for task status and priority
* JSON file storage
* CLI commands to interact with the app
* Basic features like creating, updating, deleting, and filtering tasks

---

### **Looking at the Code in More Detail**

#### **models.py (data structures)**

This file defines:

* `TaskPriority` (LOW, MEDIUM, HIGH, URGENT)
* `TaskStatus` (TODO, IN_PROGRESS, REVIEW, DONE)
* `Task` class

The Task class has things like:

* id (UUID)
* title and description
* priority and status
* dates and tags

There are also some helper methods like updating a task or checking if it’s overdue.

---

#### **storage.py (saving data)**

This part handles reading/writing to JSON.

Main things here:

* `TaskEncoder` → converts Task objects into JSON
* `TaskDecoder` → converts JSON back into Task objects
* `TaskStorage` → manages all tasks

It also keeps tasks in memory (using a dictionary), which makes things faster.

---

#### **app.py (main logic)**

This is where most of the functionality is:

* Creating tasks
* Listing tasks
* Updating status, priority, due dates
* Deleting tasks
* Managing tags
* Showing stats

So this is basically the “core” of the app.

---

#### **cli.py (user input)**

This file handles commands from the terminal.

There are commands like:

* create
* list
* update status/priority
* delete
* show task
* stats

It also formats how tasks are printed.

---

### **AI Analysis (What I Learned After Looking Deeper)**

**Architecture:**

The project follows a layered structure:

* CLI → App → Storage → Models

Each part has its own job, which makes it easier to understand.

---

**Some patterns I noticed:**

* Task acts like a data object
* Storage works like a repository
* Custom JSON encoder/decoder for saving objects
* CLI commands act kind of like command handlers

---

**How the program runs:**

```
User types command
   then
cli.py processes it
   then
app.py runs the logic
   then
storage.py saves/loads data
   then
JSON file is updated
```

---

### **Important Things I Got Wrong at First**

* I thought it might use a database → it just uses a JSON file
* I thought it might be a web app → it’s only CLI-based
* I expected more validation → it’s actually quite simple

---

### **Main Responsibilities of Each File**

* **models.py** → defines the data and rules
* **storage.py** → handles saving/loading
* **app.py** → main logic
* **cli.py** → user interaction

---

### **How Data Moves Through the CODEapp**

1. Create task → new Task object → saved to JSON
2. List tasks → loaded from memory → printed
3. Update task → changed → saved again
4. Delete task → removed → JSON updated
5. On startup → JSON file is loaded into memory

---

### **Final Summary**

This project is a simple CLI Task Manager built in Python.

**What it uses:**

* Python standard library
* JSON file storage

**Main features:**

* Create, update, delete tasks
* Task priority and status
* Tags
* Overdue tracking
* Basic statistics

---

**Overall:**

The project is actually pretty simple and well organised. It’s a good example of how to structure a small Python app without making it too complicated.
