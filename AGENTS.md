# AGENTS.md

## Work Tracking System

Tasks are tracked in `TODO.json` at repository root.

Task format:
```json
{
  "id": "TASK-001",
  "priority": 1,
  "status": "TODO",
  "dependencies": ["TASK-XXX"],
  "description": "Task description",
  "comments": []
}
```

Fields:
- `id`: Task identifier (TASK-XXX)
- `priority`: 1-5 (1=highest)
- `status`: TODO/IN_PROGRESS/BLOCKED/DONE
- `dependencies`: Array of task IDs
- `description`: Task description
- `comments`: Array of implementation notes (agents append as needed)

Manual editing. Tasks auto-increment. Keep all tasks (including DONE) in file.

## Feature Input

`TASK.md` contains feature requirements and specifications for the spec.

## Quick Reference

- Edit `TODO.json` - Manage tasks
- `jq '.tasks[] | select(.id == "TASK-XXX")' TODO.json` - Show single task
- `jq '.tasks[] | select(.status != "DONE")' TODO.json` - List incomplete tasks
- `jq '.tasks[] | select(.status == "TODO" and (.dependencies | length == 0))' TODO.json` - List ready tasks

## Planning System

`PLAN.md` documents the current plan (not yet created).