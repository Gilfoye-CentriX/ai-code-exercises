# Task Manager Codebase Analysis – Submission Summary

## 1. Initial vs. Final Understanding

**Initial (before AI):** Simple task manager with 4 files, basic Task model (id, title, status), JSON storage.

**Final (after AI + code access):** Feature-rich CLI app with UUIDs, 4 statuses (TODO, IN_PROGRESS, REVIEW, DONE), 4 priority levels, due dates, tags, and custom JSON serialization.

**Biggest surprise:** The presence of tags, due dates, and priority – features I initially assumed didn't exist.

## 2. Most Valuable Insights Per Prompt

| Part | Prompt Focus | Key Insight |
|------|--------------|-------------|
| 1 | Project Structure | File size indicates responsibility (app.py largest = orchestration) |
| 2 | Feature Location | Find similar features first – patterns repeat across codebase |
| 3 | Domain Model | Business rules often in models.py (is_overdue()) |
| 4 | Practical Application | Combine all insights – extend enums, add queries, integrate at boundaries |

## 3. Implementation Approach for Business Rule

**Rule:** Auto-abandon tasks overdue >7 days unless high priority.

**Approach:**
1. Add ABANDONED to TaskStatus enum
2. Add helpers: days_overdue(), is_high_priority(), is_abandonable()
3. Add storage methods: get_abandonable_tasks(), mark_as_abandoned()
4. Add app method: auto_mark_abandoned() – called on list/stats
5. Add CLI command: abandon for manual override

**Files modified:** models.py, storage.py, app.py, cli.py (~58 lines)

## 4. Future Strategies for Unfamiliar Codebases

**My 4-step AI workflow:**
1. Structure scan (5 min) – directories, config files, sizes
2. Feature search (10 min) – find similar functionality
3. Domain extraction (10 min) – enums, entities, business rules
4. Pattern application (15 min) – copy existing patterns

**Complementary tools:** tree, ripgrep, pytest, debugger, git log

**Key lesson:** Always search for similar features before creating new ones. The codebase already contains patterns you can reuse.

**What I'd do differently:** Scan structure first, verify assumptions by reading enums/fields, map affected components before coding, understand serialization early.