---
name: implement
description: "Implement a planned Jira task using TDD. Reads the approved plan, creates the branch, and executes red-green-refactor cycles. Usage: /implement MLID-XXXX"
user_invocable: true
arguments:
  - name: task_id
    description: "Jira task ID (e.g., MLID-1868)"
    required: true
---

# /implement $ARGUMENTS

You are starting the **implementation phase** of the software workflow. The plan has been approved — now execute it using strict TDD.

## Step 1 — Locate and read the plan

Find the plan file for `$ARGUMENTS` in `docs/agomez/plans/`:

- Search for `docs/agomez/plans/$ARGUMENTS*.md`
- If it's an epic sub-task, also read the epic plan-progress file for overall context
- If no plan file is found, **STOP** and tell the user to run `/plan $ARGUMENTS` first

Read the plan file completely. This is your implementation roadmap.

## Step 2 — Create the feature branch

Check the current git branch. If not already on the correct feature branch:

**Standalone task** (base branch is `develop`):
```bash
git checkout develop
git pull origin develop
git checkout -b feature/$ARGUMENTS-short-description
```

**Epic sub-task** (base branch is the epic branch — read from plan file):
```bash
git checkout epic/MLID-XXXX-epic-name
git pull origin epic/MLID-XXXX-epic-name
git checkout -b feature/$ARGUMENTS-short-description
```

If the branch already exists (resuming work), just check it out.

## Step 3 — Execute TDD cycles

Follow the implementation steps from the plan, applying the red-green-refactor cycle for each step:

### RED — Write a failing test first
- Create or modify the test file
- Write test cases that describe the expected behavior
- Run the test to confirm it fails: `cd apps/web && npx jest --testPathPattern="<test-file>" --no-coverage`
- The test must fail for the **right reason** (missing implementation, not syntax errors)

### GREEN — Write minimal code to pass
- Implement only what's needed to make the test pass
- Don't add extra features or optimizations yet
- Run the test to confirm it passes

### REFACTOR — Clean up while tests stay green
- Improve naming, structure, and readability
- Remove duplication
- Run tests after each change to ensure they still pass

### Commit after each cycle
```bash
git add <specific-files>
git commit -m "[$ARGUMENTS] - type(scope): description"
```

## Step 4 — Repeat for each implementation step

Work through the plan's implementation steps sequentially. Each step should result in at least one commit.

## Step 5 — Signal completion

When all implementation steps are done:

1. Run the full test suite for affected files
2. Tell the user: "Implementation complete. You can now test the UI manually. When ready, run `/verify $ARGUMENTS` to check quality gates."

## Important Rules

- **Follow the plan** — don't deviate from the approved implementation steps without discussing with the user
- **TDD is mandatory** — never write implementation code before its test
- **Commit frequently** — one commit per TDD cycle minimum
- **No `any` types** — use proper TypeScript types
- **No `console.log`** — use `logger` from `@/utils/logger`
- **Named exports only** — no default exports (except Next.js pages/layouts)
- **Don't auto-commit** — only commit when a TDD cycle is complete, never batch all changes into one commit
- **Commit format**: `[$ARGUMENTS] - type(scope): description` — no Co-Authored-By
