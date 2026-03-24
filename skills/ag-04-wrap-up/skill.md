---
name: wrap-up
description: "Delivery: merge branch, generate PR document, update plan status. Usage: /wrap-up MLID-XXXX"
user_invocable: true
arguments:
  - name: task_id
    description: "Jira task ID (e.g., MLID-1868)"
    required: true
---

# /wrap-up $ARGUMENTS

You are running the **delivery phase**. Merge the branch, generate the PR document, and update tracking documents. **You do NOT create GitHub PRs** — the user handles PR creation manually.

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
- **If the deliverable group is complete** → continue to Step 3

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

## Step 4 — Update plan status

- Update the plan file: set Status to `Done`
- If epic sub-task: update the plan-progress tracking table

## Step 5 — Present next steps to the user

Tell the user what manual steps remain for PR creation. Include the exact commands tailored to their task type:

### For standalone tasks:
```
Next steps (manual):
1. Push the branch:  git push origin -u <branch-name>
2. Create the PR:    gh pr create --base develop --head <branch-name> --title "<title>" --body-file docs/agomez/PR/$ARGUMENTS.md
```

### For epic sub-tasks (deliverable complete):
```
Next steps (manual):
1. Rebase on develop: git rebase develop && git push origin <epic-branch> --force-with-lease
2. Create the PR:     gh pr create --base develop --head <epic-branch> --title "<title>" --body-file docs/agomez/PR/$ARGUMENTS.md
```

## Important Rules

- **Git commands are OK** — merges, rebases, pushes are allowed (user will approve via permission prompt)
- **NEVER create GitHub PRs** — no `gh pr create` or any `gh` commands. The user handles PR creation manually.
- **NEVER transition Jira tickets** — only update Jira when the user explicitly asks
- **PR body from the document** — use the generated PR doc content
- **Epic sub-tasks**: if the deliverable isn't complete, STOP after the merge — don't generate a PR doc
- **Commit format**: `[$ARGUMENTS] - type(scope): description` — no Co-Authored-By
- **Always rebase before PR** — ensure clean history against the base branch
