---
name: "schedule"
description: "Create or update a scheduled task that runs automatically. Use when the user says things like 'every day', 'each morning', 'remind me in an hour', 'run this at noon', or wants to reschedule an existing task."
---

First, decide whether the user wants to **create a new** scheduled task or **change an existing** one.

## Updating an existing task

If the user wants to reschedule, edit the prompt, or pause/resume a task that already exists, call the `update_scheduled_task` tool with its `taskId`. Use `list_scheduled_tasks` if you need to look up the ID.

## Creating a new task

### 1. Analyze the session
Distill the core task into a single, repeatable objective.

### 2. Draft a prompt
Must be entirely self-contained. Include: objective, steps, file paths/URLs, expected output, constraints.
Write in second-person imperative. Never reference "the current conversation".

### 3. Choose a taskName
Short, descriptive, kebab-case (e.g. "daily-inbox-summary").

### 4. Determine scheduling
If unclear, propose one and ask the user to confirm before proceeding.

Finally, call the `create_scheduled_task` tool.
