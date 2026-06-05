# Exercise Part 4: Practical Application

## Scenario

> **Business Rule to Implement:** "Tasks that are overdue for more than 7 days should be automatically marked as **abandoned** unless they are marked as **high priority** (HIGH or URGENT)."

**Note:** The current codebase doesn't have an `ABANDONED` status. This will need to be added.

---

## 1. Files to Modify

Based on analysis of the actual codebase:

| File | Why Modify? | Specific Changes |
|------|-------------|------------------|
| `models.py` | Add `ABANDONED` to TaskStatus enum | Line ~9: Add `ABANDONED = "abandoned"` |
| `models.py` | Add `is_high_priority()` helper | New method to check priority >= HIGH |
| `models.py` | Add `is_abandonable()` method | Check if task meets criteria for abandonment |
| `storage.py` | Add `get_abandonable_tasks()` method | Query for tasks that are >7 days overdue |
| `app.py` | Add `auto_mark_abandoned()` method | Business logic for automatic abandonment |
| `app.py` | Call auto-abandon on relevant operations | e.g., after `list_tasks()`, `get_statistics()` |
| `cli.py` | Add `abandoned` filter to list command | Allow users to see abandoned tasks |

### Optional Changes

| File | Change |
|------|--------|
| `cli.py` | Add `abandon` command for manual abandonment |
| `cli.py` | Update `stats` command to show abandoned count |
| `storage.py` | Add `get_tasks_by_status(ABANDONED)` |

---

## 2. Detailed Implementation Outline

### Step 1: Update `models.py`

```python
# Add to TaskStatus enum (line ~9)
class TaskStatus(Enum):
    TODO = "todo"
    IN_PROGRESS = "in_progress"
    REVIEW = "review"
    DONE = "done"
    ABANDONED = "abandoned"      # NEW

# Add to Task class (after is_overdue method)
class Task:
    # ... existing code ...
    
    def is_high_priority(self) -> bool:
        """Return True if priority is HIGH or URGENT"""
        return self.priority in (TaskPriority.HIGH, TaskPriority.URGENT)
    
    def days_overdue(self) -> int:
        """Return number of days task is overdue (0 if not overdue)"""
        if not self.due_date or self.status == TaskStatus.DONE:
            return 0
        delta = datetime.now() - self.due_date
        return delta.days if delta.days > 0 else 0
    
    def is_abandonable(self) -> bool:
        """
        Task can be abandoned if:
        - Overdue for more than 7 days
        - Not high priority (HIGH or URGENT)
        - Not already DONE or ABANDONED
        """
        if self.status in (TaskStatus.DONE, TaskStatus.ABANDONED):
            return False
        if self.is_high_priority():
            return False
        return self.days_overdue() > 7
		

### Step 2: Update `storage.py`

# Add new method to TaskStorage class
def get_abandonable_tasks(self):
    """Return list of tasks that should be auto-abandoned"""
    return [task for task in self.tasks.values() if task.is_abandonable()]

def mark_as_abandoned(self, task_id):
    """Mark a specific task as abandoned"""
    task = self.get_task(task_id)
    if task and task.status != TaskStatus.ABANDONED:
        task.status = TaskStatus.ABANDONED
        task.updated_at = datetime.now()
        self.save()
        return True
    return False

def get_tasks_by_status(self, status):
    """Existing method - already works with ABANDONED"""
    return [task for task in self.tasks.values() if task.status == status]
	
### Step 3: Update `app.py`

# Add new method to TaskManager class
def auto_mark_abandoned(self, dry_run=False):
    """
    Automatically mark abandonable tasks as ABANDONED.
    If dry_run=True, only return list without modifying.
    """
    abandonable = self.storage.get_abandonable_tasks()
    
    if dry_run:
        return abandonable
    
    abandoned_ids = []
    for task in abandonable:
        if self.storage.mark_as_abandoned(task.id):
            abandoned_ids.append(task.id)
    
    return abandoned_ids

# Update existing methods to trigger auto-abandon
def list_tasks(self, status_filter=None, priority_filter=None, show_overdue=False):
    # Auto-abandon before listing (optional - could be configurable)
    self.auto_mark_abandoned()
    
    if show_overdue:
        return self.storage.get_overdue_tasks()
    # ... rest of existing code

# Update get_statistics to include abandoned count
def get_statistics(self):
    self.auto_mark_abandoned()  # Ensure stats reflect auto-abandoned tasks
    
    tasks = self.storage.get_all_tasks()
    total = len(tasks)
    
    # ... existing status counts ...
    
    # Add abandoned to status_counts (already handled by loop)
    
    # Add new metric
    abandoned_count = len([task for task in tasks if task.status == TaskStatus.ABANDONED])
    
    return {
        "total": total,
        "by_status": status_counts,
        "by_priority": priority_counts,
        "overdue": overdue_count,
        "completed_last_week": completed_recently,
        "abandoned": abandoned_count  # NEW
    }

# Optional: Manual abandon method
def abandon_task(self, task_id):
    """Manually abandon a task regardless of priority/overdue status"""
    return self.storage.mark_as_abandoned(task_id)
	
### Step 4: Update `cli.py`
# Add to status choices in update_status_parser (line ~49)
update_status_parser.add_argument("status", help="New status", 
    choices=["todo", "in_progress", "review", "done", "abandoned"])  # Added "abandoned"

# Add new command for manual abandon
abandon_parser = subparsers.add_parser("abandon", help="Manually abandon a task")
abandon_parser.add_argument("task_id", help="Task ID")

# Add to the command handler (after existing commands)
elif args.command == "abandon":
    if task_manager.abandon_task(args.task_id):
        print(f"Abandoned task {args.task_id}")
    else:
        print("Failed to abandon task. Task not found or already abandoned.")

# Update stats display to show abandoned count
elif args.command == "stats":
    stats = task_manager.get_statistics()
    print(f"Total tasks: {stats['total']}")
    # ... existing output ...
    print(f"Abandoned tasks: {stats['abandoned']}")  # NEW

# Optional: Add --abandoned filter to list command
list_parser.add_argument("-a", "--abandoned", help="Show only abandoned tasks", action="store_true")