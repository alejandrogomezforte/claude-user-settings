---
name: wrap-up
description: "Delivery: merge branch, generate PR document, create GitHub PR, update plan status. Usage: /wrap-up MLID-XXXX"
user_invocable: true
arguments:
  - name: task_id
    description: "Jira task ID (e.g., MLID-1868)"
    required: true
---

# /wrap-up $ARGUMENTS

You are running the **delivery phase**. Merge the branch, generate the PR, and update tracking documents.

## Step 1 — Gather context

1. Read the plan file for `$ARGUMENTS` from `docs/agomez/plans/`
2. Determine if this is a **standalone task** or **epic sub-task** (from plan file)
3. Check the current branch: `git branch --show-current`
4. Get the commit history for the branch: `git log --oneline <base-branch>..HEAD`
5. Get the diff summary: `git diff --stat <base-branch>...HEAD`

## Step 2 — Branch merge (epic sub-tasks only)

If this is an **epic sub-task**:

```bash
# Merge feature branch into epic branch
git checkout epic/MLID-XXXX-epic-name
git pull origin epic/MLID-XXXX-epic-name
git merge --no-ff feature/$ARGUMENTS-short-description
git push origin epic/MLID-XXXX-epic-name

# Clean up feature branch
git branch -d feature/$ARGUMENTS-short-description
git push origin --delete feature/$ARGUMENTS-short-description
```

Then check the epic plan-progress file:
- Update the sub-task row: set Status to `Done`, Merged to Epic to `Yes`
- **If more sub-tasks remain in this deliverable** → STOP here. Tell the user the sub-task is merged and what's next.
- **If the deliverable group is complete** → continue to Step 3 (PR from epic → develop)

## Step 3 — Generate PR document

Create a PR message document at `docs/agomez/PR/$ARGUMENTS.md` by analyzing:
- The plan file (for summary and context)
- Git log (for commits)
- Git diff (for files changed)

### PR Document Structure

```markdown
# [$ARGUMENTS] type(scope): short description

## Summary
- Bullet points of what was done and why

## Root Cause
(For bug fixes only — explain the underlying problem)

## Changes Overview
- **Files changed**: N
- **Lines added**: +N
- **Lines removed**: -N

## Files Changed

| File | Change | Description |
|------|--------|-------------|
| `path/to/file.ts` | Modified | What changed and why |

## What Changed and Why
Detailed breakdown of each change area. Include:
- Before/after examples where applicable
- Alternatives considered
- Design decisions made

## Commits
| Hash | Message |
|------|---------|
| `abc1234` | [$ARGUMENTS] - feat(scope): description |

## Test Plan
- [ ] Manual verification step 1
- [ ] Manual verification step 2

## Jira
- [$ARGUMENTS](https://localinfusion.atlassian.net/browse/$ARGUMENTS)
```

## Step 4 — Create the GitHub PR

Determine the PR base and head:

| Task type | Base | Head |
|-----------|------|------|
| Standalone | `develop` | `feature/$ARGUMENTS-short-description` |
| Epic deliverable | `develop` | `epic/MLID-XXXX-epic-name` |

Before creating the PR:
```bash
# Rebase on base branch
git checkout <base-branch>
git pull origin <base-branch>
git checkout <head-branch>
git rebase <base-branch>
git push origin <head-branch> --force-with-lease
```

Read the PR document from `docs/agomez/PR/$ARGUMENTS.md` and use its content for the PR body.

Create the PR:
```bash
gh pr create --base <base-branch> --head <head-branch> \
  --title "[title from PR doc]" \
  --body "$(cat docs/agomez/PR/$ARGUMENTS.md)"
```

**Ask the user for confirmation before creating the PR.**

## Step 5 — Update plan status

- Update the plan file: set Status to `Done`
- If epic sub-task: update the plan-progress tracking table

## Important Rules

- **Ask before creating the PR** — show the PR title and summary, get confirmation
- **Don't transition Jira tickets** — only update Jira when the user explicitly asks
- **PR body from the document** — use the generated PR doc, don't improvise
- **Epic sub-tasks**: if the deliverable isn't complete, STOP after the merge — don't create a PR
- **Commit format**: `[$ARGUMENTS] - type(scope): description` — no Co-Authored-By
- **Always rebase before PR** — ensure clean history against the base branch
