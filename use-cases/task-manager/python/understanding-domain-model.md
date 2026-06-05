# Exercise Part 3: Understanding Domain Model

## Goal: Understand the Task Manager's domain model to implement new business rules

---

## 1. Extract Domain Model (Manual Codebase Analysis)

### Core Entity Classes (Predicted from file sizes + naming)

Based on the structure from Part 1 and typical task manager patterns:

| Entity | Likely Location | Predicted Attributes |
|--------|----------------|---------------------|
| `Task` | models.py | id, title, description, status, created_at, updated_at |
| `TaskStatus` | models.py (Enum) | PENDING, IN_PROGRESS, COMPLETED, ARCHIVED |
| `TaskPriority` | models.py (Enum) | LOW, MEDIUM, HIGH, URGENT |

### Business Logic Locations (Predicted)

| Business Rule / Logic | Likely Location |
|----------------------|------------------|
| Task validation (title not empty) | models.py (`__init__` or validator) |
| Status transition rules | app.py (`complete_task()`, `archive_task()`) |
| Task completion logic | app.py (`mark_complete()`) |
| Task filtering/sorting | app.py (`list_tasks(status=...)`) |

### Terminology Specific to This Application (Predicted)

| Term | Likely Meaning |
|------|----------------|
| Task | A unit of work with an ID, title, and completion status |
| Status | Current state of a task (pending → completed) |
| Storage | JSON file persistence (no database) |
| CLI | Command-line interface commands: `add`, `list`, `complete`, `delete` |

---

## 2. Initial Understanding (Before AI Analysis)

### Entity Relationship Diagram (Sketch)


### What Each Entity Represents

| Entity | Explanation |
|--------|-------------|
| **Task** | A single actionable item with a title, description, and current state |
| **TaskStatus** | Lifecycle stage of a task (workflow state) |
| **TaskPriority** | Importance level for sorting/display purposes |

### Questions & Confusion (Before AI)

| # | Question |
|---|----------|
| 1 | Are tasks stored with auto-incrementing IDs, or UUIDs? |
| 2 | Can tasks be edited after creation? (If yes, where is `update_task()`?) |
| 3 | Is there a due date field? (Critical for task management) |
| 4 | How are tasks sorted? (By ID, created date, or priority?) |
| 5 | Is there tagging/categorization beyond priority? |
| 6 | What happens when a task is "completed" – is it deleted or just marked? |
| 7 | Can tasks be deleted permanently, or only archived? |

---

## 3. AI Analysis (Applying the Domain Understanding Prompt)

### Prompt Used

> "Analyze the domain model of this Python task manager based on typical patterns. Identify entities, relationships, business rules, and terminology. Answer my specific questions about ID generation, due dates, editing, sorting, tags, and deletion behavior."

### AI Response Summary

| Aspect | AI Prediction |
|--------|--------------|
| **ID generation** | Simple auto-increment int (from last task ID + 1) |
| **Due dates** | Unlikely (minimalist design – would be in models.py if present) |
| **Task editing** | Unlikely – would require `update_task()` in app.py (not suggested by file size) |
| **Sort order** | By ID or created_at (oldest first) |
| **Tags/categories** | Not present (adds complexity beyond 2KB models.py) |
| **Deletion behavior** | Likely permanent delete (no archive – would be 4KB+ if archiving present) |

### AI-Generated Domain Model (Predicted actual code)

```python
# models.py (predicted actual content)
from dataclasses import dataclass
from datetime import datetime
from enum import Enum
from typing import Optional

class Status(Enum):
    PENDING = "pending"
    COMPLETED = "completed"

@dataclass
class Task:
    id: int
    title: str
    status: Status = Status.PENDING
    created_at: datetime = None
    
    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.now()
    
    def complete(self):
        self.status = Status.COMPLETED
    
    def to_dict(self):
        return {
            "id": self.id,
            "title": self.title,
            "status": self.status.value,
            "created_at": self.created_at.isoformat()
        }
    
    @classmethod
    def from_dict(cls, data):
        return cls(
            id=data["id"],
            title=data["title"],
            status=Status(data["status"]),
            created_at=datetime.fromisoformat(data["created_at"])
        )