# Exercise Part 2: Finding Feature Implementation

## Feature Request: Task Export to CSV

> "Add a new feature to export all tasks to a CSV file."

---

## 1. Initial Search (Manual Codebase Exploration)

### Search Terms Used

| Search Term | Target |
|-------------|--------|
| `export` | Any existing export functionality |
| `csv` | CSV file handling |
| `write` | File write operations |
| `save` | Existing save patterns in storage.py |
| `to_json`, `to_dict` | Serialization methods |
| `file` | Generic file operations |

### Files to Examine

Based on the project structure from Part 1:

| File | Reason to Search |
|------|------------------|
| `storage.py` | Contains existing file operations (save/load) |
| `models.py` | May have `to_dict()` or serialization methods |
| `app.py` | Business logic – might have `get_all_tasks()` |
| `cli.py` | Command handling – entry point for new command |

### Existing Export/File-Related Functionality (Hypothesis)

| Location | Likely Existing Code |
|----------|---------------------|
| `storage.py` | `save_tasks()`, `load_tasks()` – likely JSON format |
| `models.py` | `Task.to_dict()` – converts task to serializable dict |
| `app.py` | `list_tasks()` – retrieves all tasks |
| `cli.py` | Command parsing for `list`, `add`, `complete` |

### Reusable Utility Functions (Predicted)

| Function | Location | Reusable For CSV Export? |
|----------|----------|-------------------------|
| `storage._write_to_file()` | storage.py | Yes – file writing logic |
| `models.Task.to_dict()` | models.py | Yes – data serialization |
| `app.get_all_tasks()` | app.py | Yes – data retrieval |
| `cli._format_output()` | cli.py | Maybe – formatting helpers |

---

## 2. Hypothesis (Before AI Analysis)

### Where Task Export Functionality Should Belong

| Component | Role in CSV Export |
|-----------|-------------------|
| `cli.py` | Add `export` command + file path argument |
| `app.py` | Add `export_to_csv(filepath)` method |
| `storage.py` | Add `write_csv(data, filepath)` or reuse existing writer |
| `models.py` | Add `to_csv_row()` method (optional) |

### Components That Need Modification


### Proposed Implementation Plan

**Step 1:** Add `to_csv_row()` or use `to_dict()` in `models.py`
**Step 2:** Add `export_to_csv(filepath)` in `app.py`
**Step 3:** Reuse `storage.py` file-writing utilities
**Step 4:** Add CLI command in `cli.py`

---

## 3. AI Analysis (Applying the Feature Location Prompt)

### Prompt Used

> "I need to add a 'Task Export to CSV' feature to this task manager. The codebase has cli.py, app.py, models.py, storage.py. Where should I implement this? Show me existing patterns I can reuse."

### AI Response Summary

| Question | AI Answer |
|----------|-----------|
| **Where to put export logic?** | `app.py` – follows existing pattern (add_task, complete_task) |
| **Where to put file writing?** | `storage.py` – already has JSON save/load, add CSV variant |
| **Where to put command?** | `cli.py` – add `do_export()` or new argument parser branch |
| **What to reuse?** | `models.Task.to_dict()` → convert to CSV row |
| **Existing pattern to follow** | `save_tasks()` → `export_to_csv()` similar signature |

### Code Location Mapping (AI Suggestion)

```python
# models.py – reuse existing
class Task:
    def to_dict(self):
        return {"id": self.id, "title": self.title, ...}

# storage.py – add new function
def export_to_csv(tasks, filepath):
    # reuse JSON-writing pattern but with CSV

# app.py – add method
class TaskApp:
    def export_csv(self, filepath):
        tasks = self.storage.load()
        storage.export_to_csv(tasks, filepath)

# cli.py – add command
if args.command == "export":
    app.export_csv(args.file)