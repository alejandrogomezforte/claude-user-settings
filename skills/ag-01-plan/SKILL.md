---
name: plan
description: "Plan a Jira task: read ticket, investigate codebase, design implementation plan, and present for approval. Usage: /plan MLID-XXXX"
user_invocable: true
arguments:
  - name: task_id
    description: "Jira task ID (e.g., MLID-1868)"
    required: true
---

# /plan $ARGUMENTS

You are starting the **planning phase** of the software workflow. Your goal is to gather context, investigate the codebase, design an implementation plan, and present it for approval — **without writing any implementation code**.

## Step 1 — Read the Jira ticket

Use the Atlassian MCP to read the Jira issue `$ARGUMENTS`:

- Summary, description, and acceptance criteria
- Story points and issue type
- Parent/epic link (determines standalone vs epic sub-task)
- Links, attachments, and subtasks

## Step 2 — Determine task type and branch strategy

Based on the Jira ticket:

| If... | Then... |
|-------|---------|
| Task has no parent epic | **Standalone task** — read `docs/agomez/dx/task-git-branch-strategy.md` |
| Task is a sub-task of an epic | **Epic sub-task** — read `docs/agomez/dx/epic-git-branch-strategy.md` and the epic plan-progress file |

Also check the current git branch with `git branch --show-current` for additional context.

## Step 3 — Investigate the codebase

Launch the `codebase-investigator` agent with a prompt that includes:

- The Jira ticket summary and description
- The acceptance criteria
- Any specific areas to investigate based on the ticket

Wait for the investigation results.

## Step 4 — Design the implementation plan

Launch the `solution-architect` agent with a prompt that includes:

- The full Jira ticket details (from Step 1)
- The codebase investigation summary (from Step 3)
- The task type (standalone vs epic sub-task)
- The branch strategy (from Step 2)
- The plan template to use:
  - Standalone: `docs/agomez/dx/individual-task-workflow.md`
  - Epic: `docs/agomez/dx/epic-workflow.md`

The architect will write the plan file to `docs/agomez/plans/`.

## Step 5 — Present the plan for approval

After the architect completes:

1. Read the plan file that was written
2. Present the full plan to the user inline
3. **STOP and wait for approval**

Tell the user:
- "Here's the implementation plan. Review it and let me know if you'd like any changes, or approve it to proceed to implementation."
- Do NOT proceed to write any implementation code

## Important Rules

- **No implementation code** — this skill only produces a plan document
- **Always use both agents** — investigator first, then architect
- **Always stop for approval** — never auto-proceed to implementation
- **Plan file location**: `docs/agomez/plans/MLID-XXXX-short-description.md` (standalone) or update `docs/agomez/plans/MLID-XXXX-plan-progress.md` (epic)
