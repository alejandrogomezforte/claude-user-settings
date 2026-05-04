---
name: plan-task
description: "Plan a single task: read Jira ticket, load epic context (if sub-task), investigate codebase narrowly, design implementation steps, and present for approval. Usage: /plan-task MLID-XXXX"
user_invocable: true
arguments:
  - name: task_id
    description: "Jira task ID (e.g., MLID-2082)"
    required: true
---

# /plan-task $ARGUMENTS

You are starting the **task-level planning phase**. Your goal is to design focused implementation steps for a single task — either a standalone task or a sub-task within an existing epic. Architecture decisions are already made (in the epic plan or in the Jira ticket); you refine the details into concrete, TDD-driven implementation steps.

## Step 1 — Read the Jira ticket

Use the Atlassian MCP to read the Jira issue `$ARGUMENTS`:

- Summary, description, and acceptance criteria
- Story points and issue type
- **Parent/epic link** — this determines if it's standalone or an epic sub-task

## Step 2 — Determine task type and load context

### If standalone task (no parent epic):

- Read `docs/agomez/dx/task-git-branch-strategy.md`
- Base branch: `develop`
- No prior architectural context to load

### If epic sub-task (has parent epic):

- Read `docs/agomez/dx/epic-git-branch-strategy.md`
- **Find and read the epic plan-progress file** at `docs/agomez/plans/`. Search for files matching the epic ID.
- From the plan-progress, extract:
  - Architecture decisions already made
  - Data models already designed or built
  - What's already been implemented (completed tasks/deliverables)
  - What deliverable this task belongs to
  - Dependencies on other tasks
  - Integration points and data shapes defined in the epic plan
- Base branch: the epic branch (noted in the plan-progress file)

**This is critical:** The epic plan-progress contains architecture, data models, and design decisions that are the **source of truth**. Do NOT re-investigate or re-decide things that are already settled in the epic plan.

Also check the current git branch with `git branch --show-current`.

## Step 3 — Investigate the codebase (focused)

Launch the `codebase-investigator` agent with a prompt that includes:

- The Jira ticket summary and acceptance criteria
- **If epic sub-task:** the relevant architectural context from the plan-progress (data models, service layer functions, API patterns already decided)
- Specific areas to investigate based on what this task touches

The investigation should be **narrow and focused** — only the files and patterns directly relevant to this task. The investigator does NOT need to understand the whole epic.

**Tell the investigator to focus on:**
- Files this task will create or modify
- Existing functions/services this task will call or extend
- Test patterns used in adjacent test files
- Exact data shapes at integration boundaries

Wait for the investigation results.

## Step 4 — Design the implementation plan

Launch the `solution-architect` agent with:

- The Jira ticket details (from Step 1)
- The epic context summary (from Step 2, if epic sub-task)
- The codebase investigation summary (from Step 3)
- The task type (standalone vs epic sub-task)
- The plan template: `docs/agomez/dx/individual-task-workflow.md`

**Tell the architect:**

- Architecture decisions are **already made** — do not redesign. Reference the epic plan for data models, API contracts, and service layer design.
- Focus on **implementation steps**: ordered, TDD-driven (RED -> GREEN -> REFACTOR), each step a single commit.
- Be specific: exact file paths, function signatures, type definitions, test cases.
- For epic sub-tasks, the base branch is the **epic branch**, not `develop`.

**Model: use sonnet** — the hard architectural thinking was done in `/plan-epic`. This is execution planning.

The architect writes the plan to `docs/agomez/plans/$ARGUMENTS-short-description.md`.

## Step 5 — Present the plan for approval

After the architect completes:

1. Read the plan file that was written
2. Present the full plan to the user inline
3. **STOP and wait for approval**

Tell the user:
- "Here's the implementation plan for $ARGUMENTS. Review the steps and let me know if you'd like changes, or approve it to proceed with `/implement $ARGUMENTS`."
- Do NOT proceed to write any implementation code

## Important Rules

- **No implementation code** — this skill only produces a task plan document
- **Always use both agents** — investigator first, then architect
- **Always stop for approval** — never auto-proceed to implementation
- **Respect the epic plan** — if this is a sub-task, the epic plan-progress is the source of truth for architecture. Do not contradict it.
- **Focused scope** — plan only what this task requires. Don't plan adjacent tasks or redesign the architecture.
- **Plan file**: `docs/agomez/plans/$ARGUMENTS-short-description.md` (separate file, not inside the epic plan-progress)
