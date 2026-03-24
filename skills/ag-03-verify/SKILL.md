---
name: verify
description: "Quality gate: run tests, type-check, format, lint, and coverage on changed files only. Reports pass/fail checklist. Usage: /verify MLID-XXXX"
user_invocable: true
arguments:
  - name: task_id
    description: "Jira task ID (e.g., MLID-1868)"
    required: true
---

# /verify $ARGUMENTS

You are running the **quality gate** before wrap-up. Check everything, report results, and fix issues.

## Step 1 — Identify scope

Find the plan file for `$ARGUMENTS` in `docs/agomez/plans/` to determine the base branch.

Determine changed files:
```bash
git diff --name-only <base-branch>...HEAD
```

Where `<base-branch>` is:
- `develop` for standalone tasks
- `epic/MLID-XXXX-*` for epic sub-tasks (read from plan file)

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
## Quality Gate — $ARGUMENTS

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
3. Commit the fixes: `[$ARGUMENTS] - chore(quality): fix lint/format/type issues`
4. Update the checklist to show all passing

## Important Rules

- **Never run `npm run format` or `npm run lint` globally** — always target only changed files
- **Always run from `apps/web/`** for lint and type checks (tsconfig.eslint.json issue at root)
- **Auto-fix what you can** — don't just report issues, fix them
- **Commit fixes separately** — quality fixes get their own commit, not mixed with feature code
