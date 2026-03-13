---
name: enhance-plan
description: Add task tracking, a skills line, and a finalize step to an implementation plan. Use when the user asks to "enhance plan", "add task tracking to plan", "add finalize to plan", "make plan trackable", or "enhance my plan".
---

# Enhance Plan

Add task tracking, a skills line, and a `/finalize` step to an implementation plan. This skill reads the current plan, extracts its steps, and adds three things:

1. **Task tracking** — A section at the top instructing the implementation session to create tasks via `TaskCreate` for each step
2. **Skills line** — An instruction to load relevant skills before making edits
3. **Finalize step** — A final step instructing to run `/finalize` after implementation

## Step 1: Read the Plan

Read the current plan. Identify all steps or units of work. Extract a short label for each one.

## Step 2: Add Task Tracking

Add a "Task Tracking" section near the top of the plan (after the title, before the first step). The section instructs the implementation session to create a task for each step at the start:

```markdown
## Task Tracking

At the start, use `TaskCreate` to create a task for each step:

1. [Step 1 label]
2. [Step 2 label]
3. ...
N. Finalize
```

Include "Finalize" as the last task.

## Step 3: Add Skills Line

Identify currently available skills from the skill list in the system prompt. Determine which skills are relevant for this plan's work by comparing the work type against each skill's trigger description.

Add an instruction to the plan: "After plan approval and before making edits, run `/skill-a`, `/skill-b`."

## Step 4: Add Finalize Step

Add a final step to the plan:

```markdown
## Step N: Finalize

Run `/finalize` to run tests, simplify code, review, and commit.
```

## Rules

- Do not rewrite or reorder existing plan content. Only add the task tracking section, skills line, and finalize step.
- If the plan already has task tracking, a skills line, or a finalize step, skip the respective addition and inform the user.
