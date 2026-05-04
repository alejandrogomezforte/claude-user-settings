---
name: implement
description: "Implement a planned Jira task using TDD. Reads the approved plan, creates the branch, and executes red-green-refactor cycles. Usage: /implement MLID-XXXX  OR  /implement docs/agomez/plans/MLID-XXXX-short-description.md"
user_invocable: true
arguments:
  - name: task_ref
    description: "Jira task ID (e.g., MLID-1868) OR a plan file path (e.g., docs/agomez/plans/MLID-1868-foo.md)"
    required: true
---

# /implement $ARGUMENTS

You are starting the **implementation phase** of the software workflow. The plan has been approved — now execute it using strict TDD.

## Step 1 — Resolve `$ARGUMENTS` to a task ID and plan file

`$ARGUMENTS` may be either a Jira task ID or a path to a plan file. Normalize before doing anything else and produce two values: `<task_id>` and `<plan_file>`.

**If `$ARGUMENTS` contains `/` or ends with `.md`** — treat it as a plan file path:
- `<plan_file>` = `$ARGUMENTS` (verify the file exists; if not, **STOP** and report the bad path)
- `<task_id>` = the first `MLID-\d+` token found in the filename (e.g., `docs/agomez/plans/MLID-1868-foo.md` → `MLID-1868`). If no Jira ID is found in the filename, **STOP** and ask the user for the task ID explicitly.

**Otherwise** — treat `$ARGUMENTS` as a Jira task ID:
- `<task_id>` = `$ARGUMENTS`
- `<plan_file>` = the result of globbing `docs/agomez/plans/<task_id>*.md`. If zero matches, **STOP** and tell the user to run `/plan-task <task_id>` first. If multiple matches, **STOP** and ask the user to pass the explicit plan file path.

For the rest of this skill, use `<task_id>` wherever a Jira ID is needed (branch names, commit messages, the `/verify` handoff). Use `<plan_file>` to read the plan.

If the plan is for an epic sub-task, also read the epic plan-progress file for overall context.

Read `<plan_file>` completely. This is your implementation roadmap.

## Step 2 — Create the feature branch

Check the current git branch. If not already on the correct feature branch:

**Standalone task** (base branch is `develop`):
```bash
git checkout develop
git pull origin develop
git checkout -b feature/<task_id>-short-description
```

**Epic sub-task** (base branch is the epic branch — read from plan file):
```bash
git checkout epic/MLID-XXXX-epic-name
git pull origin epic/MLID-XXXX-epic-name
git checkout -b feature/<task_id>-short-description
```

If the branch already exists (resuming work), just check it out.

## Step 3 — Execute TDD cycles

Follow the implementation steps from the plan. For each step:

### RED — Write a failing test first
- Create or modify the test file
- Write test cases that describe the expected behavior
- Run the test to confirm it fails: `cd apps/web && npx jest --testPathPattern="<test-file>" --no-coverage`
- The test must fail for the **right reason** (missing implementation, not syntax errors)

### GREEN — Write the full implementation and pass the test
- Write the complete, clean implementation for the current step (no stubs or throwaway code)
- Clean up naming, structure, and duplication as you go
- Run the test to confirm it passes

## Step 4 — Repeat for each implementation step

Work through the plan's implementation steps sequentially.

## Step 5 — Signal completion

When all implementation steps are done:

1. Run the full test suite for affected files
2. Tell the user: "Implementation complete. You can now test the UI manually. When ready, run `/verify <task_id>` to check quality gates."

## Important Rules

- **Follow the plan** — don't deviate from the approved implementation steps without discussing with the user
- **TDD is mandatory** — never write implementation code before its test
- **Never commit** — do NOT commit. The user will commit manually when ready
- **No `any` types** — use proper TypeScript types
- **No `console.log`** — use `logger` from `@/utils/logger`
- **Named exports only** — no default exports (except Next.js pages/layouts)
- **Commit format** (when user asks): `[<task_id>] - type(scope): description` — no Co-Authored-By
