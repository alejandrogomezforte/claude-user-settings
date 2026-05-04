---
name: verify
description: "Quality gate: run tests, type-check, format, lint, and coverage on changed files only. Reports pass/fail checklist. Usage: /verify MLID-XXXX  OR  /verify docs/agomez/plans/MLID-XXXX-short-description.md"
user_invocable: true
arguments:
  - name: task_ref
    description: "Jira task ID (e.g., MLID-1868) OR a plan file path (e.g., docs/agomez/plans/MLID-1868-foo.md)"
    required: true
---

# /verify $ARGUMENTS

You are running the **quality gate** before wrap-up. Check everything, report results, and fix issues.

## Step 1 — Resolve `$ARGUMENTS` and identify scope

`$ARGUMENTS` may be either a Jira task ID or a path to a plan file. Normalize first and produce two values: `<task_id>` and `<plan_file>`.

**If `$ARGUMENTS` contains `/` or ends with `.md`** — treat it as a plan file path:
- `<plan_file>` = `$ARGUMENTS` (verify the file exists; if not, **STOP** and report the bad path)
- `<task_id>` = the first `MLID-\d+` token found in the filename. If none, **STOP** and ask for the task ID explicitly.

**Otherwise** — treat `$ARGUMENTS` as a Jira task ID:
- `<task_id>` = `$ARGUMENTS`
- `<plan_file>` = the result of globbing `docs/agomez/plans/<task_id>*.md`. If zero matches, **STOP** and tell the user to run `/plan-task <task_id>` first. If multiple matches, **STOP** and ask the user to pass the explicit plan file path.

Use `<task_id>` for commit messages and the report header. Use `<plan_file>` to determine the base branch.

Determine changed files:
```bash
git diff --name-only <base-branch>...HEAD
```

Where `<base-branch>` is:
- `develop` for standalone tasks
- `epic/MLID-XXXX-*` for epic sub-tasks (read from `<plan_file>`)

Store the list of changed files — all subsequent checks target **only these files**.

## Step 2 — Run checks

Run the following checks. For each, record pass or fail.

### 2a. Tests
```bash
cd apps/web && npx jest --no-coverage
```

Run the full test suite (not just changed files) to catch regressions.

### 2b. Type checking
```bash
cd apps/web && npx tsc --noEmit
```

### 2c. Prettier (changed files only)
```bash
cd apps/web && npx prettier --check <changed-files>
```

If files need formatting:
```bash
cd apps/web && npx prettier --write <changed-files>
```

### 2d. ESLint (changed files only)
```bash
cd apps/web && npx eslint <changed-files>
```

If auto-fixable issues found:
```bash
cd apps/web && npx eslint --fix <changed-files>
```

### 2e. Code quality scan

Search changed files for violations:

- `console.log` or `console.warn` or `console.error` — should use `logger` instead
- `: any` or `as any` — should use proper types
- Leftover debug code, `TODO` comments from this task, `debugger` statements

### 2f. Test coverage (changed source files only)

For each changed `.ts`/`.tsx` file (excluding test files) that has a corresponding test file:
```bash
cd apps/web && npx jest --coverage --collectCoverageFrom='<file-path>' --testPathPattern='<test-file>'
```

Check that line coverage is ≥ 80%.

## Step 3 — Report results

Present a checklist:

```
## Quality Gate — <task_id>

- [x] Tests: all passing (N tests)
- [x] Types: no errors
- [x] Prettier: formatted (N files)
- [ ] ESLint: 2 warnings in xxx.ts
- [x] No console.log
- [x] No `any` types
- [x] Coverage: 87% on xxx.ts (threshold: 80%)

### Issues Found
- ESLint warning: unused import in `apps/web/xxx.ts:14`
```

## Step 4 — Fix issues

If any checks failed:

1. Fix the issues (formatting, lint, type errors)
2. Re-run the failing checks to confirm they pass
3. Commit the fixes: `[<task_id>] - chore(quality): fix lint/format/type issues`
4. Update the checklist to show all passing

## Important Rules

- **Never run `npm run format` or `npm run lint` globally** — always target only changed files
- **Always run from `apps/web/`** for lint and type checks (tsconfig.eslint.json issue at root)
- **Auto-fix what you can** — don't just report issues, fix them
- **Commit fixes separately** — quality fixes get their own commit, not mixed with feature code
